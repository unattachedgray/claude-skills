---
name: bootstrap-site
description: Scaffold a new unattached.me site with Next.js 14, PM2, Cloudflare Tunnel, and interlink integration. Use when the user wants to create a new web project for their unattached.me subdomain.
argument-hint: [app-name]
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# Bootstrap unattached.me Site

You are scaffolding a new site for the **unattached.me** family of projects. These run on **Windows 11 Pro WSL2 Ubuntu** with an **RTX 4090**, managed by **PM2** (configured to resume on reboot), and exposed via a **Cloudflare Tunnel daemon** (also managed by PM2).

## Environment Reference

| Item | Value |
|------|-------|
| **Machine** | Windows 11 Pro, WSL2 Ubuntu, RTX 4090 |
| **Projects root** | `/mnt/d/dev/` |
| **PM2** | Manages all processes, auto-resume on reboot via `pm2 startup` + `pm2 save` |
| **Cloudflare Tunnel** | Single daemon in PM2 (`cloudflared`), config at `~/.cloudflared/config.yml` |
| **Tunnel ID** | `e1f4b315-9b1b-4037-8e44-19d27b28ff5c` |
| **Tunnel cert** | `~/.cloudflared/cert.pem` |
| **Credentials file** | `~/.cloudflared/e1f4b315-9b1b-4037-8e44-19d27b28ff5c.json` |
| **Interlink repo** | `~/dev/interlink` (also at `/mnt/d/dev/interlink`) |
| **Interlink script URL** | `https://whoyou.unattached.me/interlink.js?v=7` |
| **Package manager** | **npm** (never yarn) |
| **Node heap** | `--max-old-space-size=4096` (WSL2 V8 crash workaround) |

## Step 1 — Gather Inputs

Use `AskUserQuestion` to collect the following. Pre-fill from `$ARGUMENTS` if provided.

**Required questions:**

1. **App name** (kebab-case, e.g. `my-cool-app`) — used for folder name, PM2 process name, package.json name
2. **Site title** — human-readable title for `<title>` and metadata
3. **Site description** — one-liner for SEO meta description
4. **Subdomain** — the `*.unattached.me` subdomain (e.g. `myapp` → `myapp.unattached.me`)
5. **Production port** — unique port for PM2 production server (check existing ports in `~/.cloudflared/config.yml` to avoid conflicts; current: 3000, 3100, 4174)
6. **Dev port** — unique port for `npm run dev` (avoid existing dev ports)
7. **Key dependencies** — any extra npm packages needed (e.g. `three`, `zustand`, `@google/generative-ai`)
8. **Interlink key** — the short key for this site in the interlink cube (e.g. `myapp`)

If `$ARGUMENTS` is provided, use it as the app name and infer sensible defaults, but still confirm with the user.

## Step 2 — Scaffold the Next.js Project

Create the project at `/mnt/d/dev/{app-name}/`:

```bash
cd /mnt/d/dev
npx create-next-app@14 {app-name} --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*" --use-npm
```

After scaffolding, verify the directory exists and `package.json` was created.

## Step 3 — Configure package.json Scripts

Update `package.json` scripts to match the unattached.me pattern:

```json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev -p {dev-port}",
    "build": "NEXT_DIST_DIR=.next-prod NODE_OPTIONS='--max-old-space-size=4096' next build",
    "start": "NEXT_DIST_DIR=.next-prod next start",
    "lint": "next lint"
  }
}
```

## Step 4 — Create next.config.mjs

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  distDir: process.env.NEXT_DIST_DIR || ".next",
};

export default nextConfig;
```

This enables separate dev (`.next`) and production (`.next-prod`) build directories.

## Step 5 — Configure Tailwind + Globals

Update `tailwind.config.ts` to include dark mode support:

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  darkMode: "class",
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        background: "var(--background)",
        foreground: "var(--foreground)",
      },
    },
  },
  plugins: [],
};
export default config;
```

Set up `app/globals.css` with light/dark theme support:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  color: #171717;
  background: #ffffff;
  font-family: Arial, Helvetica, sans-serif;
}

html.dark body,
.dark body {
  color: #ededed;
  background: #0a0a0a;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Step 6 — Set Up app/layout.tsx with Interlink

Copy the Geist font files from an existing project if available, or use system fonts.

```tsx
import type { Metadata, Viewport } from "next";
import Script from "next/script";
import "./globals.css";

export const viewport: Viewport = {
  width: "device-width",
  initialScale: 1,
  maximumScale: 5,
  userScalable: true,
};

export const metadata: Metadata = {
  title: "{site-title}",
  description: "{site-description}",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="font-sans antialiased" suppressHydrationWarning>
        {children}
        <Script
          src="https://whoyou.unattached.me/interlink.js?v=7"
          data-current="{interlink-key}"
          strategy="lazyOnload"
        />
      </body>
    </html>
  );
}
```

## Step 7 — Create a Starter app/page.tsx

Create a minimal but functional landing page that shows the site title and confirms the app is running. Keep it simple — the user will build from here.

## Step 8 — Create ecosystem.config.js (PM2)

```javascript
module.exports = {
  apps: [
    {
      name: "{app-name}",
      script: "node_modules/.bin/next",
      args: "start -p {prod-port}",
      cwd: "/mnt/d/dev/{app-name}",
      instances: 1,
      exec_mode: "fork",
      env: {
        NODE_ENV: "production",
        NODE_OPTIONS: "--max-old-space-size=4096",
        NEXT_DIST_DIR: ".next-prod",
      },
      max_memory_restart: "500M",
      autorestart: true,
    },
  ],
};
```

Use `fork` mode with 1 instance by default. The user can switch to `cluster` mode with more instances later.

## Step 9 — Create .env.example and .env

```env
# Add your environment variables here
# GEMINI_API_KEY=
```

Create both `.env.example` (tracked) and `.env` (gitignored) with the same template.

## Step 10 — Set Up .gitignore

```
/node_modules
/.next/
/.next-prod/
/out/
/build
/data/cache/
.DS_Store
*.pem
npm-debug.log*
.env
.env*.local
.vercel
*.tsbuildinfo
next-env.d.ts
```

## Step 11 — Create CLAUDE.md

Write a project-specific `CLAUDE.md` with:

- Project overview (name, description, purpose)
- Tech stack (Next.js 14, App Router, Tailwind, etc.)
- Commands (`npm run dev`, `npm run build`, `npm run start`, `npm run lint`)
- Dev port and production port
- Architecture notes (placeholder for the user to fill in)
- Environment variables
- Conventions:
  - Lowercase kebab-case file names
  - Keep files under 200 lines
  - npm (never yarn)
- Deployment:
  - Build: `npm run build`
  - PM2: `pm2 start ecosystem.config.js`
  - Cloudflare Tunnel handles HTTPS
- Interlink integration reference:
  - Community linking managed at `~/dev/interlink` (also at `/mnt/d/dev/interlink`)
  - To add this site to the interlink cube, edit `/mnt/d/dev/interlink/interlink.js` — add an entry to the `SITES` array
  - After updating interlink.js, bump `?v=N` in all sites' script tags and rebuild/restart them
  - The interlink cube is hosted from `whoyou.unattached.me` (source file is symlinked from `/mnt/d/dev/politmap/public/interlink.js` → `/mnt/d/dev/interlink/interlink.js`)

## Step 12 — Update Cloudflare Tunnel Config

Read `~/.cloudflared/config.yml` and add the new ingress rule **before** the catch-all `http_status:404` rule:

```yaml
  - hostname: {subdomain}.unattached.me
    service: http://localhost:{prod-port}
```

**IMPORTANT**: Ask the user for confirmation before modifying the tunnel config.

After updating, tell the user to:
1. Restart cloudflared: `pm2 restart cloudflared`
2. Add a CNAME DNS record in Cloudflare dashboard: `{subdomain}` → `e1f4b315-9b1b-4037-8e44-19d27b28ff5c.cfargotunnel.com`

## Step 13 — Install Dependencies and Build

```bash
cd /mnt/d/dev/{app-name}
npm install
# Install any extra dependencies the user requested
npm install {extra-deps}
npm run build
```

If the build fails due to V8 crash (common in WSL2), retry once.

## Step 14 — Initialize Git

```bash
cd /mnt/d/dev/{app-name}
git init
git add -A
git commit -m "Initial commit: {app-name} scaffold for {subdomain}.unattached.me"
```

## Step 15 — Print Summary

After everything is done, print a clear summary:

```
=== {app-name} scaffolded successfully ===

Location:     /mnt/d/dev/{app-name}
Dev server:   npm run dev → http://localhost:{dev-port}
Prod server:  pm2 start ecosystem.config.js → http://localhost:{prod-port}
Public URL:   https://{subdomain}.unattached.me (after DNS + tunnel restart)

Next steps:
1. Add DNS CNAME: {subdomain} → e1f4b315-9b1b-4037-8e44-19d27b28ff5c.cfargotunnel.com
2. Restart tunnel: pm2 restart cloudflared
3. Start dev: cd /mnt/d/dev/{app-name} && npm run dev
4. Start prod: cd /mnt/d/dev/{app-name} && npm run build && pm2 start ecosystem.config.js
5. Save PM2 state: pm2 save
6. Add to interlink: edit /mnt/d/dev/interlink/interlink.js SITES array
7. Build your app!
```

## Rules

- **ALWAYS use npm**, never yarn
- **Lowercase kebab-case** for all files and folders
- **Port conflicts**: Always check existing ports before assigning
- **Don't start PM2 or restart cloudflared** automatically — just configure and instruct
- **Keep it minimal** — scaffold the skeleton, let the user build the features
- **Geist font**: Copy from `/mnt/d/dev/whyhate/app/fonts/` if the user wants it, otherwise use system fonts
