# djvc.net

Source for [djvc.net](https://djvc.net) — Jack V Catalano's personal landing
page. Static HTML/CSS, no build step.

## Files

- `index.html` — page markup
- `style.css` — styling (monochrome, background photo of the Rochester
  skyline)
- `skyline-bg.jpg` — background photo
- `wrangler.jsonc` — Cloudflare Workers config (tells it to serve this
  repo's static files)

## Deploying

Hosted on Cloudflare Workers (static assets), connected directly to this
repo. Pushing to `master` triggers an automatic redeploy — no manual steps.

See `DEPLOY_GUIDE.md` for full setup details.
