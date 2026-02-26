# No-Code SPA → Next.js Migration Skill

Complete blueprint for migrating no-code platform websites (Lovable, Base44, and similar Vite + React Router SPAs) to Next.js 15+ App Router. Covers project setup, route migration, Supabase client architecture, SEO, auth, error handling, and deployment.

---

## PART 1: Understanding the Source App

### 1.1 Typical No-Code Output Structure

No-code platforms like Lovable produce a **Vite + React** SPA:

```
project/
├── src/
│   ├── pages/              # React Router page components
│   │   ├── Index.tsx        # Home page
│   │   ├── About.tsx        # About page
│   │   ├── NotFound.tsx     # 404 page
│   │   └── ...
│   ├── components/          # Reusable UI components
│   │   ├── ui/              # shadcn/ui components (Button, Card, Dialog, etc.)
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── ...
│   ├── hooks/               # Custom React hooks
│   ├── lib/                 # Utilities
│   │   └── utils.ts         # cn() helper for Tailwind
│   ├── integrations/
│   │   └── supabase/
│   │       ├── client.ts    # Supabase browser client
│   │       └── types.ts     # Auto-generated DB types
│   ├── App.tsx              # React Router setup
│   ├── main.tsx             # Entry point (ReactDOM.render)
│   └── index.css            # Global styles + Tailwind
├── .env                     # VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY
├── vite.config.ts           # Vite configuration
├── tailwind.config.ts       # Tailwind configuration
├── tsconfig.json
├── components.json          # shadcn/ui config
└── package.json
```

> **Base44 note:** Base44 uses a similar structure but with `.jsx` files, `@base44/sdk` instead of direct Supabase, and an `entities/` directory. See `skills/base44-platform.md` for details.

### 1.2 Key Characteristics

- **Router:** React Router v6 (`<BrowserRouter>`, `<Routes>`, `<Route>`)
- **Styling:** Tailwind CSS + shadcn/ui (Radix primitives)
- **Backend:** Supabase (auth, database, storage, edge functions)
- **State:** React Query (`@tanstack/react-query`)
- **Animations:** framer-motion
- **Icons:** lucide-react
- **Toasts:** Sonner or shadcn toast
- **Env vars:** `VITE_*` prefix (required by Vite)
- **Build:** Client-side SPA -- NO server-side rendering, NO SEO

### 1.3 Why Migrate to Next.js

| Problem with SPA | Next.js Solution |
|-------------------|-------------------|
| No SEO -- Google sees empty HTML | SSR/SSG renders full HTML for crawlers |
| No per-page metadata | `metadata` exports on every route |
| No sitemap or robots.txt | Dynamic `sitemap.ts` and `robots.ts` |
| No schema markup (JSON-LD) | Server-rendered JSON-LD components |
| Client-side auth only | Middleware + server-side auth (cookies) |
| Single entry point (SPA) | File-based routing with layouts |
| No API routes | Route Handlers for backend logic |
| No image optimization | Next.js Image with lazy loading + WebP |
| Poor Core Web Vitals | SSR reduces LCP, streaming reduces FCP |

---

## PART 2: Pre-Migration Checklist

### 2.1 Before You Start

1. **Clone from GitHub** -- Get the source repo locally
2. **Run the app** -- `npm run dev` to understand the app
3. **Map all routes** -- List every `<Route path="...">` in App.tsx
4. **Identify dynamic routes** -- Routes with `:id`, `:slug` params
5. **List all environment variables** -- Check `.env` for VITE_* vars
6. **Check Supabase tables** -- Understand the database schema
7. **Identify interactive components** -- Which components use hooks, state, effects
8. **Check for API calls** -- Where does the app call Supabase or external APIs

### 2.2 Route Mapping Template

```
React Router Path          -> Next.js File                       Type
/                          -> src/app/page.tsx                   Static
/about                     -> src/app/about/page.tsx             Static
/blog                      -> src/app/blog/page.tsx              Static
/blog/:slug                -> src/app/blog/[slug]/page.tsx       Dynamic
/services                  -> src/app/services/page.tsx          Static
/contact                   -> src/app/contact/page.tsx           Interactive
/admin                     -> src/app/admin/page.tsx             Protected
/admin/articles            -> src/app/admin/articles/page.tsx    Protected
```

### 2.3 Dependencies to Add/Replace

**Add:**
```
next                        # Framework
@supabase/ssr               # Server-side Supabase client
```

**Remove:**
```
react-router-dom            # Replaced by file-based routing
vite                        # Replaced by Next.js bundler
@vitejs/plugin-react        # Vite-specific
```

**Keep (no changes needed):**
```
react, react-dom            # Still React
@supabase/supabase-js       # Still used
@tanstack/react-query       # Still used for client-side caching
tailwindcss, postcss        # Still used
framer-motion               # Still used (needs 'use client')
lucide-react                # Still used
All @radix-ui/* packages    # Still used (shadcn/ui)
sonner                      # Still used
```

---

## PART 3: Project Setup

### 3.1 Install Next.js (In-Place Conversion)

**Step 1: Install Next.js**
```bash
npm install next @supabase/ssr
npm uninstall react-router-dom @vitejs/plugin-react vite
```

**Step 2: Update package.json scripts**
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

**Step 3: Create next.config.ts**
```typescript
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  env: {
    NEXT_PUBLIC_SUPABASE_URL:
      process.env.NEXT_PUBLIC_SUPABASE_URL ?? process.env.VITE_SUPABASE_URL ?? '',
    NEXT_PUBLIC_SUPABASE_ANON_KEY:
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY ?? process.env.VITE_SUPABASE_PUBLISHABLE_KEY ?? '',
  },

  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.unsplash.com' },
      { protocol: 'https', hostname: '*.supabase.co' },
    ],
  },

  async redirects() {
    return [
      // Add any renamed routes here
      // { source: '/old-path', destination: '/new-path', permanent: true },
    ]
  },

  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
        ],
      },
    ]
  },
}

export default nextConfig
```

**Step 4: Create .env.local**
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
NEXT_PUBLIC_SITE_URL=https://your-domain.com
```

**Step 5: Update tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules", "supabase"]
}
```

**Step 6: Update tailwind.config.ts**
```typescript
content: [
  "./src/pages/**/*.{ts,tsx}",     // Keep for old components
  "./src/views/**/*.{ts,tsx}",     // Moved old pages here
  "./src/components/**/*.{ts,tsx}",
  "./src/app/**/*.{ts,tsx}",       // NEW: Next.js app directory
],
```

### 3.2 Critical: Rename src/pages/ -> src/views/

**This is the most important step.** Next.js treats `src/pages/` as the Pages Router. If you leave the old page components there, Next.js will crash with routing conflicts.

```bash
mv src/pages src/views
```

Then update ALL imports across the codebase:
```
Find:    from '@/pages/
Replace: from '@/views/
```

### 3.3 Delete Source-Specific Files

```bash
rm src/App.tsx              # Replaced by app/layout.tsx + file routing
rm src/main.tsx             # Replaced by Next.js entry
rm vite.config.ts           # Replaced by next.config.ts
rm index.html               # Replaced by app/layout.tsx
```

---

## PART 4: Supabase Client Architecture (3-Client Pattern)

### 4.1 Why Three Clients?

Next.js has three execution contexts, each needing a different Supabase client:

| Context | Client | Cookies? | When Used |
|---------|--------|----------|-----------|
| Browser | `client.ts` | No (anon) | Client components, hooks, forms |
| Server | `server.ts` | Yes (cookies) | Server Components, API routes, middleware |
| Build-time | `static.ts` | No (can't access) | generateStaticParams, sitemap.ts |

### 4.2 Browser Client

```typescript
// src/lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/integrations/supabase/types'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### 4.3 Server Client

```typescript
// src/lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/integrations/supabase/types'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Can be ignored in Server Components (read-only context)
          }
        },
      },
    }
  )
}
```

### 4.4 Static Client

```typescript
// src/lib/supabase/static.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from '@/integrations/supabase/types'

export function createStaticClient() {
  return createClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### 4.5 Replacing the Original Client

The source app has a single client:
```typescript
// src/integrations/supabase/client.ts (ORIGINAL)
import { createClient } from '@supabase/supabase-js'
export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY
)
```

**Migration strategy:**
1. Keep the original types file (`src/integrations/supabase/types.ts`)
2. Create the three new clients in `src/lib/supabase/`
3. Update imports in client components: `import { createClient } from '@/lib/supabase/client'`
4. In admin/API routes: `import { createClient } from '@/lib/supabase/server'`
5. In static generation: `import { createStaticClient } from '@/lib/supabase/static'`
6. Replace `import.meta.env.VITE_*` with `process.env.NEXT_PUBLIC_*` everywhere

---

## PART 5: Route Migration

### 5.1 The Pattern: Wrapper Pages

The fastest migration approach is **thin wrapper pages** in `src/app/` that import existing components from `src/views/`:

```tsx
// src/app/about/page.tsx
import type { Metadata } from 'next'
import About from '@/views/About'

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn about our company, mission, and team.',
  alternates: { canonical: '/about' },
}

export default function AboutPage() {
  return <About />
}
```

This approach:
- Preserves ALL existing UI logic (no rewriting)
- Adds metadata for SEO
- Adds canonical URLs
- Takes 2-5 minutes per page

### 5.2 Dynamic Routes

For routes with URL parameters (e.g., `/blog/:slug`):

```tsx
// src/app/blog/[slug]/page.tsx
import type { Metadata } from 'next'
import { createStaticClient } from '@/lib/supabase/static'
import ArticleView from '@/views/blog/ArticleView'

export async function generateStaticParams() {
  const supabase = createStaticClient()
  const { data: articles } = await supabase
    .from('articles')
    .select('slug')
    .eq('is_published', true)

  return (articles ?? []).map((a) => ({ slug: a.slug }))
}

export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>
}): Promise<Metadata> {
  const { slug } = await params
  const supabase = createStaticClient()
  const { data: article } = await supabase
    .from('articles')
    .select('title, excerpt, cover_image')
    .eq('slug', slug)
    .eq('is_published', true)
    .single()

  if (!article) {
    return { title: 'Article Not Found' }
  }

  return {
    title: article.title,
    description: article.excerpt,
    alternates: { canonical: `/blog/${slug}` },
    openGraph: {
      title: article.title,
      description: article.excerpt || '',
      images: article.cover_image ? [article.cover_image] : [],
    },
  }
}

export default async function ArticlePage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  return <ArticleView slug={slug} />
}
```

### 5.3 Protected Routes (Admin)

```tsx
// src/app/admin/layout.tsx
import { redirect } from 'next/navigation'
import { createClient } from '@/lib/supabase/server'

export default async function AdminLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect('/login')
  }

  return (
    <div className="flex h-screen">
      <main className="flex-1 overflow-y-auto p-6">
        {children}
      </main>
    </div>
  )
}
```

### 5.4 Root Layout

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { Toaster } from 'sonner'
import QueryProvider from '@/components/providers/query-provider'

const inter = Inter({ subsets: ['latin'] })

const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || 'https://your-domain.com'

export const metadata: Metadata = {
  title: {
    default: 'Your Site Name',
    template: '%s | Your Site Name',
  },
  description: 'Your site description (150-160 chars)',
  metadataBase: new URL(BASE_URL),
  openGraph: {
    siteName: 'Your Site Name',
    type: 'website',
    locale: 'en_US',
  },
  twitter: {
    card: 'summary_large_image',
  },
  robots: {
    index: true,
    follow: true,
  },
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <QueryProvider>
          {children}
        </QueryProvider>
        <Toaster position="top-right" />
      </body>
    </html>
  )
}
```

---

## PART 6: Component Migration

### 6.1 The 'use client' Rule

**Needs `'use client'`:**
- Uses React hooks: `useState`, `useEffect`, `useCallback`, `useMemo`, `useRef`
- Uses browser APIs: `window`, `document`, `localStorage`
- Uses event handlers: `onClick`, `onChange`, `onSubmit`
- Uses framer-motion: `motion.div`, `AnimatePresence`
- Uses React Query: `useQuery`, `useMutation`
- Uses Next.js client hooks: `useRouter`, `usePathname`, `useSearchParams`
- Uses context: `useContext`, custom context providers

**Does NOT need `'use client'`:**
- Pure rendering (takes props, returns JSX)
- Static content (no interactivity)
- Server data fetching (async component with `await`)
- Metadata exports

### 6.2 Batch Find Components Needing 'use client'

```bash
grep -rl "useState\|useEffect\|useCallback\|useRouter\|framer-motion" src/views/ src/components/ | \
  xargs grep -L "'use client'"
```

### 6.3 Import Fixes

**`import.meta.env` -> `process.env`:**
```typescript
// Before
const url = import.meta.env.VITE_SUPABASE_URL
// After
const url = process.env.NEXT_PUBLIC_SUPABASE_URL
```

**React Router -> Next.js:**
```typescript
// Before
import { useNavigate, useParams, useLocation, Link } from 'react-router-dom'
navigate('/about')
<Link to="/about">About</Link>

// After
import { useRouter, useParams, usePathname } from 'next/navigation'
import Link from 'next/link'
router.push('/about')
<Link href="/about">About</Link>
```

### 6.4 QueryProvider Wrapper

```tsx
// src/components/providers/query-provider.tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useState } from 'react'

export default function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000,
        refetchOnWindowFocus: false,
      },
    },
  }))

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}
```

### 6.5 Sonner Toast Fix

**Critical bug:** If the Toaster component uses `useTheme()` from `next-themes` but no `ThemeProvider` exists, the app will crash.

```tsx
// Use Sonner's Toaster directly without theme integration
import { Toaster } from 'sonner'
<Toaster position="top-right" />

// Do NOT use:
// <Toaster theme={theme} /> -- crashes if no ThemeProvider
```

---

## PART 7: Middleware & Auth

### 7.1 Auth Middleware

```typescript
// src/middleware.ts
import { NextResponse, type NextRequest } from 'next/server'
import { createServerClient } from '@supabase/ssr'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) =>
            request.cookies.set(name, value)
          )
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  if (request.nextUrl.pathname.startsWith('/admin') && !user) {
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    return NextResponse.redirect(url)
  }

  if (request.nextUrl.pathname === '/login' && user) {
    const url = request.nextUrl.clone()
    url.pathname = '/admin'
    return NextResponse.redirect(url)
  }

  return supabaseResponse
}

export const config = {
  matcher: ['/admin/:path*', '/login'],
}
```

### 7.2 Login Page with Server Actions

```tsx
// src/app/login/page.tsx
'use client'

import { useActionState } from 'react'
import { loginAction } from './actions'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

export default function LoginPage() {
  const [state, formAction, isPending] = useActionState(loginAction, null)

  return (
    <div className="min-h-screen flex items-center justify-center">
      <form action={formAction} className="w-full max-w-md space-y-4 p-6">
        <h1 className="text-2xl font-bold">Sign In</h1>
        {state?.error && (
          <div className="text-red-600 text-sm">{state.error}</div>
        )}
        <div>
          <Label htmlFor="email">Email</Label>
          <Input id="email" name="email" type="email" required />
        </div>
        <div>
          <Label htmlFor="password">Password</Label>
          <Input id="password" name="password" type="password" required />
        </div>
        <Button type="submit" className="w-full" disabled={isPending}>
          {isPending ? 'Signing in...' : 'Sign In'}
        </Button>
      </form>
    </div>
  )
}
```

```typescript
// src/app/login/actions.ts
'use server'

import { redirect } from 'next/navigation'
import { createClient } from '@/lib/supabase/server'

export async function loginAction(
  _prevState: { error?: string } | null,
  formData: FormData
) {
  const email = formData.get('email') as string
  const password = formData.get('password') as string

  const supabase = await createClient()
  const { error } = await supabase.auth.signInWithPassword({ email, password })

  if (error) {
    return { error: error.message }
  }

  redirect('/admin')
}
```

---

## PART 8: SEO Enhancements

### 8.1 Dynamic Sitemap

```typescript
// src/app/sitemap.ts
import type { MetadataRoute } from 'next'
import { createStaticClient } from '@/lib/supabase/static'

const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || 'https://your-domain.com'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const supabase = createStaticClient()

  const { data: articles } = await supabase
    .from('articles')
    .select('slug, updated_at')
    .eq('is_published', true)

  const staticRoutes: MetadataRoute.Sitemap = [
    { url: `${BASE_URL}/`, lastModified: new Date(), changeFrequency: 'weekly', priority: 1.0 },
    { url: `${BASE_URL}/about`, lastModified: new Date(), changeFrequency: 'monthly', priority: 0.7 },
    // Add all your static routes here
  ]

  const articleRoutes: MetadataRoute.Sitemap = (articles ?? []).map((a) => ({
    url: `${BASE_URL}/blog/${a.slug}`,
    lastModified: new Date(a.updated_at),
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }))

  return [...staticRoutes, ...articleRoutes]
}
```

### 8.2 Robots.txt

```typescript
// src/app/robots.ts
import type { MetadataRoute } from 'next'

const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || 'https://your-domain.com'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/admin/', '/login', '/api/'],
      },
      { userAgent: 'GPTBot', allow: '/' },
      { userAgent: 'ClaudeBot', allow: '/' },
      { userAgent: 'PerplexityBot', allow: '/' },
      { userAgent: 'Google-Extended', allow: '/' },
    ],
    sitemap: `${BASE_URL}/sitemap.xml`,
  }
}
```

### 8.3 JSON-LD Schema Components

```tsx
// src/components/seo/json-ld.tsx

interface OrganizationJsonLdProps {
  name: string
  url: string
  logo: string
  description: string
}

export function OrganizationJsonLd({ name, url, logo, description }: OrganizationJsonLdProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name, url, logo, description,
  }
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}

interface ArticleJsonLdProps {
  title: string
  description: string
  url: string
  image?: string
  datePublished: string
  dateModified?: string
  author: string
}

export function ArticleJsonLd(props: ArticleJsonLdProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: props.title,
    description: props.description,
    url: props.url,
    image: props.image,
    datePublished: props.datePublished,
    dateModified: props.dateModified || props.datePublished,
    author: { '@type': 'Person', name: props.author },
  }
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}

interface FAQJsonLdProps {
  faqs: Array<{ question: string; answer: string }>
}

export function FAQJsonLd({ faqs }: FAQJsonLdProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: { '@type': 'Answer', text: faq.answer },
    })),
  }
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}
```

---

## PART 9: API Routes

### 9.1 Lead Submission

```typescript
// src/app/api/leads/submit/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const { name, email, phone, message, service } = body

    if (!name || !email) {
      return NextResponse.json({ error: 'Name and email are required' }, { status: 400 })
    }

    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return NextResponse.json({ error: 'Invalid email format' }, { status: 400 })
    }

    const supabase = await createClient()
    const { data, error } = await supabase
      .from('leads')
      .insert({ name, email, phone, message, service })
      .select()
      .single()

    if (error) throw error
    return NextResponse.json({ success: true, data })
  } catch (error) {
    console.error('Lead submission error:', error)
    return NextResponse.json({ error: 'Failed to submit lead' }, { status: 500 })
  }
}
```

### 9.2 Auth Callback

```typescript
// src/app/api/auth/callback/route.ts
import { NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get('code')

  if (code) {
    const supabase = await createClient()
    const { error } = await supabase.auth.exchangeCodeForSession(code)
    if (!error) {
      return NextResponse.redirect(`${origin}/admin`)
    }
  }

  return NextResponse.redirect(`${origin}/login?error=auth`)
}
```

### 9.3 Revalidation (ISR)

```typescript
// src/app/api/revalidate/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath } from 'next/cache'

export async function POST(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')

  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: 'Invalid secret' }, { status: 401 })
  }

  const body = await request.json()
  const { path } = body

  if (path) {
    revalidatePath(path)
    return NextResponse.json({ revalidated: true, path })
  }

  return NextResponse.json({ error: 'Path required' }, { status: 400 })
}
```

---

## PART 10: Error Handling

### 10.1 Error Boundary

```tsx
// src/app/error.tsx
'use client'

import Link from 'next/link'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center space-y-4">
        <h1 className="text-4xl font-bold">Something went wrong</h1>
        <p className="text-muted-foreground">{error.message}</p>
        <div className="flex gap-4 justify-center">
          <button onClick={reset} className="px-4 py-2 bg-primary text-white rounded">
            Try again
          </button>
          <Link href="/" className="px-4 py-2 border rounded">
            Go home
          </Link>
        </div>
      </div>
    </div>
  )
}
```

### 10.2 Global Error (uses inline styles -- CSS may not be available)

```tsx
// src/app/global-error.tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <div style={{ minHeight: '100vh', display: 'flex', alignItems: 'center', justifyContent: 'center', fontFamily: 'system-ui' }}>
          <div style={{ textAlign: 'center' }}>
            <h1 style={{ fontSize: '2rem', marginBottom: '1rem' }}>Something went wrong</h1>
            <button onClick={reset} style={{ padding: '0.5rem 1rem', background: '#000', color: '#fff', border: 'none', borderRadius: '0.25rem', cursor: 'pointer' }}>
              Try again
            </button>
          </div>
        </div>
      </body>
    </html>
  )
}
```

### 10.3 Not Found

```tsx
// src/app/not-found.tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center space-y-4">
        <h1 className="text-6xl font-bold">404</h1>
        <p className="text-xl text-muted-foreground">Page not found</p>
        <Link href="/" className="inline-block px-6 py-3 bg-primary text-white rounded-lg">
          Return to Home
        </Link>
      </div>
    </div>
  )
}
```

---

## PART 11: Common Pitfalls & Fixes

### Build Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "You're importing a component that needs `useState`" | Server Component using hooks | Add `'use client'` at top of file |
| "cookies() should be awaited" | Next.js 15+ async cookies API | Use `const cookieStore = await cookies()` |
| "Failed to collect page data for /pages/..." | `src/pages/` conflicts with Pages Router | Rename to `src/views/` |
| "import.meta.env is not defined" | Vite-only syntax | Replace with `process.env.NEXT_PUBLIC_*` |
| "Module not found: '@/pages/...'" | Old imports after rename | Update to `'@/views/...'` |
| "Cannot read properties of null (reading 'useContext')" | ThemeProvider missing | Remove `useTheme()` or add ThemeProvider |
| "Dynamic server usage: cookies" in generateStaticParams | Using server client during build | Use `createStaticClient()` instead |

### Runtime Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Hydration mismatch | Server HTML differs from client | Add `suppressHydrationWarning` or check `typeof window` |
| "useRouter() only works in Client Components" | Missing 'use client' | Add `'use client'` directive |
| Supabase auth not persisting | Middleware not refreshing session | Ensure middleware updates cookies |
| Admin page shows no data | Using server client in client component | Switch to browser client |
| 404 on page refresh | Dynamic routes without generateStaticParams | Add generateStaticParams or `dynamicParams = true` |

### Common Gotchas

- Source apps use **Tailwind v3** -- `create-next-app` installs v4. Downgrade if needed: `npm install tailwindcss@3 postcss autoprefixer --legacy-peer-deps`
- Typically **~40+ files** need `'use client'` directive
- `// @ts-nocheck` must come **BEFORE** `'use client'` for TypeScript to honor it
- Static `robots.txt`/`sitemap.xml` in `public/` will **override** dynamic Next.js routes -- delete them
- Edge Functions deploy from **Supabase** (not Vercel) -- update CORS in the source repo
- `sonner.tsx` may import `next-themes` -- remove and hardcode `theme="light"`
- Always create `.npmrc` with `legacy-peer-deps=true`
- Add `"supabase"` to `tsconfig.json` exclude array
- Admin pages need `export const dynamic = 'force-dynamic'`
- `sessionStorage`/`window` during SSR -- add `typeof window === 'undefined'` guards
- `useSearchParams` needs `<Suspense>` wrapper

---

## PART 12: Step-by-Step Workflow

### Phase 1: Setup (~30 min)
1. Install Next.js + @supabase/ssr
2. Remove react-router-dom + vite
3. Create next.config.ts, .env.local, update tsconfig
4. Rename `src/pages/` -> `src/views/`
5. Delete App.tsx, main.tsx, vite.config.ts, index.html

### Phase 2: Core Files (~1 hour)
1. Create `src/app/layout.tsx`
2. Create `src/app/globals.css`
3. Create 3 Supabase clients
4. Create `src/middleware.ts`
5. Create error boundaries
6. Create QueryProvider wrapper

### Phase 3: Route Migration (~2-4 hours)
For each route:
1. Create `src/app/<route>/page.tsx`
2. Add metadata export
3. Import the view component
4. Add `'use client'` if needed
5. Fix import paths

### Phase 4: Fix Imports (~1-2 hours)
1. Replace all `import.meta.env.VITE_*`
2. Replace React Router imports
3. Replace `<Link to=` with `<Link href=`
4. Update Supabase client imports

### Phase 5: SEO (~1-2 hours)
1. Create sitemap.ts, robots.ts
2. Create JSON-LD components
3. Verify all pages have metadata
4. Delete static sitemap/robots files

### Phase 6: Build & Fix (~1-2 hours)
1. Run `npx tsc --noEmit` -- fix TypeScript errors
2. Run `npm run build` -- fix build errors
3. Add missing `'use client'` directives
4. Verify all routes render

### Phase 7: Deploy (~30 min)
1. Create `.npmrc` with `legacy-peer-deps=true`
2. Push to GitHub
3. Deploy to Vercel
4. Set env vars in Vercel dashboard
5. Verify sitemap.xml, robots.txt, all routes

---

## PART 13: Post-Migration SEO Checklist

- [ ] Every page has unique `<title>` (50-60 chars)
- [ ] Every page has unique `<meta description>` (150-160 chars)
- [ ] Every page has `canonical` URL
- [ ] `sitemap.xml` includes all public pages
- [ ] `robots.txt` blocks /admin/ and /api/ but allows crawlers
- [ ] AI bots allowed: GPTBot, ClaudeBot, PerplexityBot
- [ ] JSON-LD schema on key content pages
- [ ] Open Graph tags for social sharing
- [ ] Images use Next.js `<Image>` with lazy loading
- [ ] All pages render full HTML server-side (view source to verify)
- [ ] Core Web Vitals: LCP < 2.5s, CLS < 0.1, INP < 200ms
- [ ] 301 redirects for renamed routes
- [ ] Submit sitemap in Google Search Console
- [ ] Heading hierarchy: single H1, logical H2->H3
- [ ] Mobile-responsive: same content on mobile and desktop
