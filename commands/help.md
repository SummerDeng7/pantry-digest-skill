# commands/help.md

**Purpose:** When the user asks for help / says "what can you do" / says
"how do I use this" — re-emit the ON-LOAD greeting from `SKILL.md`.

## When to invoke

Any of these natural-language asks (in any language):
- "help"
- "what can you do"
- "how do I use this"
- "怎么用"
- "帮助"
- The user seems lost and is asking about capabilities

## What to do

1. Re-emit the verbatim greeting block from `SKILL.md` "⚠️ FIRST CONTACT"
   section. Same content, same formatting, no preamble.
2. If the user's previous turn shows they tried something that didn't
   work, prepend one short line acknowledging what didn't work, then the
   greeting.

## What NOT to do

- Do NOT list slash commands separately — `/pantry-digest` is the only
  one, and it's in the greeting.
- Do NOT improvise a different help format — the greeting is the canonical
  introduction. Other shapes will drift over time and confuse users.
- Do NOT add a "tips" section or "advanced usage" section. The greeting
  already names natural-language verbs (add / list / remove / etc.).
