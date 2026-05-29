# Vercel → Cloudflare Pages migration

Total time: ~15 minutes. Saves $20/mo (Vercel Pro fee) and unlocks
Cloudflare Workers if you ever want server-side per-route logic.

## What's already done

- `_redirects` file written — equivalent of `vercel.json` rewrites.
  Cloudflare Pages auto-detects it.
- `_headers` file written — handles the apple-app-site-association
  Content-Type rule.
- Vercel-only files we'll leave for now (no-op on Cloudflare):
  `vercel.json` (ignored if you keep it).

## What you do (12 minutes)

1. **Sign up** at <https://pages.cloudflare.com/> (free).
   Auth with GitHub for one-click repo connect.

2. **Create a Pages project**: Cloudflare dashboard → Workers & Pages
   → Create application → Pages → "Connect to Git" →
   pick **scotty123868/mail** → branch `main`.

3. **Build settings**: leave everything blank/default.
   - Framework preset: `None`
   - Build command: (blank)
   - Build output: `/`

   This is a pure static site — Cloudflare just serves the files.

4. **Deploy**. First build takes ~30 seconds.
   You'll get a URL like `mail-xyz.pages.dev`. Verify in browser:
   - `/` shows the landing
   - `/c/test123` rewrites to the postcard page
   - `/privacy` and `/terms` resolve

5. **Custom domain**: Pages project → Custom domains → Add domain →
   `app.themailroom.club`. Cloudflare gives you a CNAME target.

6. **Update DNS**: wherever `themailroom.club` lives (Squarespace,
   Namecheap, Cloudflare DNS already?), point the `app` subdomain at
   the Pages CNAME from step 5.

   If your domain is already on Cloudflare DNS, this is a one-click
   "use this domain" prompt — no manual CNAME needed.

7. **Wait ~5 min for DNS propagation**, then verify
   <https://app.themailroom.club> serves the new build.

8. **Disable Vercel**: when verified, archive or delete the Vercel
   project. The $20/mo charge stops at the next billing cycle.

## Edge cases to verify post-migration

- **iMessage URL preview** of `/c/<token>`: visit a real card URL in
  Safari. The OG card image should still load.
- **MapKit JS on `/c/`**: the live map should render. The
  mapkit-token Edge Function is on Supabase, not Vercel, so it's
  unaffected.
- **`.well-known/apple-app-site-association`**: curl it and check
  `Content-Type: application/json`. If it returns text/plain, the
  `_headers` file isn't being applied (check syntax — Cloudflare is
  strict about indentation).

## Rollback plan

If anything breaks, swap DNS back to Vercel (you have until the next
Vercel billing cycle, so even multi-day rollback is fine). Both
deployments can coexist on different URLs while you A/B verify.

## Why we did this

Vercel Hobby is "personal use only" per their ToS. Mailroom is a
commercial product (charging via Stripe). Cloudflare Pages has no
such restriction on its free tier. Same edge CDN quality, same DX,
same git-push-to-deploy. Saves $240/year.
