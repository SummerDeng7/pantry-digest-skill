# Pantry · 茶水间

A [Claude Code](https://claude.com/claude-code) skill that generates a
beautiful, real-news AI digest webpage in the **Pantry** style — masonry
cards, cream palette, bilingual EN/ZH, click-to-expand modals with
original source links and community reactions.

> All stories are **real**, fetched live from public sources. No invented
> news, no fake engagement numbers, no fabricated quotes.

<!-- Add a screenshot once you have one:
![Pantry screenshot](docs/screenshot.png)
-->

## Install

```bash
git clone https://github.com/<your-handle>/pantry-digest ~/.claude/skills/pantry-digest
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
| `/pantry-generate [scope]` | Generate today's digest. Scope is free-form: `"this week"`, `"focus on robotics"`, `"only my custom sources"`, `"in Chinese only"`, `"to ~/Desktop/pantry.html"`. |
| `/pantry-add <name-or-url>` | Add a source. Examples: `/pantry-add https://stratechery.com/` or `/pantry-add Import AI https://jack-clark.net/ paper`. |
| `/pantry-remove <name>` | Remove a source. Confirmation required for default sources. |
| `/pantry-list` | List all configured sources, grouped by defaults vs custom. |
| `/pantry-sources [filter]` | Filtered list: `newsletters`, `priority 1`, `custom`, `search anthropic`, etc. |
| `/pantry-help` | Show the command list. |

Natural-language triggers also work: *"refresh the pantry"*, *"茶水间一下"*,
*"make me a daily AI digest"* — all route to `/pantry-generate`.

## What it does on each run

1. Runs **parallel WebSearch** queries to discover today's biggest AI stories
2. **WebFetches** the chosen articles, newsletters, and HN / X / Reddit threads
3. Extracts **real cover images** (og:image / hero image) when available;
   falls back to brand-colored SVG covers when not
4. Composes **10–12 cards** spanning model releases, products, papers,
   industry moves, funding, policy, opinion, and community signals
5. Renders a self-contained `index.html` at `AI Redbook/index.html` (or
   wherever you ask)

## Default source roster

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

## Adding your own sources

Easiest:

```
/pantry-add https://stratechery.com/
```

The agent will fetch the URL, infer a sensible name / category / brand
color, and ask you to confirm anything it can't infer.

Manual: edit `default-sources.yaml` and append under `custom:`:

```yaml
custom:
  - name: Stratechery
    short: Strat
    url: https://stratechery.com/
    kind: newsletter
    brand_color: "#2D3142"
    text_color: "#F4B860"
    priority: 1
    category: opinion
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
│   └── help.md           ← /pantry-help
├── default-sources.yaml  ← 16 default sources + your custom slot
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
- **Same layout, any sources** — your custom sources slot into the same
  masonry grid with brand-color covers

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
