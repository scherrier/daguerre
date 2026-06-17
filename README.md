# Daguerre

A single-file web AR experience that places Louis Daguerre — the inventor of photography — walking in the background of your device's live camera stream.

## What it does

When you open the page and tap the button, the rear camera fills the screen. A looping video of Daguerre walking is composited over it in real time, as if he were standing in the scene in front of you.

## How the character is positioned

**Initial placement** — Daguerre is spawned at the horizontal center of the screen (`xFrac = 0.5`) and at 90 % of the screen height from the top (`yFrac = 0.9`), placing his feet near the bottom edge of the viewport.

**Size** — His rendered height is `canvas.height × 0.30 × scale` (scale defaults to 3), making him roughly 90 % of the viewport tall. The image is drawn upward from his feet, so his head reaches close to the top of the screen.

**Direction** — He faces left (`dir = -1`); the canvas is mirrored horizontally when drawing him.

**Ground anchoring via device orientation** — At the moment the camera starts, the phone's heading and tilt (`alpha` / `beta` from the `deviceorientation` API) are recorded as the reference pose. On every subsequent frame, the deviation from that reference is converted into a screen-space offset using approximate camera field-of-view values (65 ° horizontal, 50 ° vertical). This makes Daguerre appear to stay fixed at one spot on the ground as you pan the phone. Exponential smoothing (factor 0.15) is applied to reduce jitter.

## Green screen removal

The character video (`daguerre.mp4`) is shot against a green background. A tiny WebGL pipeline runs on the GPU every frame: a fragment shader computes per-pixel "greenness" (`g − max(r, b)`) and uses `smoothstep(0.01, 0.2, greenness)` to produce soft, anti-aliased edges. Green spill on hair and clothing is suppressed by capping the green channel at `max(r, b)` on every pixel. The result is drawn onto an offscreen canvas, which is then composited onto the main canvas at the character's screen position.

## Usage

Serve the directory over HTTPS (required for camera access on mobile), open the page, and tap **Rencontrer Louis Daguerre**.
