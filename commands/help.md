# /pantry-help

**Purpose:** Show the command list and a one-paragraph quick-start.

## Argument form

```
/pantry-help
```

No arguments.

## Output (render verbatim)

> **The Pantry · 茶水间** generates a daily AI news digest webpage from
> real sources. All cards link to real articles; no invented stories,
> no fake engagement numbers. Default output: `AI Redbook/index.html`.
>
> **Commands:**
>
> - `/pantry-generate [scope]` — generate the digest. Examples:
>   - `/pantry-generate` (today, everything)
>   - `/pantry-generate this week, model releases only`
>   - `/pantry-generate focus on AI coding agents`
>   - `/pantry-generate only my custom sources`
>
> - `/pantry-add <name-or-url>` — add a source. Examples:
>   - `/pantry-add https://stratechery.com/`
>   - `/pantry-add Import AI https://jack-clark.net/ paper`
>
> - `/pantry-remove <name>` — remove a source. Confirmation required for defaults.
>
> - `/pantry-list` — show all configured sources.
> - `/pantry-sources <filter>` — filtered view (newsletters / priority 1 / custom / etc.)
> - `/pantry-help` — this message.
>
> **Natural language also works.** "Refresh the pantry" / "茶水间一下"
> / "Update my AI redbook" all route to `/pantry-generate`.
>
> **First-time setup:** nothing to install. Run `/pantry-list` to see
> the default 16 sources, then `/pantry-generate`.
