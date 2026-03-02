---
description: "Scaffold a new component, hook, or store file following aWizard GUI conventions."
---

Scaffold a new file for the awizard-gui project. Follow these rules:

1. **Components** → `src/components/ComponentName.tsx`
   - Default export, typed props interface, functional component
   - Include a `// TODO:` block for unfinished parts
   - Use Tailwind CSS 4 classes, Discord dark palette CSS variables

2. **Hooks** → `src/hooks/useHookName.ts`
   - Named export with `use` prefix
   - TypeScript return type annotation

3. **Store slices** → `src/store/sliceName.ts`
   - Use Zustand `create<T>()` pattern
   - Export the hook (e.g., `useSliceName`)

4. **Lib utilities** → `src/lib/fileName.ts`
   - Pure functions, typed inputs/outputs
   - No React imports

Every file must have a header comment:
```ts
// ─────────────────────────────────────────────────────────────────
//  FileName — brief purpose description
// ─────────────────────────────────────────────────────────────────
```
