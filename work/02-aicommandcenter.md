# Case study 2 — aicommandcenter.pro: a site an AI can read, deployed by one verified command

**Type:** case study, own product · **Status:** live ·
**Disclosure:** built by Dejan Milošević; AI agents wrote part of the code under his direction and
review.

## The problem

Two failures, both invisible to a person looking at the screen:

1. The site was a single-page app. A crawler received an almost empty shell — nothing to read, so
   nothing to cite.
2. `npm run build` replaced `index.html` with that same empty shell. Every deploy silently
   destroyed the rendered page that made the site readable. Deploying looked successful and was
   not.

## The solution

One command, and it refuses to call itself finished until the live result is proven:

1. **Build** the app from source.
2. **Render** the built page in headless Chrome and keep the HTML as the served page.
3. **Inject** the structured data (Person, Organization, ProfessionalService with offers, FAQ) from
   a single source file, so prices cannot drift between the page and the schema.
4. **Upload only what changed**, by checksum, with the page last so it never points at a missing
   asset.
5. **Verify**: bytes on the origin, the text as served through the CDN, every page and asset, the
   redirects, and the bot routing.

## Architecture

```
source → vite build → headless Chrome snapshot → JSON-LD injection
      → md5 diff → tar/scp → staging on server → backup → install → verify live
```

The verification compares content, not status codes: hashes on the origin, canonicalised text
through Cloudflare (with the CDN's own transformations undone), asset byte counts, and the
snapshot that seven bot user agents receive.

## Technologies

React, TypeScript, Vite, nginx, Cloudflare, headless Chrome via CDP, Node.js.

## The role

Sole architect and builder; author of the deployment pipeline and of the rule that a deploy is
only finished when the live page proves it. Agent-authored commits are disclosed and visible.

## Demonstration

Live: https://aicommandcenter.pro/ · the pipeline: `deploy.mjs` in the project.

## Verifiable results

- **68 checks pass** on a live deploy: origin bytes, public text, 12 pages, assets, bot routing,
  redirects, sitemap, robots.
- A real production bug was found by the pipeline itself: `/about` and six service pages were
  served as `application/octet-stream`, so browsers downloaded them instead of showing them
  (proven by a headless-Chrome download, then fixed in nginx and verified on the origin and through
  the CDN).
- The page is served as `text/html` in standards mode with one JSON-LD block; the visible text and
  the built text hash identically.

## Limits — and the one that matters

- **Structure is not discovery.** The site scores 100/100 on structural readiness, while the
  measurement lab found it in **0 of 16** real AI answers across four platforms. A readable site is
  a precondition, not a result.
- The pipeline proves that what was built is what is served; it does not prove that anyone finds
  it.
