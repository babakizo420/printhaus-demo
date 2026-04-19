# TheTalkingArts Site &mdash; Ops Runbook (Pejji Internal)

## Deployment target
- GitHub: `Pejji-ng/thetalkingarts-demo`
- Hosting target: Cloudflare Pages (note: GitHub Pages workflow exists at `.github/workflows/deploy.yml` &mdash; remove if using CF)
- Domain: `thetalkingarts.com` (already registered, on CF Registrar)
- **Current public state**: domain points to EXPIRED Shopify/Wix store (HTTP 402). Our Pejji-standard site will replace that on go-live.

## Placeholders to replace before go-live

Files containing placeholders:
- `index.html` &mdash; `{{TTA_WHATSAPP_NUMBER}}` (x3), `{{TTA_PHONE_NUMBER}}` (x1)
- `privacy.html` &mdash; `{{TTA_WHATSAPP_NUMBER}}` (x2), `{{TTA_CONTACT_EMAIL}}` (x3)
- `404.html` &mdash; none

### Sweep commands (after client sends values)
```bash
WA="2348012345678"
PHONE="801 234 5678"
EMAIL="hello@thetalkingarts.com"

sed -i "s|{{TTA_WHATSAPP_NUMBER}}|$WA|g" index.html privacy.html
sed -i "s|{{TTA_PHONE_NUMBER}}|$PHONE|g" index.html
sed -i "s|{{TTA_CONTACT_EMAIL}}|$EMAIL|g" privacy.html

grep -r "{{TTA_" . --include="*.html" || echo "All placeholders filled."
```

## CF Pages deploy checklist

1. Push `main` to `Pejji-ng/thetalkingarts-demo`
2. CF dashboard -> Workers & Pages -> Create -> Connect to Git
3. Select `Pejji-ng/thetalkingarts-demo`, branch `main`
4. Build settings: **None** preset, no build command, output dir = `.` (static)
5. Deploy
6. Custom domains -> Add `thetalkingarts.com` AND `www.thetalkingarts.com`
7. CF auto-creates CNAMEs, SSL provisions in ~3 min
8. Verify headers + 200 on both URLs

## Known items to address before client hand-off
- Contact form currently simulated (fakes success). If the client wants real email notifications, wire Web3Forms:
  1. Sign up on web3forms.com, get access key
  2. Add `action="https://api.web3forms.com/submit"` and `method="POST"` to the form
  3. Add hidden input `<input type="hidden" name="access_key" value="YOUR_KEY">`
  4. Remove the `e.preventDefault()` simulation in script.js
- GitHub Actions workflow at `.github/workflows/deploy.yml` deploys to GitHub Pages &mdash; delete or disable once CF Pages is primary
- Portfolio case studies may need real client permission before public launch (Iya Basira, Ace Logistics, Lagos Fashion Week, GlowUp Skincare)
- Testimonial from Aisha Ogundimu needs attribution permission

## Post-deploy verification
```bash
curl -sI https://thetalkingarts.com/ | grep -iE "(content-security|x-frame|strict-transport)"
curl -s https://thetalkingarts.com/ | grep -c "{{TTA_"  # should be 0
curl -sI https://thetalkingarts.com/privacy.html | head -1
```

## Add to uptime monitor after go-live
- https://thetalkingarts.com/
- https://www.thetalkingarts.com/
