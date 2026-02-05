A practical guide to setting up your own "Ralph" autonomous coding loop.

# diy-ralph

This project is currently a collection of notes, scripts and prompts that I used
to stand up a couple of "Ralph development loops". Geoffrey Huntley's [original](https://ghuntley.com/ralph/)
post is a little vague about what is _needed_ and what are _suggestions_. So here
is my list of "ingredients" and questions that you need to answer or implement
when you're getting started with your own loop:

## Getting started

1. Where are you going to run Ralph? On an old laptop? In the cloud?
    * Do you want to run Ralph in a VM, like qemu or virtualbox?
    * Do you want to run Ralph in a container, like docker or podman?
    * There's lots of options and [recommendations](https://github.com/ghuntley/how-to-ralph-wiggum/blob/main/references/sandbox-environments.md).
      Pick one and try it.
2. How do you like running long-running tasks?
    * I have a habit of using `screen` but the industry currently adores `tmux`
    * You may want to graduate to running it in `systemd` or `launchd` to ensure
      Ralph resumes working in case the machine its running on restarts.
3. How do you like tracking your "root" spec?
    * Geoffrey's [example](https://github.com/ghuntley/how-to-ralph-wiggum) uses
      `@IMPLEMENTATION_PLAN.md`
    * Anthropic's [note](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
      on long running agents recommends using JSON. I like `TASKS.jsonl` with a
      corresponding spec/plan/prd.
4. How do want Ralph to track its memories?
    * `STATUS.md` or `PROGRESS.md`? Update `TASKS.jsonl` as pending?
    * Do you want `LEARNINGS.md` to track failures/gotchas?
    * Do you want Ralph to write `specs/` and/or `docs/`?
5. Do you want Ralph to improve itself?
    * Do you want it updating `AGENTS.md` or `RALPH.md`? They will get updated
      on the next loop iteration!
    * Do you want Ralph updating `ralph.py` (more on that below)? Do you want it
      reading `~/.ralph/logs/` to find inefficiencies?
6. What coding agent do you prefer to use?
    * Claude Code, Codex are from frontier AI companies
    * Ampcode has an ad-supported free tier
    * Opencode and Aider are open source
7. How do you want to structure your codebase?
    * Where do you keep your source code? `src/`?
    * Do you want Ralph checking out code to `~` as a part of its research
      before adding new dependencies?
8. What's stopping you from getting started??

The most important thing is to just try, and "[let ralph ralph](https://github.com/ghuntley/how-to-ralph-wiggum?tab=readme-ov-file#-let-ralph-ralph)".
You will burn through API tokens like crazy. But you can pay for a monthly
subscription that will give you some allowance. [ralph.py](./ralph.py) handles
Claude code getting rate limited and sleeps until the next window.

JUST GET STARTED!

## Quick Start

Copy these files to your existing project:

1. **`RALPH.md`** - The prompt that tells Claude what to do each iteration
2. **`ralph.py`** - The loop script (no dependencies beyond Python 3.10+ stdlib)
3. **`AGENTS.md`** - Your project-specific commands (customize this heavily)

Then create these files based on the [Example Files](#example-files) section:

4. **`TASKS.jsonl`** - Your task backlog
5. **`STATUS.md`** - High-level progress tracking
6. **`LEARNINGS.md`** - Hard-won lessons for future iterations

Add these files to git:

```bash
git add RALPH.md ralph.py AGENTS.md TASKS.jsonl STATUS.md LEARNINGS.md
git commit -m "Add Ralph loop"
```

Tune these files based on your answers to the [Getting Started](#getting-started) questions.
For example, if you prefer `IMPLEMENTATION_PLAN.md` over `TASKS.jsonl`, update the
references in `RALPH.md` accordingly.

### Removing Ralph

To remove Ralph from your project:

```bash
git rm RALPH.md ralph.py AGENTS.md TASKS.jsonl STATUS.md LEARNINGS.md
git commit -m "Remove Ralph loop"
```

Git remembers, so you can always restore these files later if you want Ralph back.

## Ingredients

These are the bare _minimum_ concepts for a Ralph development loop:

1. Your `AGENTS.md` or `CLAUDE.md` file. Keep it short.
    * `AGENTS.md` can be used by other coding agents (ex: `codex` or `opencode`)
      but beware that different LLM models will interpret prompts differently.
    * `ln -s AGENTS.md CLAUDE.md` is a nice trick for writing to `AGENTS.md` and
      ensuring Claude Code reads it. Claude Code reads `CLAUDE.md` by default but
      `AGENTS.md` is more portable across different coding agents.
2. Your `RALPH.md` or `PROMPT.md` file.
3. Your `IMPLEMENTATION_PLAN.md` or `specs/architecture.md` "root" plan/prd/spec.
4. Your loop!
    * Geoffrey's [original post](https://ghuntley.com/ralph/) used `while :; do cat PROMPT.md | claude-code ; done`
    * I started with `for _ in {1..10}; do cat PROMPT.md | claude --print --dangerously-skip-permissions ; done`
    * My Ralph implementation evolved into [ralph.py](./ralph.py)

Everything else is optional.

## Example Files

### TASKS.jsonl

Each line is a JSON object with `id`, `description`, `status`, and `done_when` fields:

```jsonl
{"id": 1, "description": "Scaffold backend with health endpoint", "status": "done", "done_when": "curl localhost:8080/health returns 200 OK"}
{"id": 2, "description": "Add weather API integration", "status": "in_progress", "done_when": "GET /api/weather?city=Seattle returns current temperature"}
{"id": 3, "description": "Create frontend weather display component", "status": "todo", "done_when": "npm test passes with weather component rendering temperature"}
{"id": 4, "description": "Add 5-day forecast endpoint", "status": "todo", "done_when": "GET /api/forecast?city=Seattle returns 5 days of weather data"}
```

Status values: `todo`, `in_progress`, `done`

The `done_when` field is crucial - it tells Ralph exactly how to verify the task is complete.
Other blogs call this field `acceptance_criteria`.

### STATUS.md

Track high-level progress. Keep it compact.

```markdown
# Status

## Current State

- **Backend scaffold complete** - health endpoint, basic routing
- **Weather API researched** - using OpenWeatherMap free tier
- **Frontend not started** - blocked on API integration

## Last Completed

- Task 1: Backend scaffold (2026-02-05)
  - Created main.go with health endpoint
  - Added go.mod with correct module path
```

### LEARNINGS.md

Document surprises, failures, and workarounds. Future Ralphs read this first.

```markdown
# Learnings

Hard-won lessons from development. Future Ralphs: READ THIS FIRST.

---

### 2026-02-05: OpenWeatherMap API requires units parameter

**Context:** Weather API returned temperature in Kelvin by default.

**Solution:** Add `&units=imperial` or `&units=metric` to API calls.

**Lesson:** Always check API documentation for default units. Don't assume.

---

### 2026-02-05: Rate limiting on free tier

**Context:** Hit 60 calls/minute limit during testing.

**Solution:** Added caching layer with 10-minute TTL.

**Lesson:** Implement caching early for external APIs, even in development.
```

## Installation

`ralph.py` has no runtime dependencies beyond Python 3.10+ stdlib. The `requirements-ralph.txt`
file contains only development tools (black, mypy).

```bash
# Optional: create a virtual environment
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows

# Optional: install dev tools for linting
python -m pip install -r requirements-ralph.txt
```

You also need Claude Code installed.

## Running Ralph

```bash
# Run 10 iterations with logging
python ralph.py --log-dir ~/.ralph/logs -n 10

# Run with verbose output (see all JSON)
python ralph.py -v -n 5

# Run indefinitely
python ralph.py --log-dir ~/.ralph/logs -n 1000000000
```

## Stopping and Feedback

### Graceful Stop

To stop Ralph after the current iteration completes:

```bash
touch STOP_RALPH
```

Ralph checks for this file at the start of each iteration and exits cleanly.

### Human Feedback

Create `FEEDBACK.md` to give Ralph instructions:

```markdown
The login button is broken on mobile. Fix it before continuing with other tasks.
```

Ralph reads `FEEDBACK.md` at the start of each iteration, addresses it, then deletes
the file. This lets you steer Ralph without stopping the loop.

### Automated Feedback

Add `scripts/fetch-feedback.py` to pull feedback from a production system:

```python
#!/usr/bin/env python3
"""Fetch user feedback from production and write to FEEDBACK.md."""
import sys
# Your logic here: check Slack, email, issue tracker, etc.
# Write to FEEDBACK.md if there's actionable feedback
# Exit 0 if feedback written, 1 if no new feedback
sys.exit(1)
```

Then run Ralph with:

```bash
python ralph.py --auto-fetch-feedback-script scripts/fetch-feedback.py
```

This creates a tight iteration loop: users report issues → Ralph fixes them → deploy.

## Cost

Running Ralph and paying API per token prices is expensive. Expect to spend:

* **Claude Sonnet**: ~$10 USD/hour
* **Claude Opus**: ~$15-25 USD/hour

The `ralph_optimizer.py` script analyzes your logs to find wasteful patterns:

```bash
python ralph_optimizer.py ~/.ralph/logs/ralph-abc123.log
```

It identifies redundant file reads, late test runs, and other inefficiencies.

## Customizing RALPH.md

The prompt in `RALPH.md` controls Ralph's behavior. Small wording changes can have
big effects.

### Strong vs Weak Language

Weak language like "you can mark it in_progress if you want" leads to inconsistent
behavior. Strong language like "MUST mark it in_progress IMMEDIATELY" is more reliable.

Compare:

```markdown
# Weak (unreliable)
2. **Pick a task** - Choose a task from TASKS.jsonl. You can mark it in_progress.

# Strong (reliable)
2. **Claim the task** - Pick ONE task from TASKS.jsonl and IMMEDIATELY mark it "in_progress".
```

### Testing Prompt Changes

Use `eval_ralph.py` to test how prompt variations affect behavior:

```bash
python -m unittest eval_ralph -v
```

The eval suite runs Ralph with different prompts and verifies expected behaviors like:

* First mutation is always to TASKS.jsonl (claiming before working)
* FEEDBACK.md is read before TASKS.jsonl
* Ralph resists misdirection in task titles

You can add your own test cases by subclassing `RalphEvalTestCase` and overriding
`ralph_md_content` with your prompt variation.

## Next level

I haven't run any of these personally, but they seem to be at the cutting edge of
scaling Ralph techniques up.

* [https://github.com/ghuntley/loom](https://github.com/ghuntley/loom)
* [https://github.com/steveyegge/gastown](https://github.com/steveyegge/gastown)
* [https://github.com/mikeyobrien/ralph-orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)

## Additional references

* [obra/superpowers](https://github.com/obra/superpowers) is really popular these days. I didn't care for
  `/superpowers:brainstorm` but `/superpowers:writing-skills` was great. When I wanted
  to write a new [Skill](https://t.pottingers.us/blog/skills-vs-mcp/) it included its
  own [evals](https://newsletter.pragmaticengineer.com/p/evals) to make sure the Skill
  was an improvement over not having the Skill.
* [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) is also a popular Claude "plugin"
  (it's not an official [plugin](https://code.claude.com/docs/en/plugins)). I liked
  the plan and research produced by `/gsd:new-project`. It made an incorrect
  assumption about my git monorepo and I didn't care for the rigidity that comes
  with `/gsd:progress`, `/gsd:discuss-phase`, `/gsd:plan-phase`, `/gsd:execute-phase`,
  `/gsd:verify-work`.
* [https://cursor.com/blog/scaling-agents](https://cursor.com/blog/scaling-agents)
  sounds like Cursor has built their own internal Ralph harness that can compete with
  loom, gastown and ralph-orchestrator.
* [https://github.com/steveyegge/beads](https://github.com/steveyegge/beads) sounds
  like an improvement over `TASKS.jsonl`
