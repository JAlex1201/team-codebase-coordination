# Teammate Setup Guide

This is the one-page guide for joining a repository that already uses the team-codebase-coordination protocol. If you are the first person setting it up in a new repo, see [README.md](README.md) instead.

---

## Step 1 — Install the Claude Code skill

The skill tells your Claude agent how the coordination protocol works. Install it once; it applies to all your sessions.

**Via Claude Code:**
```
/install-skill team-codebase-coordination.skill
```

**Manual (if you don't have the packaged skill file):**
Copy `SKILL.md` from this repo into your project's `.claude/skills/team-codebase-coordination/` directory.

---

## Step 2 — Activate the pre-push hook

The repository already contains `.githooks/pre-push`. You just need to tell git to use it. Run this once per clone:

```bash
git config core.hooksPath .githooks
```

That's it. From now on, every push you make will be checked automatically:
- Branch name must follow `handle/slug` (e.g. `jb/add-login`, `pierce/fix-batch-network`)
- `PROJECT_LOG.md` must have been updated on the branch

If either check fails, the push is blocked with a message telling you what to fix.

---

## Step 3 — Pick your handle

Your handle is a short, lowercase identifier used in branch names and log entries. It should be consistent across all your work so teammates can see at a glance which branches and log entries are yours.

Examples: `julian`, `pierce`, `jake`, `jb`

Agree with your team if there is any ambiguity. The handle goes in:
- Branch names: `your-handle/short-task-slug`
- Activity log entries in `PROJECT_LOG.md`

---

## Step 4 — Read PROJECT_LOG.md before your first task

Before touching any code, read the canonical coordination file in the repo root:

```bash
cat PROJECT_LOG.md
gh pr list --state open --json number,title,headRefName,author,isDraft,updatedAt
```

This gives you the shared context, recent decisions, what has shipped, and what is currently in flight from other agents.

---

## How a normal task looks

```bash
# 1. Orient
git fetch --all --prune
cat PROJECT_LOG.md
gh pr list --state open

# 2. Create your branch
git checkout -b your-handle/your-task-slug origin/main

# 3. Do the work, commit frequently

# 4. Update PROJECT_LOG.md on your branch
#    - Add an in-progress entry at the top of the Activity log
#    - Add a Decisions entry if you made a notable architectural choice

# 5. Push — the hook verifies branch name + log update
git push -u origin your-handle/your-task-slug

# 6. Open a PR
gh pr create --title "short title" --body "..."
```

---

## Escape hatch

If you have a genuine reason to push without the checks (e.g. pushing a hotfix on `main`), protected branches are exempt automatically. For a one-off exception on a feature branch:

```bash
git push --no-verify
```

Use sparingly — skipping the hook means the log won't have a record of your change until you add it manually.

---

## Conflict in PROJECT_LOG.md?

Keep all entries from both sides and order them by timestamp. This file is append-only by design, so "accept both" is always the right resolution.
