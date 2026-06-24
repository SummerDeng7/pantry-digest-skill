# Pantry · 茶水间

A [Claude Code](https://claude.com/claude-code) skill that generates a
beautiful, real-news **daily digest webpage** in the **Pantry** style —
masonry cards, cream palette, bilingual EN/ZH, click-to-expand modals
with original source links and community reactions.

**Default topic is AI** (with a curated 16-source roster). But the skill
is topic-agnostic — point it at biotech, climate, finance, sports,
fashion, your favorite niche, and it figures out the sources.

> All stories are **real**, fetched live from public sources. No invented
> news, no fake engagement numbers, no fabricated quotes.

<!-- Add a screenshot once you have one:
![Pantry screenshot](docs/screenshot.png)
-->

## Install

```bash
git clone https://github.com/SummerDeng7/pantry-digest-skill ~/.claude/skills/pantry-digest
```

That's it. Restart Claude Code; the skill auto-loads. Run `/pantry-help`
to see all commands, or just type `refresh the pantry` in natural language.

To update later:

```bash
cd ~/.claude/skills/pantry-digest && git pull
```

## Slash commands

| Command | What it does |
|---|---|
| `/pantry-generate [scope]` | Generate today's digest. Scope is free-form: `"this week"`, `"focus on robotics"`, `"biotech news"`, `"finance digest"`, `"only my custom sources"`, `"in Chinese only"`, `"to ~/Desktop/today.html"`. |
| `/pantry-add <name-or-url>` | Add a source. Examples: `/pantry-add https://stratechery.com/` or `/pantry-add STAT News https://www.statnews.com/ news_aggregator`. |
| `/pantry-remove <name>` | Remove a source. Confirmation required for default sources. |
| `/pantry-list` | List all configured sources, grouped by defaults vs custom. |
| `/pantry-sources [filter]` | Filtered list: `newsletters`, `priority 1`, `custom`, `search anthropic`, etc. |
| `/pantry-issue [title]` | File a GitHub issue (bug / feature / question) against this repo. Uses your `gh` CLI; falls back to a pre-filled browser link. |
| `/pantry-help` | Show the command list. |

Natural-language triggers also work: *"refresh the pantry"*, *"茶水间一下"*,
*"make me a daily AI digest"*, *"daily climate digest please"* — all route
to `/pantry-generate`.

## What it does on each run

1. Runs **parallel WebSearch** queries to discover today's biggest stories
   (in your topic, AI by default)
2. **WebFetches** the chosen articles, newsletters, and HN / X / Reddit threads
3. Extracts **real cover images** (og:image / hero image) when available;
   falls back to brand-colored SVG covers when not
4. Composes **10–12 cards** spanning the categories that matter for the topic
5. Renders a self-contained `pantry-digest.html` in the current directory
   (or wherever you ask)

## Default source roster (AI topic)

| Category | Sources |
|---|---|
| Company / lab blogs | Anthropic · OpenAI · Google · Google DeepMind · Microsoft AI · Mistral AI |
| Aggregators | TechCrunch AI · Hugging Face Blog · arXiv cs.AI |
| Newsletters | Every · The Rundown · Superhuman AI · Lenny's Newsletter |
| Community | Hacker News (direct), plus X & Reddit via HN Algolia* |

*Reddit and X block direct fetching without auth. HN Algolia indexes every
X / Reddit / news URL that hits Hacker News, with real points and comment
counts — the only auth-free way to get real X / Reddit engagement data
from inside Claude Code.

## Using a non-AI topic

Three ways, pick whichever fits:

**1. One-shot — let the agent search for sources.**
```
/pantry-generate today's biotech news
```
The agent will WebSearch for biotech sources, pick credible ones (STAT,
Endpoints, NEJM, etc.), and build the digest. No setup required.

**2. Build a custom roster over time.**
```
/pantry-add https://www.statnews.com/
/pantry-add https://endpts.com/
/pantry-add https://www.nejm.org/
/pantry-generate biotech, only my custom sources
```
Future runs reuse the roster instantly.

**3. Manual — edit `default-sources.yaml` and append under `custom:`**:

```yaml
custom:
  - name: STAT News
    short: STAT
    url: https://www.statnews.com/
    kind: news_aggregator
    brand_color: "#E5184C"
    text_color: "#FFFFFF"
    priority: 1
    category: industry
```

## File layout

```
pantry-digest/
├── SKILL.md              ← skill entry, frontmatter, command router
├── AGENT.md              ← binding playbook (read first when invoked)
├── commands/
│   ├── generate.md       ← /pantry-generate
│   ├── add.md            ← /pantry-add
│   ├── remove.md         ← /pantry-remove
│   ├── list.md           ← /pantry-list
│   ├── sources.md        ← /pantry-sources
│   ├── issue.md          ← /pantry-issue
│   └── help.md           ← /pantry-help
├── default-sources.yaml  ← 16 default AI sources + your custom slot
├── template.html         ← HTML shell with __NEWS_JSON__ placeholder
├── LICENSE
└── README.md             ← this file
```

## What the agent guarantees

- **WebSearch first** — every run starts with parallel WebSearch queries
- **Real images** — preferred over generated SVG covers
- **Real engagement numbers** — HN points, comment counts, etc.; never invented
- **Real quotes** — every quote traces to a real person on the record
- **Today's news by default** — last 24h; widens to 48h or 7d only if needed
- **English by default** — toggle to Chinese with the top-right button
- **Same layout, any topic, any sources** — biotech digest looks just as
  good as an AI digest

## Refusal behavior

The skill will **refuse** to:

- Make up stories that didn't happen
- Inflate engagement numbers
- Quote a person who didn't actually say something
- Skip the language toggle (it's load-bearing)

If a source you requested is unreachable (login wall, geo-block), the
agent will tell you and ask whether to swap it for an alternative.

## Requirements

- [Claude Code](https://claude.com/claude-code) (any host: CLI, VSCode, JetBrains)
- That's it — no API keys, no Python packages, no MCP servers.

## License

MIT — see [LICENSE](LICENSE).
