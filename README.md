# nocode-to-nextjs

Migrate your Lovable or Base44 app to production-grade Next.js -- with one Claude Code agent.

This is a **Claude Code skill** (an AI agent configuration) that knows how to take a no-code platform app and convert it into a fully optimized Next.js application with Supabase backend, SEO, and Vercel deployment.

## What It Does

- Converts a Vite + React Router SPA to Next.js 15+ App Router
- Sets up the Supabase 3-client architecture (browser, server, build-time)
- Adds SSR/SSG for full SEO: per-page metadata, sitemap, robots.txt, JSON-LD schemas
- Creates auth middleware with cookie-based sessions
- Handles all the tricky migration gotchas (40+ `'use client'` directives, Tailwind version conflicts, import path rewrites, etc.)
- Deploys to Vercel with proper environment variable setup

## Supported Platforms

| Platform | Status | Notes |
|----------|--------|-------|
| [Lovable](https://lovable.dev) | Stable | Battle-tested, comprehensive coverage |
| [Base44](https://base44.com) | Beta | Core patterns covered, includes SDK replacement guide |

## Prerequisites

Before you start, make sure you have:

- [ ] **Node.js 18+** installed ([download](https://nodejs.org/))
- [ ] **A Lovable or Base44 project** exported or available on GitHub
- [ ] **A Supabase account** ([supabase.com](https://supabase.com) -- free tier works)
- [ ] **A Vercel account** ([vercel.com](https://vercel.com) -- free tier works)
- [ ] **Claude Code** installed (see below)

---

## Installing Claude Code

If you haven't used Claude Code before -- it's Anthropic's CLI tool that lets Claude work directly in your terminal and codebase. Think of it as an AI pair programmer that can read, write, and run code.

### Step 1: Install

```bash
npm install -g @anthropic-ai/claude-code
```

### Step 2: Set Up Your API Key

You'll need an Anthropic API key:

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account (or sign in)
3. Go to API Keys and create a new key
4. Set it in your terminal:

```bash
export ANTHROPIC_API_KEY=your-key-here
```

To make it permanent, add that line to your `~/.zshrc` or `~/.bashrc`.

### Step 3: Test It

```bash
cd any-project-folder
claude
```

You should see Claude Code start up and be ready to help. Type `exit` to quit.

That's it -- Claude Code is now your AI coding partner.

---

## Installing This Migration Agent

### Option A: Add to Your Project (Recommended)

Clone this repo and copy the skill files into your project:

```bash
# Clone the migration agent
git clone https://github.com/YOUR_USERNAME/nocode-to-nextjs.git /tmp/nocode-to-nextjs

# Copy into your project's Claude config
mkdir -p your-project/.claude/skills
cp -r /tmp/nocode-to-nextjs/skills/* your-project/.claude/skills/
cp /tmp/nocode-to-nextjs/CLAUDE.md your-project/.claude/CLAUDE.md

# Clean up
rm -rf /tmp/nocode-to-nextjs
```

### Option B: Add as Global Skill (Available Everywhere)

```bash
git clone https://github.com/YOUR_USERNAME/nocode-to-nextjs.git /tmp/nocode-to-nextjs

mkdir -p ~/.claude/skills
cp -r /tmp/nocode-to-nextjs/skills/* ~/.claude/skills/

rm -rf /tmp/nocode-to-nextjs
```

---

## Usage

1. Open your no-code project in the terminal:
   ```bash
   cd your-lovable-project
   ```

2. Start Claude Code:
   ```bash
   claude
   ```

3. Tell it what you want:
   ```
   Migrate this Lovable app to Next.js with Supabase and deploy to Vercel
   ```

4. The agent will:
   - Analyze your project structure and detect the platform
   - Set up Next.js with App Router
   - Create the Supabase 3-client architecture
   - Migrate all routes with SEO metadata
   - Add `'use client'` directives where needed
   - Fix all build errors
   - Set up sitemap, robots.txt, and JSON-LD
   - Deploy to Vercel

You can also ask for specific parts:
- `"Just set up the Supabase clients for Next.js"`
- `"Add SEO metadata to all my pages"`
- `"Fix the build errors after migration"`
- `"Set up auth middleware"`

---

## What Gets Migrated

| Before (No-Code SPA) | After (Next.js) |
|-----------------------|-----------------|
| No SEO -- Google sees empty HTML | Full SSR with metadata on every page |
| No sitemap | Dynamic `sitemap.xml` from your database |
| No robots.txt | Dynamic `robots.ts` allowing AI crawlers |
| Client-side routing only | File-based routing with layouts |
| Single Supabase client | 3-client architecture (browser/server/static) |
| Client-side auth only | Middleware + cookie-based auth |
| No API routes | Route Handlers for sensitive operations |
| No image optimization | Next.js Image with WebP + lazy loading |
| Vite build | Next.js build with ISR support |

## Migration Timeline

The agent handles everything, but here's roughly how long each phase takes:

| Phase | Duration | What Happens |
|-------|----------|-------------|
| Setup | ~30 min | Install deps, create configs, rename directories |
| Core Files | ~1 hour | Root layout, Supabase clients, middleware, error pages |
| Route Migration | ~2-4 hours | Page wrappers, metadata, import fixes |
| SEO | ~1-2 hours | Sitemap, robots.txt, JSON-LD schemas |
| Deploy | ~30 min | Push to GitHub, Vercel setup, env vars |

Total: roughly half a day for a typical project.

---

## Troubleshooting

If something goes wrong during migration, the agent will try to fix it automatically. But if you're stuck:

- **Build errors?** See [Supabase Troubleshooting](skills/supabase-troubleshooting.md)
- **Deployment issues?** See [Deployment Troubleshooting](skills/deployment-troubleshooting.md)
- **Base44-specific problems?** See [Base44 Platform Guide](skills/base44-platform.md)
- **Still stuck?** [Open an issue](../../issues) and describe what happened

---

## Project Structure

```
nocode-to-nextjs/
├── CLAUDE.md                           # Agent brain -- instructions + workflow
├── skills/
│   ├── migration-skill.md              # Core migration knowledge (1400+ lines)
│   ├── base44-platform.md              # Base44-specific SDK replacement patterns
│   ├── supabase-troubleshooting.md     # Supabase + Next.js common issues
│   └── deployment-troubleshooting.md   # Vercel deployment issues
├── docs/
│   ├── quick-start.md                  # Condensed guide for experienced users
│   └── adding-platforms.md             # How to contribute new platform support
├── README.md                           # You're reading this
├── CONTRIBUTING.md                     # How to contribute
└── LICENSE                             # MIT
```

---

## Contributing

Want to improve the migration agent or add support for another platform (Bolt, V0, Webflow, etc.)? See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

In short:
1. Fork the repo
2. Add your platform file to `skills/`
3. Update `CLAUDE.md` to detect the new platform
4. Submit a PR

---

## License

MIT -- see [LICENSE](LICENSE) for details.
