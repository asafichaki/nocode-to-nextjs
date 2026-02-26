# Adding Support for a New Platform

Want to add migration support for a new no-code platform? Follow this guide.

## Step 1: Create the Platform File

Create `skills/<platform-name>-platform.md` with this structure:

```markdown
# <Platform Name> Platform Migration Guide

This guide covers <Platform Name>-specific patterns. Use it **alongside**
`migration-skill.md` which contains the core Next.js migration knowledge.

---

## What <Platform Name> Generates

<Describe the file structure the platform produces>

```
project/
├── src/
│   ├── ...
├── package.json
└── ...
```

---

## Key Differences from Lovable/Base44

| Aspect | Lovable | <Platform Name> |
|--------|---------|-----------------|
| File extensions | .tsx | ... |
| Router | React Router v6 | ... |
| Backend | Direct Supabase | ... |
| Auth | Supabase Auth | ... |
| Styling | Tailwind + shadcn | ... |

---

## Platform-Specific Migration Steps

<Any additional steps needed beyond the core migration>

---

## SDK/API Replacement Patterns

<If the platform has its own SDK, show before/after code>

---

## Common Gotchas

<Platform-specific issues and their fixes>

---

## Migration Checklist

- [ ] Step 1
- [ ] Step 2
- [ ] ...
```

## Step 2: Update CLAUDE.md

Add detection signals to the Platform Detection table:

```markdown
| Signal | Platform |
|--------|----------|
| `<unique-package>` in package.json | <Platform Name> |
| `<unique-env-var>` in .env | <Platform Name> |
```

Add a routing entry:

```markdown
| <Platform Name> detected | `skills/<platform-name>-platform.md` |
```

## Step 3: Update README.md

Add the platform to the Supported Platforms table:

```markdown
| <Platform Name> | Beta | Brief description |
```

## Step 4: Test

1. Get a real project built with the platform
2. Add the skill files to the project
3. Run Claude Code and ask it to migrate
4. Verify the migration works end-to-end
5. Document any issues you find in the platform file

## Tips

- Focus on what's **different** from the core migration. Don't repeat content from `migration-skill.md`.
- Include real code examples with before/after patterns
- Document the gotchas -- those are the most valuable parts
- Use generic placeholders, never real credentials or domains
