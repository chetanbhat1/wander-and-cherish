# Deploying Wander &amp; Cherish to GitHub Pages (wanderandcherish.com)

The full static site lives in **`docs/`** — ready to publish as-is. Every page is plain
HTML with relative links, so it runs anywhere static files are served. `docs/index.html`
is the home page, so `https://wanderandcherish.com/` shows the landing screen.

```
docs/
├── index.html              ← home (served at the domain root)
├── travel.html             ← Travel hub (links to the 10 destinations)
├── arizona.html … santorini-mykonos.html   ← 10 destination pages
├── stories.html            ← Inspiring Stories (3 stories + switcher)
├── lifejourneys.html       ← Life Journeys (coming soon)
├── about.html              ← About
├── dest.css                ← shared styles for the destination pages
├── *.jpg / *.png           ← your Santorini + Rajeshree photos
├── CNAME                   ← contains "wanderandcherish.com" (do not delete)
└── .nojekyll               ← tells GitHub to serve files as-is
```

Unlike Blogger, **nothing needs pasting and no image re-hosting** — your photos are
included as files and load by relative path.

---

## Step 1 — Put the code on GitHub

If this folder isn't on GitHub yet:

```bash
cd /Users/chet/code/wander-and-cherish
git add docs DEPLOY-GITHUB-PAGES.md
git commit -m "Add GitHub Pages site"
# create a repo on github.com first (e.g. "wander-and-cherish"), then:
git remote add origin https://github.com/chetanbhat1/wander-and-cherish.git
git push -u origin master      # or: git branch -M main && git push -u origin main
```

(If the repo already has a remote, just `git add docs && git commit && git push`.)

## Step 2 — Turn on GitHub Pages

On github.com → your repo → **Settings** → **Pages**:

- **Source:** *Deploy from a branch*
- **Branch:** `main` (or `master`) and folder **`/docs`** → **Save**

Wait ~1 minute. GitHub shows a green "Your site is live at …github.io" banner.
Open that URL and click around — the whole site should work.

## Step 3 — Point wanderandcherish.com at GitHub (Bluehost DNS)

In **Bluehost → Domains → wanderandcherish.com → DNS / Zone Editor**, set these records.
First **remove the existing `A` record for `@`** — by default it points at Bluehost's
parking IP **`66.81.203.198`** (and `www` points there too). That parking record is the
single most common reason GitHub reports "Domain does not resolve to the GitHub Pages
server," so it must be deleted, along with any "parked page" / redirect on the domain.

**Apex domain (`wanderandcherish.com`) — add four A records:**

| Type | Host/Name | Value |
|------|-----------|-------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |

*(Optional but recommended — IPv6 AAAA records, same Host `@`):*
`2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`

**`www` subdomain — add one CNAME:**

| Type | Host/Name | Value |
|------|-----------|-------|
| CNAME | www | `chetanbhat1.github.io` |

(That's your GitHub username — note the trailing `.github.io`, **not** the repo name.
Also delete the old `www` record pointing at `66.81.203.198` first; if Bluehost won't
let you add a CNAME for `www` while an A record exists, remove that A record, or instead
give `www` the same four GitHub A records as the apex.)

## Step 4 — Tell GitHub about the domain &amp; enable HTTPS

Back in **Settings → Pages**:

1. Under **Custom domain**, enter `wanderandcherish.com` → **Save**.
   (The `CNAME` file in `docs/` already sets this; GitHub will verify DNS.)
2. Wait for the "DNS check successful" green check (can take 10 min–24 h while DNS
   propagates).
3. Tick **Enforce HTTPS** once it's available. GitHub issues the SSL certificate
   automatically — no cost, nothing to buy from Bluehost.

That's it. `https://wanderandcherish.com` (and `https://www.wanderandcherish.com`) will
serve the new site.

---

## Updating the site later

Edit the files in `docs/` (or re-run the build from the prototypes), then:

```bash
git add docs && git commit -m "Update site" && git push
```

GitHub redeploys within a minute.

## Notes &amp; troubleshooting

- **DNS takes time.** Apex A-record changes on Bluehost can take a few hours to
  propagate. The github.io URL works immediately in the meantime.
- **"Domain does not resolve to the GitHub Pages server" (NotServedByPagesError).** This
  almost always means DNS still points at Bluehost. Check what it currently resolves to:
  ```bash
  dig +short wanderandcherish.com
  ```
  If you see **`66.81.203.198`** (Bluehost parking) instead of the four `185.199.x.153`
  GitHub IPs, the old apex `A` record wasn't removed. Delete it, confirm only the four
  GitHub A records remain for `@`, and re-check (apex changes can take a few hours).
  Do the same for `www` — it must be a CNAME to `chetanbhat1.github.io`, not the parking IP.
  Once `dig +short wanderandcherish.com` returns the four GitHub IPs, GitHub's check goes
  green and you can enable **Enforce HTTPS**.
- **Keep the `CNAME` and `.nojekyll` files** in `docs/` — deleting `CNAME` un-sets the
  custom domain; deleting `.nojekyll` can make GitHub skip files.
- **Photos:** the 5 personal photos (Santorini + 4 Rajeshree) are bundled in `docs/`.
  Replace any with higher-resolution versions by overwriting the same filenames and
  pushing again.
- **www vs apex:** with the records above, GitHub auto-redirects `www` → apex (or set
  your preference in Settings → Pages).
```
