# Quick Start

For experienced Claude Code users who just want to get going.

## Setup (2 minutes)

```bash
# Clone and install the skill
git clone https://github.com/YOUR_USERNAME/nocode-to-nextjs.git /tmp/nocode-to-nextjs
mkdir -p your-project/.claude/skills
cp -r /tmp/nocode-to-nextjs/skills/* your-project/.claude/skills/
cp /tmp/nocode-to-nextjs/CLAUDE.md your-project/.claude/CLAUDE.md
rm -rf /tmp/nocode-to-nextjs
```

## Migrate (one command)

```bash
cd your-lovable-or-base44-project
claude
```

Then say:

```
Migrate this app to Next.js with Supabase and deploy to Vercel
```

The agent auto-detects the platform and handles everything:
- Next.js setup (App Router, TypeScript, Tailwind)
- Supabase 3-client architecture
- Route migration with SEO metadata
- Auth middleware
- Sitemap, robots.txt, JSON-LD
- Vercel deployment

## Key Files the Agent Creates

```
src/app/layout.tsx              # Root layout
src/app/page.tsx                # Home page wrapper
src/app/sitemap.ts              # Dynamic sitemap
src/app/robots.ts               # Robots configuration
src/lib/supabase/client.ts      # Browser Supabase client
src/lib/supabase/server.ts      # Server Supabase client
src/lib/supabase/static.ts      # Build-time Supabase client
src/middleware.ts               # Auth middleware
src/app/error.tsx               # Error boundary
src/app/not-found.tsx           # 404 page
.npmrc                          # legacy-peer-deps=true
next.config.ts                  # Next.js configuration
```

## Environment Variables (set in Vercel)

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_SITE_URL=https://your-domain.com
```
