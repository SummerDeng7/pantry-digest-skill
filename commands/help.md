# /pantry-help

**Purpose:** Show the command list and a one-paragraph quick-start.

## Argument form

```
/pantry-help
```

No arguments.

## Output (render verbatim)

> **The Pantry · 茶水间** generates a daily news digest webpage from real
> sources. Defaults to AI news with a curated 16-source roster; works for
> any topic — biotech, climate, finance, sports, fashion — pass it as a
> scope. All cards link to real articles; no invented stories, no fake
> engagement numbers. Default output: `pantry-digest.html` in the current
> directory.
>
> **Commands:**
>
> - `/pantry-generate [scope]` — generate the digest. Examples:
>   - `/pantry-generate` (today's AI news, default sources)
>   - `/pantry-generate biotech news today`
>   - `/pantry-generate climate this week`
>   - `/pantry-generate focus on AI coding agents`
>   - `/pantry-generate only my custom sources`
>
> - `/pantry-add <name-or-url>` — add a source (any topic). Examples:
>   - `/pantry-add https://stratechery.com/`
>   - `/pantry-add STAT News https://www.statnews.com/ news_aggregator`
>
> - `/pantry-remove <name>` — remove a source. Confirmation required for defaults.
>
> - `/pantry-list` — show all configured sources.
> - `/pantry-sources <filter>` — filtered view (newsletters / priority 1 / custom / etc.)
> - `/pantry-issue [title]` — file a bug / feature request / question against
>   the upstream skill repo. Uses your `gh` CLI account; falls back to a
>   pre-filled browser link if `gh` isn't installed.
> - `/pantry-help` — this message.
>
> **Natural language also works.** "Refresh the pantry" / "茶水间一下"
> / "Daily biotech digest please" all route to `/pantry-generate`.
>
> **First-time setup:** nothing to install. Run `/pantry-list` to see
> the default 16 AI sources, then `/pantry-generate` for AI news, or
> `/pantry-generate <other topic>` for anything else.
