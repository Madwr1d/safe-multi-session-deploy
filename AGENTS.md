# AGENTS.md — shared rules for every model in this repo

Source of truth: `Madwr1d/grok-bot-org/templates/AGENTS.shared.md`. Copy it to the repo root as `AGENTS.md`
and add a short **Repo specifics** section at the end. Keep this file short; history goes to `docs/AI-HISTORY.md`.

## Who works here
| agent (`AI-Agent`) | tool | reads |
|---|---|---|
| `claude` | Claude Code (3060ti hub, laptrade) | `CLAUDE.md` → `@AGENTS.md` |
| `grok` | Grok CLI (3080) | `AGENTS.md` natively (also `CLAUDE.md`) |
| `grok-bot` | Grok Bot desks (Electron) | this file via the repo |
| `gemini` | Gemini CLI | `GEMINI.md` → points here |
| `codex` / `chatgpt` | Codex CLI / ChatGPT | `AGENTS.md` natively |
| `cursor` | Cursor | `AGENTS.md` natively |
| `human` | Vince (owner) | everything; he decides merges to live and every deploy |

## Read first (every session, before any edit)
1. `git fetch --all --prune` and `git status --short` — know what is already changed and by whom.
2. `tail -20 AI-LEDGER.md` — what the last models did, what is live, what is mid-flight.
3. Open GitHub Issues — the task board. Pick or claim one before you start (below).
4. Check the live version if the repo deploys (`/version.json` or the repo's specifics) before building on it.

## Branches and merges
- Work only on your own branch: `<agent>/<topic>` (e.g. `claude/fix-login`, `grok/ledger-cleanup`).
- Never commit directly to the default/live branch.
- Merge to the default/live branch **only via a Pull Request** that
  - was reviewed by a **different** model (or Vince) — a model never approves its own PR, and
  - has green CI (tests + gitleaks).
- **Deploys only with Vince's OK.** No deploys while Vince streams.
- Rebase/merge from the live branch before opening a PR; resolve conflicts on your branch, not on live.

## Task board = GitHub Issues
- Claim: comment `claimed by <agent> on <device>` and add the label `in-progress`.
- One owner per issue. If it is claimed, pick another or ask in the issue — do not work it in parallel.
- Link the PR in the issue (`Closes #N`). When done or abandoned: remove `in-progress`, comment the state.
- New work you discover: open an issue instead of silently widening your PR.

## Never undo others' work
- Do not revert, delete, reformat or "clean up" code another model or Vince wrote unless the issue says so.
- Uncommitted changes you did not make belong to another session: leave them alone, work in a worktree.
- Disagree? Say it in the PR/issue. Never force-push, never rewrite shared history, never delete branches.

## Staging and secrets
- Stage by path: `git add path/to/file`. Never `git add -A` / `git add .`.
- Never commit `.env*` (except `.env.example` with placeholders), keys, tokens, seed phrases, wallet files,
  cookies, chat exports, `node_modules/`, build output. Reference secrets by **env var name** only.
- The `pre-commit` hook runs `gitleaks protect --staged --redact` and blocks on findings. Do not bypass it
  with `--no-verify`. A real false positive goes into `.gitleaks.toml` (reviewed in the PR) or gets a
  `gitleaks:allow` comment on that line.
- A key that ever reached a commit is burned: tell Vince; do not push that history.

## Commit stamp (once per clone)
```
git config core.hooksPath .githooks
git config user.name  "Madwr1d"
git config user.email "268367006+Madwr1d@users.noreply.github.com"
```
`prepare-commit-msg` appends the trailers below. Set `AI_AGENT`, `AI_MODEL`, `AI_DEVICE`, `AI_SESSION` in
your shell for a complete stamp; anything missing is filled with what is known (hostname → device,
`claude-code_*` → `claude`, `codex*` → `codex`, `gemini*` → `gemini`, else `unknown`). It never blocks a commit.
```
AI-Agent: claude
AI-Model: claude-opus-5-5
Device: 3060ti
Session: <session id or room name — never a chat-blob filename>
```
Devices: `3060ti` (hub, streams), `3080`, `laptrade`, `surface`. Plain human commits get only `Device:`.

## Push and ledger
- Push your working branch to the private GitHub repo after each logical checkpoint (no force).
- Append one line to `AI-LEDGER.md` (newest at the bottom) per merge or deploy:
  `| when (UTC) | device | ai (model) | branch@sha | what + why | deployed? |`
  Write a `DEPLOYING` line **before** a deploy and update it with the live result after.

## Machines
- The 3060ti hub streams: no heavy builds/tests/workflows there while live
  (`node ~/bin/obs-stream-status.mjs /mnt/c/Users/vince/Desktop/kick-stream-tools-hub/obs-ws.json`).
- Heavy work runs on laptrade or the Surface Book.

## Money and safety
- Never place orders, sign transactions, or move funds. Hyperliquid/wallet operations are Vince's.
- Never post publicly, spend money, or create accounts unless Vince says so.

## Keep this file short
Under ~150 lines. Session history, incident write-ups, and long notes go to `docs/AI-HISTORY.md`.

## Repo specifics
- **Purpose:** public Claude Code / agent skill (`SKILL.md` + `scripts/`) for safe deploys when several
  agent sessions share one working tree. Installed by others via `npx skills add Madwr1d/safe-multi-session-deploy`.
- **PUBLIC repo** — never commit anything private (keys, tokens, client names, logs, internal hostnames).
- **Live branch:** `master` (GitHub default). No build, no tests, no deploy target; "release" = push to `master`.
- **Check:** `bash -n scripts/*.sh` (and `shellcheck scripts/*.sh` if installed) before a PR.
- **Hot files:** `SKILL.md` (agent-facing rules, keep the frontmatter `name`/`description` valid), `scripts/deploy-guard.sh`,
  `scripts/integrate-and-deploy.sh`, `README.md` (install instructions).
