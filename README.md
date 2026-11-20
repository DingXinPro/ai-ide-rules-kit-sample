# AI IDE Rules Kit — sample

Free, public sample of **AI IDE Rules Kit**: two installable project rules in the official formats that Cursor, Windsurf, and AGENTS.md-compatible tools actually read in November 2026.

This is a documentation snapshot, checked against official docs on **20 November 2026**. It is not a lifetime-updated policy, not "50+ rules," and not an official partnership with Cursor, Windsurf, Vercel, or anyone else.

This sample is published at [https://github.com/DingXinPro/ai-ide-rules-kit-sample](https://github.com/DingXinPro/ai-ide-rules-kit-sample).

## Why this format (not a root `.cursorrules`)

Cursor official help (checked 2026-11-20) calls a root `.cursorrules` file **legacy** and says it will be deprecated. Current project rules are:

- Cursor: `.cursor/rules/*.mdc` with YAML frontmatter (`description`, `globs`, `alwaysApply`). A plain `.md` file in that folder is ignored.
- Windsurf Cascade: `.windsurf/rules/*.md` with `trigger` frontmatter (`glob`, `model_decision`, `manual`, or `always_on`). Workspace rule files have a 12,000 character limit.
- Cross-tool: a root `AGENTS.md` in plain Markdown (no frontmatter), as described at [https://agents.md/](https://agents.md/).

This sample ships all three for the two included rules.

## What is included

1. **`typescript-strict`** — `"strict": true`, no casual `any`, official TypeScript strict-family flags, narrowing for `fetch` / JSON.
2. **`nextjs-app-router`** — Next.js **16** App Router and Server Components (async `params`, `cookies()`, `headers()`; `proxy.ts` in 16; keep secrets off the client).

Each rule is provided as:

- `.cursor/rules/<name>.mdc`
- `.windsurf/rules/<name>.md`
- a section in `AGENTS.md`

The rule bodies match the corresponding files in the paid full pack.

## Install

**Pick one primary format.** If you copy both `AGENTS.md` and the `.mdc` / `.windsurf` files, many tools will load the same text twice.

### Cursor

1. Copy `.cursor/rules/` into your project root.
2. Keep the frontmatter. These sample rules use `alwaysApply: false` and file `globs`.
3. Start a new Agent chat. You can also @-mention a rule by filename.

Do not rely on a root `.cursorrules` file as the main hook.

### Windsurf

1. Copy `.windsurf/rules/` into your workspace or repo root.
2. Keep `description`, `globs`, and `trigger: glob`.

### AGENTS.md / Claude Code and other tools

Copy `AGENTS.md` to your repository root if the tool you use reads that file (Cursor, Windsurf, Devin, Claude Code, and others listed on [agents.md](https://agents.md/)). If you already installed the `.mdc` or `.windsurf` files, use a short project-specific `AGENTS.md` instead of this full sample file so the two rules are not always injected twice.

## Full pack

The paid pack adds six more rules (FastAPI + Pydantic v2, Tailwind CSS v4, Supabase Auth/RLS, agentic code review, Conventional Commits / PR, and REST/GraphQL error handling), plus Cursor, Windsurf, optional Devin, and AGENTS.md copies of all eight, with installation notes and a sources list.

- Price: **$19 USD, one-time** (no lifetime-update promise).
- Claim: **8 rules, checked against official docs in November 2026, in the current official MDC format.**
- Product URL: [https://dingster13.gumroad.com/l/ai-ide-rules-kit](https://dingster13.gumroad.com/l/ai-ide-rules-kit)
- Seller shop: [https://dingster13.gumroad.com](https://dingster13.gumroad.com)

This sample is MIT-licensed (see `LICENSE`). The paid pack has a separate license and is not free to republish as a competing pack.
