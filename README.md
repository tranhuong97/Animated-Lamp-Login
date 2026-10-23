# Lamp Login Animation

An interactive login page built with plain HTML, CSS, and JavaScript. The page starts in a dark room with a desk lamp. Pull the lamp's string to turn it on: the room lights up, fireflies drift across the screen, and a glassmorphism login card slides into view.

No frameworks, no build step, no dependencies (other than the Google Font *Outfit*).

---

## Features

- **Draggable lamp string**: pull it down to toggle the lamp on or off, using mouse, touch, or pen (Pointer Events).
- **Lighting effects**: lamp glow, light beam, desk light pool, and ambient room light, all done with CSS.
- **Fireflies**: 18 randomly generated glowing dots that float around while the lamp is on.
- **Glassmorphism login card**: blurred translucent card with username, email, and password fields, plus Google and GitHub buttons.
- **Pure CSS icons**: the user, mail, and lock input icons are drawn with pseudo-elements, so there are no image or icon-font dependencies.
- **Responsive layout**: side-by-side on desktop, stacked on screens 900px wide or narrower.

---

## Project Structure

```
.
├── index.html   # Page markup (lamp, login form, fireflies container)
├── style.css    # All styling, transitions, and animations
└── script.js    # Lamp toggle, string drag logic, firefly generation
```

---

## Getting Started

1. Put the three files in the same folder.
2. Open `index.html` in any modern browser.

Optionally, serve the folder locally:

```bash
# Python
python -m http.server 8000

# or Node
npx serve .
```

Then visit `http://localhost:8000`.

> An internet connection is needed to load the *Outfit* font from Google Fonts. Without it, the page falls back to the system `sans-serif` font.

---

## How It Works

### State

The whole scene is driven by a single class on the `#room` element:

| State | Class on `#room` | Result |
|-------|------------------|--------|
| Lamp off | *(none)* | Dark background, login card hidden |
| Lamp on | `on` | Glow, beam, room light, and fireflies visible; login card revealed |

Every visual change is a CSS transition keyed off `.room.on`, so JavaScript only needs to toggle one class.

### Pulling the string (`script.js`)

1. **`pointerdown`** on `#stringHandle` starts a drag and captures the pointer.
2. **`pointermove`** translates the handle and redraws the SVG string:
   - Horizontal movement is scaled by `0.3` and clamped to ±35px.
   - Vertical movement is only counted downward, scaled by `0.6`, and capped at 180px.
3. **`pointerup` / `pointercancel`** ends the drag. If the handle was pulled more than **30px** down (`dragY > 30`), the lamp toggles. The handle then springs back to its resting position.

The string itself is an SVG `<path>` updated with `M 0 0 L {x} {80 + y}`, where `80` is the string's resting length.

### Functions

| Function | Purpose |
|----------|---------|
| `setLamp(on)` | Sets the state, toggles the `on` class, and creates or clears fireflies. |
| `updateString(x, y)` | Redraws the SVG string path to follow the handle. |
| `resetHandle()` | Returns the handle and string to the resting position. |
| `pointerDown / pointerMove / pointerUp` | Drag handlers for the string handle. |
| `random(min, max)` | Returns a random float in the range. |
| `createFireflies()` | Clears and regenerates 18 firefly elements with random size, timing, and path. |

### Fireflies

Each firefly is a `<span class="firefly">` with randomized CSS custom properties:

- `--duration` (35–65s) and `--delay` (0–3s)
- `--x1…--x5` and `--y1…--y5`: five waypoints in `vw`/`vh` units

The `fireflyMove` keyframes move the dot between those waypoints while pulsing its opacity. Fireflies are removed when the lamp is turned off and regenerated (with new random paths) each time it turns on.

---

## Customization

| What to change | Where |
|----------------|-------|
| Number of fireflies | `for (let i = 0; i < 18; i++)` in `createFireflies()` |
| Firefly size / speed | `random(3, 7)` and `random(35, 65)` in `createFireflies()` |
| Pull distance needed to toggle | `dragY > 30` in `pointerUp()` |
| Drag sensitivity | The `0.3` and `0.6` multipliers in `pointerMove()` |
| Light color | `rgba(255, 220, 100, …)`, `rgba(255, 230, 150, …)` and similar values in `style.css` |
| Accent / button color | `.sign-in { background: #ffd600; }` |
| Lamp size | `.lamp-container { transform: scale(1.1); }` and `.lamp-stem` height |
| Card width | `.login-card { max-width: 440px; }` |
| Font | The Google Fonts `<link>` in `index.html` and `:root { font-family }` |

---

## Adding Real Login Logic

The form currently does nothing on submit. In `script.js`:

```js
loginForm.addEventListener('submit', (e) => {
  e.preventDefault();
  // Add your real login logic here.
});
```

Replace the comment with your own handler, for example:

```js
loginForm.addEventListener('submit', async (e) => {
  e.preventDefault();

  const [username, email, password] = [...loginForm.querySelectorAll('input')]
    .map((input) => input.value);

  const res = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, email, password }),
  });

  if (res.ok) {
    // redirect or update UI
  }
});
```

The Google and GitHub buttons are `type="button"` and have no behavior yet; attach your OAuth handlers to `.social-btn`.

---

## Responsive Behavior

| Condition | Behavior |
|-----------|----------|
| Width > 900px | Lamp on the left, login card on the right |
| Width ≤ 900px | Stacked layout: lamp on top (55vh, min 430px), card below, page scrolls vertically |
| Height ≤ 650px and width > 900px | Lamp scaled to 0.8 and card padding/gaps reduced |

---

## Browser Support

Works in current versions of Chrome, Edge, Firefox, and Safari. It relies on:

- Pointer Events
- CSS `backdrop-filter` (with `-webkit-` prefix), `clip-path`, and custom properties
- Optional chaining (`?.`) in JavaScript

If `backdrop-filter` is unsupported, the card still displays with a plain translucent background.

---

## Known Limitations and Ideas

- **Accessibility**: the string handle can only be operated by pointer. Consider adding `role="button"`, `tabindex="0"`, and keyboard support (Enter/Space to toggle), and respecting `prefers-reduced-motion` for the fireflies and transitions.
- **Unused CSS**: `.prompt` and `@keyframes promptArrow` are styled but there is no matching element in `index.html`. Add a hint such as "Pull the string" or remove the rules.
- **Form validation**: fields have no `required` attributes or validation yet.
- **Social icons**: Google and GitHub icons are text placeholders (`G` and `●`). Replace them with SVGs for production.
- **Heading text**: the page title in `<title>` is "Lamp Login Animation | CodeByGaurav"; update it for your own project.

---

## License

No license file is included. Add one (for example MIT) before distributing or reusing this project.
