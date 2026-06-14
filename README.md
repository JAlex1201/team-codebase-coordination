# team-codebase-coordination

A Claude Code skill for keeping multiple coding agents, each driven by a different human teammate, synchronized on one shared GitHub repository. It establishes a lightweight coordination protocol so agents can work in parallel without stepping on each other, without a central orchestrator, and without a shared session.

> **This skill works best when every contributor on your team installs it.** The protocol depends on all agents reading from and writing to the same coordination file and following the same branch naming conventions. One agent skipping the protocol is a gap in the safety net.
>
> **Joining a repo that already uses this protocol?** Go straight to [SETUP.md](SETUP.md) — it's a one-page guide covering skill installation, hook activation, and your first task.

## What this skill does

This skill defines a five-step protocol that every agent runs on every coding task in a shared repository:

1. **Orient before touching anything** - Read `PROJECT_LOG.md` (the canonical coordination file) and scan open pull requests to get the full picture of what has shipped, what is in flight, and what was recently decided.
2. **Check for collisions, then claim a lane** - Compare the files your task will touch against open PRs and in-progress log entries. If there is overlap, coordinate before starting.
3. **Log that you are starting, then work in small increments** - Add an in-progress entry to the activity log, create a single-purpose branch with a consistent naming convention, and commit frequently.
4. **Update the coordination file as you work** - Keep your log entry current, record any hard-to-reverse decisions, and correct stale context. All updates ride with the code in the same PR.
5. **Push a PR as soon as coding is done** - Unpushed local work is invisible to every other agent. Small, single-purpose PRs merge faster and conflict less.

A pre-push hook (`assets/hooks/pre-push`) acts as a deterministic backstop: it blocks pushes that bypass the branch naming convention or that skip updating the log file. It cannot make agents read each other's decisions, but it makes a skipped protocol loud instead of silent.

## Trigger phrases

Claude activates this skill when you say things like:

- "Pick up where we left off"
- "Work on the repo"
- "Add a feature"
- "Fix this"
- "Start a new task"
- "What is in flight?"
- "Check if anything conflicts with my change"
- "Open a PR for this"

It also activates on any coding task in a repository where other people's agents may be working, even when coordination is not explicitly mentioned.

## When to use it

Use this skill any time you are:

- Starting, resuming, or handing off a coding task in a shared repository
- About to commit or push code that other agents may be touching
- Opening or updating a pull request
- Trying to understand what the rest of the team's agents have in flight
- Unsure whether your change will conflict with someone else's branch

## Team installation

This skill is most effective when everyone on the team installs it before starting work. One agent running the protocol while others skip it leaves the coordination file incomplete and makes collision detection unreliable.

### Recommended rollout

1. **One person seeds the coordination file.** Before any agents start, a human writes the initial `Context` section of `PROJECT_LOG.md` using `assets/PROJECT_LOG.template.md` as a starting point. A hand-written context section measurably outperforms an agent-generated one because the human knows what the repo actually is and how it is actually organized.

2. **Every contributor installs the skill.** Each team member downloads and installs `team-codebase-coordination.skill` (see [Installation](#installation) below) before their first session.

3. **Install the pre-push hook once per repo.** One person commits `.githooks/pre-push` to the repository. Every other contributor then runs a single config line after cloning:
   ```bash
   git config core.hooksPath .githooks
   ```

4. **Agree on shared conventions.** The defaults below work for most teams. If your team changes them, every agent must use the same values:
   - Canonical file: `PROJECT_LOG.md` at repo root
   - Branch naming: `<handle>/<short-task-slug>` (e.g. `jb/vendor-pos-normalization`)
   - Default base branch: `main`

## File structure

```
SKILL.md                                # Full coordination protocol
assets/
  PROJECT_LOG.template.md               # Starting template for the canonical file
  hooks/
    pre-push                            # Git pre-push hook (install into .githooks/)
```

## The coordination protocol in detail

### PROJECT_LOG.md: the single source of truth

Every agent reads this file in full before touching any code. It contains three sections:

- **Context** - The shared mental model a new agent needs: what the repo is, where the main components live, key conventions, and active constraints. Written by a human on first setup, updated by agents when they change something that makes the existing context wrong.
- **Decisions** - Dated, attributed records of choices that are hard to reverse or that affect how other people's code must behave. Routine implementation choices do not need an entry.
- **Activity log** - Append-only, newest at top, one entry per unit of work. Each entry is timestamped and attributed. No agent edits or deletes another agent's entry.

The file reflects merged-and-pushed work. Open pull requests carry the thin layer of in-flight work that has not landed yet. Reading both gives the full picture.

#### Conflict resolution

If git reports a merge conflict in `PROJECT_LOG.md`, the resolution is always the same: keep all entries from both sides and order them by timestamp. This file is the one place in the repository where "accept both" is correct by construction.

### Branch naming convention

```
<handle>/<short-task-slug>
```

Examples: `jb/vendor-pos-normalization`, `jake/lsd-eval-harness`

Use the same handle consistently across all your branches so teammates can tell at a glance which agent owns which branch. The pre-push hook enforces this convention.

### The pre-push hook

The hook runs on every push and refuses a push that bypassed the protocol:

- The branch name must follow the `<handle>/<short-task-slug>` convention.
- The branch must update `PROJECT_LOG.md` relative to the base branch.

Base and integration branches (`main`, `master`, `develop`, `release/*`, `hotfix/*`) are exempt. Use `git push --no-verify` for a genuine one-off exception.

Defaults are configurable via environment variables:

| Variable | Default | Purpose |
|---|---|---|
| `COORD_BASE_BRANCH` | `main` | Base branch to diff against |
| `COORD_LOG_FILE` | `PROJECT_LOG.md` | Canonical file that must be updated |
| `COORD_MODE` | `block` | `block` to refuse the push, `warn` to allow it with a message |

### Installing the hook

```bash
# Committed once to the repo by any contributor:
mkdir -p .githooks
cp assets/hooks/pre-push .githooks/pre-push
chmod +x .githooks/pre-push
git add .githooks/pre-push
git commit -m "add coordination pre-push hook"

# Run once per clone by every teammate:
git config core.hooksPath .githooks
```

## Security notes

This skill requires read access to the repository and write access to open pull requests. It uses `git` and the GitHub CLI (`gh`). It does not require any additional permissions beyond what a normal development workflow already has.

The pre-push hook runs `git diff` and `grep` locally. It makes no network requests beyond the normal push.

## Installation

### Via Claude Code

1. Download `team-codebase-coordination.skill` (the packaged version of this repo).
2. In Claude Code, run:
   ```
   /install-skill team-codebase-coordination.skill
   ```
3. The skill is now available in your session.

### Manual setup

Clone this repo and point your Claude Code project at it, or copy `SKILL.md` into your project's `.claude/skills/` directory.

## Guidelines for contributors

- **One skill, one purpose.** This skill focuses on multi-agent coordination in a shared repository. Do not expand it to cover PR authorship, code review style, or deployment workflows.
- **Specific trigger phrases.** Any changes to the description frontmatter should include clear trigger phrases so Claude can activate this skill accurately.
- **Append-only examples.** When adding examples or guidance, place them in a `references/` directory rather than growing `SKILL.md`.
- **Concrete actions.** Each step should describe a specific command or action the agent takes, not abstract advice.
- **Minimal permissions.** This skill uses `git` and `gh` as the coordination channel. Do not add dependencies on external services.

## Related skills

- [pr-best-practices](https://github.com/JAlex1201/PR_Author_Best_Practices_Skill) - Author strong pull request titles and descriptions
- [pr-presubmit](https://github.com/JAlex1201/pr-presubmit) - Run a structured pre-submit checklist before opening a PR
- [pr-review](https://github.com/JAlex1201/pr-review) - Write thoughtful, collaborative code review comments
