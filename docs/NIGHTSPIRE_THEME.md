# Nightspire Theme System — aWizard DeFi Ecosystem  

> Unified theme system across `chia-cfmm`, `chia-treasure-chest`, `chia-perps`
> 
> **Status:** ✅ Validated and standardized across all projects
> **Last Updated:** March 5, 2026

---

## CSS Variable Reference

### Core Color Tokens
```css
:root {
  /* Background layers */
  --bg:           rgba(10, 12, 24, 0.88);  /* Main background */
  --bg-deep:      #060810;                 /* Deep space background */
  --bg-card:      rgba(25, 15, 20, 0.93);  /* Card background */
  --bg-primary:   #060810;                 /* Primary surface */
  --bg-secondary: rgba(25, 15, 20, 0.93);  /* Secondary surface */
  --bg-tertiary:  rgba(35, 25, 35, 0.9);   /* Tertiary surface */

  /* Text colors */
  --text-color:   #e0f7ff;    /* Primary text (legacy) */
  --text-primary: #e0f7ff;    /* Primary text */
  --text-normal:  #e0f7ff;    /* Normal text */
  --text-muted:   #8ba3b0;    /* Muted text */

  /* Accent system */
  --accent:       #00d9ff;    /* Primary accent (cyan) */
  --accent-hover: #00b8d9;    /* Accent hover state */
  --border-color: #00d9ff;    /* Border accent */

  /* Status colors */
  --success:      #4ade80;    /* Success state */
  --danger:       #f23f42;    /* Error/danger state */
  --warning:      #f0b232;    /* Warning state */
}
```

### Glow System Tokens
```css
:root {
  /* Glow colors */
  --glow-inner:   #ff6600;    /* Inner orange glow */
  --glow-outer:   #00d9ff;    /* Outer cyan glow */
  --glow-far:     #1a082b;    /* Far purple glow */

  /* Interactive states */
  --sel-fill:     #d7fcfe;    /* Selection fill */
  --sel-text:     #020d0d;    /* Selection text */

  /* Glow parameters */
  --border-w:     2.5px;      /* Border width */
  --glow-r:       10;         /* Glow radius */
  --glow-s:       30;         /* Glow spread */
  --glow-f:       36;         /* Far glow size */
  --glow-b1:      5;          /* Blur 1 */
  --glow-b2:      8;          /* Blur 2 */
}
```

---

## Utility Classes

### `.glow-card`
Primary container with Nightspire glow effect:
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
Glowing text with multi-layer shadow:
```css
.glow-text {
  color: var(--text-color);
  text-shadow:
    0 0 6px #fff,
    0 0 18px rgba(255,255,255,0.7),
    0 0 40px var(--glow-inner);
}
```

### `.glow-btn`
Interactive button with hover animations:
```css 
.glow-btn {
  padding: 10px 24px;
  font-weight: bold;
  color: var(--text-color);
  background: linear-gradient(90deg, var(--bg) 60%, var(--bg-deep) 100%);
  border: var(--border-w) solid var(--border-color);
  border-radius: 8px;
  /* Multi-layer glow shadows */
  transition: all 0.3s ease;
}
.glow-btn:hover {
  background: var(--sel-fill);
  color: var(--sel-text);
  /* Enhanced glow on hover */
}
```

### `.glow-input`
Form inputs with focus glow:
```css
.glow-input {
  padding: 10px 16px;
  color: var(--text-color);
  background: var(--bg-deep);
  border: var(--border-w) solid var(--border-color);
  border-radius: 6px;
  /* Glow shadow + focus enhancement */
}
```

---

## Implementation Status

| Project                | Nightspire Tokens | Glow Classes | .env.example | Status |
|------------------------|-------------------|--------------|--------------|--------|
| `chia-cfmm`           | ✅ Complete       | ✅ Complete  | ✅ Complete  | ✅ Ready |
| `chia-treasure-chest` | ✅ Complete       | ✅ Complete  | ✅ Complete  | ✅ Ready |  
| `chia-perps`          | ✅ Complete       | ✅ Complete  | ✅ Complete  | ✅ Ready |

**All projects:** Running on respective ports with consistent theme rendering ⚡

---

## Development Servers

| Project                | Port | URL |
|------------------------|------|-----|
| `chia-cfmm`           | 5173 | http://localhost:5173 |
| `chia-treasure-chest` | 5175 | http://localhost:5175 |
| `chia-perps`          | 5174 | http://localhost:5174 |

---

## Usage Guidelines

### Consistency Rules
1. **Always use CSS variables** — never hardcode colors
2. **Test glow classes** — `.glow-card`, `.glow-text`, `.glow-btn` should work across all projects
3. **Maintain hover states** — all interactive elements need hover feedback
4. **Respect the gradient** — background uses deep space → card hierarchy

### Component Integration
```tsx
// ✅ Good - Use utility classes
<div className="glow-card">
  <h2 className="glow-text">Nightspire Title</h2>
  <button className="glow-btn">Mystical Action</button>
</div>

// ❌ Bad - Inline styles or hardcoded colors
<div style={{backgroundColor: '#060810', border: '1px solid #00d9ff'}}>
```

### Future Expansion
- **Radix UI integration:** All projects use `@radix-ui/themes` — ensure Radix components inherit Nightspire variables
- **Animation library:** Consider `framer-motion` for advanced glow animations
- **Dark mode:** Current implementation IS the dark theme — light mode TBD

---

## Quest Completion ✅

**Validation complete:** All 3 DeFi frontends render consistently with the Nightspire theme system. Ready for Phase 1 wallet testing!