# /pantry-generate

**Purpose:** Generate today's Pantry digest as `pantry-digest.html`,
optionally filtered by a user-provided scope.

## Argument form

```
/pantry-generate [scope]
```

`[scope]` is free-form natural language. Examples:

| User input | Interpretation |
|---|---|
| `/pantry-generate` | No scope → today's AI news from default sources |
| `/pantry-generate biotech news` | Switch topic to biotech — WebSearch for sources if user has none in custom |
| `/pantry-generate climate this week` | Topic: climate; widen window to 7d |
| `/pantry-generate finance digest` | Topic: finance |
| `/pantry-generate sports today` | Topic: sports |
| `/pantry-generate model releases only` | Stay AI; filter to `category: model_release` / `opensource` |
| `/pantry-generate focus on robotics` | Stay AI; bias WebSearch toward robotics |
| `/pantry-generate this week instead of today` | Stay AI; widen window 24h → 7d |
| `/pantry-generate only my custom sources` | Skip `defaults:`, use only `custom:` block |
| `/pantry-generate to ~/Desktop/today.html` | Override output path |
| `/pantry-generate in Chinese only` | Set initial `LANG = 'zh'` (toggle still present) |

The scope is a HINT, not a strict filter — combine with judgment. If the
filter empties the result set (e.g. "robotics only" but no robotics news
today), tell the user and widen automatically.

## Steps

1. Parse the scope phrase. Extract: **topic**, time-window, category filter,
   output path, language preference, source filter.
2. **Run AGENT.md Step 0.5 (topic detection)** to decide which sources apply.
   For non-AI topics, either use matching `custom:` entries OR WebSearch
   for credible outlets in that topic AND override the hero copy in the
   final HTML to mention the actual topic instead of "AI".
3. Load the resolved source list. Apply source filter if the user said
   "only my custom sources" / "only newsletters" / "skip company blogs".
4. **Run AGENT.md Step 1 onward**, with the scope hints applied:
   - Time window → adjust `since_epoch` accordingly
   - Topic → tune WebSearch queries (see AGENT.md Step 1)
   - Category filter → bias WebSearch queries toward those subtopics
   - Language preference → flip `LANG = 'en'` to `LANG = 'zh'` if requested
5. Render the digest. Default output: `pantry-digest.html` in CWD.
   Override only if user provided a path.
6. Report back: topic, number of cards, sources cited, lead story, output path.

## Refusal

- "Make up stories" → refuse.
- "Use a paywalled site I don't have" → ask user for alternative or skip + note.

## Examples

**Example 1 — vanilla (AI default):**
```
/pantry-generate
```
→ Today's AI news, 10–12 cards, EN default, `pantry-digest.html`.

**Example 2 — different topic, ad-hoc sources:**
```
/pantry-generate today's biotech news
```
→ Topic = biotech. No biotech sources in roster, so WebSearch finds
credible outlets (STAT, Endpoints, NEJM, FierceBiotech) and uses them
for this run. Hero copy switches to "Here's what biotech is talking about."

**Example 3 — different topic, custom roster:**
User has previously added STAT, Endpoints, FierceBiotech via `/pantry-add`.
```
/pantry-generate biotech this week
```
→ Uses the user's biotech custom sources directly; widens window to 7d.

**Example 4 — AI subfocus:**
```
/pantry-generate AI coding agents this week
```
→ Topic stays AI; WebSearch queries weighted toward "AI coding agent" /
"Claude Code" / "Cursor" / "Codex" / "Devin"; window 7d.

**Example 5 — custom-only:**
```
/pantry-generate only my custom sources, this week
```
→ Skip `defaults:` block in YAML; pull only from `custom:`; 7d window.
