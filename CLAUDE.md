# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page GitHub Gist Manager with two deployment modes:

- **`gist-manager.html`** — Standalone SPA. Token in localStorage, calls GitHub API directly.
- **`worker.js`** — Cloudflare Worker. Token in httpOnly cookie, worker proxies all API calls to GitHub. Serves the same SPA with modified JS (no localStorage, calls `/api/*` endpoints).

## Technology Preferences

- HTML, JavaScript, CSS, and TailwindCSS are the preferred stack.
- All code should be delivered in a single HTML file unless otherwise specified.

## Architecture

- **Auth**: GitHub Personal Access Token stored in `localStorage`, verified via `GET /user`
- **API**: All calls to `https://api.github.com` with header `Authorization: token <token>`, `Accept: application/vnd.github+json`, `X-GitHub-Api-Version: 2022-11-28`
- **State**: Module-scoped variables inside an IIFE — `token`, `gists`, `allGists`, `selectedGistDetail`, `isEditing`, `editContent`, `editFileNames`, `editPublic`, `activeFileName`
- **Rendering**: DOM manipulation via `innerHTML` + event delegation, no virtual DOM
- **Styling**: TailwindCSS CDN, dark theme (`bg-gray-950`), Inter + JetBrains Mono fonts

## Key patterns

- Gist list: `renderGistList()` rebuilds sidebar from `gists` array; search filters `allGists` into `gists`
- Edit mode: toggled by `isEditing` flag; `renderContent()` checks it to show inputs vs static text
- File rename: `editFileNames` tracks current filenames; on input change, `editContent` keys are migrated; on save, old names not in `editFileNames` are marked `null` (deleted)
- Save: new gists use `POST /gists`, existing use `PATCH /gists/{id}`; loading state via `setSaving()` prevents double-submit
- Draft gists: `isNew: true`, `id: '__new__'`, shown in list with blue badge; cancelled drafts are removed from arrays
