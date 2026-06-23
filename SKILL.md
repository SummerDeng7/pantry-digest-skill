---
name: pantry-digest
description: "Generate a daily digest webpage from real news sources (茶水间 · The Pantry style — masonry cards, cream palette, bilingual EN/ZH, click-to-expand modals with original source links and community reactions). Defaults to AI news with a curated 16-source roster (Anthropic, OpenAI, Google, DeepMind, Microsoft, Mistral, TechCrunch, HuggingFace, arXiv, Every, The Rundown, Superhuman, Lenny's, HN, plus X/Reddit via HN Algolia). Works for ANY topic — pass a topic scope (biotech, climate, finance, sports, fashion, etc.) and the agent will search for the right sources or use what you've added. All stories are real, no invented data. Output: pantry-digest.html in the current directory. Commands — /pantry-generate, /pantry-add, /pantry-remove, /pantry-list, /pantry-sources, /pantry-help. Trigger words — pantry, the pantry, 茶水间, refresh the pantry, daily digest, AI digest, AI 日报, news digest page, daily roundup, generate the pantry, pantry 一下."
---

# The Pantry · 茶水间 Skill

A **daily digest generator**. Pulls real news from real sources, writes a
self-contained `pantry-digest.html` page in **The Pantry** style: masonry
cards, cream palette, bilingual EN/ZH (loads in English by default,
toggleable), real source links, community reactions, click-to-expand modals.

**Default topic is AI** (with a curated 16-source roster). But this skill
is topic-agnostic — point it at biotech, climate, finance, sports,
fashion, your favorite niche, and it will search for the right sources or
use the ones you've added with `/pantry-add`.

> **All stories are real.** Fetched live from public sources every run.
> No invented news, no fake engagement numbers, no fabricated quotes.

## Files in this skill

- **This file (`SKILL.md`)** — frontmatter for the harness, command router, behavior contract
- **[`AGENT.md`](AGENT.md)** — full 10-step execution playbook
- **[`commands/`](commands/)** — one `.md` per `/pantry-*` slash command, each with its own action contract
- **[`default-sources.yaml`](default-sources.yaml)** — default 16-source AI roster; `custom:` block holds user-added sources of any topic
- **[`template.html`](template.html)** — the rendered HTML shell with `__NEWS_JSON__` and hero placeholders

## Slash commands

When the user types `/pantry-*`, read the matching file under `commands/`
**before acting** — that file is the source of truth for that command's behavior.

| Command | What it does | Playbook file |
|---|---|---|
| `/pantry-generate [scope]` | Generate the digest page. With no scope → today's AI news from default sources. With scope → respects user's filter (e.g. "biotech this week", "only model releases", "focus on robotics", "fashion news", "in Chinese only"). | [`commands/generate.md`](commands/generate.md) |
| `/pantry-add <name> <url> [options]` | Add a new source to `default-sources.yaml` under `custom:`. Asks for missing brand color / category if not provided. Works for any topic. | [`commands/add.md`](commands/add.md) |
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
- "Make a daily biotech digest" / "Today's climate news as a page"
- "Pantry 一下"
- Bare `/pantry-digest`

## Behavior contract (binding, across all commands)

1. **WebSearch is the first tool.** Always issue ≥3 parallel WebSearch queries
   before any WebFetch. Use WebSearch results to drive the targeted WebFetch calls.
2. **Default scope is today, default topic is AI.** When the user gives no
   scope: search the last 24h, use the AI defaults from `default-sources.yaml`.
   Widen the time window only if <8 stories.
3. **Off-topic scope → adapt sources.** If the user names a non-AI topic
   (biotech, finance, sports, etc.), do NOT force the AI default sources.
   Either (a) use only the user's `custom:` entries if they match the topic,
   or (b) WebSearch to discover topic-appropriate sources on the fly, or
   (c) ask the user which 2–3 sources they want.
4. **Real images first.** Try og:image → source thumbnail → favicon →
   `brandCover()` SVG fallback. Set both `image` (real URL) and
   `imageFallback` (SVG) so broken images degrade gracefully.
5. **User-defined sources are respected.** If the user names sources or
   provides a list, merge them in. Maintain the same Pantry layout.
6. **No invented data.** Every URL must be real and reachable; every quote
   traces to a real person on record; every engagement number is the real
   number HN / the platform reports.
7. **Output location.** Default `pantry-digest.html` in CWD, override on user request.
8. **Default language is English.** Don't change the `let LANG = 'en'` line
   unless the user explicitly says "in Chinese only."

## How the agent should walk in

1. **If the user typed `/pantry-<command>`**, read `commands/<command>.md` first.
2. **Otherwise**, read [`AGENT.md`](AGENT.md) end-to-end — it has the full
   10-step execution flow, the news-item JSON schema, the integration-card
   formatting rules, the HN Algolia query templates, and the fact-checking bar.

This file is just the entry pointer + command router.
