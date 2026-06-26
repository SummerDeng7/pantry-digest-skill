# commands/add.md — natural-language "add a source"

**Purpose:** Add a new source to `default-sources.yaml` under the `custom:`
block so it appears in all future `/pantry-digest` runs.

## How users invoke this

Natural language only. Any of these phrasings should route here:

- "add Stratechery as a source"
- "add https://stratechery.com/ to my sources"
- "新加一个信源 importai.net"
- "把 Ben's Bites 加进来"
- "添加 Import AI 这个 newsletter"

## Argument extraction

From the user's message, extract whatever is present:

| User said | Extracted |
|---|---|
| "add https://stratechery.com/" | URL only; infer `name=Stratechery` from page title |
| "add Ben's Bites https://bensbites.com newsletter" | name + URL + kind |
| "add Import AI" | name only; ask user for the URL in one follow-up |
| "add Stratechery https://stratechery.com opinion #2D3142" | name + URL + category + brand color |

## YAML entry the agent will write

```yaml
custom:
  - name: Stratechery               # required, display name on the card
    short: Strat                     # 2-12 chars, shown on the SVG cover
    url: https://stratechery.com/
    kind: newsletter                 # one of: lab_blog, news_aggregator, newsletter, social, forum, paper, mirror_api
    brand_color: "#2D3142"
    text_color: "#F4B860"
    priority: 1                      # 1 = must check on every generate run
    category: opinion                # see schema in default-sources.yaml
    note: "Added by user on YYYY-MM-DD."
```

## Steps

1. Parse the arguments. Identify which of `name`, `url`, `category`, `brand_color`,
   `kind` were given.
2. **Fetch the URL once** to confirm it's reachable and to grab the site's
   favicon / theme color for a sensible default brand color (if missing).
3. If any of `name` / `url` / `category` is still missing, ask the user in ONE
   follow-up message — list each missing field as a question.
4. Default values when the user gave a URL but no other hints:
   - `name` ← page `<title>` or domain
   - `short` ← first 8 chars of name, no spaces
   - `kind` ← infer from URL pattern (substack.com → newsletter, arxiv.org → paper,
     `/blog` in path → lab_blog, otherwise news_aggregator)
   - `brand_color` ← favicon's dominant color, or `#3D4A5C` (slate fallback)
   - `text_color` ← `#FFFFFF` for dark brand colors, `#1A1A1A` for light ones
   - `priority` ← 1
   - `category` ← infer from `kind` (newsletter → digest, lab_blog → product, paper → paper)
5. Append the YAML entry under `custom:` in `default-sources.yaml`.
   - Use proper indentation matching the existing entries.
   - Do NOT touch the `defaults:` block.
   - If `custom:` was `[]`, replace it with a real list.
6. Confirm to the user: "Added <name> as a <category> source. Next
   `/pantry-digest` will include it."

## Refusal

- URL doesn't resolve / 403s on first fetch → tell the user, don't add.
- Source already exists (same URL or same name in either `defaults:` or
  `custom:`) → tell the user, suggest they ask to remove the old one first or
  rephrase to overwrite.
- Source clearly outside any news / digest use case → confirm with the user
  before adding.

## Examples

**Example 1 — minimum input:**
```
User: add https://www.ben-evans.com/newsletter as a source
```
Agent fetches the URL, infers `name=Benedict Evans`, `kind=newsletter`,
`category=opinion`, derives a brand color from the favicon, asks the user
"Confirm: name 'Benedict Evans', newsletter, opinion category, brand color
#1A2B3C?" Once confirmed, writes the entry.

**Example 2 — full spec:**
```
User: add Import AI from https://jack-clark.net/ — it's a paper / research newsletter
```
Agent writes immediately — all fields present. Replies: "Added Import AI as
a paper source. Next `/pantry-digest` will include it."
