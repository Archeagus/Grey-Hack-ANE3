# ANE3 — Episode 11.2

## Mission Runner / LibScan

### Component Contract

**Mission Runner coordinates.**  
**LibScan researches.**

Mission Runner evaluates whether mission work is prepared enough to hand to ExpScan.

LibScan owns vulnerability research for libraries discovered through MapScan.

### Readiness Flow

`MissionScan -> MapScan -> LibScan -> Mission Runner -> ExpScan`

### Record-Ready Builds

- `mission-runner v0.1.3 bld1003`
- `libscan1 v0.1.11 bld1011`
- `lib_functions v0.1.6 bld1006`

### Key Files

`/ANE3/DB/mission-queue.db`  
`/ANE3/DB/targets.db`  
`/ANE3/DB/services.db`  
`/ANE3/DB/libraries.db`  
`/ANE3/DB/exploits.db`  
`/ANE3/MISSIONS/active-mission.db`

### Core Rule

> Mission Runner does not prepare the work.  
> It determines whether the preparation is sufficient to proceed.
































---

# RECORDING CONTROL — 11.2
## Keep Below Viewport

### Segment Goal

Demonstrate how Mission Runner evaluates preparation state, prove that incomplete work is gated instead of executed, let LibScan complete the missing research, and then establish a clean READY handoff for ExpScan.

**Viewer-facing runtime target:** ~10 minutes  
**Commentary style:** organic / screen-driven  
**Runtime is diagnostic, not a hard production target.**

---

## Recording Target 1 — Introduce Mission Runner's Revised Role

### Goal

Re-establish Mission Runner as a coordinator rather than another preparation or execution tool.

### Capture

- `mission-runner v0.1.3 bld1003` is the record-ready build.
- Mission Runner now evaluates readiness from existing ANE3 state.
- It no longer:
  - seeds MapScan targets,
  - offers manual MapScan target submission,
  - writes ExpScan execution fields,
  - tells the operator to blindly run a fixed MapScan -> LibScan -> ExpScan sequence.

### Core Explanation

Mission Runner asks:

> “Is this mission actually ready for attended execution?”

It does not answer that question by doing MapScan or LibScan's work itself.

### Done When

The viewer understands that Runner is now a **readiness gate and coordination layer**, not a preparation engine.

---

## Recording Target 2 — Show the Readiness Dashboard

### Trigger

`mission-runner`

or, if useful for a clean first view:

`mission-runner --dashboard`

### Goal

Make ANE3 preparation state visible at the mission level.

### Capture

Readiness categories:

- `READY_FOR_EXPSCAN`
- `TARGET_QUEUED`
- `MAPPING`
- `WAITING_FOR_RESEARCH`
- `BLOCKED`
- `UNPREPARED`

Use the actual counts from the recording save.

### Explain

The dashboard is not another scheduler.

It is reading evidence already produced by MissionScan, MapScan, and LibScan and translating that evidence into an operator-facing readiness state.

### Done When

The viewer can see that “available mission” and “ready mission” are not the same thing.

---

## Recording Target 3 — Inspect the Mission That Has Advanced Farthest

### Trigger

Mission Runner:

`[2] List available missions`

Then inspect the most advanced mission, expected to be `M-000007` if the restored recording save follows the smoke-tested path.

### Goal

Connect the aggregate dashboard to one concrete mission.

### Capture

For the selected mission, show as useful:

- mission ID / type
- objective
- public target
- LAN target
- MapScan target record
- mapped open services
- LibScan research state
- exploit candidate count
- Runner readiness

### Recording-Save Rule

Do not force `M-000007` if another mission is actually farther along.

Follow the live ANE3 state.

### Done When

The audience can see **why** Runner assigned the readiness state it did.

---

## Recording Target 4 — Prove Mission Runner Is Fail-Closed

### Trigger

Attempt to select a mission that is not `READY_FOR_EXPSCAN`.

Expected case:

`WAITING_FOR_RESEARCH`

### Goal

Demonstrate that Runner will let us inspect incomplete work but will not activate it for ExpScan.

### Capture

- Operator attempts selection.
- Runner refuses the handoff.
- `active-mission.db` remains unchanged / no new active mission is established.
- The refusal is tied to the actual readiness evidence rather than an arbitrary hardcoded restriction.

### Core Principle

> We can inspect unprepared work.  
> We cannot accidentally execute it through Mission Runner.

### Done When

The fail-closed control boundary has been demonstrated on screen.

---

## Recording Target 5 — Let LibScan Complete the Missing Preparation

### Trigger

`libscan1 --once`

### Goal

Show that Runner's readiness state identifies a real missing dependency rather than merely displaying a label.

### Capture

- LibScan selects an unresearched library from ANE3 knowledge.
- It reconnects to the recorded sample.
- It researches vulnerability zones / unsafe checks.
- It persists results into `exploits.db`.
- LibScan does not execute an exploit.

### Architectural Point

Runner did not “call” LibScan or take over its work.

Runner exposed:

`WAITING_FOR_RESEARCH`

LibScan independently performs the research it owns.

### Outcome Possibilities

After research, the mission may become:

`READY_FOR_EXPSCAN`

or:

`BLOCKED`

if research completes with no usable candidate.

Follow the actual result.

### Done When

The audience has seen a readiness dependency change because the component that owns that dependency actually did its job.

---

## Recording Target 6 — Re-Evaluate and Establish the Active Mission

### Trigger

Return to:

`mission-runner`

Then list / inspect the mission again.

### Goal

Show the readiness transition and, if ready, establish the attended ExpScan handoff.

### Ideal Recording Path

`WAITING_FOR_RESEARCH`
`-> LibScan research`
`-> READY_FOR_EXPSCAN`

If the live save produces `BLOCKED`, explain that result honestly and select another mission only if doing so remains natural and within scope.

### Capture

When a mission is READY:

- Runner reports `READY_FOR_EXPSCAN`.
- Operator deliberately selects it.
- `active-mission.db` is written as selected context.
- Runner does **not** write ExpScan execution results.
- The active handoff contains readiness/context, not a claim that execution has happened.

### Core Handoff

`prepared mission`
`-> operator selection`
`-> active-mission.db`
`-> ExpScan`

### Done When

A READY mission has been intentionally established as the active execution context.

---

# 11.2 Exit / 11.3 Setup

### Segment Takeaway

Mission Runner now gives ANE3 an attended control point between preparation and execution.

It can show incomplete work, explain why it is incomplete, refuse premature execution, and establish a handoff only after the required preparation exists.

### Transition Goal

Organic transition concept:

> The mission is mapped. The library research exists. Mission Runner says the work is ready. Now we can finally hand it to the component that actually executes: ExpScan.

**Next:** `11.3 — ExpScan`

---

# Recording Guardrails

- Use the actual state left by the 11.1 recording.
- Do not force the smoke-test counts or mission ordering.
- `M-000007` is the expected through-line, not a scripted requirement.
- Do not describe Mission Runner as a scheduler for MapScan or LibScan.
- Do not let Runner write or imply ExpScan execution state.
- Do not execute an exploit during 11.2.
- Do not start solving the LAN/router-pivot problem.
- If LibScan yields `BLOCKED` instead of `READY_FOR_EXPSCAN`, treat that as valid system behavior rather than a failed recording.
- Keep source walkthroughs focused on the readiness delta; do not re-explain entire established components.
- A clean final section significantly below ~5 minutes or above ~15 minutes should trigger a presentation-boundary review after recording, not forced pacing during recording.
