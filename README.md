# Daguerre

A single-file WebXR experience that places Louis Daguerre — the inventor of photography — in the user's live camera feed, then lets them "develop" a still photo of the scene into a stylized daguerreotype.

## What it does

Tap the button, point your phone at the ground to find a surface, then tap to place a life-size video of Daguerre there. He plays through a short walking animation once and freezes on his last frame. From there you can capture the scene — camera feed plus Daguerre composited together — and the app turns it into a vintage daguerreotype-style still you can save.

## Platform support

Placement and tracking use the WebXR Device API (`immersive-ar` session with the `hit-test` feature). **This currently only works in Chrome on Android (ARCore).** Safari on iOS does not implement WebXR, so there is no AR experience on iPhone yet.

## How placement works

A WebXR hit-test source continuously raycasts against detected real-world surfaces. While nothing is placed, a reticle tracks the ground under wherever the phone is pointed, and the "Placer Louis Daguerre ici" button enables once a surface is found. Tapping it bakes Daguerre's position and facing (rotated to face the camera at that moment) into a fixed transform — he's added to the AR scene in world space, not parented to the camera, so he stays put as you move around. That transform is set once and never recalculated.

## Daguerre character & green screen removal

`daguerre.mp4` is shot against a green background and rendered as a textured plane sized to a fixed real-world height (1.72 m). A custom shader removes the green per pixel every frame: it computes "greenness" (`g − max(r, b)`), uses `smoothstep` for soft anti-aliased edges, and caps the green channel to suppress spill on hair and clothing.

The video plays once — it does not loop. When it ends, it holds on the last frame and reveals a helper message plus the "Créer un daguerréotype" button.

## Capturing the daguerreotype

Tapping "Créer un daguerréotype" grabs the current camera frame and the rendered AR scene, composites them together, and runs the result through a multi-pass filter: grayscale luminance extraction, blur-based local contrast, a bloom mask on highlights, a sigmoid tone-mapping curve with lifted blacks, monochrome grain, a silver-blue color tint, a vignette and center glow, and randomized scratch/dust marks — all applied uniformly across both the camera background and Daguerre. The result is shown full-screen with an option to save the image or resume.

## Usage

Serve the directory over HTTPS (required for camera and WebXR access on mobile), open the page in Chrome on Android, and tap **Rencontrer Louis Daguerre**.
