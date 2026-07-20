---
name: PC Builder HTML data bugs
description: Root causes of blank content in pc-builder.html and how they were fixed.
---

## Bugs in pc-builder.html (all in the DB object array literals)

1. **Missing commas** between object entries (`}` followed by `{id:` with no `,`):
   - r319, ca329, ca331, ca334 entries
   - Caused `SyntaxError: Unexpected token '{'` → entire `<script>` block failed to parse → `init()` never called

2. **Double comma** (`},,`):
   - c141 entry ended with `},,` creating a sparse array hole (`undefined` element)
   - `renderComponents()` called `item.id` on the undefined element → `TypeError` crash mid-render

**Why:** The original HTML file (`debugmi530-del/pc-builder-html`) was hand-edited and had copy-paste errors in the data arrays.

## Fix approach
- Served via `artifacts/pc-builder/public/app.html` (static, no Vite processing)
- `artifacts/pc-builder/index.html` is a JS redirect: `window.location.replace('app.html')`
- **Why public/**: Vite plugins (`@replit/vite-plugin-dev-banner` loads Tailwind CDN, cartographer injects scripts) interfered with the standalone HTML

## Detection method
```bash
# Check syntax
node --check /tmp/script.js
# Find double commas
grep -n "},,$" file.html
# Find missing commas between entries
node -e "lines where line ends with } and next line starts with {id:"
```
