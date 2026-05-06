# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page GitHub Gist Manager with two deployment modes:

- **`gist-manager.html`** — Standalone SPA. Token in localStorage, calls GitHub API directly from the browser.
- **`_worker.js`** — Cloudflare Pages deployment. Token in httpOnly cookie, all API calls proxied through the worker. The file is auto-detected by Cloudflare Pages as a Workers function.


## Technology Preferences

- HTML, JavaScript, CSS, and TailwindCSS are the preferred stack.
- All SPA code should be in a single HTML file (`gist-manager.html`).
- The worker runs on Cloudflare's runtime — no Node.js APIs, only Web Standard APIs.

## Architecture — SPA (`gist-manager.html`)

- **Auth**: GitHub Personal Access Token stored in `localStorage`, verified via `GET /user`
- **API**: Direct `fetch` to `https://api.github.com` with `Authorization: token <token>`
- **State**: Module-scoped variables inside an IIFE
- **Rendering**: DOM manipulation via `innerHTML` + event delegation, no virtual DOM

## Architecture — Worker (`_worker.js`)

- **Serves**: The SPA as an inline HTML template literal (no KV or static assets needed)
- **Auth**: `POST /login` verifies token against GitHub, sets `gh_token` cookie (httpOnly, Secure, SameSite=Lax)
- **API proxy**: All `/api/*` requests are forwarded to `https://api.github.com/*` with the token from the cookie
- **Logout**: `POST /logout` clears the cookie

The inline HTML in the worker is a modified version of `gist-manager.html` with these differences:
- No `localStorage`, no `token` variable
- `api()` calls `/api` + path instead of `https://api.github.com` + path
- Login/logout use `POST /login` and `POST /logout` endpoints
- `initAuth()` tries `/api/user` — if it fails, shows the login modal

## Key patterns (shared)

- Gist list: `renderGistList()` rebuilds sidebar; search filters `allGists` into `gists`
- Edit mode: toggled by `isEditing` flag; `renderContent()` shows inputs vs static text
- File rename: `editFileNames` tracks names; old names not in the list are marked `null` on save
- Save: new gists → `POST /gists`, existing → `PATCH /gists/{id}`; `setSaving()` prevents double-submit
- Draft gists: `isNew: true`, added to `allGists` on create, removed from arrays on cancel
