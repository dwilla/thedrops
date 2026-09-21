# thedrops.fm

A web player for live DJ rides — an Icecast audio stream (broadcast from djay Pro) paired with a simple static site, used for group cycling rides on Zwift.

## Architecture

This is intentionally a **static site**, not a webapp:

- `index.html` — the player page: an `<audio>` element pointing at the Icecast stream URL, plus a small script that polls the Icecast `status-json.xsl` endpoint to show live/offline state. Stream (`https://stream.thedrops.fm/drops`) and status URLs are constants at the top of the script. Brand colors are CSS variables in `:root`.
- No backend, database, or build step required. Hostable for free on GitHub Pages, Netlify, or Cloudflare Pages.

Add a lightweight serverless function later only if needed (e.g. to proxy the Icecast status endpoint if CORS can't be opened on the streaming host, or to send "we're live" push notifications). Not needed for launch.

## Streaming pipeline (context for whoever's on the decks)

djay Pro does not broadcast to Icecast natively. Audio is routed through an intermediary encoder (e.g. BUTT — "Broadcast Using This Tool," or Audio Hijack) into the Icecast server. Things to confirm with whoever sets up the stream:

1. **Track metadata** — the site doesn't display the current track (it's posted to Zwift chat separately), but whether title updates reach Icecast at all depends on the encoder tool's config, not djay Pro itself.
2. **HTTPS** — the Icecast stream must be served over https. If it's only http, browsers will block it as mixed content on an https page (thedrops.fm should be https via GitHub Pages/Netlify/Cloudflare by default).
3. **CORS on `status-json.xsl`** — needed if the page polls listener count / live status client-side. Icecast 2.4+ ships a permissive `Access-Control-Allow-Origin: *` by default, but confirm the streaming host hasn't locked it down.

## Local development

No build tooling needed. Serve it locally (opening the file via `file://` can break the status polling):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploying

### Option A: GitHub Pages (simplest, free)

1. Push this repo to GitHub (already done — `origin` is `dwilla/thedrops`).
2. In the repo settings → **Pages**, set the source to the `main` branch, root directory.
3. GitHub will publish to `https://dwilla.github.io/thedrops` — then point the custom domain (below) at it.
4. Add a `CNAME` file to the repo root containing just `thedrops.fm` so GitHub Pages knows the custom domain (GitHub can also write this for you when you set the custom domain in the Pages settings UI).

### Option B: Netlify or Cloudflare Pages (a bit more flexible, still free tier)

1. Connect the GitHub repo in the Netlify/Cloudflare dashboard.
2. Build command: none. Publish directory: repo root (or `/public` if you move files there).
3. Every push to `main` auto-deploys.
4. Add the custom domain in the site settings.

Cloudflare Pages is a reasonable default if you also want to eventually add a small serverless function (Cloudflare Workers) for the CORS-proxy or notification use case mentioned above, since it's the same platform.

### Custom domain (thedrops.fm) DNS

At your domain registrar, point thedrops.fm at whichever host you pick:

- **GitHub Pages**: `A` records for the apex domain to GitHub's IPs (`185.199.108.153`, `.109.153`, `.110.153`, `.111.153`), or a `CNAME`/`ALIAS` for `www` to `dwilla.github.io`.
- **Netlify**: `CNAME` to the Netlify subdomain they assign, or use Netlify DNS directly.
- **Cloudflare Pages**: if the domain's nameservers are already on Cloudflare, this is a couple of clicks in the dashboard — no manual record juggling.

## Notes / gotchas

- Browsers block autoplaying audio — the page needs an explicit "play" tap, it can't auto-start when someone loads the page.
- Since rides are scheduled, not 24/7, the page should handle an "offline" state gracefully (e.g. "not live right now — next ride: [time]") rather than showing a dead player.

## Possible future additions (not needed for launch)

- Ride schedule (could start as a hardcoded list, or embed a Zwift Companion/Discord/Google Calendar widget).
- Email signup (Mailchimp/ConvertKit embed — still static-compatible).
- Archive of past sets (link out to Mixcloud/SoundCloud rather than hosting audio files here).
- Chat, RSVP tracking, or "we're live" push notifications — would need a small backend/serverless layer if these become priorities.
