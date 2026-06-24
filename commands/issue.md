# /pantry-issue

**Purpose:** Help the user file a GitHub issue against the pantry-digest
skill's upstream repo (`SummerDeng7/pantry-digest-skill`) — for bug
reports, feature requests, or questions about how the skill behaves.

## Argument form

```
/pantry-issue [one-line title]
```

- No args → fully interactive: ask the user for title, type, and body.
- Title given → use it as the issue title, ask for type and body.

## Upstream repo

The issues go to: `SummerDeng7/pantry-digest-skill`
Issues page: <https://github.com/SummerDeng7/pantry-digest-skill/issues>

## Steps

### 1. Pre-flight checks

Run these in parallel, capture results, decide a path before asking the user anything:

- `gh --version` → confirms `gh` CLI is installed
- `gh auth status` → confirms a GitHub account is logged in, and which one is active

**Branch by result:**

| Pre-flight result | Action |
|---|---|
| `gh` not installed | Skip to **Fallback A — browser URL** |
| `gh` installed, no auth | Skip to **Fallback B — ask to auth or use browser** |
| `gh` installed, auth OK | Proceed to step 2 |

### 2. Active-account heads-up

If `gh auth status` shows the active account is an Enterprise Managed
User (typical sign: account name like `<name>_microsoft`, `<name>_google`,
or any account where the description says "Enterprise" / "EMU"):

> Tell the user one sentence: "Your active gh account is
> `<name>_microsoft` — the issue will be filed under that handle. If
> you'd rather use your personal account, switch with
> `gh auth switch -u <other-account>` first, then re-run me."

Do **not** auto-switch their account. Wait for them to either confirm
"go ahead" or switch and re-invoke.

### 3. Gather issue content (interactive)

Ask the user in ONE message:

> Three quick things and I'll file it:
>
> 1. **Type** — bug / feature request / question / other?
> 2. **Title** — one line (if not already given on the command)
> 3. **Description** — what happened, what you expected, anything
>    relevant. Or just paste the error / behavior you saw.
>
> I'll add a small environment footer for you (OS, Claude Code host).

If the user only answers a subset, fill the rest with safe defaults
(type → "question", title → first 60 chars of body).

### 4. Compose the issue body

Format:

```markdown
<user's description, verbatim>

---

**Environment** (auto-filled)

- OS: <platform from `uname -a` or `$OSTYPE` — keep one line>
- Skill version: <output of `cd ~/.claude/skills/pantry-digest && git rev-parse --short HEAD` if it's a git checkout, else "unknown">
- Claude Code host: <best guess — `code` / `claude` CLI / Desktop — only if obvious; otherwise omit this line>

Filed via `/pantry-issue`.
```

Keep the footer short. Three lines, no more.

### 5. Pick labels based on type

| User said | Labels |
|---|---|
| bug | `bug` |
| feature / feature request | `enhancement` |
| question | `question` |
| other / unclear | (no label) |

If the repo doesn't have these labels yet, `gh issue create --label` will
fail — in that case retry without `--label` and tell the user.

### 6. Show a preview and confirm

Before firing `gh`, show the user the exact composed issue:

```
About to file this issue:

  Repo: SummerDeng7/pantry-digest-skill
  Title: <title>
  Labels: <labels or "(none)">
  Body:
  ---
  <full body, indented>
  ---

  Account: <gh active account>

Type "go" to file it, "edit" to tweak, or "cancel" to abort.
```

Wait for the user's word. Don't fire on assumption.

### 7. File the issue

```bash
gh issue create \
  --repo SummerDeng7/pantry-digest-skill \
  --title "<title>" \
  --body "<body>" \
  --label "<labels-if-any>"
```

Use a HEREDOC for the body to preserve newlines:

```bash
gh issue create --repo SummerDeng7/pantry-digest-skill \
  --title "..." \
  --label "..." \
  --body "$(cat <<'EOF'
<body>
EOF
)"
```

### 8. Report back

```
✅ Issue #<number> filed: <URL>

Filed under: <gh active account>
```

If the user used a personal account, mention they can star / watch the
repo to see your reply.

## Fallback A — `gh` not installed

> I can't file the issue automatically because the `gh` CLI isn't installed
> on this host. You have two options:
>
> 1. **Install gh** — see <https://cli.github.com/>, then re-run `/pantry-issue`.
> 2. **File it in your browser.** Click here to open a pre-filled issue:
>    <https://github.com/SummerDeng7/pantry-digest-skill/issues/new?title=<url-encoded-title>&body=<url-encoded-body>>
>
> If you want option 2, give me the title and description and I'll
> generate the URL.

If the user picks option 2, build the URL by URL-encoding the title and
body and showing the user a clickable link. Don't try to open the browser
for them.

## Fallback B — no gh auth

> `gh` is installed but not signed in. You have two options:
>
> 1. **Sign in** — run `gh auth login` in your terminal, then re-run `/pantry-issue`.
> 2. **File it in your browser.** Give me the title and description and
>    I'll generate a pre-filled issue link.

## Refusal

- User asks to file an issue against a different repo without authorization:
  ask "Did you mean the pantry-digest repo? If you want to file against
  another repo, tell me which one and I'll switch." Don't silently re-target.
- Title or body contains what looks like a secret (token, password, private
  key prefix like `gho_`, `ghp_`, `sk-`, `AKIA`, etc.): refuse, tell the user
  to redact, ask them to re-submit. This is a hard rule — issues are public.
- User says "make up reproduction steps to make this look like a real bug":
  refuse.

## Examples

**Example 1 — bug, interactive:**
```
User: /pantry-issue
Agent: (pre-flight OK) Three quick things...
User: bug. Sources list isn't loading. Just blank when I run /pantry-list.
Agent: (composes, previews) About to file...
User: go
Agent: ✅ Issue #5 filed: https://github.com/SummerDeng7/pantry-digest-skill/issues/5
```

**Example 2 — feature with inline title:**
```
User: /pantry-issue Add RSS feed support
Agent: Got the title. Type is feature request? And what's the use case?
User: yes, feature. Right now I have to use webfetch on Substack pages...
Agent: (composes, previews) ...
```

**Example 3 — no gh installed:**
```
User: /pantry-issue
Agent: I can't file automatically (gh not installed). Want me to make a
       pre-filled browser link? Tell me title + description.
User: yes. Title: "Spanish UI support". Description: "Would be great..."
Agent: Here: https://github.com/SummerDeng7/pantry-digest-skill/issues/new?title=...
```
