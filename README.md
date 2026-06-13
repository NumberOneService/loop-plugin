# `/loop` — a bounded multi-agent collaboration loop for Claude Cowork

`/loop <goal>` puts a team of **six specialist agents** to work on a goal and has them
iterate together — build, critique, fix, re-check — until a **verifier** confirms the goal
is actually finished (or a safety limit stops them). It runs on **Claude Fable 5**.

## The six agents

| Agent | Role |
|---|---|
| `planner` | Turns your goal into tasks + a testable "done" checklist (acceptance criteria) |
| `researcher` | Gathers facts and sources, separates solid facts from guesses |
| `builder` | Produces and updates the actual deliverable |
| `critic` | Red-teams the work for errors, bad assumptions, and edge cases |
| `integrator` | Merges the critique into a clear change list for the next build |
| `verifier` | Checks the work against the checklist and declares DONE or NOT-DONE |

The loop repeats `builder → critic → integrator → verifier` each round. It stops when the
verifier says **DONE**, when it hits the round limit, when it stops making progress, or when
it needs your approval for something risky.

---

## How to install it

### Option A — install as a plugin (recommended, full and reusable)

1. Put this whole `loop-plugin/` folder in a place Cowork or Claude Code can read — either a
   **GitHub repo** or a **local folder** on your machine.
2. **In Claude Cowork (desktop app):** open **Customize → Plugins → Add marketplace**, point it
   at the repo/folder, then install the **`loop`** plugin.
3. **Or via the Claude Code command line:**
   ```bash
   # point at the folder that contains .claude-plugin/marketplace.json
   claude plugin marketplace add /full/path/to/loop-plugin
   claude plugin install loop@loop-marketplace
   ```
4. Start a new conversation. Type `/loop` and you should see it in the menu.

> Heads up: Cowork and Claude Code keep **separate** plugin lists, and Cowork's custom-plugin
> install has historically been a bit flaky (installs occasionally lost after an app restart or
> update). If `/loop` disappears, just re-install it. None of this affects your data — it is only
> about whether the command shows up.

### Option B — quick test, command only (cheapest way to try the workflow)

If you just want to try it fast: in Cowork open **Customize → Skills**, add a new skill, and paste
in the contents of `loop/commands/loop.md`. This gives you `/loop` **without** the six separate
agent files — so the orchestrator will play all six roles inside one conversation instead of
spawning real separate agents. It is cheaper and simpler, but the roles share one context (less
isolation). Good for a first try; use Option A for the real thing.

---

## How to use it

```
/loop write a 600-word blog post on signs your dishwasher needs professional repair, with sources
```

```
/loop build a small Python script that reads a CSV of Google Ads search terms and flags likely negative keywords. max 3 rounds
```

Tip: add `max N rounds` in your goal to cap the work (default is 5).

---

## ⚠️ Read before you run it: cost

This is powerful but **not cheap**, for two stacking reasons:

1. **Subagents multiply tokens.** A run that spawns several subagents can use **roughly 7×** the
   tokens of a normal single-agent chat, because each agent keeps its own context.
2. **Fable 5 is a premium model.** On Claude subscription plans it counts as **2× usage**, and it
   is only **free from June 9–22, 2026** — after that it draws on usage credits until capacity
   allows it back as a standard feature. (Pricing reference: $10 / million input tokens,
   $50 / million output tokens.)

Stacked together, a six-agent Fable loop can burn through usage **fast**. Two ways to cut cost:

- **Use Option B** (command only) for everyday tasks — no subagent fan-out.
- **Change the model** per agent (see below): run most agents on a cheaper model and reserve
  Fable for the hardest roles (e.g. `planner`, `critic`, `verifier`).

## Changing the model

Each agent file (`loop/agents/*.md`) has a `model:` line in its header:

```yaml
model: claude-fable-5
```

Change it to `sonnet`, `haiku`, or `inherit` (use the conversation's model) on any agent you want.
The `/loop` command already falls back to Sonnet automatically if a Fable agent declines a request
(Fable 5 has safety classifiers that can occasionally refuse).

## Adding or removing agents

Agents are just files. Drop a new `something.md` into `loop/agents/` (copy an existing one's
header format) and reference it in `loop/commands/loop.md`. Delete a file to remove that agent.

---

## What this loop is — and is not

- It is an **orchestrated, turn-taking** team: the agents hand work back and forth in rounds. It is
  **not** six models chatting simultaneously in real time.
- It is **bounded on purpose.** There is no true "run forever." The round limit, the no-progress
  stop, and the verifier gate exist so it cannot quietly loop and drain your credits.
- It **stops and asks you** before anything irreversible or external (sending, publishing, paying,
  deleting, changing live settings). It will not take those actions on its own.

## Things to double-check in your setup

- That your Cowork plan actually exposes **Fable 5** in the model picker (it rolls out in stages).
- That per-agent `model: claude-fable-5` is honored in your Cowork build. If an agent ignores the
  setting, it will run on your default model — still works, just not necessarily on Fable.

---

Built as a starting point. Edit the agent prompts and the loop in `loop/commands/loop.md` to match
how you actually like work done.
