# Deployment Troubleshooting (Vercel)

Common issues when deploying a migrated Next.js app to Vercel.

---

## 1. Build Fails: Peer Dependency Conflicts

**Symptom:** `npm install` fails during Vercel build with peer dependency errors.

**Fix:** Create `.npmrc` in your project root:
```
legacy-peer-deps=true
```

This is almost always needed when migrating from no-code platforms because their dependency trees have peer conflicts that aren't issues in development but fail in strict CI/CD.

---

## 2. Static Files Override Dynamic Routes

**Symptom:** `sitemap.xml` or `robots.txt` returns static content instead of dynamically generated content.

**Cause:** Files in `public/robots.txt` or `public/sitemap.xml` take precedence over `src/app/robots.ts` and `src/app/sitemap.ts`.

**Fix:** Delete the static files:
```bash
rm public/robots.txt
rm public/sitemap.xml
```

The dynamic Next.js routes will now serve correctly.

---

## 3. Environment Variables Not Available During Build

**Symptom:** Build fails because `process.env.NEXT_PUBLIC_*` is undefined during `next build`.

**Cause:** Environment variables must be set in Vercel **before** triggering the build.

**Fix:**
1. Go to Vercel dashboard > Your Project > Settings > Environment Variables
2. Add all required variables:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `NEXT_PUBLIC_SITE_URL`
3. Set them for **Production**, **Preview**, and optionally **Development**
4. Redeploy (existing deployments don't get new env vars)

**Via CLI:**
```bash
# Set for production
echo "your-value" | vercel env add NEXT_PUBLIC_SUPABASE_URL production

# Set for preview
echo "your-value" | vercel env add NEXT_PUBLIC_SUPABASE_URL preview
```

---

## 4. Preview vs Production Environment Variables

**Symptom:** Preview deployments (pull request branches) fail but production works, or vice versa.

**Cause:** Environment variables are scoped per environment in Vercel.

**Fix:** Set variables for all needed environments:
- **Production:** Your live site (main/master branch)
- **Preview:** Pull request and branch deployments
- **Development:** `vercel dev` local development

Some variables may differ between environments (e.g., `NEXT_PUBLIC_SITE_URL` is different for preview vs production).

---

## 5. Domain DNS Propagation

**Symptom:** Custom domain returns 404 or shows the wrong site after DNS change.

**Fix:**
1. Check current DNS with `dig your-domain.com` or `nslookup your-domain.com`
2. Set low TTL (300 seconds) before switching DNS
3. Add domain in Vercel dashboard first: Domains > Add Domain
4. Update DNS records at your registrar to point to Vercel
5. Wait for propagation (usually 5-30 minutes, can take up to 48 hours)
6. Verify with `curl -I https://your-domain.com`

**DNS records to set:**
- For apex domain: `A` record pointing to `76.76.21.21`
- For subdomain: `CNAME` record pointing to `cname.vercel-dns.com`

---

## 6. Vercel Hobby Plan Limitations

**Things to watch out for on the free plan:**

| Limitation | Hobby Plan | Pro Plan |
|------------|-----------|----------|
| Serverless function timeout | 10 seconds | 60 seconds |
| Edge function size | 1 MB | 4 MB |
| Build duration | 45 minutes | 45 minutes |
| Bandwidth | 100 GB/month | 1 TB/month |
| Deployments | Unlimited | Unlimited |
| Team members | 1 | Unlimited |

**Common issues:**
- Serverless functions timing out on complex Supabase queries -- optimize queries or upgrade
- Large middleware bundles exceeding Edge function size limits

---

## 7. Git Author Email Mismatch

**Symptom:** First deploy to Vercel fails with email verification error.

**Cause:** Vercel Hobby plan checks that the git author email matches your Vercel account.

**Fix:**
- Ensure your git config email matches your Vercel account email
- Or connect via Vercel CLI: `vercel` (interactive setup)
- Or connect the GitHub repo directly in Vercel dashboard (recommended)

---

## 8. Build Fails: "Could Not Find a Production Build"

**Symptom:** Vercel deploy fails saying it can't find the build output.

**Cause:** Build command or output directory misconfigured.

**Fix:** In Vercel project settings:
- Build Command: `npm run build` (or `next build`)
- Output Directory: `.next` (default, usually auto-detected)
- Install Command: `npm install` (or `npm ci`)

---

## 9. Images Not Loading in Production

**Symptom:** `<Image>` components show broken images in production.

**Cause:** Remote image domains not configured in `next.config.ts`.

**Fix:**
```typescript
// next.config.ts
images: {
  remotePatterns: [
    { protocol: 'https', hostname: 'images.unsplash.com' },
    { protocol: 'https', hostname: '*.supabase.co' },
    // Add any other image domains your app uses
  ],
},
```

---

## 10. ISR (Incremental Static Regeneration) Not Working

**Symptom:** Pages don't update after content changes in Supabase.

**Fix options:**

**Option A: Time-based revalidation**
```typescript
// In your page or layout
export const revalidate = 3600 // Revalidate every hour
```

**Option B: On-demand revalidation**
Create an API route that calls `revalidatePath()` and trigger it from Supabase webhooks or your admin panel.

**Option C: Force dynamic rendering**
```typescript
export const dynamic = 'force-dynamic'
```
This skips caching entirely -- use for admin pages or frequently changing content.
