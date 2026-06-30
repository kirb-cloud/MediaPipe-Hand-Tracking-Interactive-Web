# Hand-Tracking Web Simulation (TouchDesigner)

An interactive TouchDesigner project that uses **MediaPipe hand tracking** on a live webcam feed to drive a **simulated spider-web / proximity surface**, where the web reacts to the position and movement of the viewer's hands.

## Demo

https://github.com/user-attachments/assets/28ba6f54-6d4d-450b-bfb0-bda33673616b

## Network Screenshots

**Full hand-tracking network**
![Hand tracking network](images/hand_trackin_web_project_images.png)

**Webcam → hand_tracking → web_sim signal flow**
![Top-level signal flow](images/web_image_2.png)

**Proximity / web simulation network**
![Web simulation network](images/web_image_3.png)

**Render network (camera, geometry, composite)**
![Render network](images/web_image_4.png)

**hand_tracking COMP detail**
![hand_tracking detail](images/Screenshot_2026-06-30_053204.png)

## Project File

The full TouchDesigner project is included: [`handtrackingProject.toe`](handtrackingProject.toe)

## Overview

```
MediaPipe (webcam in) → hand_tracking (COMP) → web_sim (COMP) → output
```

1. **MediaPipe** — A webcam TOP feeds a MediaPipe CHOP/COMP that detects hand landmarks in real time.
2. **hand_tracking** — Takes the raw MediaPipe landmark data and processes it into clean, usable per-hand channels.
3. **web_sim** — Consumes the processed left/right hand data to deform and render a reactive "web" surface, composited into the final output.

## Network Breakdown

### Top Level
- `MediaPipe` — webcam Video In TOP + MediaPipe hand-landmark detector. Outputs raw landmark CHOP data (pink wire) and tracking/confidence data (blue wire).
- `hand_tracking` — a COMP containing the landmark-cleanup network (see below).
- `web_sim` — a COMP containing the rendering/simulation network (see below).

### Inside `hand_tracking`
Raw landmark data (`in1`) is split into two symmetric chains, one per hand:

**Left hand chain:**
`select1 → shuffle1` (and two auxiliary selects/shuffles for additional landmark groups) `→ merge1 → math1 → math2 → flip` → combined with `noise1` → **`left_hand_final`**

**Right hand chain:**
`select4 → shuffle2` (with `select5/6 + shuffle3/...` feeding additional landmark groups) `→ merge2 → math3 → math4 → flip1` → combined with `noise2` → **`right_hand_final`**

- The **select/shuffle** nodes isolate and reorder specific landmark channels (e.g., fingertip, wrist, palm points) from the raw 21-point MediaPipe hand skeleton.
- The **merge** nodes recombine the selected groups into a single channel set per hand.
- The **math** nodes rescale/remap/offset the landmark values (e.g., normalizing screen-space coordinates into simulation space).
- **noise1 / noise2** inject subtle procedural noise into each hand's signal for organic motion / jitter reduction or stylized wobble.
- **flip / flip1** mirror or invert the relevant axes before final output.
- Outputs: `left_hand_final` and `right_hand_final` CHOPs, exported up to the parent COMP for use by `web_sim`.

### Inside `web_sim`
- `sopto1` and `sopto2` — convert geometry (likely point clouds driven by the left/right hand CHOPs) into renderable SOPs.
- `merge3` — combines both hand-driven SOP networks into a single geometry stream.
- `proximity1` — computes proximity/distance relationships between points (e.g., web strands reacting based on how close a hand point is to a web node).

### Rendering Network (`render1` / `comp1` area)
- `cam1` — scene camera.
- `constant1` — a simple circle/point generator (likely used as a reference or particle shape).
- `geo1` — main geometry object, fed by `popto1` (Points to... TOP/SOP conversion) and `constant1`.
- `render1` — renders `geo1` from `cam1`.
- `in2` — secondary input layer (e.g., background or UI layer).
- `comp1` — final composite of `render1` and `in2`, producing the finished output image.

### Misc
- `chopto1` / `chopto2` — convert CHOP data to TOP textures (likely used to visualize or pass landmark data as image data for further GPU-side processing, e.g., feeding shaders).

## Requirements
- TouchDesigner (2022.x or later recommended)
- A connected webcam
- MediaPipe plugin/component for TouchDesigner (hand landmark model)

## Usage
1. Open the `.toe` project file in TouchDesigner.
2. Ensure your webcam is selected as the Video In TOP source inside the `MediaPipe` component.
3. Enable hand tracking and confirm `left_hand_final` / `right_hand_final` channels are updating in the `hand_tracking` COMP.
4. View the composited output from `comp1` inside `web_sim` — the web/surface should visibly react as hands move in front of the camera.

## Notes / Demo
A reference video (`882634dd9c6a4261a17a1d03d4e4c808.MOV`) is included showing the project running live, demonstrating the web reacting to hand movement in real time.

## Possible Improvements
- Add confidence-based smoothing/filtering before the `math` nodes to reduce jitter when tracking confidence is low.
- Expose `noise1` / `noise2` parameters to a UI panel for live VJ-style tweaking.
- Add support for more than two hands / multi-person tracking.
