# TechNexi Website

Marketing site for TechNexi and its flagship product, Atlas — plus the Ventures, Careers, and Contact pages.

Live at [technexi.com](https://technexi.com).

## Tech stack

Plain static HTML/CSS/JS — no build step, no bundler, no framework. Each page is a self-contained `.html` file with inline styles and a small `<script type="text/x-dc">` block for interactivity (form state, modals, etc.), hydrated at runtime by a single shared runtime file, `support.js`.

## Project structure

```
public/                  ← deployed site (this is what actually goes live)
  index.html               Home (Atlas + Early Access modal)
  contact/index.html       Contact page
  careers/index.html       Careers page + application modal
  ventures/index.html      Ventures page
  support.js                Shared component runtime
  CNAME                     Custom domain config for GitHub Pages

*.dc.html, index.html      ← root-level source copies (edited via the page-builder tool)
.github/workflows/         ← GitHub Actions deploy workflow
```

The root-level `.dc.html` files are editable source copies kept in sync with their `public/` counterparts. Only `public/` is actually deployed — when editing page logic, update both.

## Running locally

The site uses root-relative-within-directory paths, so it needs to be served (not opened via `file://`), with `public/` as the web root:

```bash
cd public
python3 -m http.server 8080
```

Then visit `http://localhost:8080/`, `/contact/`, `/careers/`, `/ventures/`.

## Deployment

Deploys automatically via GitHub Actions (`.github/workflows/deploy-pages.yml`) on every push to `main`, using GitHub Pages' native Actions-based deployment (no build step — it just uploads `public/` as-is).

The custom domain (`technexi.com`) is configured via the `CNAME` file and DNS records pointing at GitHub Pages, managed through Cloudflare (the domain's authoritative DNS).

## Form integrations

The Contact, Early Access, and Careers forms all submit to a shared external backend (a Google Apps Script Web App) that writes submissions to a spreadsheet and sends email notifications to the site owners. The endpoint URL is configured as `GOOGLE_SCRIPT_URL` in each page's script block.

Requests are sent as `GET` with the payload as query parameters (chosen for reliability — Apps Script's POST response relay proved unreliable in testing), with a `formType` field routing to the correct handler:

| Form | `formType` | Fields |
|---|---|---|
| Contact | `contact` | `name`, `email`, `message`, `userAgent` |
| Early Access | `earlyAccess` | `email`, `company`, `userAgent` |
| Careers | `career` | `role`, `name`, `email`, `portfolio`, `whyRole`, `userAgent` |

The backend responds with `{"success": true}` or `{"success": false, "error": "..."}`, which the form UI uses to show its success/error state.
