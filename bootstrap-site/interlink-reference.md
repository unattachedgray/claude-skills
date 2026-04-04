# Interlink Integration Reference

## What is Interlink?

A 3D rotating cube widget that provides cross-site navigation between all unattached.me sites. It floats in the bottom-right corner of every site.

## Architecture

- **Source file**: `/mnt/d/dev/interlink/interlink.js`
- **Hosted at**: `https://whoyou.unattached.me/interlink.js` (via symlink in politmap project)
- **Symlink**: `/mnt/d/dev/politmap/public/interlink.js` → `/mnt/d/dev/interlink/interlink.js`
- **All sites load from the single hosted URL** — no local copies needed
- **Cache busting**: Bump `?v=N` query param when widget is updated

## Current Sites (as of last check)

| Key | Label | URL |
|-----|-------|-----|
| `home` | Blog | `https://unattached.me` |
| `whoyou` | Who Are You? | `https://whoyou.unattached.me` |
| `whyhate` | Why Do They Hate Us? | `https://whyhate.unattached.me` |
| `tess` | Chess Chess Tess | `https://tess.unattached.me` |

## Adding a New Site

1. Edit `/mnt/d/dev/interlink/interlink.js`
2. Add entry to the `SITES` array:
   ```javascript
   { key: "myapp", label: "My App Name", url: "https://myapp.unattached.me" },
   ```
3. Bump the `?v=N` in ALL sites' `<Script>` tags
4. Rebuild and restart all sites via PM2

## Integration by Framework

### Next.js (App Router)
```tsx
// app/layout.tsx
import Script from "next/script";

// Inside <body>, after {children}:
<Script
  src="https://whoyou.unattached.me/interlink.js?v=7"
  data-current="myapp"
  strategy="lazyOnload"
/>
```

### React + Vite
```html
<!-- index.html, before </body> -->
<script src="https://whoyou.unattached.me/interlink.js?v=7" data-current="myapp" defer></script>
```

### Plain HTML / WordPress
```html
<script src="https://whoyou.unattached.me/interlink.js?v=7" data-current="myapp" defer></script>
```

## data-current Attribute

Set to the site's `key` from the SITES array. The current site's cube face renders with inverted colors (light bg) and is not clickable.
