# Synteric Robotics — Marketing Site

Static marketing site for Synteric Robotics. No build step, no bundler, no CI.

**Live:** https://syntericrobotics.com/ (the old `robleuscaesar.github.io/syntericrobotics-site/` URL now 301s here)

## Deploy

GitHub Pages serves this repo's `main` branch from the root directory.

```bash
git add -A
git commit -m "Describe the change"
git push origin main
```

Pages rebuilds automatically on push — live in roughly one minute. Hard-refresh
(`Ctrl+Shift+R`) if you still see the old page; the CDN caches aggressively.

Check deploy status:

```bash
gh api repos/RobleusCaesar/syntericrobotics-site/pages/builds/latest --jq '.status, .error.message'
```

## Layout

```
index.html                     Landing page          -> /
request-access/index.html      Pilot request form    -> /request-access/
support.js                     Claude Design runtime (generated — do not edit)
_ds/
  react.production.min.js      Self-hosted React 18.3.1 UMD
  react-dom.production.min.js  Self-hosted ReactDOM 18.3.1 UMD
  industry-411e43cb.../        Exported design system (styles.css, bundle, manifest)
img/                           7 WebP page images + og-card.jpg (social preview)
favicon.svg, favicon-32.png    Site mark
.nojekyll                      Serve files verbatim (see below)
```

## How the pages work

These are **not** hand-written HTML. They are Claude Design exports: `support.js` is a
client-side runtime that reads the `<x-dc>` template, the `{{ }}` bindings, the
`sc-if` / `sc-for` elements and the `DCLogic` class in each page, and renders the result
with React. Consequences worth knowing:

- **The page is blank until JavaScript runs.** There is no server-side or pre-rendered
  HTML. Crawlers that don't execute JS see an empty shell — which is why the `<title>`,
  meta description and Open Graph tags live in a real `<head>`, not inside the template.
- **`loading="lazy"` does not work here.** The runtime applies `loading` after `src`,
  which leaves images permanently deferred — they never load, even scrolled into view.
  Verified in-browser. Don't re-add it. WebP conversion already cut images ~92%, so
  eager loading is cheap.
- **React is self-hosted** from `_ds/`, not a CDN, so the site has no third-party runtime
  dependency. `support.js`'s `loadReactUmd()` short-circuits when `window.React` and
  `window.ReactDOM` already exist, so the two `<script>` tags in `<head>` are all that's
  needed — the generated runtime is untouched. The `integrity=` hashes on those tags are
  the ones `support.js` itself ships for these exact builds.
- Google Fonts (Archivo, IBM Plex Mono) is the one remaining third-party request.

## Conventions

- **Relative asset paths only.** `./img/x.webp` from the root page, `../img/x.webp` from
  `/request-access/`. Never root-absolute `/img/x.webp`. The site lives at the subpath
  `/syntericrobotics-site/` and now lives at the domain root — root-absolute
  paths work at exactly one of those and silently 404 at the other.
  - The **only** intentional absolute URLs are `og:url` and `og:image`, because OG
    crawlers do not reliably resolve relative URLs. Both are flagged in-file and must be
    updated whenever the canonical domain changes.
- **No build step.** Files are committed exactly as served.
- **`.nojekyll` stays at root.** Without it Jekyll ignores any path starting with an
  underscore — which would silently delete the entire `_ds/` directory, including React.

## Form backend

`/request-access/` posts to **FormSubmit.co** (no account required):

- `POST https://formsubmit.co/ajax/<recipient>` with `Content-Type: application/json`
  and `Accept: application/json`. The recipient is set in `request-access/index.html`.
- Payload is the six form fields plus `_subject: "Synteric pilot request"` and a
  `markets` string. **The MARKETS buttons are component state, not form inputs** — they
  are serialized explicitly. If you touch the form, keep that or market data is lost
  silently on every lead.
- Success is FormSubmit's JSON `success: "true"` (a *string*, not a boolean). Anything
  else shows the error state and leaves the form up for retry.
- Double-submit is guarded by a `sending` flag.

### Activation status

FormSubmit consumes the **first** submission to a new address to send a confirmation
email; that first one is never delivered. Sequence:

1. Submit once live → confirmation email arrives.
2. Click the activation link in that email.
3. Submit again → this one actually lands.

**Recipient alias — outstanding.** After activation FormSubmit issues a random alias for
the address. Swap the raw email in `request-access/index.html` for that alias and push,
so the address isn't sitting in public page source for scrapers. Until that swap the
plain address is visible in the deployed HTML.

## Roadmap

### Phase 1 — Pipeline (done)

Repo → GitHub → Pages, verified live.

### Phase 2 — Real site (done)

Claude Design export integrated. Images converted PNG → WebP at native resolution
(26.42 MB → 1.18 MB, 37–39.5 dB PSNR, visually lossless). `uploads/` dropped — all six
files were byte-identical duplicates of `img/` and unreferenced. `image-slot.js` dropped —
64 KB of canvas-editor scaffolding with no `<image-slot>` element on either page. Header
nav gained a `max-width: 599px` rule because the four-item nav overflowed a 375px viewport
and clipped the REQUEST ACCESS CTA to zero visible pixels.

### Phase 3 — Custom domain (done)

Cut over to `syntericrobotics.com` — `CNAME` added in `74da45f`, DNS configured, HTTPS
enforced, domain state verified. `www` and the old `robleuscaesar.github.io/...` URL both
redirect to the apex. The absolute `og:url` / `og:image` tags were repointed to the new
domain afterwards.

The relative-path convention did its job: the move from the `/syntericrobotics-site/`
subpath to the domain root needed **zero** path edits.

## Maintenance

**The files in this repo are the source of truth.** They began as a Claude Design export
(`Synteric Landing.dc.html` / `Synteric Request Access.dc.html` in Rob's Downloads) that was
transformed once on import — paths rewritten, pages split, form wired, nav breakpoint added.
That transform was a one-shot. Edit the files here directly; do not regenerate from the
export, or every change since import is lost.

### Outstanding

- **FormSubmit recipient alias.** The form still posts to the plain address rather than the
  random alias FormSubmit issues after activation, so the address is readable in deployed
  page source. Swap it in `request-access/index.html` and push.
