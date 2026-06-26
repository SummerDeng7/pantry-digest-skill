# commands/remove.md — natural-language "remove a source"

**Purpose:** Remove a source from `default-sources.yaml`.

## How users invoke this

Natural language only. Any of these phrasings should route here:

- "remove Stratechery"
- "delete the source Mistral"
- "把豆包从信源里删掉"
- "kill source stratechery.com"

## Argument extraction

From the user's message, extract the **identifier**. Matching is fuzzy but
must be unambiguous. Accepts:

- Display name (e.g., `Stratechery`)
- Short label (e.g., `Strat`)
- Hostname (e.g., `stratechery.com`)
- Full URL

## Steps

1. Read `default-sources.yaml`.
2. Search both `defaults:` and `custom:` for entries matching the identifier.
   Match priority: exact `name` → exact `short` → URL host substring → URL exact.
3. **If 0 matches** → tell the user, list 3 closest names by Levenshtein
   distance, suggest they try one of those.
4. **If >1 match** → tell the user the matches and ask which to remove.
   Don't auto-pick.
5. **If exactly 1 match**:
   - If it's in `defaults:`, ask for explicit confirmation:
     "This is a default source. Remove it permanently? You can re-add later
     by saying 'add <name> as a source'."
   - If it's in `custom:`, remove immediately and confirm.
6. Rewrite the YAML preserving order and formatting of other entries.

## Refusal

- Argument is empty or one character → ask for a more specific name.
- The match is the last remaining source in the file → refuse, tell the
  user to add a replacement first.

## Examples

**Example 1 — unambiguous custom source:**
```
User: remove Stratechery
```
Agent finds one match in `custom:`, removes it, replies "Removed Stratechery."

**Example 2 — ambiguous:**
```
User: remove Google
```
Agent finds `Google Blog` and `Google DeepMind`, replies:
"Two matches: (1) Google Blog, (2) Google DeepMind. Which one?"

**Example 3 — default source confirmation:**
```
User: delete Anthropic from sources
```
Agent replies: "Anthropic is in `defaults:`. Confirm removal? It's the
primary signal source — most generates lean on it heavily." Waits for
"yes" before writing.
