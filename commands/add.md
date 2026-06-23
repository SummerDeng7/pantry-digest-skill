# /pantry-add

**Purpose:** Add a new source to `default-sources.yaml` under the `custom:`
block so it appears in all future `/pantry-generate` runs.

## Argument form

```
/pantry-add <name-or-url> [extra hints]
```

The agent must be tolerant of how the user gives input. All of these
should work:

| User input | Behavior |
|---|---|
| `/pantry-add https://stratechery.com/` | Infer `name=Stratechery` from the URL; ask for category and brand color in one follow-up if not derivable. |
| `/pantry-add Ben's Bites https://bensbites.com newsletter` | Use exactly what was provided. |
| `/pantry-add Import AI` | URL missing — ask the user for the URL in a single follow-up. |
| `/pantry-add Stratechery https://stratechery.com opinion #2D3142` | Full spec — name, URL, category, brand color. Write immediately. |

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
   `/pantry-generate` will include it."

## Refusal

- URL doesn't resolve / 403s on first fetch → tell the user, don't add.
- Source already exists (same URL or same name in either `defaults:` or
  `custom:`) → tell the user, offer `/pantry-remove` to swap or
  `/pantry-add <name> as a replacement` to overwrite.
- Source clearly off-topic for an AI digest → confirm with the user before
  adding.

## Examples

**Example 1 — minimum input:**
```
/pantry-add https://www.ben-evans.com/newsletter
```
Agent fetches the URL, infers `name=Benedict Evans`, `kind=newsletter`,
`category=opinion`, derives a brand color from the favicon, asks the user
"Confirm: name 'Benedict Evans', newsletter, opinion category, brand color
#1A2B3C?" Once confirmed, writes the entry.

**Example 2 — full spec:**
```
/pantry-add Import AI https://jack-clark.net/ paper #5B6B7A
```
Agent writes immediately — all fields present. Replies: "Added Import AI as
a paper source. Next generate will include it."
