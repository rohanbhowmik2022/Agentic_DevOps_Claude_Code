# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static HTML/CSS personal portfolio website. Single-page site with sections for About, Services, Courses, Books, Community, and Contact. Will be deployed to AWS using S3 and CloudFront, provisioned with Terraform.

**Stack:** Pure static HTML5 + CSS3. No JavaScript, No build step, No framework. Font Awesome 6.5.0 is loaded via CDN.

## Repository Layout

- `index.html` — Single-page portfolio (Navbar, Hero, About, Services, Courses, Books, Community, Contact, Footer)
- `privacy.html`, `terms.html` — Legal pages with their own inline `<style>` blocks (do **not** link `style.css`)
- `style.css` — All styles for `index.html` only
- `images/` — Static assets (`logo.png`, `image.png` hero, `signature.png`, book covers, `dmi-course.jpg`)
- `.github/workflows/deploy.yml` — GitHub Actions deploy pipeline (S3 sync + CloudFront invalidation)
- `README.md` — DMI deployment context, NOT a product readme

## Architecture & Conventions

### Page structure (index.html)
Sections are anchored by `id` for in-page nav: `home`, `about`, `services`, `courses`, `book`, `community`, `contact`. Navbar uses `onclick="goToSection('...')"` and the hamburger calls `toggleMenu()`. **Both functions and the `<span id="year">` auto-update are referenced in markup but the `<script>` block is missing** — if asked to "fix" mobile menu / year / smooth nav, add an inline `<script>` near `</body>` defining them.

### Legal pages
`privacy.html` and `terms.html` are fully self-contained with inline CSS and do **not** share `style.css`. Keep this pattern when editing — do not extract their styles into the shared file.

### CSS conventions
- Mobile-first responsive breakpoints at `900px` (navbar collapse), `768px` (hero), `600px` (about/legal)
- No JavaScript in this project
- All images go in the images/ directory
- Brand accent color: `#facc15` (yellow); primary text `#111` / `#111827`; dark backgrounds for legal pages
- Duplicate `@keyframes fadeUp` and `.contact-btn` rules exist in `style.css` — minor existing duplication, leave unless asked to clean up

### Deployment (`.github/workflows/deploy.yml`)
- Triggers on push to `main`
- Uses **OIDC** (`aws-actions/configure-aws-credentials@v4`) — no long-lived AWS keys
- Syncs repo to `s3://pravinmishradmi-site-production` in `eu-north-1` with `--delete` (destructive — anything not in the repo is removed)
- Excludes: `.git/`, `.github/`, `.claude/`, `terraform/`, `.mcp.json`, `*.md`, `CLAUDE.md`
- Invalidates CloudFront distribution `E3V6O6MRE2E21P` on `/*` after each deploy
- Role ARN: `arn:aws:iam::533267262133:role/github-actions-deploy`

## Common Commands

This is a no-build project, so the typical dev cycle is just **edit → refresh browser**. Useful commands from the repo root:

```bash
# Local preview (Python 3)
python -m http.server 8000

# Local preview (Node)
npx serve .

# Local preview (Nginx, mirrors prod)
sudo cp -r . /var/www/html/ && sudo nginx -s reload
```

There are no test, lint, or build commands — verify changes by opening the page in a browser and checking responsive breakpoints (900/768/600 px).

## DMI Deployment Context (from README)

If working on this as a DMI student exercise, the **mandatory ownership proof** must be added to the footer before deploying:

```html
<p><strong>Deployed by:</strong> DMI Cohort 2 | Rahul Sharma | Group 4 | Week 1 | 16-01-2026</p>
```

This line must be visible in the deployment screenshot submission.

## Things to Watch Out For

- **No JS file exists** but `index.html` calls `toggleMenu()`, `goToSection()`, and references `id="year"`. The mobile menu therefore does not currently work. Don't "fix" this by removing the `onclick` handlers unless explicitly asked — the missing `<script>` is the actual gap.
- **`--delete` on S3 sync** is destructive. Never add files to the repo root that should not be deployed (e.g., local notes, scratch files); either add them to the `aws s3 sync --exclude` list in `deploy.yml` or keep them out of the repo.
- **External images**: course cards in `index.html` reference `https://img-c.udemycdn.com/...` — these are third-party, can break or change, and require internet access at view time.
- **PII / contact info** in `index.html` (email, phone, address in legal pages) is real contact info for the site owner — edit with care.
