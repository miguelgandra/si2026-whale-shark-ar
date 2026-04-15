# 🦈 Whale Shark AR – SI2026

**Augmented Reality experience for the _Linking Three-Dimensional Movement Patterns to Foraging Behaviour of Whale Sharks_ poster at Sharks International 2026.**

Point your phone at the printed conference poster to bring it to life with a swimming whale shark, interactive videos, and a high-resolution Azores map.

---

## 📱 How to Use (Conference Attendees)

1. **Open the AR experience** on your smartphone browser:  
   `https://<your-github-username>.github.io/<repo-name>/`

2. **Tap "Tap anywhere to start"** on the splash screen.

3. **Point your camera** at the printed SI2026 poster.  
   Hold the phone steady and ensure the entire poster is in frame.  
   Good lighting improves tracking accuracy.

4. **Interact:**

   | Element | What to do |
   |---|---|
   | 🦈 Whale shark + fish school | Watch them orbit the poster automatically |
   | 📡 iPilot tag (3D model) | Observe the biologging device rotating above the poster |
   | **▶ Play buttons** (4 images) | Tap any button in the Foraging Ecology section to watch the behaviour video |
   | 🗺 **Azores map area** | Tap the map figure (top of poster) to open a high-resolution view |
   | ✉ Email / LinkedIn | Tap the contact buttons on the left panel |
   | **✕ button** | Tap the red X to close any open video or map panel |

5. **Re-scanning:** If the AR content disappears, move the camera away and slowly return to point at the poster.

---

## 🗂 Project Structure

```
/
├── index.html                  ← Main AR experience (entry point)
├── coordinate-finder.html      ← Debug tool for refining hit-box positions
├── targets.mind                ← MindAR image target (generated from poster)
├── SI2026 - poster.png         ← Poster image used as AR reference
├── 3D models/
│   ├── whale_shark.glb         ← Whale shark (orbits poster)
│   ├── the_fish_particle.glb   ← Fish school particle system
│   └── iPilot.glb              ← Biologging tag (rotates on Y-axis)
├── media/
│   ├── profile.png             ← Author profile photo
│   ├── azores-map.png          ← High-resolution Azores distribution map
│   ├── lunge-feeding.mp4       ← Behaviour video 1
│   ├── stationary-feeding.mp4  ← Behaviour video 2
│   ├── intraspecific-aggregation.mp4  ← Behaviour video 3
│   └── snipefish.mp4           ← Behaviour video 4
└── README.md
```

---

## 🛠 Technical Overview

### Stack

| Library | Version | Purpose |
|---|---|---|
| [A-Frame](https://aframe.io) | 1.6.0 | WebXR / 3D scene framework |
| [MindAR](https://hiukim.github.io/mind-ar-js-doc/) | 1.2.5 | Image-target recognition |
| [A-Frame Extras](https://github.com/c-frame/aframe-extras) | 7.4.0 | GLTF animation mixer |

### Architecture

`index.html` is a self-contained, single-file application. All CSS and JavaScript are inlined following the example boilerplate's conventions.

**Custom A-Frame components:**

| Component | Responsibility |
|---|---|
| `clickable` | Opens a URL in a new tab on click (email, LinkedIn) |
| `ar-handler` | Master orchestrator: target detection, show/hide content, video & map popup state machine |

**3D Scene structure (relative to detected image target):**

```
mindar-image-target [ar-handler]
│
├── #static-poster            (poster plane, 1.0 × 1.41 units)
│
├── #orbit-parent             (invisible rotator, Y-axis, 20 s/loop)
│   ├── #whale-shark          (gltf-model, offset +X = radius)
│   └── #fish-particle        (gltf-model, near shark)
│
├── #ipilot-model             (gltf-model, Y-axis rotation, 10 s/loop)
│
├── #side-content             (profile + email + LinkedIn, left of poster)
│
├── #video-buttons            (4 semi-transparent play overlays)
│
├── #panel-overlay            (full-poster dark dim, z = 0.025)
│
├── #video-popup              (floating video panel, z = 0.55)
│
├── #azores-hitbox            (invisible clickable plane over map figure)
│
└── #azores-popup             (high-res map panel, z = 0.55)
```

**Whale shark orbit mechanism:**  
`#orbit-parent` animates a continuous Y-axis rotation. `#whale-shark` and `#fish-particle` are children offset along the local +X axis. As the parent rotates, children naturally face the tangential direction of travel.

**Video popup:**  
Four `<video>` elements are declared in `<a-assets>`. On tap, `ar-handler.openVideo()` updates the single `#video-screen` plane's `material.src` dynamically and calls `.play()` on the selected video element.

**Mobile video unlock:**  
`unlockVideos()` is called on the starter-overlay tap. It briefly plays and pauses all four `<video>` elements to satisfy the browser's "user gesture required" requirement, ensuring later programmatic `.play()` calls succeed.

### Coordinate System

The poster plane is anchored at `(0, 0, 0)` with dimensions `1.0 × 1.41` A-Frame units:

```
         +0.705  (top)
            │
  -0.5 ────┼──── +0.5
  (left)   │   (right)
            │
         -0.705 (bottom)
         
 Z = 0    → poster surface
 Z = 0.01 → just above (hitboxes, buttons)
 Z = 0.35+ → 3D models floating in front
 Z = 0.55  → popup panels
```

### Refining Coordinates

Open **`coordinate-finder.html`** in a browser. Click anywhere on the poster image to instantly receive the corresponding A-Frame `position` string. Use the labelled section buttons to get prompted for each interactive element:

- Azores map centre → `#azores-hitbox`
- Video image centres (×4) → `#video-btn-1` through `#video-btn-4`
- iPilot tag image centre → `#ipilot-model` (use Z ≈ 0.35)

---

## 🚀 Deployment (GitHub Pages)

### 1. Initialise the repository

```bash
git init
git remote add origin https://github.com/<your-username>/<repo-name>.git
```

### 2. Check file sizes

```bash
find . -name "*.glb" -o -name "*.mp4" | xargs ls -lh
```

**Actual asset sizes (measured):**

| File | Size | Notes |
|---|---|---|
| `3D models/whale_shark.glb` | 37 MB | ✅ Under GitHub 50 MB limit |
| `3D models/iPilot.glb` | 34 MB | ✅ Under GitHub 50 MB limit |
| `3D models/the_fish_particle.glb` | 1.1 MB | ✅ Fine |
| `SI2026 - poster.png` | **24 MB** | ⚠️ See performance note below |
| `media/stationary-feeding.mp4` | 12 MB | ✅ Fine |
| `media/snipefish.mp4` | 9 MB | ✅ Fine |
| `media/lunge-feeding.mp4` | 5.5 MB | ✅ Fine |
| `media/intraspecific-aggregation.mp4` | 4.6 MB | ✅ Fine |
| `targets.mind` | 1.4 MB | ✅ Fine |
| `SI2026 - poster.pdf` | 45 MB | ❌ Excluded by `.gitignore` (not needed for AR) |

> **No files exceed the 50 MB threshold.** Git LFS is not strictly required.  
> However, both `.glb` models are in the 34–37 MB range and may trigger GitHub's soft warning. If you plan to iterate on the models frequently or the repository grows, setting up LFS is a good practice:

> ⚠️ **Performance note – Poster PNG (24 MB):**  
> The `SI2026 - poster.png` used as the visual texture in the AR scene is 24 MB, which will be slow to download on mobile networks. **Strongly recommended:** export a compressed JPEG version at ~70–80% quality for use in `index.html`. The `targets.mind` recognition file is already pre-processed and unaffected.
>
> Quick compression command (requires `ffmpeg` or use [Squoosh](https://squoosh.app)):
> ```bash
> # Creates a ~2–4 MB JPEG suitable for the AR texture
> ffmpeg -i "SI2026 - poster.png" -q:v 3 "SI2026 - poster-web.jpg"
> ```
> Then update `index.html`: change `src="SI2026%20-%20poster.png"` to `src="SI2026%20-%20poster-web.jpg"` in **both** the `<img id="posterImage">` tag and the `<a-plane id="static-poster">` `src` attribute.

<details>
<summary><strong>Optional: Set up Git LFS</strong></summary>

```bash
# Install Git LFS (once)
brew install git-lfs          # macOS
# or: sudo apt install git-lfs  (Linux)

git lfs install

# Track large file types
git lfs track "*.glb"
git lfs track "*.mp4"
git lfs track "*.mind"

# This creates/updates .gitattributes
git add .gitattributes
git commit -m "Configure Git LFS for large binary assets"
```

After this, all future `git add` commands for `.glb`, `.mp4`, and `.mind` files will automatically be handled by LFS.

</details>

### 3. Push and enable GitHub Pages

```bash
git add .
git commit -m "Initial commit: SI2026 Whale Shark AR experience"
git push -u origin main
```

Then in your GitHub repository:  
**Settings → Pages → Source: Deploy from branch → `main` / `(root)` → Save**

Your experience will be live at:  
`https://<your-username>.github.io/<repo-name>/`

---

## ⚙️ Configuration Cheatsheet

### Change orbit speed
In `index.html`, find `#orbit-parent` and modify `dur` (milliseconds per full orbit):
```html
animation="... dur: 20000 ..."   <!-- 20 s = default -->
```

### Change whale shark scale
```html
<a-entity id="whale-shark" scale="0.12 0.12 0.12" ...>
```

### Change iPilot rotation speed
```html
<a-entity id="ipilot-model" animation="... dur: 10000 ...">
```

### Enable video audio
Remove the `muted` attribute from the `<video>` elements in `<a-assets>`.

### Add Google Analytics
Replace `G-XXXXXXXXXX` (appears twice near the top of `index.html`) with your GA4 Measurement ID.

### Developer mode (exclude from Analytics)
Visit `<your-url>?dev=true` once. Your session is then excluded until you clear `localStorage`.

---

## 📝 Acknowledgements

Built with [A-Frame](https://aframe.io) and [MindAR](https://hiukim.github.io/mind-ar-js-doc/).  
Developed for **Sharks International 2026**.

*Miguel Gandra · m3gandra@gmail.com · [LinkedIn](https://www.linkedin.com/in/miguel-gandra/)*
