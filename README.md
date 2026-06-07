# RemotePad website

Plain static HTML/CSS for the RemotePad landing page. No build step, no
JavaScript framework, no node_modules. Edit, commit, push — GitHub Pages
publishes in ~30 seconds.

## What lives here

| Path | Purpose |
|---|---|
| `index.html` | Landing page |
| `privacy.html` | Privacy Policy |
| `terms.html` | Terms of Service |
| `404.html` | Not-found page (GitHub Pages auto-serves it) |
| `mac/index.html` | Permanent install short link. Meta-refresh redirects to the latest GitHub Releases DMG. |
| `styles.css` | Single design-token CSS file, mirrors the iOS palette, supports light + dark via `prefers-color-scheme` |
| `assets/icon.svg` | App icon (placeholder — swap for the commissioned PNG/SVG before public launch) |

## Deploying to GitHub Pages (one-time, ~10 min)

GitHub Pages serves any static site for free, with HTTPS. We use it as the
public face of the project — the source code stays in a private repo.

### 1. Push the website to a **public** GitHub repo

This directory should become the **root** of its own repo. Suggested name:
`remotepad-website` (matches the placeholders in the code).

```
# from inside the website/ directory:
cd /Users/aditya/Desktop/RemoteControlMacApp/remote-control-suite/website
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:YOUR_USERNAME/remotepad-website.git
git push -u origin main
```

> The repo **must be public** for Pages to serve it on the free GitHub
> plan. Private-repo Pages requires GitHub Pro/Team. Since the website
> contains no secrets, public is the right default.

### 2. Enable GitHub Pages

On GitHub:

1. `remotepad-website` → **Settings → Pages**
2. **Source:** *Deploy from a branch*
3. **Branch:** `main`, **Folder:** `/ (root)`
4. **Save**

GitHub builds and publishes within ~30 seconds. Your URL will be:

```
https://YOUR_USERNAME.github.io/remotepad-website/
```

### 3. Replace the `REPLACE_USER` tokens

Two files in the suite hardcode `REPLACE_USER` and need to be swapped to
your actual GitHub username:

- **`website/mac/index.html`** — two URLs in the meta refresh and the
  manual download link. Update both, commit, push.
- **`ios/RemotePadIOS/RemotePadIOS/AppConfig.swift`** — `siteBase`
  constant. Also update if you used a different repo name than
  `remotepad-website`.

After editing both, smoke-test:

```bash
# Should return 200:
curl -I "https://YOUR_USERNAME.github.io/remotepad-website/"

# Should 302 / meta-refresh to a GitHub Releases URL:
curl -I "https://YOUR_USERNAME.github.io/remotepad-website/mac/"
```

### 4. Smoke test in a browser

Open `https://YOUR_USERNAME.github.io/remotepad-website/` — you should see
the landing page. Click around: Privacy, Terms, the "Send to your Mac"
flow (until a DMG is up on Releases, the `/mac` link will 404 inside
GitHub — that's expected; it'll resolve once you ship your first DMG).

## Local preview

No build step. Just:

```
cd website
python3 -m http.server 4000
# Open http://localhost:4000
```

## Every release

When `tools/build-mac-dmg.sh` ships a new DMG to GitHub Releases:

- `mac/index.html` points at `releases/latest/download/RemoteHostMac.dmg`,
  which **GitHub auto-resolves to the most recent release's asset**.
- **Nothing in this repo needs to change.** Push a release in the
  source repo, the website automatically picks it up.

## Migrating to a custom domain later (15 min)

When you decide to buy `remotepad.app` (or anything else):

1. **Register the domain** at Cloudflare Registrar (~$20/yr).
2. **Add a `CNAME` file** at the root of the website repo with the bare
   domain on one line:
   ```
   remotepad.app
   ```
3. **DNS records in Cloudflare**:
   - `A` record for `remotepad.app` → `185.199.108.153, 185.199.109.153,
     185.199.110.153, 185.199.111.153` (GitHub Pages IPs)
   - `CNAME` for `www.remotepad.app` → `YOUR_USERNAME.github.io`
4. In the GitHub Pages settings, **add the custom domain** so HTTPS gets
   re-issued.
5. **Update `AppConfig.swift`** — change `siteBase` to
   `"https://remotepad.app"`, ship a new iOS version.
6. (Optional) Set up **Cloudflare Email Routing** for
   `feedback@remotepad.app` → your Gmail. Swap the email back in
   `AppConfig.swift` + the HTML pages.

Existing v1 users still hitting `YOUR_USERNAME.github.io/remotepad-website/mac`
will keep working forever — the github.io URL is a permanent fallback.

## Before launching publicly

- [ ] Replace `assets/icon.svg` with the commissioned production icon
- [ ] Add a 1200×630 PNG `assets/og-image.png` for social link previews
- [ ] Add the real App Store link to the two "Download on the App Store"
      buttons in `index.html` (search for `id="app-store-btn"`)
- [ ] Add Cloudflare Web Analytics or Plausible (privacy-respecting, no
      cookies) if you want traffic numbers
- [ ] Update the copyright year if launching after 2026
