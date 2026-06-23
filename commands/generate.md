# /pantry-generate

**Purpose:** Generate today's Pantry digest as `AI Redbook/index.html`,
optionally filtered by a user-provided scope.

## Argument form

```
/pantry-generate [scope]
```

`[scope]` is free-form natural language. Examples:

| User input | Interpretation |
|---|---|
| `/pantry-generate` | No scope → today's news, all sources, all categories |
| `/pantry-generate model releases only` | Filter to `category: model_release` and `opensource` |
| `/pantry-generate focus on robotics` | Add "robotics" to WebSearch queries, prioritize related sources |
| `/pantry-generate this week instead of today` | Widen scope from 24h → 7d |
| `/pantry-generate only my custom sources` | Skip `defaults:`, use only `custom:` block |
| `/pantry-generate to ~/Desktop/pantry.html` | Override output path |
| `/pantry-generate in Chinese only` | Set initial `LANG = 'zh'` (still keep toggle) |

The scope is a HINT, not a strict filter — combine with judgment. If the
filter empties the result set (e.g. "robotics only" but no robotics news
today), tell the user and widen automatically.

## Steps

1. Parse the scope phrase. Extract: time-window, category filter, output
   path, language preference, source filter.
2. Load `default-sources.yaml`. Apply source filter if the user said
   "only my custom sources" / "only newsletters" / "skip company blogs".
3. **Run AGENT.md Step 1 onward**, with the scope hints applied:
   - Time window → adjust `since_epoch` accordingly
   - Category filter → bias WebSearch queries toward those topics
   - Language preference → flip the `LANG = 'en'` initial value in the
     final HTML if user said "in Chinese only"
4. Render the digest. Default output: `AI Redbook/index.html` in CWD.
   Override only if user provided a path.
5. Report back: number of cards, sources cited, lead story, output path.

## Refusal

- "Make up stories" → refuse.
- "Use a paywalled site I don't have" → ask user for alternative or skip + note.

## Examples

**Example 1 — vanilla:**
```
/pantry-generate
```
→ Today's news, 10–12 cards, all categories, EN default, `AI Redbook/index.html`.

**Example 2 — focused topic:**
```
/pantry-generate AI coding agents this week
```
→ WebSearch queries weighted toward "AI coding agent" / "Claude Code" /
"Cursor" / "Codex" / "Devin"; time window 7d; output as usual.

**Example 3 — custom-only:**
```
/pantry-generate only my custom sources, this week
```
→ Skip `defaults:` block in YAML; pull only from `custom:`; 7d window.
