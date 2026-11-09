# Portfolio: single-file site (Lumora-based)

`index.html` is the whole site: inline CSS, one inline `<script type="module">`, and a `CONTENT` object at the top of the script that holds every placeholder. No build step.

## Run it locally

Serve the folder over http. Opening the file directly (`file://`) blocks the images.

```
npx serve .
# or
python3 -m http.server 8000
```

The page shows a red banner (localhost and file:// only) if a required asset fails, and names the URL.

## Files

- `index.html`: the site.
- `assets/HERO/Gemini_Generated_Image_wm4setwm4setwm4s-removebg-preview_upscaled.png`: hero photo (required). 2000×2000 transparent cut-out, used as-is.
- `assets/HERO/eyes/`: **unused** (the eye-follow effect was removed). Eye layers cut from the hero photo: `eye-{L,R}-sclera.png` (eye whites with the iris painted out), `-iris.png`, `-glint.png`, `-shadow.png`. The geometry is in the `EYES` constant.
  - `eyes.py` regenerates the layers and `eyes.json`, and `preview.py` renders a contact sheet of gaze states. Both need `numpy`, `opencv-python-headless` and `pillow`. **Re-run them if the photo changes.** Then paste the new geometry into `EYES`.
- `assets/3D/person_bust.stl`: no longer used (the 3D bust was removed).
- `assets/PALETTE/COLORS.png`: palette reference. Not loaded by the page.

The paths differ from the spec's lowercase `assets/hero/` layout. They live in the `PATHS` constant at the top of the script. Files were not moved.

## Hero: halftone + dispersing dots

- **Halftone:** you're drawn as fine blue dots (5 px cells). A faint dot field covers the hero, strongest near you and thinning toward the headline. Everything is drawn by one WebGL2 shader on `#halftone`. The `<img>` stays in the DOM, invisible, for its alt text.
- **Cursor:** dots near the cursor are pushed away and scatter with a little jitter, leaving a gap that follows the cursor. When the cursor leaves, they spring back with a slight overshoot.
- **Life:** while the cursor is in the hero, the dots around it never stand still. The area sways in a slow twist (`HT.life.twirl`), each dot circles its own spot with its own phase and speed (`orbit`), and the gap breathes ±15% (`breathe`). The motion fades out with distance (`reach`).
- **Click / tap:** a bigger burst blows the dots out, then lets them spring back (≈1.2 s).
- **Entrance:** the dots grow in as a wave from the bottom. **Reduce motion:** a still halftone. **No WebGL2:** the plain colour photo.

Tuning is in `HT`: `cursor.push`/`radius`, `burst.push`/`radius`, `follow`, `spring`, `jitter`, and colours. Keep `push` below 1.6 × `radius`, or dots can fold over each other. The canvas renders at up to 1.5× device pixels, because each pixel checks 9 dots.

## Header glass

The nav bar's glass is set by the `--g-*` tokens at the top of the CSS (edit them by hand): blur, saturation, brightness, tint over the hero, tint after scrolling, chip tint, edge line, top highlight, shadow, refraction and lens edge width.

- **Refraction:** any value other than 0 bends the backdrop at the pill's edge through an SVG displacement map drawn from the pill's shape (redrawn on resize). It works in Chromium browsers only (Chrome, Edge, Arc). Safari and Firefox show the frosted glass without the lens.

## Connecting the form

`FORM_ENDPOINT` (top of the script) is `""`, which makes the form a stub: it shows "Sending…" for 600 ms and then the success state. Set it to a URL (for example a Formspree endpoint) and the form will `POST` JSON there.

## Placeholders and assets to supply

Every `[[PLACEHOLDER: …]]` and `[[IMAGE: …]]` string is in `CONTENT`. Ctrl-F `PLACEHOLDER` to find them.

Optional files, enabled by flipping the matching flag in `HAVE` (top of the script):

- `assets/hero/person.png`: no longer used. The hero photo is the transparent PNG above.
- `assets/proof/logo-N.svg`, `assets/proof/avatar-N.webp`: client logos and testimonial avatars. Currently text and placeholder tiles.
- `assets/resume.pdf`: the résumé button shows a note until this exists.
- `assets/logo.svg`: the logo. The page uses a placeholder "A" mark for now.

## Works and case studies

The Works section is one full-width row per project, in the order of `CONTENT.projects`: Xpro Delivery, WMS Tactical, Sagana app, SukiTuki, Sagana website. Each row has a CSS device frame (phone, landscape tablet or browser) on a tinted panel. "View case study" opens a full-screen overlay at `#work/<slug>`. Deep links work, Back closes it, and Forward reopens it.

- **Copy:** every word of every case study is in `CONTENT.projects[]`. Design projects (`kind: "design"`) have `overview`, `problem`, `decisions`, `flow` (the "How it works" SVG, drawn from `lanes[].steps`), `structure` (grey-box spec + legend), `iterations` (before/after sliders), `gallery`, `style`, `howBuilt` (projects 1–2) and `learned`. The website (`kind: "engineering"`) has `does`, `stack`, `stackNote`, `challenges` and `gallery`. `related` cross-links Sagana app ↔ Sagana website.
- **Images:** all paths are in one map, `WORKS_MEDIA`, just below `CONTENT`. A file is `assets/works/<dir>/<key>.webp`, and each entry is `[width, height, alt text]`. Captions are in `WORKS_CAPTIONS`. Screen keys used in `CONTENT.projects` (`cover`, `decisions[].screens`, `gallery`…) must exist there. A key with no entry renders "Screens coming soon", so a project still opens before its images exist.
- **Adding or replacing screens:** export WebP (quality ~82, phone screens at native 390–412 px wide, tablet at 1280 wide) into `assets/works/<slug>/`, add `` "key": [w, h, `alt`] `` to that project's `WORKS_MEDIA` entry, then use the key in `CONTENT.projects`. Screens taller than the frame scroll inside it automatically.
- **Sources:** the WebP files are copies. The originals in `assets/XPRO DELIVERY`, `assets/WMS`, `assets/SAGAN MOBILE APP` and `assets/SukiTuki` are untouched. Real names, photos, customer names, company names and plates were redacted **on the copies only**. Unused originals still contain them, so don't publish those folders.
- **Overlay behaviour:** it has its own scroller (`data-lenis-prevent`), and the page's Lenis is stopped while it's open. It also has a sticky Close button plus Escape, a focus trap, and inert page content, and focus returns to the row's button on close. Opening from a row animates the mockup into the hero (FLIP). Reduced motion gets a fade.
- **Draft and placeholder marks:** `[[DRAFT: …]]` is copy written from what the screens show; confirm or rewrite it. `[[PLACEHOLDER: …]]` is a fact only you know. Both render as visible highlighted marks until replaced. Ctrl-F `[[DRAFT` and `[[PLACEHOLDER` in `index.html`.

### Still to fill in (works)

- **Years:** Sagana app and Sagana website context lines and overview, `[[PLACEHOLDER: year]]`.
- **Tools and team:** Xpro, WMS, SukiTuki tools. Team size for Xpro, WMS, Sagana app and Sagana website. Your role on the Sagana website.
- **"How I built it":** the stack for Xpro and WMS (left as `[[PLACEHOLDER: stack…]]`; not guessed).
- **Every `[[DRAFT]]`:**
  - the problem and its "before" state for each project;
  - the "why" of each key decision;
  - the iteration notes, including which version came first for the Sagana pairs;
  - the Xpro block-out caption;
  - the style-guide reasoning;
  - all "What I learned" bullets.
- **SukiTuki:** the client brief. There are no iterations: no earlier versions were supplied, so the section is hidden. Add an `iterations` entry if you have them.

Still to supply in `CONTENT`: `location` city, the five project names/summaries, `email` (empty = copy-email button hidden) and `socials` (empty = links hidden; a commented-out LinkedIn / GitHub / Dribbble / Behance template sits right above it).

## Simplifications versus the spec

- **Footer watermark:** rises as specified, but the letter-spacing change is not implemented.
- **Hero card and testimonial auto-advance:** use `setTimeout` (4 s), not `gsap.delayedCall`.
- **Hover springs:** approximated with `gsap.to` and `back.out`, not the spec's eases exactly.
- **Copy email:** copies the placeholder string until the real address is set in `CONTENT.email`.
- **Banner:** shown on `file://` as well as localhost. Otherwise a `file://` visit fails silently.

## Verification

Checked in headless Chrome over `http://127.0.0.1`:

- Section order: home, proof, works, about, skills, manifesto, contact (footer). One `<h1>`.
- No console errors or exceptions at 1440 px, 360 px, or with `prefers-reduced-motion`.
- No horizontal scroll at 360 px or 1440 px.
- Loader: counts from 000, then slides away. Removed only after the count reaches 100.
- Hero (CDP mouse/touch input, SwiftShader WebGL): the halftone renders, the cursor push was captured live, and the click burst was checked on a slowed copy of the page. No exceptions at 1440 px, on an emulated phone with a tap, or with reduced motion.
- Works (Nov 6): no errors and no horizontal overflow at 1440, 1024 and 390. Every case study opens from its row button and by deep link (`#work/<slug>`), Escape closes it, and focus returns to the row. Back and Forward close and reopen it. The slider responds to arrow keys. A tall WMS screen scrolls inside the tablet frame. The overlay scrolls with the wheel while the page stays put. Tab stays inside the overlay. "Work" lights up in the header while in Works.
- Request modal: opens from the header, locks scroll, makes the rest of the page inert, and closes on Escape.
- Menu: opens with 5 items.

Not verified: real devices, touch input, screen readers, measured colour contrast, the halftone on a phone-sized screen, real-GPU performance, and how the motion *feels* (screenshots can't show motion).
