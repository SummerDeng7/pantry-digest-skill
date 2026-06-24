# Pantry — Agent Playbook

This is the binding behavior contract for the `pantry-digest` skill. Read it
end-to-end before doing any fetching.

## What this skill produces

A single self-contained `pantry-digest.html` file rendered in the Pantry style
(瀑布流卡片网格 · 奶油色调 · 中英双语切换 · Fraunces 衬线 + Inter 无衬线 ·
点击卡片打开模态层 · 模态层包含原文链接 + 网络评论). Default output path:
`pantry-digest.html` in the current working directory, unless the user
specifies otherwise.

The HTML inherits all styles, i18n and rendering logic from
[`template.html`](template.html). Your job is to **fetch real news**,
**select 10–12 stories**, **build a JSON array**, and **substitute it into
the template's placeholders**. No CSS or render-logic edits.

**Topic-agnostic.** The skill's defaults are tuned for AI news (16-source
roster in `default-sources.yaml`), but the same pipeline works for any
news topic — biotech, climate, finance, sports, fashion, gaming, etc. See
"Topic detection" below.

---

## Command routing — read FIRST when the user types a slash command

If the user's message begins with `/pantry-<verb>`, the command-specific
playbook in [`commands/<verb>.md`](commands/) is the source of truth for
**what to do** in that turn. This file (AGENT.md) describes the underlying
mechanics — how to fetch news, render the page, handle integration cards.

Mapping:

| Slash command | Playbook | Touches |
|---|---|---|
| `/pantry-generate [scope]` | `commands/generate.md` | Reads `default-sources.yaml`, runs Steps 1–9 below, writes `pantry-digest.html` |
| `/pantry-add <name/url>` | `commands/add.md` | Writes to `default-sources.yaml` under `custom:` only |
| `/pantry-remove <name>` | `commands/remove.md` | Edits `default-sources.yaml`, asks before touching `defaults:` |
| `/pantry-list` | `commands/list.md` | Read-only — renders the source table |
| `/pantry-sources [filter]` | `commands/sources.md` | Read-only — filtered table |
| `/pantry-issue [title]` | `commands/issue.md` | Files a GitHub issue against the upstream repo via `gh`; falls back to a pre-filled browser URL |
| `/pantry-help` | `commands/help.md` | Static help text |

**Natural-language triggers** (without a slash) — "refresh the pantry,"
"茶水间一下," "make me an AI digest," "update my pantry-digest page" — all route
to `/pantry-generate` with the user's words treated as the `[scope]` arg.

If a `/pantry-*` verb is unknown, list the valid ones (run `commands/help.md`).

---

## Execution flow — follow in order, do not skip

### Step 0 — Confirm scope

If the user has not specified otherwise, **default to "today's news"**:

- Compute `today_utc_midnight` and `since_epoch = today_utc_midnight - 86400`
  (last 24 h). If the day yields <8 stories, widen to last 48 h, then last 7 d.
- Tell the user one sentence: "I'll search the last 24h from <N> sources;
  expect 10–12 cards."

If the user offered custom sources or said "use my list," load
`default-sources.yaml` **and** the user's additions (merge `custom:` block).

### Step 0.5 — Topic detection (which sources apply?)

Decide the topic from the scope argument:

| Scope hint | Topic | Sources to use |
|---|---|---|
| No scope, or contains AI / LLM / model / agent / Anthropic / etc. | **AI (default)** | All `defaults:` + any `custom:` |
| Contains "biotech" / "pharma" / "FDA" / etc. | biotech | `custom:` matches only; if none, WebSearch for credible biotech sources and use those |
| Contains "climate" / "energy" / "carbon" / etc. | climate | same pattern |
| Contains "finance" / "markets" / "Fed" / etc. | finance | same pattern |
| Contains "sports" / "NBA" / "soccer" / etc. | sports | same pattern |
| Anything else off-AI | as named | same pattern |
| `"only my custom sources"` | any | `custom:` only, regardless of topic |

**When a non-AI topic is detected and no matching custom sources exist:**

1. Tell the user one sentence: "No biotech sources in your roster — I'll
   search for credible biotech outlets and use those for this run. Add them
   permanently with `/pantry-add` later if you want."
2. Run a WebSearch like:
   `WebSearch("best biotech news websites 2026 site:nature.com OR site:statnews.com OR site:endpts.com OR site:nejm.org")`
3. Use the returned outlets as ad-hoc sources for this run. Don't write
   them to `default-sources.yaml` (that's `/pantry-add`'s job).
4. Update the hero copy in the rendered HTML — replace "AI" wording with
   the actual topic. Specifically, override these i18n keys when emitting
   the final HTML:
   - `hero-title` EN → `Pull up a chair. <em>Here's what {Topic} is talking about.</em>`
   - `hero-title` ZH → `搬把椅子过来,<em>看看{Topic}圈在聊什么。</em>`
   - `hero-sub` → adjust the source list mentioned

If `default-sources.yaml` has user-added sources that match the topic
(e.g., user added STAT News and Endpoints), prefer those over WebSearch
discovery.

### Step 1 — **WebSearch first**, always

WebSearch is the primary discovery tool. **Run it before any WebFetch.**

Run **at least 4 parallel WebSearch queries** in a single message, tuned
to the topic detected in Step 0.5.

**For the default AI topic, queries MUST follow this priority order — the
order both decides search effort AND how many cards each tier deserves
(see Step 5):**

> **P1 (highest) — Products, tools, and model releases / updates.** The
> single most important tier. Major US and Chinese labs:
>
> ```
> WebSearch("Anthropic OR OpenAI OR Google DeepMind release update <today's date>")
> WebSearch("Microsoft Copilot OR Gemini OR ChatGPT OR Claude OR Mistral release <today's date>")
> WebSearch("ByteDance Doubao OR Alibaba Qwen OR Tencent Hunyuan OR DeepSeek release <today's date>")
> WebSearch("AI coding agent OR new AI product launch <today's date>")
> ```
>
> **P2 — Industry moves and opinion.** What people in the field think about
> recent releases, where the industry is heading.
>
> ```
> WebSearch("AI industry analysis <today's date> Stratechery OR The Information")
> WebSearch("AI takes OR commentary <today's date> Andrej Karpathy OR Yann LeCun OR Demis Hassabis")
> ```
>
> **P3 — Papers and research.** What academia / research labs are working
> on. Lower volume — usually 1–2 cards.
>
> ```
> WebSearch("arXiv AI paper <today's date> OR NeurIPS OR ICLR")
> ```
>
> **P4 (lowest) — Funding and open-source releases.** Cover only if there
> is a genuinely large or strategic story. Skip routine seed rounds and
> minor GitHub drops.

**For other topics**, generalize the same shape — strongest signal first,
academic/research mid, fundraising last:
```
WebSearch("<topic> news <today's date>")
WebSearch("<topic> <typical-subcategory-1> <today's date>")
WebSearch("<topic> <typical-subcategory-2> <today's date>")
```

For example, biotech → ("biotech news 2026", "FDA approval 2026", "pharma
M&A 2026"); finance → ("markets today", "Fed rate decision", "earnings
this week"); sports → ("NBA news today", "Premier League news", etc.).

Add user-specified topic queries if they asked for an angle ("focus on robotics,"
"only model releases," etc.).

**WebSearch output is the seed.** Use the URLs WebSearch returns to drive your
WebFetch calls — don't fetch source homepages blindly.

If WebSearch is unavailable in the current environment (returns "no
capability"), fall back to **WebFetch on the sources directly** AND surface
this limitation to the user in one sentence ("WebSearch is offline in this
session, so I'm fetching source homepages directly — coverage may be narrower").

### Step 2 — Pull from priority-1 sources (parallel)

Read `default-sources.yaml`. For every source with `priority: 1`, fire a
parallel WebFetch in a single message. Typical prompt:

> "List the most recent posts since {date}. Return up to 8 with title,
> date, author, canonical URL, and a one-sentence summary. Plain text."

For **Reddit and X**: do not WebFetch directly (blocked / requires auth).
Use **HN Algolia API** as a proxy:

```
https://hn.algolia.com/api/v1/search?tags=story&numericFilters=created_at_i>{since_epoch}&query=Anthropic&hitsPerPage=15
https://hn.algolia.com/api/v1/search?tags=story&numericFilters=created_at_i>{since_epoch}&query=twitter.com&hitsPerPage=15
https://hn.algolia.com/api/v1/search?tags=story&numericFilters=created_at_i>{since_epoch}&query=reddit.com&hitsPerPage=15
```

Each Algolia hit gives you a real X/Reddit/article URL **with real HN points
and comment counts** — that is the closest proxy to "X/Reddit hot posts"
available without auth.

### Step 3 — Fetch article bodies in parallel

For the ~15 most promising URLs from Step 1–2, WebFetch each in parallel
asking for "title, date, author, the first 2–3 paragraphs of body text
verbatim, and any quoted statement from a person."

If a URL 404s, retry with a search-derived URL. Don't guess slugs.

### Step 4 — Image extraction (REAL IMAGES FIRST)

For each story, attempt to extract the **real article hero / OG image**
in this order:

1. **Real article hero image** — While WebFetching the article body, also
   ask: "Return the og:image / twitter:image URL if present, or the URL of
   the first content image in the post."
2. **Source homepage card thumbnail** — If the article page hides the
   image, refetch the source listing page and ask for "the thumbnail image
   URL associated with the headline '<title>'."
3. **Logo / favicon** — As a last resort, fetch the source's apple-touch-icon
   or favicon at a usable size (e.g., `https://www.anthropic.com/apple-touch-icon.png`).
4. **`brandCover()` SVG fallback** — Only if 1–3 all fail. Use the source's
   `brand_color` / `text_color` / `short` fields from the YAML.

When using a real image URL: **verify it's a direct image link** (ends in
.png/.jpg/.webp or is from a known CDN). Don't use page URLs as image src.

If unsure whether an image will load, include the brandCover fallback in
the data and prefer the real URL — the renderer can be told to try the real
URL first and swap on `onerror`. To enable that, set the news item's
`image` to the real URL and `imageFallback` to the brandCover SVG data URI.

### Step 5 — Compose the digest

Aim for **10–12 cards** covering **at least 4 categories**. **Card count
per tier MUST follow the priority order from Step 1.** This is the single
most important shape decision — it's how a reader feels the digest.

**For the default AI topic, target distribution:**

| Priority | Card count | What goes here | Canonical `category` values |
|---|---|---|---|
| **P1 — Products / Tools / Models** | **5–6 cards** | Major lab releases, model updates, new products and dev tools. Heavy emphasis. | `product`, `model_release`, `tool` |
| **P2 — Industry & Opinion** | **3–4 cards** | Industry shifts, partnerships, commentary, hot takes, where things are heading | `industry`, `opinion`, `policy`, `community` (1 integrated X/HN card lives here), `digest` (cross-newsletter scan if ≥3 newsletters converge) |
| **P3 — Research** | **1–2 cards** | Papers worth flagging, lab benchmarks, methodology contributions | `paper`, `safety`, `benchmark` |
| **P4 — Funding / Open Source** | **0–1 cards each** | Only include if genuinely big (≥$500M round, or a real frontier-tier open-source drop). Skip noise. | `funding`, `opensource` |

**If you can't fill P1 with 5–6 cards from real news**, lower the total
card count to 9–10 rather than padding with P3/P4 — readers come for
products and models.

**Don't double-count.** Each card has exactly one `category`. A "Google
releases DiffusionGemma" story is `model_release`, not both
`model_release` AND `paper`, even though there's an arXiv link.

**For other (non-AI) topics**, generalize the same shape: dominant tier
gets ~half the cards, secondary tier gets ~third, the rest are taper.

### Step 6 — Build the NEWS array

Each card is one object in the array. Schema:

```js
{
  id: 1,                                    // unique integer 1..N
  tag: { en: 'Model Release', zh: '模型发布' },  // display label on the card
  category: 'model_release',                // REQUIRED. Canonical category for filter chips.
                                            // One of: product, model_release, tool, industry,
                                            // opinion, policy, community, digest, paper, safety,
                                            // benchmark, funding, opensource, workflow.
                                            // The renderer writes this onto <article data-category="...">
                                            // and the filter chips key off it.
  aspectRatio: '4/5',                       // vary: '4/5' '3/4' '1/1' '5/4' '16/11'
  image: 'https://.../hero.jpg',            // REAL image URL preferred
  imageFallback: 'data:image/svg+xml,...',  // brandCover() SVG — optional
  headline: { en: '...', zh: '...' },       // short tagline, on the card
  title:    { en: '...', zh: '...' },       // long form, in the modal
  source: 'Anthropic',                      // exact display name
  sourceInitial: 'A',                       // 1-char badge
  date: '2026.06.23',                       // YYYY.MM.DD
  body: {
    en: [ '<p>...</p>', '<div class="pull">...</div>', '<p>...</p>' ],
    zh: [ '<p>...</p>', '<div class="pull">...</div>', '<p>...</p>' ]
  },
  sourceUrl: 'https://...',                 // canonical article URL
  comments_data: [
    { author: 'name', platform: 'X|HN|Reddit|wsj.com|...',
      time: '2h', av: 'av-1',
      text: { en: '...', zh: '...' } }
  ]
}
```

**Aspect-ratio mix rule:** never give two adjacent cards the same ratio.
Vary so the waterfall stays uneven.

### Step 7 — Integration-card rules (X · HN · newsletter cross-source)

If you make a community card or newsletter digest card, **each item inside
the body MUST be its own `<a class="src">` block**, not buried in prose.
Format per item:

```html
<a class="src" href="<canonical-or-search-url>" target="_blank" rel="noopener">
  <div class="src-head">
    <span class="src-badge">PLATFORM · @handle</span>
    <span class="src-meta">DATE · context</span>
  </div>
  <div class="src-title">"verbatim quote or exact headline"</div>
  <div class="src-body">one-sentence why-it-matters.</div>
  <span class="src-stat">real engagement numbers, with <strong>bold</strong> on the digits</span>
</a>
```

The whole `<a>` is clickable. Style is already in `template.html`
(`.modal-body .src`). Don't invent fake numbers; if you don't have real
engagement, omit `src-stat` and keep the title + body.

### Step 8 — Fact-checking & honesty bar

Before finalizing:

- Every `sourceUrl` must be a real URL you successfully WebFetched.
- Every quote in `<div class="pull">` and in `comments_data[].text` must
  trace to a real person who actually said it on the record (article body,
  press release, public tweet). If you only have a paraphrase, mark it
  clearly as paraphrase, not in quotes.
- Real engagement numbers (HN points, comment counts) only — never invent.
- Dates use the real article publication date in `YYYY.MM.DD`.
- If a story is from a non-English source, keep the source name in its
  original script (e.g., 量子位, 36Kr).

### Step 9 — Render the page

1. Read `template.html`.
2. Substitute:
   - `__NEWS_JSON__` → `JSON.stringify(NEWS_array, null, 2)`
   - `__HERO_EYEBROW_EN__` → e.g., `2026 / 06 / 23 · Tuesday`
   - `__HERO_EYEBROW_ZH__` → e.g., `2026 / 06 / 23 · 周二`
   - `__HERO_COUNT__` → number of cards (e.g., `11`)
   - `__HERO_SOURCES__` → number of unique sources cited
3. Write the result to the user's target path
   (default `pantry-digest.html`, override via user instruction).
4. Report back to the user with: card count, source count, what the lead
   card is, and the absolute path.

### Step 10 — Default language

Page loads in **English first** (set by the template's `let LANG = 'en'`).
Language toggle in the top-right swaps to Chinese live. Do not change this.

---

## Image-rendering hint to bake into the data

The template's `<img>` tag uses simple `src=`. For real-image + fallback,
emit the image src like this in `image`:

```
image: 'https://real-article-hero.jpg'
```

And separately set `imageFallback` to a brandCover SVG data URI. The skill
should also add a tiny `onerror` handler to the rendered `<img>` so that
broken real images degrade gracefully to brandCover. Patch `renderCard()`
and the modal `<img>` to include:

```html
<img src="${n.image}"
     ${n.imageFallback ? `onerror="this.onerror=null;this.src='${n.imageFallback}'"` : ''}
     alt="${pick(n.headline)}" loading="lazy">
```

If you didn't get a real image at all, set `image` directly to a
`brandCover()` data URI and omit `imageFallback`.

---

## What to refuse / clarify

- **"Make stories up"** → refuse. Skill premise is real news only.
- **"Inflate engagement numbers"** → refuse.
- **"Use a source that requires login I don't have"** → ask user for either
  an alternative source or permission to skip and note the gap.
- **"Skip the language toggle"** → ask before removing; it's a load-bearing
  feature.

## Lookup quick reference

| Need | Best route |
|------|-----------|
| Find today's stories | WebSearch (parallel queries) |
| Read a specific article | WebFetch the URL |
| What's hot on X this week | HN Algolia `?query=twitter.com&numericFilters=created_at_i>{epoch}` |
| What's hot on Reddit this week | HN Algolia `?query=reddit.com&numericFilters=created_at_i>{epoch}` |
| What's hot on HN | WebFetch `https://news.ycombinator.com/best` or `/` |
| Real article hero image | Ask WebFetch for `og:image` while fetching body |
| User wants their own sources | Read user's path, merge with default-sources.yaml |
