# ANE3 — Episode 11.1

## MissionScan / MapScan

### Component Contract

**MissionScan understands the work.**  
**MapScan maps the target.**

MissionScan identifies actionable public mission targets and reconciles them into ANE3 preparation.

MapScan remains responsible for deciding whether a target should be scanned, reused, refreshed, or retried.

### Preparation Flow

`MISSION -> MissionScan -> target-seeds.db -> MapScan`

### Record-Ready Builds

- `missionscan1 v0.1.3 bld1003`
- `mapscan3 v0.1.14 bld1014`
- `lib_functions v0.1.6 bld1006`

### Key Files

`/ANE3/MISSIONS/manifest.db`  
`/ANE3/DB/mission-queue.db`  
`/ANE3/DB/target-seeds.db`  
`/ANE3/DB/targets.db`  
`/ANE3/DB/services.db`

### Core Rule

> MissionScan says **ANE3 has a reason to care** about a target.  
> MapScan decides **what preparation that target requires**.
































---

# RECORDING CONTROL — 11.1
## Keep Below Viewport

### Segment Goal

Demonstrate how an actionable mission target enters ANE3 preparation, prove that MissionScan and MapScan retain separate ownership, and show that the mission/target relationship remains durable after MapScan processes the work.

**Target runtime:** ~10 minutes  
**Commentary style:** organic / screen-driven  
**Do not treat timings as hard edit points.**

---

## Recording Target 1 — Establish the MissionScan Integration Change
**Approx. 1:30**

### Goal

Give the audience a reason for revisiting MissionScan without turning the section into another full source-code walkthrough.

### Capture

- The save has been reverted to the pre-smoke-test state.
- `missionscan1 v0.1.3 bld1003` is the record-ready build.
- This revision adds proactive reconciliation of actionable public mission targets into ANE3 preparation.
- MissionScan has gained an **integration responsibility**, not a mapping responsibility.

### Useful implementation focus

Show only the code necessary to explain:

- post-queue reconciliation,
- durable mission/public-target identity,
- creation of `MISSION` target seeds.

Avoid rereading unrelated MissionScan parsing/mail-intake code already established in prior episodes.

### Done When

The viewer understands **why MissionScan changed** and does not leave thinking MissionScan now performs network mapping.

---

## Recording Target 2 — Show MissionScan Creating Actionable Target Work
**Approx. 2:00**

### Trigger

`missionscan1 --reparse`

### Goal

Demonstrate the new integration behavior in the restored save.

### Capture

- Mission Contract threads synchronize.
- Mission queue is rebuilt.
- Available missions with valid public IPs are reconciled.
- Nonzero `Targets seeded` appears on the first preparation pass.
- Briefly inspect `target-seeds.db` if useful.

### Relationship to expose

`<public_ip>|MISSION|<mission_id>|NORMAL|QUEUED|...`

Important fields:

- public IP
- `source=MISSION`
- `source_ref=<mission_id>`
- MapScan queue state

### Do Not Over-Promise

Smoke testing produced six seeded targets, but use the **actual recording-save result**.

### Done When

The audience has seen an actionable mission become durable preparation work for ANE3.

---

## Recording Target 3 — Establish the MissionScan / MapScan Ownership Boundary
**Approx. 1:00**

### Goal

Explain why MissionScan creating a MapScan seed does not blur component ownership.

### Core Explanation

MissionScan determines:

> “ANE3 has a reason to care about this public target.”

MapScan determines:

- scan,
- reuse,
- refresh,
- retry.

MissionScan should not make MapScan freshness or scan-policy decisions.

### Screen Support

Prefer explaining this while looking at the seed / databases already produced rather than stopping for another large code walkthrough.

### Done When

A viewer could explain why MissionScan writes target preparation work without saying:

> “MissionScan now scans mission targets.”

---

## Recording Target 4 — Demonstrate Reconciliation / Idempotence
**Approx. 1:15**

### Trigger

`missionscan1`

### Goal

Show that repeating mission synchronization does not create duplicate MapScan work.

### Expected Pattern

Smoke testing produced:

`Targets seeded: 0`  
`Targets reconciled: 6`

Use the actual recording output.

### Explain

“Reconciled” means MissionScan already recognizes the durable mission/public-target relationship.

The identity is not merely:

`this row is currently QUEUED`

It is the relationship between:

`MISSION + mission_id + public_ip`

### Done When

The audience understands MissionScan can run repeatedly without polluting `target-seeds.db`.

---

## Recording Target 5 — Let MapScan Consume the Work on Its Own Terms
**Approx. 2:15**

### Trigger

`mapscan3 --once`

### Goal

Show the second half of the integration contract.

### Capture

- MapScan discovers queued mission-derived work through its normal scheduler.
- MapScan—not MissionScan—decides what that target currently requires.
- It may perform an authoritative scan or reuse a valid snapshot depending on state.
- Mapping results remain MapScan-owned data.

### Architectural Flow

`mission exists`
`-> MissionScan creates reason-to-care`
`-> target-seeds.db`
`-> MapScan scheduler`
`-> MapScan-owned preparation decision`

### Commentary Guardrail

Do not say MissionScan “ordered a scan.”

MissionScan requested preparation.  
MapScan interpreted and processed that request according to MapScan policy.

### Done When

The audience has visibly crossed from **mission understanding** into **target mapping** while the ownership boundary remains clear.

---

## Recording Target 6 — Prove the Relationship Survives MapScan Processing
**Approx. 2:00 including transition**

### Trigger

`missionscan1`

### Goal

Demonstrate that MissionScan reconciliation is based on durable mission/target identity rather than MapScan's transient queue status.

### Capture

After MapScan changes one seed to a processed state:

- MissionScan runs again.
- `Targets seeded` remains zero for existing relationships.
- The mission target is still recognized as reconciled.
- No duplicate mission request is created merely because MapScan changed the operational state.

### Core Proof

`MapScan changes operational state`
`-> MissionScan reruns`
`-> no duplicate seed`
`-> relationship remains reconciled`

### Done When

We have live evidence that MissionScan and MapScan can independently update their own state without producing duplicate work.

---

# 11.1 Exit / 11.2 Setup

### Segment Takeaway

MissionScan now gets actionable mission targets into the preparation system, while MapScan retains ownership of how those targets are mapped and maintained.

### Transition Goal

End with the preparation queue in a mixed state suitable for Mission Runner.

Organic transition concept:

> We now know what work exists, and MapScan has started preparing the target intelligence. The next question is which of these missions is actually ready to act on.

**Next:** `11.2 — Mission Runner / LibScan`

---

# Recording Guardrails

- Record-ready builds have already passed smoke testing; do not imply this is first-ever execution.
- Follow the restored save's actual counts and generated data.
- Do not force the smoke-test result of exactly six mission targets if the recording save differs.
- Do not advance LibScan in 11.1.
- Do not select a mission in Mission Runner during 11.1.
- Do not enter ExpScan during 11.1.
- Avoid a second full MissionScan source walkthrough; focus on the integration delta.
- MapScan scan/reuse/refresh policy remains MapScan-owned.
- If an unexpected result appears, investigate only far enough to keep the 11.1 architectural story accurate; do not manufacture the expected state.
