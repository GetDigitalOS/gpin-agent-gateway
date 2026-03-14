# /setup-docs

Perform this exact sequence of one-time setup steps for this project in Claude Code:

---

## Step 1: Handle CLAUDE.md

**CLAUDE.md is the persistent system prompt for every Claude Code session in this workspace.** It is read automatically at the start of every session.

- If `CLAUDE.md` does not exist → create it starting with `# {PROJECT_NAME}` followed immediately by the mandatory block below.
- If `CLAUDE.md` already exists:
  - Check if it already contains the exact heading `## Docs-First Rule (MANDATORY)`.
  - If yes → do nothing to that section (don't duplicate).
  - If no → insert the block below immediately after the first `# ` or `## ` heading (append at end if no heading exists).

### The block to insert (do not alter any wording):

```
## Docs-First Rule (MANDATORY)

This block is the **standing instruction** for every Claude Code session, every sub-agent, and every /command invocation in this workspace.

### Core Rule (enforced on every single interaction)

**For every plan, build, feature, refactor, bugfix, or code change request:**

1. **Immediately and fully read the entire `/docs` folder** before doing anything else.
   - Treat `/docs` as the single source of truth for architecture, decisions, specifications, planning, examples, and reference material.
   - Respect this exact layout (do not create folders outside this structure without explicit approval):
     ```
     docs/
     ├── architecture/
     ├── examples/
     ├── planning/
     ├── reference/
     ├── specifications/
     └── (any .md files explicitly added at top level)
     ```
   - If a `/docs` folder does not yet exist, note that and proceed — but flag it as needing backfill.

2. **Maintain documentation as you go — lean & zero-bloat rule**
   - Only update or add to `/docs` when the change is **meaningful** for a new developer or future team handoff.
   - Prefer **editing an existing file** over creating a new one.
   - Never add filler, repetition, or "nice-to-have" sections.
   - Keep every file concise, scannable, and accurate.
   - If something becomes outdated → mark it `[DEPRECATED — see new location]` or move it to a `docs/archive/` subfolder. Never delete history.
   - After any code change that affects design, specs, architecture, or workflow, **always include** in your response:
     > "Documentation update required: [exact file(s) + 1-sentence summary of what changed]"

3. **Documentation philosophy (enforced)**
   - Living, minimal, high-signal only.
   - One source of truth per topic.
   - No duplicate information between files.
   - Update immediately when the project evolves — never let docs drift from reality.
   - Goal: A new senior developer could be fully productive in <2 hours just by reading `/docs`.

### How this interacts with hub sync

- This `Docs-First Rule` block is **hub-canonical** — `hub sync` may re-insert it if removed.
- Everything else in this `CLAUDE.md` is **project-owned** — hub will never overwrite it.
- If hub updates this block, accept the update. It will never conflict with project-specific content.
```

---

## Step 2: Create docs/docs-maintenance.md (only if missing)

Create the file at exactly `docs/docs-maintenance.md` with this exact content:

```markdown
# Docs Maintenance Rules (for AI + Humans)

These rules are binding for every Claude Code session and every code change in this project.

## Always
- Read the entire /docs folder before starting any task.
- Update docs only when the change matters for a future developer or team handoff.
- Prefer editing existing files over creating new ones.
- Keep additions concise — prefer <10 lines when possible.
- Use tables, lists, and diagrams only when they reduce total words.
- Flag documentation drift explicitly: "Documentation update required: [file] — [what changed]"

## Never
- Add tutorial-style fluff, repetition, or overview sections that already exist elsewhere.
- Create a new subfolder unless a genuinely new category appears (e.g., new `security/` or `deployment/` concern with 3+ related files).
- Let docs get out of sync with code — if you changed it, update the doc in the same session.
- Delete history — use `[DEPRECATED]` markers or move to `docs/archive/`.

## Approved top-level structure (do not add folders without explicit approval)
- `architecture/`   — system design, ADRs, data models, diagrams
- `examples/`       — working code samples, patterns, usage demos
- `planning/`       — roadmaps, milestones, feature specs in progress
- `reference/`      — API docs, config schemas, environment variables, quick-ref tables
- `specifications/` — finalized feature specs, acceptance criteria
- `archive/`        — deprecated content (never delete, just move here)
- `docs-maintenance.md` — this file

## File naming
- Lowercase, hyphen-separated: `auth-flow.md`, not `AuthFlow.md`
- Be specific: `supabase-rls-patterns.md`, not `database.md`
- ADRs: `adr-001-framework-choice.md` (zero-padded number + slug)

## When to create a new file vs. edit an existing one
- Edit existing: adding a section, updating a pattern, correcting outdated info
- New file: entirely new topic with no existing home, OR existing file exceeds ~200 lines and has a clean split point
- When in doubt: edit existing
```

---

## Step 3: Backfill the /docs folder

Using this `CLAUDE.md`, the existing codebase, and any project context visible in the session, populate the `/docs` folder with meaningful, accurate documentation.

**Backfill priorities (in order):**

1. **`docs/architecture/`** — What is this project? What are its key systems, data flows, and deployment model? Include a brief system overview and any non-obvious architectural decisions already made.

2. **`docs/specifications/`** — What does this project do? What are the core features, user flows, and acceptance criteria for work that already exists?

3. **`docs/reference/`** — What does a developer need to get started? Environment variables, key commands, config schema, external service dependencies.

4. **`docs/planning/`** — What's next? If there's an active roadmap or known next steps, document them. Even a simple `roadmap.md` with 3-5 bullet points is better than nothing.

5. **`docs/examples/`** — Are there patterns, conventions, or non-obvious code approaches in this project that a new developer would need to know? Document 1-3 of the most important ones.

**Backfill rules:**
- Only create files you can populate with real, accurate content — no placeholder stubs.
- Infer from the codebase, not from imagination.
- If you're uncertain about something, note the uncertainty inline: `[VERIFY: is this still the deployment target?]`
- Keep each file scannable — use headers, short paragraphs, and tables.

---

## After completing all steps

Output:
1. A clean tree view of the `/docs` folder (including all new/updated files)
2. A summary of exactly what was created or modified in `CLAUDE.md`, `docs/docs-maintenance.md`, and `/docs/`
3. The 2-3 most important docs files for the project owner to review and expand next, with a one-sentence reason for each

---

## Hub Integration Notes

This command is hub-canonical and synced to all Tier 2+ projects via `hub sync`.

- **Safe to re-run** — idempotent. Re-running after docs exist will only add missing sections.
- **Tier targeting**: Sync to Tier 2+ projects. Tier 1 (static/marketing) gets a lighter stub: just the `Docs-First Rule` block in CLAUDE.md and a minimal `docs/reference/` with setup instructions.
- **CLAUDE.md ownership**: Hub owns only the `Docs-First Rule` block. All other content in CLAUDE.md is project-owned and will never be overwritten by `hub sync`.
- **When to run**: Once per project during initial retrofit (`hub retrofit`), or on-demand when a project's docs are missing or stale.
