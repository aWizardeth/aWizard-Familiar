# Skill: The Nightspire Design System

> The canonical theme specification for all aWizard / Arcane BOW sites.
> Source of truth: `src/index.css` in https://github.com/aWizardxch/The-Nightspire
>
> **All new frontends** (chia-cfmm, chia-treasure-chest, chia-perps, bow-app) MUST import
> or replicate this token set to maintain visual consistency across the ecosystem.

---

## Philosophy

Dark arcane aesthetic — deep navy/obsidian backgrounds, cyan-orange glow borders, electric
accent on interactive elements. The "glow" is the brand: every card, button, and input
pulses with bioluminescent energy from a dark void.

---

## CSS Custom Properties (`:root`)

Paste this block verbatim into any new site's `src/index.css` after `@import "tailwindcss"`:

```css
@import "tailwindcss";

/* ── Glow theme variables ──────────────────────────────────── */
:root {
  /* Core glow colours */
  --text-color:   #e0f7ff;
  --border-color: #00d9ff;
  --glow-inner:   #ff6600;   /* orange inner glow */
  --glow-outer:   #00d9ff;   /* cyan outer glow */
  --sel-fill:     #d7fcfe;   /* selected/hover fill */
  --sel-text:     #020d0d;   /* text on sel-fill */

  /* Border & glow intensity knobs */
  --border-w: 2.5px;
  --glow-r:   10;   /* inner radius multiplier (px) */
  --glow-s:   30;   /* spread multiplier */
  --glow-far: #1a082b;
  --glow-f:   36;
  --glow-b1:  5;
  --glow-b2:  8;

  /* Background layers */
  --bg:       rgba(10, 12, 24, 0.88);
  --bg-deep:  #060810;                   /* darkest — page bg */
  --bg-card:  rgba(25, 15, 20, 0.93);   /* card surfaces */

  /* Semantic aliases (used across components) */
  --bg-primary:   #060810;
  --bg-secondary: rgba(25, 15, 20, 0.93);
  --bg-tertiary:  rgba(35, 25, 35, 0.9);
  --text-primary: #e0f7ff;
  --text-normal:  #e0f7ff;
  --text-muted:   #8ba3b0;
  --accent:       #00d9ff;
  --accent-hover: #00b8d9;
  --success:      #4ade80;
  --danger:       #f23f42;
  --warning:      #f0b232;

  /* Discord component aliases */
  --background-secondary:        rgba(25, 15, 20, 0.93);
  --background-modifier-accent:  rgba(0, 217, 255, 0.15);
}
```

---

## Body Defaults

```css
body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
  background: var(--bg);
  background-color: var(--bg-deep);
  color: var(--text-color);
  font-family: 'gg sans', 'Noto Sans', Helvetica, Arial, sans-serif;
  overflow-x: hidden;
}
```

---

## Utility Classes

These 4 classes are the design system primitives. Use them everywhere instead of
writing custom box-shadows.

### `.glow-card`
Base surface for all content panels:
```css
.glow-card {
  background: var(--bg-card);
  border: var(--border-w) solid var(--border-color);
  border-radius: 12px;
  box-shadow:
    0 0 calc(var(--glow-r) * 1px) var(--glow-inner),
    0 0 calc(var(--glow-s) * 1px) calc(var(--glow-b1) * 1px) var(--glow-outer),
    0 0 calc(var(--glow-f) * 1px) calc(var(--glow-b2) * 1px) var(--glow-far);
}
```

### `.glow-text`
Headers and highlighted labels:
```css
.glow-text {
  color: var(--text-color);
  text-shadow:
    0 0 6px #fff,
    0 0 18px rgba(255,255,255,0.7),
    0 0 40px var(--glow-inner);
}
```

### `.glow-btn` / `.glow-btn.active`
Primary interactive buttons:
```css
.glow-btn {
  padding: 10px 24px;
  font-weight: bold;
  color: var(--text-color);
  background: linear-gradient(90deg, var(--bg) 60%, var(--bg-deep) 100%);
  border: var(--border-w) solid var(--border-color);
  border-radius: 8px;
  box-shadow:
    0 0 calc(var(--glow-r) * 1px) var(--glow-inner),
    0 0 calc(var(--glow-s) * 1px) calc(var(--glow-b1) * 1px) var(--glow-outer);
  cursor: pointer;
  transition: background 0.2s, color 0.2s, box-shadow 0.2s;
}
.glow-btn:hover {
  background: var(--sel-fill);
  color: var(--sel-text);
  box-shadow:
    0 0 calc(var(--glow-r) * 2px) var(--glow-inner),
    0 0 calc(var(--glow-s) * 1.5px) calc(var(--glow-b1) * 1px) var(--glow-outer),
    0 0 6px rgba(255,255,255,0.4);
}
.glow-btn.active {
  background: var(--sel-fill);
  color: var(--sel-text);
  border-color: var(--border-color);
  box-shadow:
    0 0 calc(var(--glow-s) * 1px) var(--glow-outer),
    0 0 calc(var(--glow-f) * 1px) calc(var(--glow-b2) * 1px) var(--glow-far);
}
```

### `.glow-input` / `.glow-input:focus`
Form inputs and text areas:
```css
.glow-input {
  padding: 10px 16px;
  color: var(--text-color);
  background: var(--bg-deep);
  border: var(--border-w) solid var(--border-color);
  border-radius: 6px;
  box-shadow:
    0 0 calc(var(--glow-r) * 1px) var(--glow-inner),
    0 0 calc(var(--glow-s) * 1px) calc(var(--glow-b1) * 1px) var(--glow-outer);
  outline: none;
  transition: border 0.2s, box-shadow 0.2s;
}
.glow-input:focus {
  box-shadow:
    0 0 calc(var(--glow-r) * 2px) var(--glow-inner),
    0 0 calc(var(--glow-s) * 1.5px) calc(var(--glow-b1) * 1px) var(--glow-outer);
}
```

### Scrollbar (Discord-style)
```css
::-webkit-scrollbar       { width: 8px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background-color: var(--bg-tertiary); border-radius: 4px; }
```

---

## Element Colour Palette

Use these as inline styles or in Tailwind. Both hex (for CSS) and Tailwind class equivalents
are listed.

| Element    | Hex        | Tailwind card classes                   | Tailwind text colour            |
|------------|------------|-----------------------------------------|---------------------------------|
| Fire       | `#ff6b35`  | `bg-orange-500/80 border-orange-400`    | `text-orange-400`               |
| Water      | `#00b4d8`  | `bg-blue-500/80 border-blue-400`        | `text-blue-400`                 |
| Nature     | `#4caf50`  | `bg-green-500/80 border-green-400`      | `text-green-400`                |
| Electric   | `#ffd600`  | `bg-yellow-400/80 border-yellow-300`    | `text-yellow-300`               |
| Shadow     | `#9c27b0`  | `bg-purple-700/80 border-purple-500`    | `text-purple-400`               |
| Ice        | `#b3e5fc`  | `bg-cyan-400/80 border-cyan-300`        | `text-cyan-300`                 |
| Spirit     | `#f8bbd0`  | `bg-pink-400/80 border-pink-300`        | `text-pink-300`                 |
| Arcane     | `#00d9ff`  | `bg-indigo-500/80 border-indigo-400`    | `text-indigo-300`               |
| Corruption | `#e53935`  | `bg-red-900/80 border-red-700`          | `text-red-400`                  |
| Neutral    | `#aaa`     | `bg-zinc-600/80 border-zinc-500`        | `text-zinc-400`                 |

---

## Status / Semantic Colours (inline style)

```tsx
// Connected / success state
style={{ color: '#4ade80' }}  // green
// Error / disconnected
style={{ color: '#f23f42' }}  // discord danger red
// Accent highlight
style={{ color: '#00d9ff' }}  // cyan
// Muted / secondary text
style={{ color: 'var(--text-muted)' }}  // #8ba3b0
// Warning
style={{ color: '#f0b232' }}  // amber
// Wallet connect gradient button
background: 'linear-gradient(135deg, #00d9ff, #ff6600)'
```

---

## Common Inline Patterns

### Element badge
```tsx
<span
  style={{
    background: `${elementHex}20`,
    color: elementHex,
    border: `1px solid ${elementHex}40`,
  }}
  className="inline-block px-1 py-0.5 rounded font-bold text-xs"
>
  {element}
</span>
```

### Accent card (e.g. "selected" state)
```tsx
style={{ background: 'rgba(0,217,255,0.08)', border: '1px solid rgba(0,217,255,0.3)', color: '#00d9ff' }}
```

### Danger card
```tsx
style={{ background: 'rgba(242,63,66,0.12)', border: '1px solid rgba(242,63,66,0.4)', color: '#f87171' }}
```

### Success card
```tsx
style={{ background: 'rgba(74,222,128,0.08)', border: '1px solid rgba(74,222,128,0.3)', color: '#4ade80' }}
```

### Purple/arcane card (lobbies, actions)
```tsx
style={{ background: 'rgba(139,92,246,0.18)', border: '1px solid rgba(139,92,246,0.5)', color: '#c4b5fd' }}
```

### Loading dots (purple, animate-bounce)
```tsx
{[0,1,2].map((i) => (
  <span key={i} className="w-2 h-2 rounded-full bg-purple-500 animate-bounce"
    style={{ animationDelay: `${i * 150}ms` }} />
))}
```

---

## Typography

- **Font**: `'gg sans', 'Noto Sans', Helvetica, Arial, sans-serif`
- **Text hierarchy**:
  - Page title: `glow-text text-lg font-bold`
  - Section header: `text-sm font-semibold` + `var(--text-color)`
  - Body: `text-sm` + `var(--text-color)`
  - Muted: `text-xs` + `var(--text-muted)` (`#8ba3b0`)
  - Monospace: `font-mono text-xs` (addresses, hashes, debug)

---

## Vite Config Requirements

Every new project that is a Vite+React site needs:

```ts
// vite.config.ts
import tailwindcss from '@tailwindcss/vite';
plugins: [react(), tailwindcss()],
```

And `@import "tailwindcss"` as the first line of `index.css` (not `@tailwind base/components/utilities`).

---

## Where This Theme Lives

| Repo / Project              | CSS file                   | Status   |
|-----------------------------|----------------------------|----------|
| `The-Nightspire`            | `src/index.css`            | ✅ Source |
| `chia-treasure-chest`       | `src/index.css` — TODO     | ⬜ Pending |
| `chia-cfmm`                 | `src/index.css` — TODO     | ⬜ Pending |
| `chia-perps` (new)          | `src/index.css` — TODO     | ⬜ Pending |
| `bow-app` (Next.js)         | `app/globals.css`          | ✅ Aligned |

---

## How aWizard Uses This Skill

When scaffolding a new frontend component:
1. Check if it needs a surface → use `.glow-card` class
2. Check if it's a heading → use `.glow-text` class
3. Check if it's a CTA button → use `.glow-btn` or the gradient inline style
4. Check if it's an input → use `.glow-input` class
5. Status indicators → use success/danger/accent colours from this file
6. Never hardcode background colours without checking this palette first

When creating a new project's `src/index.css`:
- Start with the full `:root` block above
- Add `body` defaults
- Add the 4 utility classes
- Add scrollbar styles
- Done — Tailwind handles the rest via `@import "tailwindcss"`

---

_Source: https://github.com/aWizardxch/The-Nightspire — `src/index.css`_
_Last updated: 2026-03-05_
