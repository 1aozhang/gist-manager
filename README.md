# GitHub Gist Manager

A single-page web application for managing GitHub Gists — list, view, create, edit, and comment on gists using the GitHub REST API.

## Features

- **Token-based authentication** — uses a GitHub Personal Access Token
- **Gist list** — browse all gists in a scrollable sidebar with search/filter
- **View mode** — view gist content with file tabs for multi-file gists
- **Edit mode** — edit gist description, file names, and file content inline
- **Create new gists** — create gists locally first, only saved to GitHub on explicit save
- **File management** — add, rename, and delete files within a gist while editing
- **Visibility toggle** — switch between public and secret gists during edit
- **Comments** — view and post comments on existing gists
- **Saving indicator** — loading overlay and disabled buttons prevent duplicate submissions

## Deployment Options

### Option A: Static HTML

Open `gist-manager.html` directly in a browser. The token is stored in `localStorage`. API calls go directly to `api.github.com`.

### Option B: Cloudflare Worker

Deploy `worker.js` as a Cloudflare Worker:

```bash
npx wrangler deploy worker.js
```

In this mode:
- The token is stored in an **httpOnly, Secure** cookie — never exposed to JavaScript
- All GitHub API calls are proxied through the worker — the browser never contacts GitHub directly
- The worker verifies tokens on login and injects the `Authorization` header server-side

## Token Permissions

The token requires the **gist** permission to:
- Read gists and comments
- Create and update gists
- Post comments

## Technical Stack

| Layer | Technology |
|-------|-----------|
| UI | TailwindCSS (CDN) |
| Logic | Vanilla JavaScript (ES6+) |
| API | GitHub REST API v2022-11-28 |
| Auth (static) | Token in `localStorage` + `Authorization` header |
| Auth (worker) | Token in httpOnly cookie + server-side proxy |

## File Structure

```
.
├── gist-manager.html   # Standalone SPA (localStorage token)
├── worker.js           # Cloudflare Worker (cookie-based token)
├── CLAUDE.md           # Claude Code guidance
└── README.md           # This file
```
