# Hydria — Marketing & Legal Site

Self-contained static site for the Hydria app. No build step, no external JS, no dependencies.
Just HTML + one CSS file. Deploy as-is to any static host.

## Files
```
site/
  index.html     ← marketing landing page (deep-ocean brand, hero, features, pricing, App Store badge placeholder)
  privacy.html   ← privacy policy (HealthKit, local + iCloud, no tracking, children's privacy, contact, eff. 2026-06-24)
  terms.html     ← terms of use (Apple Licensed-Application EULA points + auto-renewable subscription terms)
  style.css      ← shared styles for all three pages
  README.md      ← this file
```

## Intended final URLs (these are what the iOS app links to — they MUST resolve)
The app currently hardcodes these exact paths, so the deployed structure must match:

| Page | Final URL | Source file |
|---|---|---|
| Landing / Marketing | `https://hydria.app/` | `index.html` |
| **Privacy Policy** | `https://hydria.app/privacy` | `privacy.html` |
| **Terms of Use (EULA)** | `https://hydria.app/terms` | `terms.html` |

> ⚠️ The app links to **`/privacy`** and **`/terms`** with **no `.html`**. Make sure the host
> serves `privacy.html` at `/privacy` and `terms.html` at `/terms`. Three easy options:
>
> 1. **Rename on deploy** (simplest for plain static roots): copy `privacy.html` → `privacy/index.html`
>    and `terms.html` → `terms/index.html`, so `/privacy` and `/terms` resolve as directories.
> 2. **nginx `try_files`** (matches the portfolio's VPS setup):
>    ```nginx
>    server {
>      server_name hydria.app www.hydria.app;
>      root /var/www/hydria;
>      index index.html;
>      location = /privacy { try_files /privacy.html =404; }
>      location = /terms   { try_files /terms.html   =404; }
>      location / { try_files $uri $uri/ =404; }
>    }
>    ```
> 3. **Cloudflare Pages / Netlify / GitHub Pages**: add a `_redirects` (or `[[redirects]]`) rule
>    mapping `/privacy → /privacy.html` and `/terms → /terms.html` (200 rewrite).

## Deploy (portfolio VPS pattern)
The portfolio hosts other static marketing/legal sites on VPS boxes (nginx + Let's Encrypt). Same
recipe here:

```bash
# 1. point DNS: hydria.app A record → the VPS IP (and www CNAME → hydria.app)
# 2. copy the site
rsync -avz --delete -e "ssh -i ~/.ssh/my_key.pem" \
  /Users/HUGOMORICEAU/hydria/marketing/site/ root@<VPS_IP>:/var/www/hydria/
# 3. nginx server block as above (add the /privacy and /terms try_files lines), then:
ssh -i ~/.ssh/my_key.pem root@<VPS_IP> 'nginx -t && systemctl reload nginx'
# 4. TLS:
ssh -i ~/.ssh/my_key.pem root@<VPS_IP> 'certbot --nginx -d hydria.app -d www.hydria.app'
```

After deploy, verify the two URLs the app depends on actually return 200:
```bash
curl -sI https://hydria.app/privacy | head -1   # expect: HTTP/2 200
curl -sI https://hydria.app/terms   | head -1   # expect: HTTP/2 200
```

## Brand notes
- Accents: aqua `#39D4F5` → blue `#1E94E2` → indigo `#286FF6` on a deep-ocean `#04111E` background.
- System SF-Rounded-style font stack (no web-font download).
- All imagery is CSS-generated (gradients/shapes) — **no copyrighted or third-party assets**, nothing to license.
- The "Download on the App Store" badge is a styled placeholder; swap in Apple's official badge SVG
  and the real App Store link once the app is live.
