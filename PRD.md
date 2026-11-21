# PRD: Line-Art Walking Gallery

**Repo:** github.com/fadilafidina/gallery
**Status:** Draft v0.2
**Reference:** `7b1f9456-foo-bar.html` ("fadila's lil gallery")

---

## 1. Summary

A single-page, side-scrolling art gallery in a line-drawing style. A small line-drawn bird walks along a long, horizontally scrolling world. The visitor moves it left and right with the mouse, keyboard, or touch. Photos hang on the wall. When the bird stops near a photo, the visitor can open it in a full-screen viewer, page through the set, and close it. Footstep sounds play as the bird walks. The background is a Sydney sky that follows the real time of day and local weather.

The look is ethereal and built entirely from line work: thin ink outlines, flat pale fills, and generous white space.

## 2. Goals and non-goals

**Goals**
- Recreate the walk-and-browse feel of the reference: smooth, playful, a little slow on purpose.
- Make the art the focus. Photos are the content, the world is the frame.
- Keep the visual system line-based everywhere: thin strokes, no drop shadows, one accent colour.
- Include the Sydney sky: time-of-day and weather from the reference.
- Ship as a static site with no backend. Adding a photo means editing one JSON file and adding one image.

**Non-goals (v1)**
- No accounts, comments, uploads, or CMS.
- No e-commerce or print sales.
- No 3D.
- No second character or alternate theme.

## 3. How close to the reference?

**Recommendation: match the interaction model and the Sydney sky closely, and rebuild the visuals in line style.**

| Keep close to reference | Change for the line-art theme |
|---|---|
| Horizontal world with a camera that follows the walker | All visuals drawn as thin ink lines, not filled cards |
| Walk with A/D, arrows, mouse-follow, hold-to-walk, tap on touch | Frames: 1 px ink outline around each photo, no shadow |
| Idle → sit → sleep behaviour, "z" bubbles when dozing | Palette: pale paper base, one accent colour |
| Nearest photo lifts, prompt to open, click-to-walk-and-open | Sky: Sydney sun, moon, clouds and rain drawn as line art |
| Full-screen viewer with prev / next / close and a counter | Cursor: custom line-art cursor |
| Footsteps tied to the gait | Bird: pecks and small hops, as in the reference |

Not worth copying: embedding every image, the font, and the cursors as base64 inside one HTML file. That makes the file about 5 MB and hard to edit. Split them out (see §8).

## 4. What the reference does (baseline behaviour)

- **Movement:** keys A/D or arrows set a target speed. Mouse-follow moves the bird toward the pointer's world x position. Hold or tap on touch walks to a point. Speed eases in and out.
- **Camera:** eases toward the bird, clamped to the world bounds, with slight parallax on the foreground layer.
- **Gait:** legs swing with the walk phase, the body bobs and tilts, and the head pecks forward. A short hop (flap) can be triggered.
- **Sound:** a footstep plays each time the leg-swing sign flips while the bird walks fast enough.
- **Idle:** after about 5 seconds the bird sits. After about 11 seconds it dozes and shows "z" bubbles. Any movement wakes it.
- **Photos:** each piece is a framed image with a label (title, caption, note, camera EXIF). The nearest photo lifts and a prompt appears. The bird looks up at it when still.
- **Viewer:** full-screen overlay with the photo, its label, prev / next, and close (button or outside click). A counter shows position in the set.
- **Sky:** the window shows a sky that follows Sydney's sunrise and sunset, with sun, moon, stars, clouds, rain, snow and fog from current weather. Weather comes from a public weather API, and the sky falls back to clock-only if the request fails.
- **Intro:** title overlay with a start button.
- **HUD:** brand label top-left, pills top-right, a counter, and a hint at the bottom. The hint changes for touch devices.

## 5. Visual direction: ethereal line art

- **Palette (starting point, to refine):** paper `#F7F4EE`, ink `#1D1B19`, soft grey `#8B857C`, line `#D8D2C8`, accent `#E8552D`.
- **Lines:** every object is a stroke. Frames, the bird, clouds, hills, the sun and moon, and the wall are all line drawings with flat, pale fills at most.
- **Sky:** the Sydney window keeps the reference's time-of-day and weather logic, redrawn in line art.
- **Typography:** one clean monospace or sans for UI, one handwritten face for notes only.
- **Motion:** slow, eased, with a gentle float on the wall. Respect `prefers-reduced-motion`.

## 6. Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| F1 | Bird walks left and right across a world wider than the viewport | Must |
| F2 | Input: A/D and arrow keys, mouse-follow, hold-to-walk, touch tap-to-walk | Must |
| F3 | Camera follows the bird, clamped to world bounds, with smooth easing | Must |
| F4 | Photos hang at fixed positions with a title, caption, and optional note | Must |
| F5 | Nearest photo lifts when the bird is in range, and an "open" prompt shows | Must |
| F6 | Clicking a photo walks the bird to it and opens the viewer | Must |
| F7 | Full-screen viewer: prev / next, close by button or outside click, keyboard Esc and arrows | Must |
| F8 | Counter shows position in the set (for example `03 / 12`) | Must |
| F9 | Footstep sound synced to the gait, with a mute toggle | Must |
| F10 | Audio starts only after the first user gesture (browser autoplay rules) | Must |
| F11 | Idle: sit after about 5 s, sleep after about 11 s, wake on movement | Should |
| F12 | Intro overlay with a start button | Should |
| F13 | Touch hint text and touch-friendly controls | Should |
| F14 | Sydney sky: time of day from local sunrise and sunset, weather from a public API, with clock-only fallback | Must |
| F15 | Deep links: `#photo=3` opens photo 3 directly | Could |

## 7. Content, sound, and technical approach

**Stack (proposal):** Vite + vanilla JS. No backend.

**Hosting:** Netlify, which is already connected to this repo. GitHub Pages is an alternative, and it is free for public repositories.

**Repo layout (proposed):**
```
gallery/
  index.html
  src/
    main.js          # loop, input, camera
    walker.js        # bird sprite and gait
    gallery.js       # wall, pieces, lift, prompt
    viewer.js        # full-screen viewer
    sky.js           # Sydney sky, time and weather
    audio.js         # footsteps, mute
  public/
    photos/          # original images (to be provided)
    audio/
      step-left.wav  # generated footstep sample
      step-right.wav # generated footstep sample
  content/
    photos.json      # one entry per photo
  PRD.md
```

**Sound:** the two footstep samples are generated locally (a short low thump with soft gravel noise, alternating left and right pitch). They can be replaced with real recordings later.

**`photos.json` entry (proposed):**
```json
{
  "id": "03",
  "src": "/photos/03.jpg",
  "thumb": "/photos/03-thumb.jpg",
  "width": 3000,
  "height": 2000,
  "title": "Kamikochi",
  "caption": "Morning mist over the river",
  "note": "shot at 6am",
  "date": "2026-10-05",
  "camera": { "make": "", "model": "", "lens": "", "exposure": "" }
}
```

**Performance:** lazy-load photos near the viewport. Thumbnails on the wall, full images in the viewer. Target 60 fps on a laptop and usable on a mid-range phone.

**Accessibility:** keyboard-only navigation, visible focus, `prefers-reduced-motion` reduces camera ease and parallax, and the mute toggle is always visible.

## 8. Milestones

| Milestone | Scope | Done when |
|---|---|---|
| M0 | Repo scaffold, Vite, Netlify deploy, empty world | Site loads on Netlify |
| M1 | Bird with gait, keyboard and mouse movement, camera follow | Bird walks smoothly across a test world |
| M2 | Photos from `photos.json`, lift and prompt, viewer with prev / next and counter | All photos open and page correctly |
| M3 | Footstep audio, mute, first-gesture unlock, intro overlay | Sound matches the walk and respects mute |
| M4 | Sydney sky (time, weather, fallback) | Sky matches local time and weather |
| M5 | Idle sit and sleep, touch support, hint text, reduced motion | Mobile and reduced motion checked |

## 9. Open questions

1. **Photos:** the Instagram profile could not be fetched (the site's robots rules block automated access). Options: you export your data from Instagram ("Download your information") and I use those files, or you drop the images into `public/photos`. How many, and what aspect ratios?
2. **Reference photos:** the reference HTML embeds its photos as base64. I have not extracted them yet, because that file is only in my sandbox and I can't reach it from your Mac. If you want those, copy the HTML into `~/src/gallery` and I'll extract them.
3. **Character art:** should the bird be hand-drawn SVG (matching the line style) or simple geometric shapes like the reference?
4. **Netlify:** confirm Netlify as the host, or switch to GitHub Pages.
