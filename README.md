# aep-skills

A Claude Code plugin that bundles three AEP (Agent Enhancement Proposal) skills and the shared AEP spec, distributed as a self-contained marketplace so it can be installed and updated on any machine with the `/plugin` commands.

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| `publish-aep` | `/publish-aep <rule-description>` | Publish a new AEP (searches for duplicates first). |
| `improve-plan` | `/improve-plan` | Review the current plan-mode plan against AEPs and edit it in place. |
| `deep-review` | `/deep-review <PR-number-or-URL>` | Deep PR review against CLAUDE.md rules and AEPs; posts/edits a review comment. |

The AEP spec the skills follow ships in `plugins/aep-skills/reference/tootik-client.md` and is referenced from the skills via `${CLAUDE_PLUGIN_ROOT}`, so it travels and versions with the plugin. The `tootik-client` wrapper ships in `plugins/aep-skills/bin/` and is added to the Bash tool's `PATH` automatically while the plugin is enabled — it is a small POSIX `sh` script that downloads and runs a prebuilt `cmd/tootik-client`, so no binary is committed and it works on Linux, macOS and Windows.

The wrapper connects to a tootik instance identified by a **host**, remembered globally in `${CLAUDE_CONFIG_DIR:-~/.claude}/aep-skills/host`. It injects `-host` into every call; on the first run with no host configured it prints instructions to ask the user for the host and save it (the host is never guessed).

The account it authenticates as defaults to the OS user name. Where that is not the account name on the instance — a shared devbox, a CI runner, any login that differs — write the right one to `${CLAUDE_CONFIG_DIR:-~/.claude}/aep-skills/user` and the wrapper injects `-user` as well. With no such file the call is unchanged, so the common case needs nothing. Note that both files sit under `CLAUDE_CONFIG_DIR`: a second Claude account has its own, and needs its own copy.

> **PATH note:** the wrapper is named `tootik-client`. Claude Code's ordering of the plugin `bin/` relative to the rest of your `PATH` is not documented, so a separate `tootik-client` earlier on `PATH` (e.g. `~/go/bin/tootik-client`, or `tootik-client.exe` on Windows) could shadow it. If skills end up running the wrong binary, remove or rename the other one.

## Install

```
/plugin marketplace add dimkr/aep-skills
/plugin install aep-skills@aep-skills
```

(`dimkr/aep-skills` is the GitHub `owner/repo`; adjust if you fork or rehost.)

## Update

```
/plugin update aep-skills@aep-skills
```

Or enable auto-update from `/plugin` → Marketplaces. Releases are cut by bumping `version` in both `.claude-plugin/marketplace.json` and `plugins/aep-skills/.claude-plugin/plugin.json`.

## Prerequisites (assumed present on the target device — not installed by this plugin)

- A reachable tootik instance at the configured host for `tootik-client` to talk to.
- `gh` (GitHub CLI, authenticated) and `git` (used by `deep-review` to clone the repo and post comments), plus a Python 3 interpreter on `PATH` (AEP ID computation).
- On Windows: a POSIX shell environment.

## Manual setup after install

### Plan-mode wiring for `improve-plan` (optional)

If you want the plan-improvement skill to run automatically before `ExitPlanMode`, add a line to your global `~/.claude/CLAUDE.md` Planning section invoking `/improve-plan`.

## Cross-platform notes

- No compiled binaries are committed to this repo — only text and the portable POSIX `sh` wrapper.
- The `bin/tootik-client` wrapper avoids hardcoded macOS paths and downloads a prebuilt binary, so it works on Linux, macOS and Windows with no per-OS build step.
