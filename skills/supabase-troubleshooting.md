# Supabase + Next.js Troubleshooting

Common issues when connecting Supabase to a Next.js app deployed on Vercel.

---

## 1. "Invalid API key" After Deployment

**Symptom:** App works locally but Supabase calls fail in production.

**Cause:** Environment variables not set in Vercel, or using the wrong prefix.

**Fix:**
- Set `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` in Vercel dashboard (Settings > Environment Variables)
- Make sure they're set for **both** Production and Preview environments
- Don't use `VITE_*` prefix -- Next.js requires `NEXT_PUBLIC_*`
- Redeploy after adding env vars (they don't take effect on existing deployments)

---

## 2. Auth Session Not Persisting Across Pages

**Symptom:** User logs in successfully but appears logged out on page navigation.

**Cause:** Middleware isn't refreshing the Supabase auth cookies.

**Fix:** Your `src/middleware.ts` must create a Supabase server client that reads AND writes cookies. The key is the `setAll` function:

```typescript
setAll(cookiesToSet) {
  cookiesToSet.forEach(({ name, value }) =>
    request.cookies.set(name, value)
  )
  supabaseResponse = NextResponse.next({ request })
  cookiesToSet.forEach(({ name, value, options }) =>
    supabaseResponse.cookies.set(name, value, options)
  )
},
```

See the full middleware pattern in `migration-skill.md` Part 7.

---

## 3. RLS Blocking All Queries

**Symptom:** `SELECT` queries return empty arrays, `INSERT` operations fail silently or return errors.

**Cause:** Row Level Security (RLS) is enabled on the table but no policies are defined. With RLS enabled and no policies, ALL access is denied.

**Fix options:**

**Option A: Create appropriate policies**
```sql
-- Allow public read access
CREATE POLICY "Public read" ON your_table
  FOR SELECT USING (true);

-- Allow authenticated users to insert
CREATE POLICY "Auth insert" ON your_table
  FOR INSERT WITH CHECK (auth.role() = 'authenticated');
```

**Option B: Disable RLS temporarily (development only)**
```sql
ALTER TABLE your_table DISABLE ROW LEVEL SECURITY;
```

**Important:** Never disable RLS in production. Always create proper policies.

---

## 4. "relation does not exist" Error

**Symptom:** `relation "your_table" does not exist` error from Supabase.

**Causes:**
1. Table name is case-sensitive -- check exact name in Supabase dashboard
2. Table is in a different schema (e.g., `auth` schema, not `public`)
3. Typo in the table name
4. Migration hasn't been run yet

**Fix:** Check the exact table name in your Supabase dashboard > Table Editor. Use that exact name in your query.

---

## 5. Edge Function CORS Errors

**Symptom:** Edge Function calls fail with CORS error after deploying to a new domain.

**Cause:** Edge Functions have CORS allowlists. Your new Vercel domain isn't in the list.

**Fix:** Update CORS headers in each Edge Function:

```typescript
const corsHeaders = {
  'Access-Control-Allow-Origin': '*', // Or specific domains
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
  'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
}

// Handle OPTIONS preflight
if (req.method === 'OPTIONS') {
  return new Response('ok', { headers: corsHeaders })
}
```

For better security, list specific domains instead of `*`:
```typescript
const allowedOrigins = [
  'https://your-project.vercel.app',
  'https://your-domain.com',
  'http://localhost:3000',
]
```

---

## 6. `cookies()` Must Be Awaited

**Symptom:** Build error: "cookies() should be awaited before using its value"

**Cause:** Next.js 15+ made `cookies()` async.

**Fix:**
```typescript
// Before (Next.js 14)
const cookieStore = cookies()

// After (Next.js 15+)
const cookieStore = await cookies()
```

This affects the server Supabase client. Make sure `createClient()` in `server.ts` is `async`.

---

## 7. "Dynamic server usage" in generateStaticParams

**Symptom:** Build error: "Dynamic server usage: cookies" inside `generateStaticParams`.

**Cause:** Using the server client (which reads cookies) in a build-time function.

**Fix:** Use the static client instead:

```typescript
// Wrong
import { createClient } from '@/lib/supabase/server'

// Right
import { createStaticClient } from '@/lib/supabase/static'

export async function generateStaticParams() {
  const supabase = createStaticClient() // No cookies needed at build time
  // ...
}
```

---

## 8. Service Role Key Exposure

**Symptom:** Security vulnerability -- service role key visible in browser.

**Cause:** Using `NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY` as an env var name.

**Fix:** NEVER prefix the service role key with `NEXT_PUBLIC_`. It should only be:
- `SUPABASE_SERVICE_ROLE_KEY` (server-only, not exposed to browser)
- Only used in Route Handlers (`route.ts`) and Server Actions
- Never imported in client components

---

## 9. Supabase Types Generation

**Symptom:** TypeScript errors because `Database` type is missing or outdated.

**Fix:** Generate fresh types:

```bash
# Install Supabase CLI if needed
npm install -g supabase

# Login
supabase login

# Generate types (replace with your project ref)
supabase gen types typescript --project-id your-project-ref > src/integrations/supabase/types.ts
```

Run this whenever you change your database schema.

---

## 10. Connection Pooling on Vercel Serverless

**Symptom:** "Too many connections" errors in production under load.

**Cause:** Each serverless function invocation creates a new database connection. Supabase has connection limits.

**Fix:**
1. Use Supabase's built-in connection pooler (Supavisor)
2. In Supabase dashboard: Settings > Database > Connection Pooling
3. Use the pooler connection string for server-side clients
4. The anon key / browser client already uses the pooler by default

---

## 11. Realtime Subscriptions Not Working

**Symptom:** `supabase.channel()` or `.on()` subscriptions don't receive updates.

**Causes:**
1. Realtime not enabled for the table (enable in Supabase dashboard > Database > Replication)
2. RLS policies blocking the subscription
3. Using server client for realtime (must use browser client)

**Fix:**
```typescript
// Realtime MUST use the browser client (in a 'use client' component)
'use client'
import { createClient } from '@/lib/supabase/client'

const supabase = createClient()
supabase
  .channel('changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'your_table' }, (payload) => {
    console.log('Change:', payload)
  })
  .subscribe()
```
