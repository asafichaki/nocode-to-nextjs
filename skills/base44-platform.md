# Base44 Platform Migration Guide

This guide covers Base44-specific patterns. Use it **alongside** `migration-skill.md` which contains the core Next.js migration knowledge.

---

## What Base44 Generates

Base44 produces a React SPA similar to Lovable, but with key differences:

```
project/
├── src/
│   ├── pages/              # Route components (Home.jsx, Settings.jsx)
│   ├── components/         # Reusable UI + ui/ subfolder (shadcn-like)
│   ├── api/                # Base44 SDK client + entity imports
│   │   └── entities.js     # Entity definitions (BlogPost, User, etc.)
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Base44 integration + app config
│   └── utils/              # Utility functions
├── entities/               # JSON schema data model definitions
├── functions/              # TypeScript backend functions (Deno runtime)
├── package.json            # Dependencies including @base44/sdk
├── vite.config.js          # Vite configuration
├── tailwind.config.js      # Tailwind CSS config
└── index.html              # Entry point
```

---

## Key Differences: Base44 vs Lovable

| Aspect | Lovable | Base44 |
|--------|---------|--------|
| File extensions | `.tsx` (TypeScript) | `.jsx` (JavaScript), sometimes `.tsx` |
| Backend | Direct Supabase calls | `@base44/sdk` abstraction layer |
| Database | Supabase (you own it) | Base44 managed NoSQL (you do NOT own it) |
| Auth | Supabase Auth | Base44 built-in auth |
| Entity pattern | Direct Supabase queries | `Entity.list()`, `Entity.create()`, etc. |
| Edge Functions | Supabase Edge Functions (Deno) | Base44 backend functions (Deno) |
| Data models | Supabase tables (SQL) | JSON schema files in `entities/` |
| Env vars | `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY` | Base44 SDK handles internally |
| Export | Full frontend + backend access | Frontend only (backend stays on Base44) |

---

## The Critical Extra Step: Database Migration

Unlike Lovable (where you already own the Supabase database), Base44 requires you to:

1. **Create a new Supabase project** at [supabase.com](https://supabase.com)
2. **Map Base44 entities to Supabase tables** using the JSON schemas in `entities/`
3. **Create SQL tables** matching each entity
4. **Export data from Base44** and import into Supabase
5. **Set up Supabase Auth** to replace Base44's built-in auth
6. **Replace all `@base44/sdk` calls** with Supabase client calls

### Entity to Table Mapping

Base44 entities (in `entities/` directory) are JSON schemas. Convert them to SQL:

```
Base44 Entity (entities/blog_post.json)     ->  Supabase Table
{                                                CREATE TABLE blog_posts (
  "name": "BlogPost",                             id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  "fields": {                                      title TEXT NOT NULL,
    "title": { "type": "string" },                 content TEXT,
    "content": { "type": "text" },                 author TEXT,
    "author": { "type": "string" },                is_published BOOLEAN DEFAULT false,
    "is_published": { "type": "boolean" }          created_at TIMESTAMPTZ DEFAULT now(),
  }                                                updated_at TIMESTAMPTZ DEFAULT now()
}                                                );
```

### Set Up Row Level Security (RLS)

After creating tables, enable RLS and add policies:

```sql
-- Enable RLS
ALTER TABLE blog_posts ENABLE ROW LEVEL SECURITY;

-- Public read access for published posts
CREATE POLICY "Public can read published posts"
  ON blog_posts FOR SELECT
  USING (is_published = true);

-- Authenticated users can manage posts
CREATE POLICY "Authenticated users can manage posts"
  ON blog_posts FOR ALL
  USING (auth.role() = 'authenticated');
```

---

## SDK Replacement Patterns

### CRUD Operations

```javascript
// BEFORE (Base44)
import { BlogPost } from '@/api/entities'

// List
const posts = await BlogPost.list()
const filtered = await BlogPost.list({ is_published: true })

// Create
const newPost = await BlogPost.create({ title: 'Hello', content: '...' })

// Update
await BlogPost.update(postId, { title: 'Updated' })

// Delete
await BlogPost.delete(postId)

// Get one
const post = await BlogPost.get(postId)
```

```typescript
// AFTER (Supabase via Next.js)
import { createClient } from '@/lib/supabase/client'
const supabase = createClient()

// List
const { data: posts } = await supabase.from('blog_posts').select('*')
const { data: filtered } = await supabase.from('blog_posts').select('*').eq('is_published', true)

// Create
const { data: newPost } = await supabase.from('blog_posts').insert({ title: 'Hello', content: '...' }).select().single()

// Update
await supabase.from('blog_posts').update({ title: 'Updated' }).eq('id', postId)

// Delete
await supabase.from('blog_posts').delete().eq('id', postId)

// Get one
const { data: post } = await supabase.from('blog_posts').select('*').eq('id', postId).single()
```

### Authentication

```javascript
// BEFORE (Base44)
import base44 from '@base44/sdk'

// Login
await base44.auth.loginViaEmailPassword(email, password)

// Get current user
const user = await base44.auth.me()

// Logout
await base44.auth.logout()

// Check if logged in
const isLoggedIn = base44.auth.isLoggedIn()
```

```typescript
// AFTER (Supabase Auth)
import { createClient } from '@/lib/supabase/client'
const supabase = createClient()

// Login
const { error } = await supabase.auth.signInWithPassword({ email, password })

// Get current user
const { data: { user } } = await supabase.auth.getUser()

// Logout
await supabase.auth.signOut()

// Check if logged in
const { data: { session } } = await supabase.auth.getSession()
const isLoggedIn = !!session
```

### File Storage

```javascript
// BEFORE (Base44)
import base44 from '@base44/sdk'
const url = await base44.storage.uploadFile(file)
```

```typescript
// AFTER (Supabase Storage)
import { createClient } from '@/lib/supabase/client'
const supabase = createClient()

const { data, error } = await supabase.storage
  .from('uploads')
  .upload(`files/${file.name}`, file)

const { data: { publicUrl } } = supabase.storage
  .from('uploads')
  .getPublicUrl(`files/${file.name}`)
```

---

## Backend Functions Migration

Base44 backend functions (in `functions/` directory) run on Deno. Migrate them to Supabase Edge Functions:

1. Create a `supabase/functions/` directory in your Next.js project
2. Copy function logic, adapting imports:

```typescript
// BEFORE (Base44 function)
import { serve } from "https://deno.land/std/http/server.ts"
import base44 from "base44-sdk/server"

serve(async (req) => {
  const data = await base44.entities.BlogPost.list()
  return new Response(JSON.stringify(data))
})
```

```typescript
// AFTER (Supabase Edge Function)
import { serve } from "https://deno.land/std/http/server.ts"
import { createClient } from "https://esm.sh/@supabase/supabase-js@2"

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
  )
  const { data } = await supabase.from('blog_posts').select('*')
  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json' },
  })
})
```

Deploy with: `supabase functions deploy function-name`

---

## Community Bridge: base44-to-supabase-sdk

The community has built a drop-in replacement SDK that can serve as an intermediate step. Search for `base44-to-supabase-sdk` on GitHub. It maps Base44's `Entity.list()`, `Entity.create()`, etc. to Supabase calls, allowing a gradual migration without rewriting all entity calls at once.

**Recommended approach:**
1. Start with the bridge SDK for quick migration
2. Gradually replace bridge calls with direct Supabase client calls
3. Remove the bridge once all calls are converted

---

## Migration Checklist (Base44-Specific)

- [ ] Created new Supabase project
- [ ] Mapped all entities to Supabase tables (SQL)
- [ ] Set up RLS policies for each table
- [ ] Migrated data from Base44 to Supabase
- [ ] Replaced `@base44/sdk` imports with Supabase client
- [ ] Replaced all entity CRUD calls
- [ ] Replaced Base44 auth with Supabase Auth
- [ ] Migrated backend functions to Edge Functions
- [ ] Migrated file storage to Supabase Storage
- [ ] Removed `@base44/sdk` from package.json
- [ ] Tested all functionality end-to-end
