# 🐉 Interactive Dragon Wallpaper

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![SVG](https://img.shields.io/badge/SVG-FFB13B?style=flat-square&logo=svg&logoColor=black)
![License: MIT](https://img.shields.io/badge/License-MIT-cyan.svg?style=flat-square)

A zero-dependency, hardware-accelerated interactive desktop wallpaper featuring a serpentine dragon that tracks the user's cursor in real time. Built entirely with vanilla HTML, CSS, and JavaScript using inline SVG rendering.

![Demo](assets/demo.gif)

---

## Description

The dragon is composed of **40 chained SVG segments** (`<use>` elements) whose positions are resolved per frame via a **spring-damped follow algorithm**. Each segment trails the one ahead of it using polar-coordinate chasing:

1. **Lead segment** (index `0`) interpolates toward the cursor position with linear easing (`÷ 10` damping factor), offset by a **Lissajous drift** (`cos(3f)` / `sin(4f)`) that produces organic idle motion when the cursor is stationary.
2. **Trailing segments** (`1 … N-1`) each compute the angle to their predecessor via `atan2`, then advance along that vector scaled by a distance factor `(50 - i) / 5`, with a secondary `÷ 4` damping pass — producing the characteristic undulating, serpentine motion.
3. **Idle re-centering**: when the cursor remains idle long enough for `rad > 60`, the pointer target drifts back toward the viewport center at 5% per frame, causing the dragon to autonomously orbit.

Each segment is progressively **scaled down** (`(162 + 4(1 - i)) / 100`) and **faded** (`opacity = 1 - i/N`) from head to tail, creating a natural depth taper.

---

## Features

- **Real-time cursor tracking** — sub-frame pointer event capture drives the head segment with smooth easing
- **Chained inverse-kinematics** — 40-segment chain with per-link `atan2` angle resolution and distance-scaled spring damping
- **Lissajous idle animation** — organic drift pattern via `cos(3f)` × `sin(4f)` Lissajous curves when the cursor is stationary
- **Automatic re-centering** — dragon migrates back to screen center after sustained cursor inactivity
- **Progressive opacity & scale taper** — head-to-tail gradient for visual depth
- **Neon glow effects** — CSS `drop-shadow` and `filter` chains with a `cyanFire` keyframe animation on the head for a flickering flame aesthetic
- **SVG `<use>` instancing** — three reusable SVG symbol definitions (`Cabeza`, `Aletas`, `Espina`) instantiated via `<use>` to minimize DOM weight
- **Viewport-responsive** — dynamic resize listener recalculates dimensions; the SVG canvas fills `100vw × 100vh`
- **Zero dependencies** — no frameworks, no build tools, no npm packages
- **60 FPS render loop** — single `requestAnimationFrame` callback with no forced reflows

---

## Tech Stack

| Layer     | Technology                                       |
| --------- | ------------------------------------------------ |
| Structure | HTML5                                            |
| Graphics  | Inline SVG (`<use>`, `<defs>`, linear gradients) |
| Animation | Vanilla JavaScript (`requestAnimationFrame`)     |
| Styling   | CSS3 (keyframes, `drop-shadow`, `filter`)        |

No build tools, bundlers, transpilers, or external dependencies.

---

## Project Structure

```
interactive-dragon-wallpaper/
├── dragon.html      # Entry point — SVG <defs> for dragon parts (Cabeza, Aletas, Espina)
├── style.css        # Layout reset, neon glow filters, cyanFire keyframe animation
├── script.js        # Animation loop, chain physics, pointer tracking, segment instantiation
└── README.md        # This file
```

### File Responsibilities

| File          | Role                                                                                                                                                                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `dragon.html` | Declares three SVG symbol groups inside `<defs>`: **Cabeza** (head with eyes/mouth), **Aletas** (wings with cyan-to-dark linear gradient), **Espina** (spine segments with bidirectional gradients). Hosts the `<g id="screen">` render target.        |
| `script.js`   | Initializes a 40-element chain array. Assigns SVG symbols to segments (index 1 → `Cabeza`, 8 & 14 → `Aletas`, all others → `Espina`). Runs the `requestAnimationFrame` loop computing per-segment transforms (translate → rotate → scale) and opacity. |
| `style.css`   | Full-viewport black background, `drop-shadow` glow on the SVG canvas, `cyanFire` flicker animation on the head's highlight path, neon stroke outlines on body segments.                                                                                |

---

## Installation & Setup

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, Safari)
- **No** Node.js, npm, or build step required

### Run Locally

```bash
# Clone the repository
git clone https://github.com/Adwaith-Balakrishnan/interactive-dragon-wallpaper.git
cd interactive-dragon-wallpaper

# Open in browser (any of the following)
# Option 1: Direct file open
start dragon.html          # Windows
open dragon.html           # macOS
xdg-open dragon.html       # Linux

# Option 2: Local HTTP server (recommended for consistent behavior)
python -m http.server 8000
# Then navigate to http://localhost:8000/dragon.html
```

### Use as Desktop Wallpaper

This project outputs a self-contained HTML page, compatible with wallpaper engines that support web-based wallpapers:

| Software                                                                        | Platform       | Setup                                                                                   |
| ------------------------------------------------------------------------------- | -------------- | --------------------------------------------------------------------------------------- |
| [Wallpaper Engine](https://store.steampowered.com/app/431960/Wallpaper_Engine/) | Windows        | Open Wallpaper Engine → **Edit** → **Open from File** → select `dragon.html`            |
| [Lively Wallpaper](https://www.rocksdanister.com/lively/)                       | Windows (free) | Drag & drop `dragon.html` into the Lively window, or **Add Wallpaper** → **Local File** |
| [Plash](https://github.com/nicklockwood/Plash)                                  | macOS          | Set URL to the local file path or hosted URL                                            |
| [Komorebi](https://github.com/cheesecakeufo/komorebi)                           | Linux          | Point to `dragon.html` in the wallpaper config                                          |

---

## Configuration & Customization

All tunable parameters live in [`script.js`](script.js). No config files — edit the source directly.

### Segment Count

```js
const N = 40; // Total chain segments (head + body + tail)
```

Increase for a longer dragon (higher GPU cost); decrease for a shorter, snappier one.

### Tracking Speed (Easing)

```js
// Lead segment — line 53-54
e.x += (ax + pointer.x - e.x) / 10; // ÷ 10 = damping factor (lower = faster)
e.y += (ay + pointer.y - e.y) / 10;

// Trailing segments — line 59-60
e.x += (ep.x - e.x + (Math.cos(a) * (50 - i)) / 5) / 4; // ÷ 4 = chain stiffness
e.y += (ep.y - e.y + (Math.sin(a) * (50 - i)) / 5) / 4;
```

| Parameter                      | Location   | Effect                                    |
| ------------------------------ | ---------- | ----------------------------------------- |
| `/ 10` (lead damping)          | Line 53–54 | Lower value → head snaps to cursor faster |
| `/ 4` (chain stiffness)        | Line 59–60 | Lower value → body follows more tightly   |
| `(50 - i) / 5` (link distance) | Line 59–60 | Controls spacing between segments         |
| `frm += 0.003`                 | Line 72    | Idle Lissajous animation speed            |
| `rad > 60`                     | Line 73    | Idle timeout before re-centering begins   |
| `* 0.05`                       | Line 74–75 | Re-centering interpolation speed          |

### Scale & Opacity Taper

```js
const s = (162 + 4 * (1 - i)) / 100; // Segment scale (head ≈ 1.66×, tail ≈ 1.06×)
const opacity = 1 - i / N; // Linear fade: head = 1.0, tail → 0.0
```

### Glow & Color Theme

All color values are in [`style.css`](style.css):

| Property       | Selector                     | Default                  | Purpose                  |
| -------------- | ---------------------------- | ------------------------ | ------------------------ |
| Body glow      | `svg { filter }`             | `rgba(0, 251, 255, 0.6)` | Global neon aura         |
| Eye flicker    | `@keyframes cyanFire`        | `#00fbff` / `#0088ff`    | Head highlight animation |
| Segment stroke | `#Aletas path, #Espina path` | `#00fbff`                | Neon outline on body     |
| Background     | `body, html` / `svg`         | `#000000`                | Wallpaper background     |

To change the dragon's color scheme, replace all instances of `#00fbff` (cyan) with your desired accent color.

### Asset Replacement

The dragon is built from three SVG symbol groups defined in the `<defs>` block of [`dragon.html`](dragon.html):

| Symbol ID | Assignment         | Description                                |
| --------- | ------------------ | ------------------------------------------ |
| `Cabeza`  | Segment 1          | Head — silhouette with eye cutouts         |
| `Aletas`  | Segments 8, 14     | Wings — gradient-filled fin shapes         |
| `Espina`  | All other segments | Spine — small bidirectional gradient barbs |

To replace a body part, edit or swap the `<path d="...">` data within the corresponding `<g id="...">` group. Ensure the new paths are centered around the origin `(0, 0)` for correct rotation.

---

## License

This project is licensed under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2025 Adwaith Balakrishnan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
