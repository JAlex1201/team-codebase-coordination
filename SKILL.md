---
name: team-codebase-coordination
description: Coordination protocol for multiple coding agents, each driven by a different human teammate, contributing to one shared GitHub repository. Use this whenever you are about to start a coding task, write code, commit, push, or open a pull request in a shared repo. Before doing any work, use it to read the repository's canonical coordination file (PROJECT_LOG.md) so you know the shared context, recent decisions, and what other teammates' agents have in flight. Use it to update that same file as you work, and to push a pull request to GitHub as soon as a piece of coding is done so the repo history stays complete and other agents can see your change. Trigger this even when the user only says things like "pick up where we left off", "work on the repo", "add a feature", or "fix this", as long as the work happens in a shared codebase that other people's agents also touch.
---

# Team Codebase Coordination

Multiple humans each drive their own coding agent against one shared GitHub repository. You are one of those agents. The other agents cannot see your local context, your plans, or your uncommitted work. They see two things only: what is on GitHub, and what is written in the repository's canonical coordination file. This protocol keeps the whole team in sync through those two channels, so no central server or shared session is required.

## The single source of truth: PROJECT_LOG.md

The repository contains one canonical file, `PROJECT_LOG.md` at the repo root, that every agent reads first and updates as it works. It holds the shared context, the decision history, and a running activity log. Treat it as authoritative. If the code and the file disagree, the file tells you what the team intended, and the disagreement is itself worth flagging.

The file reflects merged-and-pushed work. Anything still in an open pull request is not in it yet, so you also glance at open PRs for the thin layer of in-flight work that has not merged. The file plus open PRs together give you the full picture.

### Why the file stays usable instead of turning into a merge-conflict magnet

Two rules, both mandatory:

1. **The activity log is append-only.** Add your own timestamped, attributed entry. Never edit or delete another agent's entry. Newest entries go at the top of the log section.
2. **Updates ride with the code.** Update `PROJECT_LOG.md` on your working branch and let it merge in the same PR as your change, so log updates serialize through merges instead of racing.

If git ever reports a conflict in `PROJECT_LOG.md`, the resolution is always the same: keep every entry from both sides and order them by timestamp. This is the one file in the repo where "accept both" is correct by construction.

### First run: if PROJECT_LOG.md does not exist yet

The file should be seeded once, by a human, before agents start relying on it. A person writes the initial Context section ("what this repo is and how it is organized"), because a hand-written context section measurably outperforms an agent-generated one. Start from `assets/PROJECT_LOG.template.md`, fill in Context, and commit it.

If you are an agent and `PROJECT_LOG.md` is genuinely missing, do not generate a rich file from your own assumptions. Instead, create a minimal scaffold from the template with an empty Context section and a single "log initialized" activity entry, open it as its own small PR, and tell the human that Context needs to be filled in. Keeping bootstrap in a dedicated PR avoids two agents racing to create whole-file copies that would conflict on merge.

## Required tools

This skill assumes the GitHub CLI (`gh`) and `git` are available. Check once at the start of a session:

```bash
gh --version && git --version && gh auth status
```

If `gh` is missing or not authenticated, tell the human and stop, because without it you cannot see other agents' in-flight pull requests.

## Step 1: Read before you touch anything

Run this every time you start a task, resume work, or have been idle.

```bash
git fetch --all --prune

# The canonical file is the first thing you read, in full.
# If this errors because the file is missing, go to "First run" above.
cat PROJECT_LOG.md

# The thin live layer: anything not yet merged into the file
gh pr list --state open --json number,title,headRefName,author,isDraft,updatedAt
```

Read the whole canonical file: the context section, the recent decisions, and the most recent activity entries. Then scan open PRs for work that has not landed in the file yet. For any PR that looks related to your task, check what it touches:

```bash
gh pr view <number>
gh pr diff <number> --name-only
```

You now know the shared context, what was recently decided, what shipped, and what is coming down the pipeline.

## Step 2: Check for collisions, then claim your lane

Decide which files your task will touch. Compare that set against the in-progress entries in the activity log and the files in every open PR.

- **No overlap:** proceed.
- **Overlap with an open PR:** do not start a parallel edit of the same files. Choose one: branch from that PR's head and build on it, wait for it to merge, or comment on the PR to coordinate with the teammate who owns it. Tell the human which you chose and why.
- **Your task depends on something still in an open PR:** branch from that PR's head rather than from `main`, and note the dependency in your log entry and PR description.

When overlap is unclear, ask the human rather than risk a tangled merge.

## Step 3: Log that you are starting, then work in small increments

Add an in-progress entry to the activity log (top of the log section) describing what you are about to do, then create a branch named so the owner and purpose are obvious:

```
<handle>/<short-task-slug>
```

Examples: `jb/vendor-pos-normalization`, `jake/lsd-eval-harness`. Use the same `<handle>` consistently so teammates can tell at a glance which agent owns which branch.

Keep each branch single-purpose. Commit frequently with clear messages.

## Step 4: Update PROJECT_LOG.md as you work

Update the canonical file on your branch, so it merges atomically with your code:

- **Activity log:** keep your in-progress entry current. When the work is done, update its status to done and note the PR number.
- **Decisions:** when you make a choice that is hard to reverse, affects how other people's code must behave, or would make a teammate later ask "why was it done this way?", add a dated, attributed entry to the Decisions section. Record the decision, the alternatives you weighed, and the consequence. Routine implementation choices do not need an entry.
- **Context:** if you changed something that makes part of the Context section wrong (a new convention, a moved module, a changed contract), append a correction. Do not silently delete what was there; supersede it with a dated note so the history is intact.

Append, attribute, timestamp. Never overwrite another agent's words.

## Step 5: Push a PR as soon as the coding is done

The moment a coherent piece of work is finished and the log is updated, push it and open a PR. Unpushed local work is invisible to every other agent and is missing from the repo history, which is the whole thing this protocol exists to prevent.

```bash
git push -u origin <branch>
gh pr create --title "<concise title>" --body "<use the structure below>"
```

Push as soon as coding is done rather than batching. Keep PRs small and single-purpose: a reviewer, human or agent, handles a 50-line PR well and a 2000-line PR badly. If you want work visible even earlier, open it as a draft (`--draft`) on the first meaningful commit and mark it ready (`gh pr ready <number>`) when done.

### PR description structure

```
## What
One or two sentences on the change.

## Why
The problem being solved. Reference the relevant Decisions entry in PROJECT_LOG.md.

## In-flight dependencies
PRs or branches this builds on or might conflict with, e.g. "builds on #42".
Write "none" if independent.

## Files touched
The main paths, so other agents can scan for overlap quickly.
```

## Step 6: Re-read before you finish

Right before marking a PR ready or handing back to the human, re-run Step 1. A PR may have merged into `PROJECT_LOG.md` that conflicts with yours, or a new decision may change your approach. If something landed that affects you, rebase or coordinate before declaring done.

## Enforcement: the pre-push hook

The skill shapes agent behavior but cannot guarantee an agent consults it on every task. A pre-push hook is the deterministic backstop for the times it does not. The hook is bundled at `assets/hooks/pre-push`. It runs on every push, with no model judgment involved, and refuses a push that bypassed the protocol:

- The branch name must follow the `<handle>/<short-task-slug>` convention.
- The branch must update `PROJECT_LOG.md` relative to the base branch (so every unit of work leaves a trace).

Base and integration branches (`main`, `master`, `develop`, `release/*`, `hotfix/*`) are exempt, and `git push --no-verify` bypasses the hook for a genuine one-off exception. The hook cannot make an agent read other people's decisions; it only makes a skipped protocol loud instead of silent. The orientation step still has to happen at the start of work, which is where the real collision protection comes from.

Install once, shared through the repo so teammates run a single config line:

```bash
mkdir -p .githooks
cp assets/hooks/pre-push .githooks/pre-push
chmod +x .githooks/pre-push
# commit .githooks/pre-push, then each teammate runs once after cloning:
git config core.hooksPath .githooks
```

Defaults are configurable via `COORD_BASE_BRANCH`, `COORD_LOG_FILE`, and `COORD_MODE` (`block` or `warn`).

## What this protocol deliberately does not do

- It does not keep a mutable "who is working on what" table. Live status lives in the append-only activity log plus open PRs, neither of which requires editing shared lines.
- It does not require agents to message each other directly. The canonical file and GitHub PRs are the message bus.
- It does not assume a single orchestrator. Every agent runs this same protocol independently; coordination emerges from everyone reading and writing the same file and the same GitHub state.

## Quick reference

| Need | Action |
| --- | --- |
| Get oriented | `cat PROJECT_LOG.md`, then `gh pr list --state open` |
| Inspect a related PR | `gh pr diff <number> --name-only` |
| Announce you are starting | append an in-progress entry to the activity log |
| Record a decision | add a dated entry to the Decisions section of PROJECT_LOG.md |
| Maintain history | push a PR as soon as coding is done |
| Resolve a log conflict | keep all entries from both sides, order by timestamp |

## Conventions the team should agree on once

Defaults below. The team can change them by editing this skill, but every agent must use the same values:

- Canonical file: `PROJECT_LOG.md` at repo root. Start from `assets/PROJECT_LOG.template.md`.
- Activity log: append-only, newest at top, each entry timestamped and attributed.
- Branch names: `<handle>/<short-task-slug>`.
- PRs: pushed as soon as coding is done, small, single-purpose.
- Default base branch: `main`.
