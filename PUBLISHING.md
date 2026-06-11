# Tuxera Marketing Dashboard — Publishing Guide

This document explains how the dashboard is hosted, updated, and published. Keep this file in the source repository so the process is never lost.

## The two-repo setup

**Repo 1 — Source (Internal):** `github.com/tuxera-main/tuxera-marketing-dashboard`
Holds all versioned dashboard files (v43.html, v44.html, ...) and the GitHub Actions workflow. Only visible to Tuxera GitHub users. This is where version history lives.

**Repo 2 — Site (Public):** `github.com/tuxera-main/tuxera-marketing-dashboard-site`
Holds only one file: `dashboard.html`. GitHub Pages serves it publicly. No source history, no sensitive code.

**Live URL (share this with colleagues):**
https://tuxera-main.github.io/tuxera-marketing-dashboard-site/dashboard.html

## How to publish a new dashboard version

1. Get the new version file from Claude, named with the next version number (for example v45.html)
2. Go to Repo 1 → Add file → Upload files
3. Upload the new vXX.html file, commit directly to main
4. Done. GitHub Actions automatically:
   - Finds the highest-numbered vXX.html file
   - Copies it to Repo 2 as dashboard.html
   - The live URL updates within 1-2 minutes

Old versions stay in Repo 1 as history. Never delete them unless cleaning up.

## How the automation works

The workflow file lives at `.github/workflows/publish.yml` in Repo 1. It triggers on any HTML file pushed to main, picks the latest version by number, and pushes it to Repo 2 using a personal access token.

## The deploy token (important!)

The workflow authenticates with a GitHub personal access token (classic) stored as a repository secret:

- **Secret name:** `SITE_REPO_TOKEN`
- **Location:** Repo 1 → Settings → Secrets and variables → Actions
- **Token type:** Classic personal access token with `repo` scope
- **Created by:** varjo-droid (Maija)
- **Expiration:** check your GitHub token settings; if deploys start failing with 403 errors, the token has likely expired

### If deploys fail with "Permission denied" or 403:

1. Create a new classic token: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token
2. Note: `dashboard-deploy`, Scope: check `repo` only
3. Copy the token
4. Repo 1 → Settings → Secrets and variables → Actions → edit `SITE_REPO_TOKEN` → paste new token
5. Actions tab → failed run → Re-run all jobs

Note: fine-grained tokens require Tuxera org admin approval. Classic tokens work without approval.

## Known facts and decisions

- Repo 2 is public because not all Tuxera employees have GitHub access. The dashboard HTML contains no API keys (users enter their own Anthropic key).
- The Zapier webhook URL is visible in the public file. Acceptable risk for now; can be moved to a GitHub Secret injected at deploy time if needed.
- Translation request and stale content alerts go via Zapier webhook to email (maija.varjoranta@ and mark.fletcher@tuxera.com). Both use form-encoded fields: title, team, url, message, alert_type.
- Teams channel notifications are not possible via Zapier (no approval). Future option: server-side script with Microsoft Graph API once an internal server is available.
- Dashboard data refresh (sitemaps, CTA scan, WordPress content) happens client-side when the dashboard is open, via corsproxy.io. Full server-side sync is a future improvement.

## Version history

| Version | Date | Changes |
|---------|------|---------|
| v43 | 2026-06-10 | First hosted version. CTA scanner, localisation, Monday.com links, Zapier email alerts |
| v44 | 2026-06-11 | Fixed stale content alert format so Zapier emails actually send |
