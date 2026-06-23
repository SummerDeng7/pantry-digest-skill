# /pantry-list

**Purpose:** Show all sources currently configured in `default-sources.yaml`.

## Argument form

```
/pantry-list
```

No arguments. (Use `/pantry-sources <scope>` for filtered views.)

## Output format

Render a compact two-column grouped list, EN/ZH-neutral:

```
DEFAULT SOURCES (16)
────────────────────────────────────────────
Anthropic           lab_blog     model_release  P1
OpenAI              lab_blog     model_release  P1
Google Blog         lab_blog     model_release  P1
Google DeepMind     lab_blog     paper          P1
Microsoft AI        lab_blog     product        P2
Mistral AI          lab_blog     opensource     P2
TechCrunch AI       news_aggreg  industry       P1
Hugging Face Blog   news_aggreg  benchmark      P2
arXiv cs.AI         paper        paper          P3
Every               newsletter   workflow       P1
The Rundown         newsletter   digest         P1
Superhuman AI       newsletter   product        P1
Lenny's Newsletter  newsletter   opinion        P2
Hacker News         forum        community      P1
HN Algolia          mirror_api   community      P1

CUSTOM SOURCES (0)
────────────────────────────────────────────
(none — add with /pantry-add)
```

If `custom:` has entries, list them in the same format.

Closing line: "Run `/pantry-generate` to use these sources." or
"Add more with `/pantry-add <name> <url>`."

## Steps

1. Read `default-sources.yaml`.
2. Render the grouped list above. Truncate `kind` to fit (e.g.
   `news_aggregator` → `news_aggreg`).
3. Show counts in each group header.
4. End with a one-line action hint.
