---
name: pantry-digest
description: "Generate a Pantry-style AI news digest webpage (茶水间 · The Pantry) — masonry cards, cream palette, bilingual EN/ZH, click-to-expand modals with real source links and community reactions. Pulls real news from company blogs (Anthropic, OpenAI, Google, DeepMind, Microsoft, Mistral), aggregators (TechCrunch, HuggingFace, arXiv), newsletters (Every, The Rundown, Superhuman, Lenny's), and X/Reddit via HN Algolia. Defaults to today's news. Commands — /pantry-generate, /pantry-add, /pantry-remove, /pantry-list, /pantry-sources, /pantry-help. Trigger words — pantry, the pantry, 茶水间, refresh the pantry, AI digest, AI 日报, AI news page, daily AI roundup, generate the pantry, 生成 AI 新闻网页, pantry 一下."
---

# The Pantry · 茶水间 Skill

Generates a self-contained `index.html` page in **The Pantry** style:
masonry cards, cream palette, bilingual EN/ZH (loads in English by default,
toggleable), real source links, community reactions, click-to-expand
modals. **All stories are real, fetched live from public sources.**

## Files in this skill

- **This file (`SKILL.md`)** — frontmatter for the harness, command router, behavior contract
- **[`AGENT.md`](AGENT.md)** — full 10-step execution playbook
- **[`commands/`](commands/)** — one `.md` per `/pantry-*` slash command, each with its own action contract
- **[`default-sources.yaml`](default-sources.yaml)** — default 16-source roster; the `custom:` block at the bottom holds user-added sources
- **[`template.html`](template.html)** — the rendered HTML shell with `__NEWS_JSON__` and hero placeholders

## Slash commands

When the user types `/pantry-*`, read the matching file under `commands/`
**before acting** — that file is the source of truth for that command's behavior.

| Command | What it does | Playbook file |
|---|---|---|
| `/pantry-generate [scope]` | Generate the digest page. With no scope → today's news. With scope → respects user's filter (e.g. "only model releases", "focus on robotics", "in Chinese only"). | [`commands/generate.md`](commands/generate.md) |
| `/pantry-add <name> <url> [options]` | Add a new source to `default-sources.yaml` under `custom:`. Asks for missing brand color / category if not provided. | [`commands/add.md`](commands/add.md) |
| `/pantry-remove <name>` | Remove a source from `default-sources.yaml`. Refuses on ambiguous matches. | [`commands/remove.md`](commands/remove.md) |
| `/pantry-list` | List all currently active sources, grouped by `defaults:` vs `custom:`, with their category and priority. | [`commands/list.md`](commands/list.md) |
| `/pantry-sources [scope]` | Same as `/pantry-list` but filterable by category, priority, or kind. | [`commands/sources.md`](commands/sources.md) |
| `/pantry-help` | Show the command list and quick-start. | [`commands/help.md`](commands/help.md) |

## When to invoke (no command typed)

If the user types any of these natural-language triggers, route to `/pantry-generate`:

- "Refresh the pantry"
- "生成今天的茶水间" / "更新茶水间"
- "Make me a daily AI digest" / "AI news page for today"
- "What's everyone in AI talking about — make me a page"
- "Update my AI redbook"
- "Pantry 一下"
- Bare `/pantry-digest`

## Behavior contract (binding, across all commands)

1. **WebSearch is the first tool.** Always issue ≥3 parallel WebSearch queries
   before any WebFetch. Use WebSearch results to drive the targeted WebFetch calls.
2. **Default scope is today.** Search the last 24h. Widen only if <8 stories.
3. **Real images first.** Try og:image → source thumbnail → favicon →
   `brandCover()` SVG fallback. Set both `image` (real URL) and
   `imageFallback` (SVG) so broken images degrade gracefully.
4. **User-defined sources are respected.** If the user names sources or
   provides a list, merge them into `default-sources.yaml`'s `custom:` block
   for this run. Maintain the same Pantry layout and styling.
5. **No invented data.** Every URL must be real and reachable; every quote
   traces to a real person on record; every engagement number is the real
   number HN / the platform reports.
6. **Output location.** Default `AI Redbook/index.html` in CWD, override on user request.
7. **Default language is English.** Don't change the `let LANG = 'en'` line.

## How the agent should walk in

1. **If the user typed `/pantry-<command>`**, read `commands/<command>.md` first.
2. **Otherwise**, read [`AGENT.md`](AGENT.md) end-to-end — it has the full
   10-step execution flow, the news-item JSON schema, the integration-card
   formatting rules, the HN Algolia query templates, and the fact-checking bar.

This file is just the entry pointer + command router.
