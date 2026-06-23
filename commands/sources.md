# /pantry-sources

**Purpose:** Filtered view of `default-sources.yaml`. Same data as
`/pantry-list`, but takes a filter argument.

## Argument form

```
/pantry-sources [filter]
```

Recognized filters:

| Filter | Behavior |
|---|---|
| `/pantry-sources newsletters` | Show only `kind: newsletter` |
| `/pantry-sources blogs` / `/pantry-sources labs` | Show only `kind: lab_blog` |
| `/pantry-sources papers` | Show only `kind: paper` |
| `/pantry-sources priority 1` / `/pantry-sources p1` | Show only `priority: 1` |
| `/pantry-sources category model_release` | Filter by category |
| `/pantry-sources custom` | Show only entries from `custom:` block |
| `/pantry-sources defaults` | Show only entries from `defaults:` block |
| `/pantry-sources search anthropic` | Substring match on name or URL |

Combine with comma if needed: `/pantry-sources newsletters, priority 1`.

## Output format

Same as `/pantry-list`, but with the filter shown at the top:

```
FILTER: newsletters · priority 1
────────────────────────────────────────────
Every          newsletter   workflow    P1
The Rundown    newsletter   digest      P1
Superhuman AI  newsletter   product     P1

3 of 16 sources shown.
```

If 0 matches → tell the user, suggest broader filter.

## Steps

1. Parse the filter phrase. Multiple filters AND together.
2. Read `default-sources.yaml`, apply filters across both `defaults:` and
   `custom:` (unless the filter explicitly restricts to one).
3. Render the table.
4. End with a count and an action hint
   (`/pantry-generate only newsletters` if the filter is itself a useful generate scope).
