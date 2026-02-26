# Contributing

Thanks for your interest in improving this migration agent! Here's how you can help.

## Ways to Contribute

### 1. Add Support for a New Platform

We currently support Lovable and Base44. If you've migrated an app from another no-code platform (Bolt, V0, Webflow, Bubble, etc.), you can add support:

1. Create a new file: `skills/<platform-name>-platform.md`
2. Follow the structure of `skills/base44-platform.md` as a template
3. Include:
   - What the platform generates (file structure)
   - Key differences from other platforms
   - Platform-specific migration steps
   - SDK/API replacement patterns (if applicable)
   - Platform-specific gotchas
4. Update `CLAUDE.md` to detect the new platform (add signals to the Platform Detection table)
5. Update `README.md` to list the new platform in the Supported Platforms table

See [docs/adding-platforms.md](docs/adding-platforms.md) for a detailed template.

### 2. Improve Existing Documentation

- Fix errors or outdated patterns
- Add new troubleshooting entries
- Improve code examples
- Clarify confusing sections

### 3. Report Issues

Found a bug or a migration step that doesn't work? Open an issue with:
- The source platform (Lovable, Base44, etc.)
- The specific error or problem
- Steps to reproduce
- Your Node.js and Next.js versions

## Guidelines

- Keep the tone friendly and practical
- Use generic placeholders (e.g., `your-domain.com`, `your-project.supabase.co`) -- never include real credentials, domains, or project IDs
- Test your migration patterns on a real project before submitting
- Keep code examples focused and minimal

## Pull Request Process

1. Fork the repo
2. Create a branch: `git checkout -b add-bolt-support`
3. Make your changes
4. Test by using the skill files with Claude Code on a real project
5. Submit a PR with a clear description of what you changed and why

## Code of Conduct

Be kind, be helpful, be respectful. We're all here to make no-code migrations easier.
