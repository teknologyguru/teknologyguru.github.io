# djvc.net — Migration Guide: Static VM → Cloudflare Workers

This walks you from where you are now (old iWeb HTML, hosted on a Linux VM
at home, repo at `teknologyguru/teknologyguru.github.io`) to a Cloudflare
Workers (static assets) site that redeploys automatically every time you
`git push`.

---

## 0. Before you start

- You already have: a GitHub repo, a Cloudflare account, and `djvc.net`
  registered/managed in that Cloudflare account.
- New files provided: `index.html`, `style.css`, `wrangler.jsonc`,
  `skyline-bg.jpg` (background photo), `favicon.svg` (tab icon), `_headers`
  (cache rules for the photo). 
- Replace the alias placeholder in `index.html` (`REPLACE_ME@simplelogin.co`)
  with your real SimpleLogin alias before pushing.
- Decide now whether you want to keep the old `gallery/` photos directory
  in the repo. It's dropped from the new homepage's nav, but the files can
  stay in the repo and still be reachable at `djvc.net/gallery/` if you
  don't delete them — nothing forces you to remove them.

---

## 1. Update the GitHub repo

1. Clone the repo locally if you don't already have it:
   ```
   git clone https://github.com/teknologyguru/teknologyguru.github.io.git
   cd teknologyguru.github.io
   ```
2. Remove the old iWeb-generated cruft (safe to delete once you're happy
   with the new site):
   - `djvc.net_files/` (iWeb's asset dump)
   - `Scripts/` (if it's iWeb-generated; check first if anything in there
     is actually yours)
   - Old `_config.yml` if it was only there for Jekyll defaults you're not
     using anymore — a plain static site doesn't need Jekyll at all, so
     this file becomes optional either way.
3. Copy in the six new files (`index.html`, `style.css`, `wrangler.jsonc`,
   `skyline-bg.jpg`, `favicon.svg`, `_headers`) at the repo root,
   overwriting the old `index.html`.
4. Commit and push:
   ```
   git add -A
   git commit -m "Replace iWeb site with new minimal static homepage"
   git push origin master
   ```

At this point GitHub Pages (if still enabled on the repo) would already
serve the new design at `teknologyguru.github.io` — but the goal is to
move hosting to Cloudflare Pages instead, so the next steps set that up.

---

## 2. Create the Cloudflare Worker (git-connected static site)

Cloudflare has moved its default git-connected static hosting flow from the
older "Pages" product to "Workers" with static assets. The workflow is the
same idea — connect the repo, auto-deploy on push — just a renamed path.

1. Add a `wrangler.jsonc` file (provided) to the **repo root**. This tells
   Cloudflare where your static files live:
   ```jsonc
   {
     "name": "djvc-dot-net",
     "compatibility_date": "2026-08-01",
     "assets": {
       "directory": "./",
       "not_found_handling": "404-page"
     }
   }
   ```
   Commit and push this alongside `index.html` and `style.css` — without
   it, the deploy step below will fail.
2. Log into the Cloudflare dashboard → **Workers & Pages** →
   **Create application** → **Create a Worker** → **Connect to Git** (or
   similar wording; Cloudflare's UI has iterated on this a few times).
3. Authorize Cloudflare's GitHub App if prompted, and select the
   `teknologyguru/teknologyguru.github.io` repository.
4. Configure the project:
   - **Project name**: whatever you like (e.g. `djvc-dot-net`) — this
     becomes part of the default `*.workers.dev` URL, it does not affect
     your custom domain.
   - **Build command**: leave blank (nothing to build for a static site)
   - **Deploy command**: `npx wrangler deploy` (this is the default and
     is correct)
5. Click **Deploy**. Cloudflare will pull the repo and publish it to a
   generated `*.workers.dev` URL. Confirm it loads and looks right before
   moving on.

---

## 3. Point djvc.net at the Worker

1. Open the Worker's settings → **Domains & Routes** (this replaces the
   old Pages "Custom domains" tab).
2. Click **Add** → **Custom domain** and enter `djvc.net` (and
   `www.djvc.net` if you want that to work too).
3. Because `djvc.net` is already on this same Cloudflare account, Cloudflare
   will create the necessary DNS record automatically — accept that.

   **If you get an error like "Hostname 'djvc.net' already has externally
   managed DNS records (A, CNAME, etc.)"**: your zone still has the old
   record pointing at your home VM. Go to **DNS → Records** in the
   dashboard, find the record where **Name** is `djvc.net` (or `@`) — it's
   likely an A record pointing at your VM's IP — and delete it (and any
   `www` record if you're also connecting that). Leave MX/email records
   alone. Then retry adding the custom domain.
4. Wait for the DNS/SSL status to show **Active** (usually a couple of
   minutes, sometimes a bit longer for certificate issuance). Note that
   even once it's active, your browser or a fetch tool may still show
   stale/cached content for a bit — hard-refresh or check in an incognito
   window before assuming something's wrong.

---

## 4. Verify auto-deploy

1. Make a small, visible change locally (e.g., tweak the tagline text).
2. Commit and push it:
   ```
   git add -A
   git commit -m "Test auto-deploy"
   git push origin master
   ```
3. In the Cloudflare dashboard, watch the Worker's **Deployments** tab — a
   new deployment should kick off within seconds of the push and finish
   in well under a minute for a static site like this.
4. Reload `djvc.net` (hard-refresh / incognito to dodge cache) and confirm
   the change is live.

From here on, every `git push` to `master` = a new deployment. No manual
steps, no server to maintain.

**Unrelated git error you might hit:** if a commit fails with something
like `error: Couldn't get agent socket?` / `fatal: failed to write commit
object`, that's a local SSH-agent/commit-signing issue on your machine —
nothing to do with this deploy setup. Either start your SSH agent
(`eval "$(ssh-agent -s)"` then `ssh-add`) or, if you don't need signed
commits, disable signing for this repo: `git config commit.gpgsign false`.

---

## 5. Decommission the old VM (optional, once you're confident)

- Stop the web server process/service on the home VM.
- Keep the VM around for a bit as a fallback if you want, but there's no
  functional reason to keep serving traffic from it once Cloudflare Pages
  is confirmed working and DNS has fully propagated.
- Double check nothing else on your network (email, monitoring, etc.)
  depends on that VM also being reachable at `djvc.net` before you fully
  retire it.

Note: Cloudflare's dashboard naming has shifted between "Pages" and
"Workers" more than once recently as they've unified the two products —
if the exact button labels above don't match what you see, look for
"Connect to Git" / "Domains & Routes" under Workers & Pages; the
underlying flow (repo → auto-deploy → custom domain) stays the same.

---

## Ongoing workflow

To update the site going forward:
```
# edit index.html / style.css locally
git add -A
git commit -m "describe your change"
git push origin master
```
Cloudflare picks it up automatically — that's the whole deploy process.