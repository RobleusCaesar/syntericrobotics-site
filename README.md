# Synteric Robotics — Marketing Site

Static marketing site for Synteric Robotics. Plain HTML/CSS, no build step, no framework.

**Live:** https://robleuscaesar.github.io/syntericrobotics-site/

Currently a holding page. The real site lands in Phase 2 (below).

## Deploy

There is no build and no CI. GitHub Pages serves this repo's `main` branch from the root directory.

```bash
git add -A
git commit -m "Describe the change"
git push origin main
```

Pages rebuilds automatically on push — live in roughly one minute. Hard-refresh
(`Ctrl+Shift+R`) if you still see the old page; the CDN caches aggressively.

Check deploy status any time:

```bash
gh api repos/RobleusCaesar/syntericrobotics-site/pages/builds/latest --jq '.status, .error.message'
```

## Conventions

- **Relative asset paths only.** Use `./assets/logo.svg`, `styles.css` — never
  root-absolute `/assets/logo.svg`. The site lives at the subpath
  `/syntericrobotics-site/` today and moves to the domain root in Phase 3.
  Root-absolute paths work at exactly one of those and silently 404 at the other;
  relative paths survive both.
- **No build step.** Plain static files committed as-is, unless an exported design
  genuinely requires tooling.
- **`.nojekyll` stays at root.** It tells Pages to serve files verbatim rather than
  running them through Jekyll — which would otherwise ignore any file or directory
  whose name starts with an underscore (e.g. `_ds/`).

## Roadmap

### Phase 1 — Pipeline (done)

Repo → GitHub → Pages, verified live. Holding page only: `index.html`, `.nojekyll`,
this README.

### Phase 2 — Real site (next)

Replace the placeholder with the exported Claude Design build (Rob provides the files).
Before pushing, audit **every** asset path — `href`, `src`, `url()` in CSS, and any
JS-constructed paths — against the relative-path rule above. Verify locally by serving
the directory, not by opening `index.html` from disk.

### Phase 3 — Custom domain (`syntericrobotics.com`) — on Rob's go-ahead only

1. Add a `CNAME` file at repo root containing exactly `syntericrobotics.com`.
2. DNS at the registrar:
   - Apex `@` → four A records pointing at the GitHub Pages IPs
     (`185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`).
   - `www` → CNAME to `robleuscaesar.github.io`.
3. In Settings → Pages, set the custom domain and wait for the DNS check to pass,
   then tick **Enforce HTTPS** (the certificate can take up to ~24h to issue).
