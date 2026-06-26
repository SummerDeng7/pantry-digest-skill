# Pantry · 茶水间

A [Claude Code](https://claude.com/claude-code) skill that generates a
beautiful, real-news **daily digest webpage** in the **Pantry** style —
masonry cards, cream palette, bilingual EN/ZH, click-to-expand modals
with original source links and community reactions.

**Default topic is AI** (with a curated 56-source roster spanning model
releases, opinion, media heat, investment, and academic frontier). But
the skill is topic-agnostic — point it at biotech, climate, finance,
sports, fashion, your favorite niche, and it figures out the sources.

> All stories are **real**, fetched live from public sources. No invented
> news, no fake engagement numbers, no fabricated quotes.

<!-- Add a screenshot once you have one:
![Pantry screenshot](docs/screenshot.png)
-->

## Install

```bash
git clone https://github.com/SummerDeng7/pantry-digest-skill ~/.claude/skills/pantry-digest
```

That's it. Restart Claude Code; the skill auto-loads. On first activation
the skill greets you with what it can do.

To update later:

```bash
cd ~/.claude/skills/pantry-digest && git pull
```

## How to use it

There is one slash command:

```
/pantry-digest [scope]
```

`[scope]` is free-form natural language. Examples:

| You type | What happens |
|---|---|
| `/pantry-digest` | Today's AI news, all 56 default sources, English UI, `pantry-digest.html` in CWD |
| `/pantry-digest biotech this week` | Switch topic to biotech, widen window to 7 days |
| `/pantry-digest focus on AI coding agents` | Stay AI but bias toward coding-agent subtopic |
| `/pantry-digest only my custom sources` | Skip defaults, use just what you've added |
| `/pantry-digest in Chinese only` | Render with Chinese UI as initial language |
| `/pantry-digest to ~/Desktop/today.html` | Override output path |

**Everything else is natural language.** No more slash commands to memorize.
Just say what you want:

| Say something like… | And the skill will… |
|---|---|
| "add Stratechery as a source" | Add it to your custom sources |
| "list my sources" | Show all current sources, grouped by default / custom |
| "show only newsletters" | Filtered view |
| "remove ByteDance" | Remove the source (with confirmation for defaults) |
| "file a bug" | Open an interactive issue-filing flow via `gh` |
| "help" / "what can you do" | Re-display the onboarding greeting |

Natural-language triggers for the digest itself also work: *"refresh the
pantry"*, *"茶水间一下"*, *"make me a daily AI digest"*, *"daily climate
digest please"* — all route to `/pantry-digest`.

## What it does on each run

1. Runs **parallel WebSearch** queries to discover today's biggest stories
   (in your topic, AI by default).
2. **WebFetches** the chosen articles, newsletters, and HN / X / Reddit threads.
3. Extracts **real cover images** (og:image / hero image) when available;
   falls back to brand-colored SVG covers when not.
4. Composes **10–15 cards** spanning the 5 priority tiers (Model/Product
   Release, Opinion/Experience, Media Heat, Investment/Insight, Academic
   Frontier).
5. Renders a self-contained `pantry-digest.html` in the current directory
   (or wherever you ask).

## Default source roster (AI topic, 56 sources, 5 priority tiers)

| Tier | Sources |
|---|---|
| **P1 — Model / Product Release** (19 sources) | Anthropic · Claude Blog · Anthropic Engineering · OpenAI · Google DeepMind · Google AI · Microsoft AI · Meta AI · xAI · Mistral · DeepSeek · Qwen · Doubao · Hunyuan · Kimi · GLM · ERNIE · MiniMax · HN Algolia |
| **P2 — Opinion / Experience** (12 sources) | Every · Lenny's · Latent Space · Interconnects · Import AI · The Batch · Ahead of AI · Ben's Bites · Exponential View · One Useful Thing · Anthropic Institute · Claude's Corner |
| **P3 — Media Heat** (12 sources) | Superhuman · TLDR AI · The Rundown · The Neuron · AI Weekly · Hacker News · r/ML · r/LocalLLaMA · r/artificial · X #AITwitter · 机器之心 · 量子位 |
| **P4 — Investment / Insight** (6 sources) | a16z · a16z Newsletter · Sequoia · YC · Stanford HAI Index · CB Insights |
| **P5 — Academic Frontier** (7 sources) | arXiv cs.AI · HF Daily Papers · Semantic Scholar · OpenReview · Anthropic Research · Alignment Science Blog · Anthropic Red Team |

(Reddit and X block direct fetching without auth. The HN Algolia source
indexes every X / Reddit / news URL that hits Hacker News, with real points
and comment counts — the only auth-free way to get real X / Reddit
engagement data from inside Claude Code.)

## Using a non-AI topic

Three ways, pick whichever fits:

**1. One-shot — let the agent search for sources.**
```
/pantry-digest today's biotech news
```
The agent will WebSearch for biotech sources, pick credible ones (STAT,
Endpoints, NEJM, etc.), and build the digest. No setup required.

**2. Build a custom roster over time.**
```
User: add https://www.statnews.com/ as a source
User: add https://endpts.com/
User: add https://www.nejm.org/
User: /pantry-digest biotech, only my custom sources
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
├── SKILL.md              ← skill entry, onboarding greeting, command router
├── AGENT.md              ← binding playbook (read first when invoked)
├── commands/
│   ├── digest.md         ← /pantry-digest playbook
│   ├── add.md            ← natural-language "add a source"
│   ├── remove.md         ← natural-language "remove a source"
│   ├── list.md           ← natural-language "list my sources"
│   ├── sources.md        ← natural-language "filter sources"
│   ├── issue.md          ← natural-language "file an issue"
│   └── help.md           ← natural-language "help"
├── default-sources.yaml  ← 56 default AI sources + your custom slot
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

- [Claude Code](https://claude.com/claude-code) v2.1.3+ (any host: CLI, VSCode, JetBrains)
- That's it — no API keys, no Python packages, no MCP servers.

## License

MIT — see [LICENSE](LICENSE).
