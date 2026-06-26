# commands/list.md — natural-language "list my sources"

**Purpose:** Show all sources currently configured in `default-sources.yaml`.

## How users invoke this

Natural language only. Any of these phrasings should route here:

- "list sources"
- "list my sources"
- "show me the sources"
- "看看信源"
- "现在有哪些信源"
- "我加了哪些信源"

For filtered views (only newsletters / priority 1 / etc.) → route to
`commands/sources.md` instead.

## Output format

Render a compact grouped list, EN/ZH-neutral:

```
DEFAULT SOURCES (55)
────────────────────────────────────────────
Anthropic Newsroom    lab_blog      model_release  P1
Claude Blog           lab_blog      product        P1
…
HN Algolia            mirror_api    community      P1

CUSTOM SOURCES (0)
────────────────────────────────────────────
(none — say "add <name> as a source" to add one)
```

If `custom:` has entries, list them in the same format.

Closing line: "Run `/pantry-digest` to use these sources." (or, if
`custom:` is empty: "Say 'add <name> <url>' to add your own.")

## Steps

1. Read `default-sources.yaml`.
2. Render the grouped list above. Truncate `kind` to fit (e.g.
   `news_aggregator` → `news_aggreg`).
3. Show counts in each group header.
4. End with a one-line action hint.
