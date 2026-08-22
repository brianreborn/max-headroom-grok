---
name: max-headroom
description: >
  Install the latest Headroom release from github.com/headroomlabs-ai/headroom
  into a local `projects` or `meta-projects` directory, batch-install missing
  system prerequisites (one privilege-escalation prompt; fallback to
  `max-headroom.sh` or `max-headroom.ps1`), then write `max-headroom.script` so
  this session can engage the installed package. Use when the user wants to
  install Headroom, wrap Grok with Headroom, clone headroomlabs-ai/headroom,
  set up token compression, or run /max-headroom.
---

# Max Headroom

Repo: `https://github.com/headroomlabs-ai/headroom`
PyPI: `headroom-ai` (CLI ships **only** from the Python package, not npm)

Two modes. Detect, then run **one**:

1. **Engage** — a `max-headroom.script` exists and `headroom --version` works: read the script and follow **Session protocol**. Do not reinstall.
2. **Install** — otherwise run the install flow, then write the script and engage.

Detect first. Status-report each phase in one line. Ask only when a choice or privilege is required. Never ask "continue?"

**Resume on ruach (2026-08-22):** If `headroom` is missing or `max-headroom.script` is missing, **walk the operator through install** using `~/freebsd-mac-grok/REMNANT.md` (Headroom remnant) and this skill, unless they say the docs showed something to address first. Checkout `~/projects/headroom` @ v0.36.4 already exists — do not wipe it. Last `pip` wheel failed on `ort` / maturin (`ORT_DYLIB_PATH` match, FreeBSD). No `uv`. Use `python3.12`. Walk steps; do not jump to MAC `oneenforce`.

## Detect platform

```bash
uname -s 2>/dev/null || echo WINDOWS
```

| Result contains | Platform | Admin fallback |
| --- | --- | --- |
| `MINGW` / `MSYS` / `CYGWIN` / no `uname` on Windows | Windows | `max-headroom.ps1` |
| `Darwin` | macOS | `max-headroom.sh` |
| anything else | Linux | `max-headroom.sh` |

## Parent directory

Resolve `$PARENT` without asking when possible. First existing directory wins:

1. `$HOME/meta-projects`
2. `$HOME/projects`
3. `$HOME/Meta-Projects`
4. `$HOME/Projects`
5. `<workspace>/meta-projects` or `<workspace>/projects` if either exists as a directory

If **both** `meta-projects` and `projects` exist at the same rank (home or workspace), use `meta-projects`.
If none exist, create `$HOME/projects` (no prompt).

Checkout path: `$PARENT/headroom`

## Probe prerequisites — all at once

Run one probe. Collect every missing item before installing anything.

```bash
echo "OS=$(uname -s 2>/dev/null || echo unknown) ARCH=$(uname -m 2>/dev/null)"
command -v git; command -v curl; command -v gh; command -v uv; command -v rustc; command -v python3
python3 --version 2>/dev/null
uv python list 2>/dev/null | head
```

On Windows PowerShell, equivalent: `Get-Command git, curl, gh, uv, rustc, python -ErrorAction SilentlyContinue; python --version`

**Need (install if missing):**

| Need | Why | Prefer (no admin) | OS package fallback |
| --- | --- | --- | --- |
| `git` | clone/checkout | — | `git` |
| `curl` or `gh` | resolve latest release | — | `curl` and/or `gh` |
| Python 3.10–3.13 | Headroom wheels; **prefer 3.13** (LiteLLM $ savings; 3.14+ is unsupported for pricing) | `uv python install 3.13` | `python3.13` / `python@3.13` |
| `uv` | recommended CLI installer | `curl -LsSf https://astral.sh/uv/install.sh \| sh` | — |
| C++ toolchain | only if a **source** build is required (no matching wheel) | — | Linux: `build-essential`; macOS: Xcode CLT; Windows: VS Build Tools "Desktop development with C++" |
| Rust (`rustup`) | only if a **source** build is required | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` | Windows: `winget install Rustlang.Rustup` |

Wheels exist for Linux (`x86_64` / `aarch64`), macOS (arm64 + Intel), and Windows (`win_amd64`) on current releases. Treat source-build deps as **conditional**: add them to the missing list only after a wheel install has failed, or when `uname -m` / OS has no matching wheel.

### User-space first, then one admin batch

Install everything that does not need privilege **without asking** (`uv`, `uv python`, `rustup`).

Then, if any OS packages remain:

1. Show the full missing list and the **single** combined install command.
2. Ask once for privilege escalation.
3. Run that one command (one `sudo` / one elevated PowerShell).

Package-manager detection (first that exists): `apt-get`, `dnf`, `yum`, `pacman`, `zypper`, `apk`, `brew`, `winget`, `choco`.

Examples of **one** command:

```bash
# Debian/Ubuntu
sudo apt-get update && sudo apt-get install -y git curl python3.13 python3.13-venv build-essential

# Fedora
sudo dnf install -y git curl python3.13 gcc gcc-c++ make

# macOS (Homebrew is usually already user-writable — no sudo)
brew install git curl python@3.13

# Windows (elevated)
winget install --id Git.Git --id Rustlang.Rustup -e --accept-package-agreements --accept-source-agreements
```

### If privilege escalation fails

Write `$PWD/max-headroom.sh` (Unix) or `$PWD/max-headroom.ps1` (Windows) containing **only** the remaining privileged commands, then stop and tell the user to run it:

Unix:

```bash
#!/usr/bin/env bash
set -euo pipefail
# Run with: sudo bash ./max-headroom.sh
<the batched package install>
```

Windows:

```powershell
# Run elevated: Right-click → Run with PowerShell (Admin), or:
# Start-Process powershell -Verb RunAs -ArgumentList '-File', "$PSCommandPath"
<the batched package install>
```

Do not proceed with git clone / `uv tool install` until those packages exist. When the user confirms they ran the script, re-probe and continue. Do not regenerate the script unless the missing set changed.

## Latest stable release

"Best release" = latest **non-draft, non-prerelease** GitHub release.

```bash
curl -fsSL https://api.github.com/repos/headroomlabs-ai/headroom/releases/latest
```

If `gh` is authenticated, prefer `gh api repos/headroomlabs-ai/headroom/releases/latest`.

Read `tag_name` (e.g. `v0.36.4`). Strip the leading `v` → `$VERSION` (e.g. `0.36.4`).
If the API fails, fall back to `git ls-remote --tags --refs https://github.com/headroomlabs-ai/headroom.git` and pick the highest `vX.Y.Z` that is not a pre-release suffix (`a`, `b`, `rc`, `pre`).

## Checkout

```bash
REPO=https://github.com/headroomlabs-ai/headroom.git
```

- If `$PARENT/headroom/.git` exists: `git fetch --tags origin` and `git checkout --detach "$TAG"` (or the tag ref). Do not wipe uncommitted user changes; if the tree is dirty, stash or ask once.
- Else: `git clone --branch "$TAG" --depth 1 "$REPO" "$PARENT/headroom"`

Announce: checkout path + tag.

## Install the CLI

Prefer a **wheel** of the same version as the checkout (avoids compiling):

```bash
uv tool install --python 3.13 "headroom-ai[all]==$VERSION"
uv tool update-shell   # only if `command -v headroom` fails after install
```

If 3.13 is unavailable, use the newest available 3.10–3.13 (`--python 3.12`, …). Never install onto 3.14+ for the CLI.

If PyPI lacks that version, install from the checkout:

```bash
uv tool install --python 3.13 --from "$PARENT/headroom" "headroom-ai[all]"
```

If `uv` is missing after the probe (should not happen), `pipx install --python python3.13 "headroom-ai[all]==$VERSION"` or `python3.13 -m pip install --user "headroom-ai[all]==$VERSION"`.

Verify:

```bash
command -v headroom
headroom --version
headroom doctor
```

Record the **absolute** path of `headroom` (`command -v headroom` / `(Get-Command headroom).Source`). MCP clients often do not inherit interactive `PATH`.

## Write `max-headroom.script`

Read `${SKILL_DIR}/references/protocol-template.md` (this skill directory). Substitute real values. Write the filled file to **all** of:

1. `$PARENT/headroom/max-headroom.script` (next to the checkout)
2. `${SKILL_DIR}/max-headroom.script` (session home for later `/max-headroom`)
3. `$PWD/max-headroom.script` if cwd is not already (1) or (2)

On later Engage, search those three paths in order and load the first that exists.

## Engage this Grok session

Do **not** spawn a nested `grok` if this session is already Grok Build.

1. If `http://127.0.0.1:8787/health` is not healthy, start the proxy in the background:

   ```bash
   headroom proxy --port 8787
   ```

   Default profile is `coding` (prefix-cache safe). Do not change it unless the user asked.

2. Register Headroom MCP for Grok, using the **absolute** binary:

   ```bash
   headroom mcp install
   ```

   If that does not write `~/.grok/config.toml`, add:

   ```toml
   [mcp_servers.headroom]
   command = "<absolute-headroom>"
   args = ["mcp", "serve"]
   ```

   MCP tools typically appear only after a Grok restart. Still register now.

3. Durable inference routing (next Grok launch). Prefer, in order:

   ```bash
   headroom wrap grok-build
   ```

   If `grok-build` is not a wrap target:

   ```bash
   headroom wrap grok
   ```

   If wrap would start a second proxy, pass `--no-proxy` (this session already started one). `unwrap` is `headroom unwrap grok-build` (or `grok`).

4. Tell the user **once**:
   - Checkout path, version, `headroom` binary
   - Proxy: `http://127.0.0.1:8787` (dashboard `/dashboard`)
   - `max-headroom.script` path
   - Restart Grok so wrap + MCP take effect on inference
   - This session can still use MCP (after reload) or `POST http://127.0.0.1:8787/v1/compress`

### Session protocol (always, after Engage)

Follow the loaded `max-headroom.script`:

- Prefer Headroom MCP tools `headroom_compress`, `headroom_retrieve`, `headroom_stats` when they exist.
- If a tool/log/file result is large and MCP is unavailable, `POST /v1/compress` on loopback with the real model name. For multi-turn, pass `config.frozen_message_count` and resend **previously forwarded** messages, not pristine originals.
- When output contains `<<ccr:…>>` markers, call `headroom_retrieve` (or the CCR retrieve endpoint) before answering from the compressed stub.
- Do not reinstall, unwrap, or change `HEADROOM_SAVINGS_PROFILE` unless the user asked.
- `headroom doctor` / `headroom perf` only when diagnosing.

Canonical docs if a step fails: `https://headroom-docs.vercel.app/docs/installation` and `https://headroom-docs.vercel.app/docs/grok-build`.
