# Cursor Cloud Agents, Automations & BugBot

Reference guide for teams evaluating Cursor Cloud Agents — setup, comparisons with local runs, Automations, BugBot, and cost implications.

---

## Quick Links

| Resource | URL |
|----------|-----|
| Cloud Agents dashboard | https://cursor.com/dashboard/cloud-agents |
| Agent status (mobile-friendly) | https://cursor.com/agents |
| Cloud (mobile access) | https://cursor.com/cloud |
| GitHub integration | https://cursor.com/dashboard/integrations |
| Automations | https://cursor.com/automations |
| Automation marketplace | https://cursor.com/marketplace |
| BugBot | https://cursor.com/dashboard/bugbot |
| Usage & billing | https://cursor.com/dashboard/usage |

**Org constraints:** Long-running agents are not enabled. No native mobile app — use the web URLs above.

---

## Cloud Agents — Getting Started

1. **Connect GitHub** at [integrations](https://cursor.com/dashboard/integrations).
2. Open [Cloud Agents](https://cursor.com/dashboard/cloud-agents) → **New** → select your repo. 
3. **Environment setup** — Cursor runs install commands and dependencies. When ready, save the environment; agents reuse it for future runs.
4. **Launch an agent** — **New agent** → confirm repo and branch → choose model and MCPs (if needed).
5. **Example prompt:**
   > Fix [bug description]. Acceptance criteria: [what "done" looks like]. Run tests. Open a PR against main.
6. **Close your laptop** — the agent runs on Cursor's servers. When finished, it opens a PR on GitHub. Track progress at [cursor.com/agents](https://cursor.com/agents).

### FAQ

| Question | Answer |
|----------|--------|
| Does Cloud run in Max Mode? | **Yes** — Max Mode is always on for Cloud Agents. |

---

## Local vs Cloud — Cost & Performance

### Benchmark (Composer 2.5, add test cases)

| | Local | Cloud |
|---|-------|-------|
| **Cost** | $0.94 | $0.69 |
| **Time** | 8m 14s | 3m 15s |
| **Tokens** | ~2.5M | ~1.6M |

Cloud was ~36% fewer tokens and ~2.5× faster. Blended rate in this test: Local ~$0.38/1M tokens, Cloud ~$0.43/1M tokens — Cloud had a higher per-token rate but lower total cost because it did less work.

### Why Local used more tokens

1. More agent loops over a longer session (8m vs 3m).
2. IDE may inject extra context each step (open files, codebase index, prior chat, rules, MCPs).
3. Slower feedback can lead to more exploratory tool calls.

### Why Cloud was leaner (this task)

1. Fresh VM with task-focused context (no prior chat history).
2. Environment may already be cached from earlier setup runs.
3. Shorter path to completion → fewer tool calls.

### Billing pools (same model, different $/token)

| Pool | Used by | Approx. rates |
|------|---------|---------------|
| **Auto + Composer** | Local Composer 2.5 (typical) | Input ~$1.25/1M, Output ~$6.00/1M |
| **API** | Cloud Composer 2.5 (Max Mode, no toggle) | Input ~$0.50/1M, Output ~$2.50/1M |

The Auto + Composer pool has a separate larger included allowance — it is not necessarily cheaper per token. Output-heavy Local runs can show higher $ cost per token. Verify which pool each run used on [usage dashboard](https://cursor.com/dashboard/usage).

### What this test proves (and does not)

| Proves | Does not prove |
|--------|----------------|
| For a small, bounded task, Cloud can be cheaper if it finishes faster with fewer tokens | Cloud is always cheaper |
| Max Mode always-on did not hurt here (narrow task, quick completion) | Large multi-file tasks, cold VM boot, or expensive models (e.g. Opus) won't flip the result |

### When Local is usually cheaper

1. Small task, you are at the keyboard, few agent turns.
2. Usage stays within the generous Auto + Composer included allowance.
3. No VM boot / clone / install overhead.

### When Cloud can win

1. Narrow task that finishes fast in a warm, pre-configured VM.
2. Local agent wanders or runs long (8+ minutes).
3. Local pulls in extra IDE context on every step.

### Fairer follow-up tests (optional)

1. Re-run the same prompt in a fresh Local chat (no prior history) and compare token counts.
2. Local with Max Mode ON + Composer 2.5 to match Cloud's API pool billing, then compare tokens only.
3. Repeat with a harder task (multi-file refactor) where Cloud's clone + Max Mode overhead shows up.

**Bottom line:** Cloud cost less in this test because it finished in 3 minutes with 1.6M tokens; Local took 8 minutes and 2.5M tokens, and may have been billed from the Auto + Composer pool at higher output rates.

**Rule of thumb:** At keyboard + small scope → **Local**. Away from keyboard + repeatable + parallel → **Cloud**.

---

## Automations

Automations run Cloud Agents automatically on triggers or schedules — set up once, run without manual launch.

**Pattern:** *When X happens (trigger), run this agent with these instructions (action).*

### Use cases

| Trigger | Action |
|---------|--------|
| PR opened | Review, add tests, fix lint |
| PR review complete | Apply fixes |
| Jira issue created | Triage, implement easy fixes |
| Schedule (nightly / weekly) | Security audit, performance audit, dependency bumps |
| Custom (Slack, webhook, etc.) | Any integration-specific workflow |

### Three components

1. **Trigger** — when to run (schedule, PR opened, Slack message, webhook, …)
2. **Agent instructions** — prompt (same style as a normal agent task)
3. **Tools** — allowed actions (open PR, comment on PR, post to Slack, MCP, …)

**Cost:** Same as Cloud Agents — each automation run initiates a Cloud Agent.

**Getting started:** Use marketplace templates at [cursor.com/marketplace](https://cursor.com/marketplace) and tune for your repo.

---

## BugBot

Automatic PR reviewer for GitHub-connected repos. Leaves inline comments with explanations and suggested fixes.

1. Built-in review instructions + optional custom standards.
2. **Custom standards:** add `.cursor/BUGBOT.md` in the repo (tests, style, security, patterns, …).
3. **Priority order:** Team rules → Repository rules → `.cursor/BUGBOT.md` → User rules (personal Cursor rules).

### BugBot + Cloud Autofix loop

PR opened → Bugbot reviews → findings → Cloud Agent fixes → new PR/branch → human merge. Good for an automated first pass, not fully autonomous merges.

---

## Best Use Cases

### For Cloud

Pick Cloud when:

1. You are not at the keyboard (AFK, mobile, laptop off).
2. Work is repeatable (Automations on triggers/schedules).
3. Work can run in parallel (many stories → many PRs).
4. Tasks need a real environment (build, test, verify in VM).
5. Paired with BugBot (find → Autofix → PR loop).

| Use case | Why |
|----------|-----|
| **AFK / "close the laptop"** — bug fixes with clear acceptance criteria, test coverage gaps, small scoped refactors, docs tied to code | Runs on Cursor's VM; manage from phone at [cursor.com/agents](https://cursor.com/agents) |
| **Automations** — PR review, CI failure on main, Jira triage, nightly scans, Slack #bugs, Sentry/PagerDuty alerts | Define once; agents run 24/7 without clicking Launch |
| **BugBot autofix loop** — review → fix → PR | Automated first pass; human merges non-trivial changes |
| **Parallel fan-out** — N agents on N branches (e.g. 5 Jira stories → 5 PRs) | Local = one machine; Cloud = batch throughput |

### For Local

Pick Local when actively coding, iterating fast, or the task is tiny and uncommitted:

1. Quick one-liner fix while actively coding
2. Exploring unfamiliar code (interactive steering)
3. Sensitive WIP not pushed yet (Cloud needs GitHub)
4. Tight budget, tiny task (avoids VM overhead and PR ceremony)
5. Need to switch models mid-task (Cloud is less flexible)
