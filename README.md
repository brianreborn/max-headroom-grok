# max-headroom-grok

Grok **plugin** for the `/max-headroom` skill: install [Headroom](https://github.com/headroomlabs-ai/headroom) (`headroom-ai` on PyPI), write `max-headroom.script`, wrap Grok.

License: **Light-ware** (`LICENSE` — BSD-4-Clause plus a non-binding ask to help keep the lights on). Not PGP-signed yet.

This skill does **not** vendor Headroom. Headroom’s own license applies to the cloned tree and PyPI package.

## Install (Grok)

```sh
grok plugin install brianreborn/max-headroom-grok --trust
```

Or copy `skills/max-headroom/` to `~/.grok/skills/max-headroom/`.

## What the skill does when you run it

Plugin install only copies prompts. When `/max-headroom` **runs**, the agent may:

- `curl`/`gh` GitHub (`headroomlabs-ai/headroom` releases)
- `git clone` that repo under `~/projects` or `~/meta-projects`
- install user-space `uv` / `rustup` (their install scripts)
- `uv tool install` or `pip install` from PyPI (`pypi.org`)
- ask **once** for sudo/`pkg`/`apt`/… for missing OS packages, or write `max-headroom.sh` / `.ps1`
- start `headroom proxy` on `127.0.0.1:8787`
- write `~/.grok/config.toml` MCP/wrap entries

No install-time hooks. No telemetry in this skill. Credentials: none required for the skill itself; Headroom/Grok tokens are yours.

Install remnant (checkout exists; CLI and `max-headroom.script` do not): **[REMNANT.md](REMNANT.md)**.

## Marketplace

Catalog-ready shape: `skills/`, `.grok-plugin/plugin.json`, `LICENSE`, public repo. Pin a 40-character commit SHA if you PR the xAI plugin index. Do not pin `main`.
