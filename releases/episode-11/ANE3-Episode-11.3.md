# ANE3 — Episode 11.3

## ExpScan

### Component Contract

**ExpScan executes.**

ExpScan consumes prepared mission context, correlates MapScan service data with LibScan exploit knowledge, and performs exactly one attended exploit attempt per selection.

It does not blindly sweep candidates, prepare targets, research libraries, submit mission completion, or solve LAN traversal automatically.

### Execution Flow

`Mission Runner -> active-mission.db -> ExpScan -> one attended exploit -> persisted result`

### Record-Ready Builds

- `expscan2 v0.1.3 bld1003`
- `mission-runner v0.1.3 bld1003`
- `lib_functions v0.1.6 bld1006`

### Key Files

`/ANE3/MISSIONS/active-mission.db`  
`/ANE3/DB/services.db`  
`/ANE3/DB/exploits.db`  
`/ANE3/DB/exploit-results.db`  
`/ANE3/MISSIONS/<mission_id>.db`

### Core Rule

> Prepared intelligence narrows the choices.  
> ExpScan still executes only one deliberate candidate at a time.
































---

# RECORDING CONTROL — 11.3
## Keep Below Viewport

### Segment Goal

Demonstrate the final attended execution stage of the current ANE3 workflow: consume a READY mission handoff, correlate prepared intelligence into exploit candidates, execute one candidate, persist the result, and follow the resulting access far enough to discover the next operational barrier.

**Viewer-facing runtime target:** ~10 minutes  
**Commentary style:** organic / screen-driven  
**Runtime is diagnostic, not a hard production target.**

---

## Recording Target 1 — Establish the ExpScan Execution Boundary

### Goal

Explain what ExpScan is responsible for now that MissionScan, MapScan, LibScan, and Mission Runner have already done their work.

### Capture

- `expscan2 v0.1.3 bld1003` is the record-ready build.
- ExpScan reads the active Mission Runner context.
- It correlates open MapScan services, matching library identity, and usable LibScan exploit rows.
- It presents candidates for attended selection.
- It executes one candidate only.

### Boundaries to Reinforce

ExpScan does **not** sweep every exploit automatically, perform MapScan freshness decisions, perform LibScan research, auto-launch bash, delete logs, submit the mission, or pivot automatically through the LAN.

### Done When

The viewer understands that ExpScan is the **execution layer**, not another discovery or preparation layer.

---

## Recording Target 2 — Launch Through the READY Mission Context

### Trigger

`expscan2`

### Goal

Demonstrate the Mission Runner -> ExpScan contract.

### Capture

- mission ID
- mission type
- objective
- public IP
- LAN target
- `Runner state: READY_FOR_EXPSCAN`

### Explain

ExpScan is not guessing which mission to attack. The operator already selected a prepared mission through Mission Runner, and ExpScan is consuming that attended handoff.

### Guardrail

If Runner context is unexpectedly not READY, do not bypass the readiness gate just to preserve the recording.

### Done When

The audience can see that execution begins from an intentional, prepared mission context.

---

## Recording Target 3 — Explain the Candidate Correlation

### Goal

Show how the candidate list represents combined ANE3 knowledge rather than blind exploit probing.

### Capture

- port
- library/version
- vulnerability zone
- unsafe check
- prior history/status

### Core Relationship

`open MapScan service`
`+ matching library`
`+ LibScan exploit knowledge`
`= ExpScan candidate`

### History

If prior history exists, explain it as accumulated exploit knowledge. If the reverted save shows `unknown/untested`, that is equally useful.

### Done When

The viewer understands why these candidates exist and why ExpScan does not automatically execute all of them.

---

## Recording Target 4 — Execute Exactly One Candidate and Persist the Result

### Trigger

Select one candidate deliberately.

### Goal

Demonstrate the core ExpScan behavior under real mission context.

### Capture

As available:

- selected target port
- library
- zone
- unsafe check
- one overflow execution
- result classification
- privilege classification
- optional operator notes

### Persistence Order

`execute`
`-> classify`
`-> persist`
`-> interactive Shell handoff, if applicable`

The result should be written before ExpScan gives terminal control to a returned Shell.

### Possible Results

The recording may produce `SHELL`, `PASSWORD`, `COMPUTER`, `FILE`, `NO_RESULT`, or another native result type.

Follow the actual result. Do not force the smoke-tested Shell candidate or memory zone.

### Done When

Exactly one attended exploit attempt has been executed and its result has been recorded.

---

## Recording Target 5 — Follow the Foothold Far Enough to Discover the Next Barrier

### Goal

If the exploit yields useful access, investigate far enough to determine whether the mission objective is actually reachable.

### Expected Through-Line

For the current file-retrieval mission:

- public target: `219.109.10.103`
- objective LAN host: `192.168.2.4`
- objective file: `/home/Coos/sales_report.bin`

Follow the recording save's actual state.

### If a Shell Is Returned

Capture as useful:

- where the Shell landed,
- privilege level,
- visible LAN context,
- whether the objective host is directly reachable,
- whether exposed services exist on the objective host.

### Core Narrative

> A successful exploit is not the same thing as successful objective access.

The important distinction is:

`FOOTHOLD`

versus:

`OBJECTIVE REACHABILITY`

### Episode Boundary

If we rediscover the closed-port / LAN-topology barrier, **stop there**.

Do not begin solving router exploitation, pivot automation, LAN traversal, or multi-hop execution.

### Done When

The recording has identified what prevents the current ANE3 workflow from completing the mission after a successful foothold.

---

# 11.3 Exit / 11.X Setup

### Segment Takeaway

The current ANE3 chain can now carry work from mission discovery through preparation, readiness gating, and one attended exploit execution.

If the objective still cannot be reached, that is not a failure of ExpScan. It reveals the next problem domain:

**network topology and pivoting.**

### Outro Preparation

Do not script the final outro before the session result is known.

Build `11.X` after recording is complete and SRT/transcript review confirms what happened on screen.

Potential final progression:

`DISCOVER`
`-> PREPARE`
`-> SELECT`
`-> EXECUTE`
`-> DISCOVER THE NEXT BARRIER`

---

# Recording Guardrails

- Use the active mission state produced by the 11.2 recording.
- Do not force the smoke-tested candidate number, memory zone, unsafe value, or exact exploit result.
- Execute exactly one candidate per attended selection.
- If the selected candidate yields no useful result, follow the live result honestly rather than manufacturing a Shell.
- Do not re-run candidates merely to reproduce the smoke-test outcome unless that becomes a natural continuation of the recording.
- Persistence happens before interactive Shell handoff.
- Returning from a remote Shell is not evidence that the exploit failed.
- Do not claim objective completion merely because a Shell was acquired.
- Do not solve router/pivot architecture during Episode 11.
- Keep code review focused on the ExpScan execution delta.
- A clean final section significantly below ~5 minutes or above ~15 minutes should trigger a presentation-boundary review after recording, not forced pacing during recording.
