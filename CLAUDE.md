# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file WebXR app (`index.html`) that places a life-size video of Louis Daguerre in the user's live camera feed via AR plane-hit placement, then lets the user "develop" a still photo of the scene into a stylized daguerreotype.

There is no build step, no package manager, and no test suite. `index.html` is the entire application: HTML, CSS, and JavaScript in one file. The only external dependency is three.js, loaded from a CDN via a plain `<script>` tag (pinned to `three@0.160.0`).

## Running it

Camera access (and WebXR) requires a secure context, so the directory must be served over HTTPS — opening the file directly (`file://`) or over plain HTTP will not work on a phone. Any static file server with TLS (e.g. a tunnel like ngrok, or deploying to a static host) is sufficient; there is nothing to compile.

There is no lint or test command in this repo.

## Platform constraint

The AR session is built entirely on the WebXR Device API (`navigator.xr.requestSession('immersive-ar', { requiredFeatures: ['hit-test'] })`). **This only works in Chrome on Android (ARCore).** Safari on iOS does not implement WebXR at all, so this app currently does not run on iPhone — there is no fallback path. Any cross-platform fix has to replace the AR session/tracking layer (see "Architecture" below for what would need to change), not just tweak this file.

## Architecture (all in `index.html`'s single `<script>` block)

The code is one un-modularized script, organized top-to-bottom into these sections — useful to know since changes in one often require touching code further down:

1. **Three.js scene/renderer setup** — `scene`, `camera`, `renderer` created up front; `renderer.xr.enabled = true` is what allows `renderer.xr.setSession()` later.
2. **Reticle** — a ring mesh showing where Daguerre will be placed; visibility/position is driven entirely by the WebXR hit-test loop (`onXRFrame`), not by any independent logic.
3. **Daguerre character mesh** — a `THREE.PlaneGeometry` textured with a `THREE.VideoTexture` of `daguerre.mp4`, using a custom `ShaderMaterial` that does real-time chroma-key removal of the video's green background (computes `green = g - max(r,b)`, alpha from `smoothstep`, plus green-spill suppression). The video does **not** loop — it plays once and holds its last frame (`ended` event reveals the "Créer un daguerréotype" CTA).
4. **Placement** (`placeDaguerre`) — on tap, reads the current reticle pose, computes a yaw so Daguerre faces the camera, and bakes that into the mesh's matrix once (`matrixAutoUpdate = false`). The mesh is added to `scene` (world space), not parented to the camera, so it stays fixed in place as the user moves — it is *not* recomputed per frame after placement.
5. **WebXR session lifecycle** (`startAR`, `onSessionEnd`) — requests `hit-test` (required) plus `dom-overlay` and `camera-access` (optional). `dom-overlay` is what lets the regular HTML UI (`#ar-ui` and its children) render on top of the immersive session without any manual z-index/compositing work — this only exists because of that feature; losing WebXR also loses this for free.
6. **XR frame loop** (`onXRFrame`) — runs hit-testing every frame (only while nothing is placed yet) and triggers capture when `captureNextFrame` is set.
7. **Capture** (`renderSceneToDataURL`) — the trickiest part of the file. An XR session's swap-chain framebuffer is cleared after every present, so `renderer.domElement.toDataURL()` would just return blank. The workaround: re-render the three.js scene into an off-screen `WebGLRenderTarget` to get Daguerre's pixels, and separately pull the camera's own video frame via `XRWebGLBinding.getCameraImage()` (requires the optional `camera-access` feature) into a manual framebuffer read. Both pixel buffers need a manual Y-flip (WebGL's origin is bottom-left) before being composited onto a 2D canvas. This whole read must happen synchronously inside the same `onXRFrame` callback — the camera texture is only valid for that one frame.
8. **Daguerreotype filter** (`processDaguerreotype`) — a multi-pass 2D canvas image pipeline applied to the captured composite: grayscale luminance extraction → CSS-blur-based local contrast (high-pass of a blurred grayscale copy added back in) → a separately blurred bloom mask for highlights → a sigmoid S-curve tone-mapping LUT with lifted blacks → monochrome grain → silver-blue color tint → bloom screen-blend → vignette + center glow → random scratch/dust overlay. All steps run on the already-composited (camera + Daguerre) image so both receive identical treatment.

## Known discrepancy

`README.md` describes an earlier implementation (device-orientation-based ground anchoring, looping video, no WebXR, no daguerreotype capture) that no longer matches `index.html`. Trust the code over the README for how positioning, capture, and the video lifecycle actually work today.
