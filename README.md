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

## Local Usage

1. Open `gist-manager.html` directly in a browser
2. Create a [GitHub Personal Access Token](https://github.com/settings/tokens) with the `gist` scope
3. Paste the token and click 确认

The token is stored in `localStorage`. All API calls go directly from the browser to `api.github.com`.

## Cloudflare Deployment

The project supports one-click Git deployment via Cloudflare Pages.

### Setup

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com) → **Workers & Pages**
2. Click **Create** → **Pages** → **Connect to Git**
3. Authorize and select your fork of this repository
4. Configure build settings:
   - **Build command**: (leave empty)
   - **Output directory**: (leave empty)
5. Click **Save and Deploy**

Cloudflare Pages automatically detects `_worker.js` in the repository root and runs it as a Worker. Every push to the default branch triggers a new deployment.

### How it works

- The token is stored in an **httpOnly, Secure** cookie — never exposed to browser JavaScript
- All GitHub API calls are proxied through the worker — the browser only talks to the worker's domain
- The worker reads the token from the cookie, attaches the `Authorization` header, and forwards requests to `api.github.com`

### Routes

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Serves the SPA |
| `POST` | `/login` | Verifies token against GitHub, sets cookie |
| `POST` | `/logout` | Clears the cookie |
| `*` | `/api/*` | Proxies to `api.github.com` with auth |

## Token Permissions

The token requires the **gist** permission to:
- Read gists and comments
- Create and update gists
- Post comments

## Technical Stack

| Layer | Technology |
|-------|-----------|
| UI | TailwindCSS (CDN) |
| Logic | Vanilla JavaScript |
| API | GitHub REST API v2022-11-28 |
| Auth (local) | Token in `localStorage` |
| Auth (deployed) | Token in httpOnly cookie, server-side proxy |

## File Structure

```
.
├── gist-manager.html   # Standalone SPA — open in browser

├── _worker.js          # Cloudflare Pages entry point
├── CLAUDE.md           # Claude Code guidance
└── README.md           # This file
```
