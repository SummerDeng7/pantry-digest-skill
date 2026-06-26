# commands/sources.md — natural-language "filter sources"

**Purpose:** Filtered view of `default-sources.yaml`. Same data as
"list my sources", but with a filter applied.

## How users invoke this

Natural language asking for a *subset* of sources. Route here when the user
asks for a filter:

- "show only newsletters"
- "list my priority 1 sources"
- "show only Chinese sources"
- "filter by category opinion"
- "search for anthropic"
- "show my custom sources only"

## Recognized filters

| Filter phrase | Behavior |
|---|---|
| "newsletters" | Show only `kind: newsletter` |
| "blogs" / "labs" / "company blogs" | Show only `kind: lab_blog` |
| "papers" | Show only `kind: paper` |
| "priority 1" / "p1" / "must-check sources" | Show only `priority: 1` |
| "model release sources" / "category model_release" | Filter by category |
| "my custom sources" / "what I added" | Show only entries from `custom:` block |
| "defaults" / "built-in" | Show only entries from `defaults:` block |
| "search <term>" / "find <term>" | Substring match on name or URL |

Multiple filters combine with AND. e.g. "newsletters that are priority 1".

## Output format

Same as `commands/list.md`, but with the filter shown at the top:

```
FILTER: newsletters · priority 1
────────────────────────────────────────────
Every          newsletter   workflow    P1
The Rundown    newsletter   digest      P1
Superhuman AI  newsletter   product     P1

3 of 56 sources shown.
```

If 0 matches → tell the user, suggest broader filter.

## Steps

1. Parse the filter phrase. Multiple filters AND together.
2. Read `default-sources.yaml`, apply filters across both `defaults:` and
   `custom:` (unless the filter explicitly restricts to one).
3. Render the table.
4. End with a count and an action hint (e.g. "Try `/pantry-digest only
   newsletters` to make a newsletter-only digest").
