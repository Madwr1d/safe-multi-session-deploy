# safe-multi-session-deploy

A Claude Code / agent **skill** that makes deploying safe when **multiple agent
sessions share one working tree** — the situation that quietly breaks real teams
running several AI coding sessions at once.

It prevents the three failure modes that actually happen in production:

1. **Clobbering unsaved work** — a deploy uploads the whole working tree, so
   another session's in-progress edits ship (or yours get overwritten).
2. **Wrong-project deploy** — `.vercel/project.json` is linked to a stale or
   duplicate project, so the build succeeds but the real site never updates and
   you *think* you shipped.
3. **Invisible provenance** — after a deploy you can't tell *which* session's
   build is actually live, or whether the alias even moved.

It also includes the git-native version of "merge every session's work into one
build before shipping" (per-session branches → integration branch → real 3-way
merge with conflict detection → verified deploy).

## Install

```bash
npx skills add madclaw1/safe-multi-session-deploy
```

Then it's available to your agent like any other skill. (It shows up on
[skills.sh](https://skills.sh) via install telemetry.)

## What's inside

- **`SKILL.md`** — the rules an agent follows: identify a session id, session-tag
  commits, the pre-deploy guard checklist, and the integration-branch workflow.
- **`scripts/deploy-guard.sh <domain> [project] [public-dir]`** — single-session
  safe deploy for Vercel: verifies the linked project owns the domain, checks for
  foreign uncommitted edits, builds as a gate, stamps `version.json`
  `{session, sha, builtAt, project}`, deploys, then **verifies the live
  `version.json` matches** what it built.
- **`scripts/integrate-and-deploy.sh <domain> <project>`** — merges every
  `session/*` branch into one `pre` branch (stops on conflict instead of silently
  clobbering), stamps multi-session provenance, deploys, and verifies live.

## Why

Born from a real incident: four agent sessions on one repo, a deploy that went to
the wrong Vercel project, and no way to tell whose code was live. This skill is
the checklist + scripts that stop all three.

## License

MIT
