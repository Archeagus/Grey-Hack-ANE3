# ANE3 Project-State Checkpoint

**Project:** Advanced Network Exploitation Engine 3.0 (ANE3)  
**Checkpoint date:** 2026-07-26 (Asia/Taipei)  
**Checkpoint position:** Episode 8 sections 8.0–8.3 recorded; Episode 8.4 design complete enough to begin implementation, but Mission Organizer production code has not yet been drafted in this chat.  
**Authority:** This document consolidates the ANE3 decisions, observations, source files, tests, and recording status established in the conversation through the successful mission-mail probe and the subsequent Mission Organizer schema discussion.

> This checkpoint does not fill gaps by inference. Anything not established strongly enough is marked **VERIFY AGAINST SOURCE** or **UNRESOLVED**.

---

## Status Legend

| Status | Meaning |
|---|---|
| **LOCKED** | An architectural, accountability, naming, or workflow decision explicitly accepted as the current ANE3 direction. |
| **IMPLEMENTED** | Present in the current versioned source or deployed/tested in Grey Hack. |
| **OBSERVED** | Demonstrated by a live Grey Hack test or actual output, but not necessarily generalized beyond what was observed. |
| **PLANNED** | Intended for the current Episode 8 implementation sequence but not yet implemented. |
| **BACKLOG** | Deferred intentionally to Episode 9 or later. |
| **UNRESOLVED** | A material decision, contract, defect, or risk remains open. |
| **VERIFY AGAINST SOURCE** | The conversation does not establish enough evidence to state the item authoritatively. Check the authoritative external source, current in-game source, or recorded episode outline before relying on it. |

---

# 1. Current Architectural Model and Design Philosophy

## 1.1 Core model

**LOCKED**

> **MapScan maps. LibScan researches. ExpScan executes. Heartbeat observes. Mission Runner coordinates.**

Mission Organizer provides the mission-intake and mission-record layer that feeds Mission Runner.

ANE3 is a collection of specialized components with:

- one clear responsibility per component;
- shared runtime primitives rather than duplicated infrastructure;
- persistent operational data;
- lease-based state reporting;
- explicit ownership of files and responsibilities;
- version/build accountability;
- external authoritative source files and controlled in-game deployment;
- an operator-visible workflow rather than indiscriminate autonomous exploitation.

## 1.2 Separation of responsibility

**LOCKED**

No component should absorb the internal responsibility of another component merely because it can technically access the same data.

- MapScan does not research or execute exploits.
- LibScan does not map targets or execute exploits.
- ExpScan does not map or research.
- Heartbeat does not launch, stop, kill, restart, or impersonate worker components.
- Mission Runner coordinates; it does not become the implementation of MapScan, LibScan, ExpScan, Heartbeat, or Mission Organizer.
- Mission Organizer owns mission intake and normalized mission records; it does not remain connected to mail or become the interactive execution console.
- Shared libraries provide reusable plumbing only; they do not own component-specific behavior or databases.

## 1.3 External source authority and deployment boundary

**LOCKED**

The authoritative development source lives outside Grey Hack:

- in a VS Code workspace;
- on a local network share;
- using the **Greybel VS** extension;
- with every externally preserved iteration carrying version and build information in its filename.

Examples:

```text
heartbeat-v0.1.11-bld1011.src
mapscan3-v0.1.11-bld1011.src
libscan1-v0.1.11-bld1011.src
lib_functions-v0.1.6-bld1006.src
```

Current chat artifacts include `-record-ready` in some filenames, but the long-term naming rule is the version/build suffix, not the reuse of a generic filename.

**LOCKED**

Inside Grey Hack, deployment uses canonical filenames and paths. Transfer is currently manual copy-and-paste. This is an intentional deployment boundary, not an accidental limitation.

**BACKLOG**

GitHub is expected to become an external development and release authority later, but it will not eliminate the controlled in-game deployment boundary.

## 1.4 Root cleanliness and operational grouping

**LOCKED**

The `/ANE3` root contains purpose-specific subfolders rather than loose runtime files.

Operational purpose takes precedence over file extension when deciding where a file belongs. Mission-related files may use different extensions while remaining grouped under `MISSIONS`.

## 1.5 Persistent versus one-shot behavior

**LOCKED**

- Heartbeat is persistent.
- MapScan 3 can run persistently or once.
- LibScan 1 can run persistently or once.
- ExpScan 2 is interactive/attended and not intended to be an indiscriminate background exploit sweeper.
- Mission Runner is intended to be a persistent operational console/dashboard.
- Mission Organizer is one-shot: one process, one mail login, one fetch, required reads, persistence, then exit.

---

# 2. Components and Exact Responsibilities

## 2.1 Shared library: `lib_functions`

### Responsibility

**LOCKED**

`lib_functions` contains lean ANE3 runtime plumbing only.

It must not contain:

- scanner behavior;
- database schema ownership;
- exploit logic;
- target selection;
- manifest ownership;
- rigid component-specific file wrappers.

### Current implementation

**IMPLEMENTED**

Current authoritative chat artifact:

```text
lib_functions-v0.1.6-bld1006-record-ready.src
```

Canonical in-game source:

```text
/home/Archeagus/ANE3/SRC/LIB/lib_functions.src
```

Current metadata:

```text
ANE3_LIB_NAME = "lib_functions"
ANE3_LIB_VERSION = "0.1.6"
ANE3_LIB_LAST_VERSION = "0.1.5"
ANE3_LIB_LAST_BLD = 1005
ANE3_LIB_BLD = 1006
```

Implemented shared primitives include:

```text
version_score()
calc_bld()
ane_root()
ane_path()
init_folder()
init_file()
safe_content()
append_line()
contains_line()
append_unique()
split_fields()
get_field()
has_flag()
restart_command()
color_text()
print_log()
log_event()
state_expires()
log_state()
shutdown()
manifest_line()
manifest_version()
manifest_bld()
check_runtime_policy()
update_required_exit()
activity_color()
bld_color()
```

### Accountability

**LOCKED**

The shared library does not own the manifest. Heartbeat owns and refreshes the manifest. Shared helpers only read a caller-provided manifest file.

### Compile-time behavior

**OBSERVED / IMPLEMENTED**

Importing scripts compile against an embedded copy of the shared library. Updating `lib_functions.src` does not update already deployed commands dynamically. Dependent commands must be redeployed to embed the new library version.

---

## 2.2 MapScan 3

### Responsibility

**LOCKED**

MapScan 3 is the persistent target/service mapper.

> **MapScan maps.**

It:

- accepts or discovers target IPs;
- resolves routers and ports;
- records target and service information;
- identifies library/version combinations exposed by services;
- queues new library/version combinations for LibScan;
- writes target reports;
- reports its state to Heartbeat.

It does not:

- research vulnerability zones as its primary responsibility;
- execute overflows;
- classify exploit outcomes;
- coordinate the mission workflow.

### Current implementation

**IMPLEMENTED**

Current authoritative chat artifact:

```text
mapscan3-v0.1.11-bld1011-record-ready.src
```

Canonical command:

```text
mapscan3
```

Canonical in-game source:

```text
/home/Archeagus/ANE3/SRC/mapscan3.src
```

Current metadata:

```text
SCRIPT_NAME = "mapscan3"
SCRIPT_COMMAND = "mapscan3"
SCRIPT_VERSION = "0.1.11"
SCRIPT_LAST_VERSION = "0.1.10"
SCRIPT_LAST_BLD = 1010
SCRIPT_BLD = 1011
```

Compile-time import:

```greyscript
import_code("/home/Archeagus/ANE3/SRC/LIB/lib_functions.src")
```

Current state identity:

```text
scanner_name = "mapscan"
state_file = /ANE3/STATES/state_mapscan.txt
```

Current run modes:

```text
mapscan3
mapscan3 --once
mapscan3 <ip_address>
mapscan3 <ip_address> --once
```

### Current data ownership/use

**IMPLEMENTED**

MapScan currently initializes or consumes:

```text
/ANE3/DB/mission-queue.db
/ANE3/DB/target-seeds.db
/ANE3/DB/targets.db
/ANE3/DB/services.db
/ANE3/DB/libraries.db
/ANE3/TARGETS/<ip>.txt
```

Current initialized formats:

```text
mission-queue.db  # ip|source|status|notes
target-seeds.db   # ip|source|status|notes
targets.db        # ip|source|status|last_scan
services.db       # ip|port|state|service|lan_ip|lib_key|last_seen
libraries.db      # lib_key|lib_name|version|sample_ip|sample_port|status|last_seen
```

### Important current contract

**IMPLEMENTED**

MapScan’s `find_next_ip()` reads the first field of `/ANE3/DB/mission-queue.db` as an IP address.

This creates a material unresolved contract with the proposed Mission Runner-ready mission queue. See Sections 10 and 11.

---

## 2.3 LibScan 1

### Responsibility

**LOCKED**

LibScan 1 is the persistent library vulnerability researcher.

> **LibScan researches.**

It:

- reads library/version combinations discovered by MapScan;
- reconnects to a sample target/service;
- scans the library;
- identifies vulnerability zones and unsafe checks;
- writes exploit candidates;
- reports state to Heartbeat.

It does not:

- map target networks;
- execute exploit candidates;
- classify shells, passwords, or argument requirements.

### Current implementation

**IMPLEMENTED**

Current authoritative chat artifact:

```text
libscan1-v0.1.11-bld1011-record-ready.src
```

Canonical command:

```text
libscan1
```

Canonical in-game source:

```text
/home/Archeagus/ANE3/SRC/libscan1.src
```

Current metadata:

```text
SCRIPT_NAME = "libscan1"
SCRIPT_COMMAND = "libscan1"
SCRIPT_VERSION = "0.1.11"
SCRIPT_LAST_VERSION = "0.1.10"
SCRIPT_LAST_BLD = 1010
SCRIPT_BLD = 1011
```

Compile-time import:

```greyscript
import_code("/home/Archeagus/ANE3/SRC/LIB/lib_functions.src")
```

Current state identity:

```text
scanner_name = "libscan"
state_file = /ANE3/STATES/state_libscan.txt
```

Current run modes:

```text
libscan1
libscan1 --once
```

### Current data ownership/use

**IMPLEMENTED**

LibScan reads:

```text
/ANE3/DB/libraries.db
```

LibScan writes:

```text
/ANE3/DB/exploits.db
```

Current formats:

```text
libraries.db
# lib_key|lib_name|version|sample_ip|sample_port|status|last_seen

exploits.db
# lib_key|zone|unsafe_value|result_type|status|last_tested|notes
```

LibScan records candidates as untested. It does not run them.

---

## 2.4 ExpScan 2

### Responsibility

**LOCKED**

ExpScan 2 is the attended exploit execution and classification component.

> **ExpScan executes.**

It will:

- receive a selected mission/target or direct execution request;
- read service and exploit candidate data;
- present one exploit candidate at a time;
- let the operator decide whether to test it;
- run one overflow attempt;
- preserve meaningful native terminal output;
- let the operator classify the result;
- write normalized exploit results;
- report state to Heartbeat.

It will not:

- sweep every exploit automatically;
- clear/redraw the terminal during active native overflow output;
- automatically launch `/bin/bash` unless that behavior is explicitly selected;
- automatically clean logs unless explicitly selected;
- replace MapScan or LibScan.

### Planned data flow

**PLANNED**

Reads:

```text
/ANE3/DB/services.db
/ANE3/DB/exploits.db
```

Writes:

```text
/ANE3/DB/exploit-results.db
/ANE3/STATES/state_expscan.txt
```

### Planned result classifications

**PLANNED — exact labels not yet implemented**

Candidate classification vocabulary discussed:

```text
NO_RESULT
PASSWORD_FOUND
USER_SHELL
ROOT_SHELL
REQUIRES_ARGUMENT
REQUIRES_PASSWORD
REQUIRES_ACTIVE_USER
UNKNOWN_OUTPUT
SKIPPED
```

Earlier conceptual wording also included:

```text
active user
active root
password
argument format
unknown
no useful output
```

**UNRESOLVED**

The final canonical label set and serialized exploit-results schema have not been locked in implemented source.

### Terminal behavior

**LOCKED**

Grey Hack overflow execution may emit meaningful native terminal-only output that cannot be captured programmatically. ExpScan 2 therefore receives a dedicated visible terminal and uses append-only output during active testing.

### Current implementation state

**PLANNED**

Executable name:

```text
expscan2
```

Canonical source path:

```text
/home/Archeagus/ANE3/SRC/expscan2.src
```

No current production source/version/build is implemented in this checkpoint.

---

## 2.5 Heartbeat

### Responsibility

**LOCKED**

Heartbeat is the persistent ANE3 sentinel and system-health console.

> **Heartbeat observes.**

It:

- creates and owns the ANE3 source manifest;
- creates and owns the shared run log;
- creates and owns its own state file;
- reads worker state files without creating them;
- interprets leases and stale state;
- compares running embedded builds with source-manifest builds;
- reports health and update status.

It does not:

- launch scanners;
- stop scanners;
- kill scanners;
- restart scanners;
- create worker state files;
- claim a worker is initialized when it has never checked in.

### Current implementation

**IMPLEMENTED**

Current authoritative chat artifact:

```text
heartbeat-v0.1.11-bld1011-record-ready.src
```

Canonical command:

```text
heartbeat
```

Canonical in-game source:

```text
/home/Archeagus/ANE3/SRC/heartbeat.src
```

Current metadata:

```text
SCRIPT_NAME = "heartbeat"
SCRIPT_COMMAND = "heartbeat"
SCRIPT_VERSION = "0.1.11"
SCRIPT_LAST_VERSION = "0.1.10"
SCRIPT_LAST_BLD = 1010
SCRIPT_BLD = 1011
```

Compile-time import:

```greyscript
import_code("/home/Archeagus/ANE3/SRC/LIB/lib_functions.src")
```

Current identity and polling:

```text
scanner_name = "heartbeat"
state_grace = 3
manifest_refresh_interval = 3600
state_poll_interval = 5
```

### Current owned files

**IMPLEMENTED**

```text
/ANE3/MANIFEST/ane3.manifest
/ANE3/LOG/run.log
/ANE3/STATES/state_heartbeat.txt
```

### Current read-only worker state files

**IMPLEMENTED**

```text
/ANE3/STATES/state_mapscan.txt
/ANE3/STATES/state_libscan.txt
/ANE3/STATES/state_expscan.txt
```

### Current dashboard rows

**IMPLEMENTED**

The current `v0.1.11` dashboard displays:

```text
MapScan
LibScan
ExpScan
```

It does not yet display Mission Organizer or other future workers.

### Current manifest entries

**IMPLEMENTED**

Heartbeat currently refreshes entries for:

```text
lib_functions
mapscan3
libscan1
expscan2
mission-runner
heartbeat
```

**UNRESOLVED DEFECT / GAP**

`mission-organizer` is not currently included in the manifest refresh list, even though Mission Organizer is now a defined ANE3 component.

### Watcher-for-the-watcher model

**LOCKED ARCHITECTURE / NOT YET IMPLEMENTED**

- Heartbeat watches all operational ANE3 workers.
- Mission Runner has a dedicated monitor for Heartbeat.
- Mission Runner is therefore the watcher for the watcher.
- Heartbeat does not add itself as a row in its own dashboard.
- Heartbeat writes `/ANE3/STATES/state_heartbeat.txt`; Mission Runner evaluates that state independently.

### UI backlog

**BACKLOG**

1. Normal ANE3 report text should be explicitly white.
2. Grey Hack’s default teal should be reserved for native terminal-only output, including `nmap`-like or overflow output.
3. Consider restrained background-color emphasis for state/freshness after all core components pass functional testing.

---

## 2.6 Mission Runner

### Responsibility

**LOCKED**

Mission Runner is ANE3’s persistent operational console and workflow coordinator.

> **Mission Runner coordinates.**

It will:

- display mission and system status;
- request a one-shot Mission Organizer refresh;
- list actionable missions;
- show mission objectives and targets;
- queue or launch a selected mission;
- coordinate MapScan, LibScan, and ExpScan work;
- provide a dedicated monitor for Heartbeat;
- maintain an interactive lower section/dashboard;
- calculate dashboard totals from mission facts rather than duplicate mutable totals in every mission record.

It does not:

- become the mission-mail parser;
- perform MapScan’s mapping;
- perform LibScan’s research;
- perform ExpScan’s exploit execution;
- become Heartbeat.

### Planned startup menu

**PLANNED**

```text
1. Launch a queued mission
2. Update available missions
3. Add known IPs to MapScan queue
```

A refresh/dashboard option and exit option have also been discussed, but the final menu has not been implemented.

### Planned dashboard emphasis

**PLANNED**

- missions logged;
- completed;
- queued;
- failed or unresolved;
- attempted but not resolved with client;
- income and expenses;
- current wallet, if the GreyScript API permits;
- ANE3 earnings derived from completed missions once mission payout data is known.

### Financial accountability

**LOCKED**

`realized_earnings` or `realized_payout` is not stored as a mutable mission field.

Mission facts retain:

```text
mission_level
mission_payout
status
```

Because there are no partial payouts:

```text
completed mission  -> realized value equals mission_payout
other status       -> realized value equals 0
```

The aggregate calculation belongs in Mission Runner or a later accounting component.

**UNRESOLVED**

Whether the final calculation is implemented directly in Mission Runner or delegated to a future accounting library/component has not been locked.

### Current implementation state

**PLANNED**

Executable name:

```text
mission-runner
```

Canonical source:

```text
/home/Archeagus/ANE3/SRC/mission-runner.src
```

No current production version/build is implemented in this checkpoint.

---

## 2.7 Mission Organizer

### Responsibility

**LOCKED**

Mission Organizer is the one-shot mission intake, normalization, persistence, and outcome source of truth.

It will:

- log into in-game mail using the private credential module;
- fetch the mailbox once;
- identify mission threads;
- read the complete current thread for every relevant mission;
- create stable ANE3 mission IDs;
- preserve the raw full thread;
- normalize mission data;
- detect current mission lifecycle state;
- record attempt and unsuccessful-submission counts;
- create/update the mission manifest;
- generate operational queue data;
- report state;
- exit after synchronization.

It will not:

- retain/reuse a persistent mail object;
- delete mission mail automatically;
- skip a known thread solely because its `mail_id` is already known;
- guess payout from mail text;
- store calculated realized earnings as mutable mission data.

### One-shot lifecycle

**LOCKED**

```text
Mission Runner requests refresh
→ launch Mission Organizer once
→ login to mail
→ fetch once
→ identify/read relevant threads
→ preserve raw and normalized data
→ update operational indexes
→ process ends
```

### Current implementation state

**PLANNED**

Executable name:

```text
mission-organizer
```

Canonical source:

```text
/home/Archeagus/ANE3/SRC/mission-organizer.src
```

Production Mission Organizer source has not yet been created.

### Probe precursor

**IMPLEMENTED / OBSERVED / RETIRED**

Versioned probe artifact:

```text
mission-mail-probe-v0.1.0-bld1000.src
```

It was deployed in-game under the intentionally short-lived executable name:

```text
test
```

The user chose `test` for brevity and to lock its limited longevity.

The probe:

- imported the shared library;
- imported the private credentials binary;
- logged in once;
- fetched once;
- preserved all raw previews;
- allowed one selected full-thread read;
- did not send, delete, or modify mail;
- exited.

The test was very successful and Episode 8.3 is recorded.

---

# 3. Canonical Directory and File Structure

## 3.1 Runtime root

**LOCKED / IMPLEMENTED**

Shared runtime path helpers use:

```greyscript
home_dir + "/ANE3"
```

Canonical account-specific import paths currently use:

```text
/home/Archeagus/ANE3/...
```

The runtime and deployment currently assume compilation and execution under the `Archeagus` account.

## 3.2 Consolidated canonical tree

Status is shown inline because some files are implemented and others are planned.

```text
/home/Archeagus/ANE3/
├── SRC/                                             [LOCKED]
│   ├── LIB/                                         [LOCKED / IMPLEMENTED]
│   │   └── lib_functions.src                       [IMPLEMENTED]
│   ├── heartbeat.src                               [IMPLEMENTED]
│   ├── mapscan3.src                                [IMPLEMENTED]
│   ├── libscan1.src                                [IMPLEMENTED]
│   ├── expscan2.src                                [PLANNED]
│   ├── mission-organizer.src                       [PLANNED]
│   └── mission-runner.src                          [PLANNED]
│
├── STATES/                                          [LOCKED / IMPLEMENTED]
│   ├── state_heartbeat.txt                         [IMPLEMENTED]
│   ├── state_mapscan.txt                           [IMPLEMENTED]
│   ├── state_libscan.txt                           [IMPLEMENTED]
│   ├── state_expscan.txt                           [PLANNED]
│   └── state_mission_organizer.txt                 [PLANNED; probe bootstrapped/tested]
│
├── MANIFEST/                                        [LOCKED / IMPLEMENTED]
│   └── ane3.manifest                               [IMPLEMENTED; Heartbeat-owned]
│
├── LOG/                                             [LOCKED / IMPLEMENTED]
│   └── run.log                                     [IMPLEMENTED; Heartbeat-owned]
│
├── DB/                                              [LOCKED / PARTLY IMPLEMENTED]
│   ├── mission-queue.db                            [IMPLEMENTED by MapScan as IP-first queue;
│   │                                                future Mission Runner contract UNRESOLVED]
│   ├── target-seeds.db                             [IMPLEMENTED]
│   ├── targets.db                                  [IMPLEMENTED]
│   ├── services.db                                 [IMPLEMENTED]
│   ├── libraries.db                                [IMPLEMENTED]
│   ├── exploits.db                                 [IMPLEMENTED]
│   └── exploit-results.db                          [PLANNED]
│
├── TARGETS/                                         [IMPLEMENTED]
│   └── <ip-address>.txt                            [IMPLEMENTED MapScan report]
│
├── MISSIONS/                                        [PLANNED; manifest placeholder probe-tested]
│   ├── manifest.db                                 [PLANNED source-of-truth index]
│   ├── M-000001.db                                 [PLANNED normalized mission record]
│   ├── M-000002.db                                 [PLANNED normalized mission record]
│   └── RAW/                                        [PLANNED]
│       ├── M-000001.txt                            [PLANNED exact latest full thread]
│       └── M-000002.txt                            [PLANNED exact latest full thread]
│
├── CONFIG/                                          [LOCKED / IMPLEMENTED]
│   └── PRIVATE/                                    [LOCKED / IMPLEMENTED]
│       └── mail_credentials                        [IMPLEMENTED by user; compiled binary]
│
└── TMP/                                             [LOCKED / IMPLEMENTED by probe]
    └── mail-fetch-last.txt                         [IMPLEMENTED diagnostic capture]
```

## 3.3 Mission storage layers

**LOCKED DIRECTION / NOT YET IMPLEMENTED**

Three separate layers:

```text
RAW full thread
→ exact source evidence

Normalized mission record
→ durable mission source of truth

Manifest and operational queue/index
→ efficient Mission Runner access
```

Proposed paths:

```text
/ANE3/MISSIONS/RAW/M-000001.txt
/ANE3/MISSIONS/M-000001.db
/ANE3/MISSIONS/manifest.db
```

## 3.4 Mission queue path conflict

**UNRESOLVED — MUST BE RESOLVED BEFORE 8.4 CODE IS LOCKED**

Current MapScan 3 consumes:

```text
/ANE3/DB/mission-queue.db
```

with:

```text
ip|source|status|notes
```

The proposed Mission Runner-ready queue was drafted as mission-centric, for example:

```text
mission_id|priority|status|mission_type|target_ip|target_lan_ip
```

These formats are incompatible because MapScan expects field zero to be a valid IP.

No authoritative resolution has yet been locked.

---

# 4. Executables, Source Naming, Versions, Builds, State, Config, and Deployment

## 4.1 Canonical executable names

**LOCKED**

```text
heartbeat
mapscan3
libscan1
expscan2
mission-organizer
mission-runner
```

Temporary diagnostic executable:

```text
test
```

`test` was the deployed name of the disposable mission-mail probe and is not a permanent ANE3 command.

## 4.2 Canonical in-game source filenames

**LOCKED**

```text
heartbeat.src
mapscan3.src
libscan1.src
expscan2.src
mission-organizer.src
mission-runner.src
LIB/lib_functions.src
```

## 4.3 External source naming

**LOCKED**

Every preserved external iteration includes version and build:

```text
<component>-v<major.minor.patch>-bld<build>.src
```

Examples:

```text
heartbeat-v0.1.11-bld1011.src
mapscan3-v0.1.11-bld1011.src
libscan1-v0.1.11-bld1011.src
lib_functions-v0.1.6-bld1006.src
```

Current chat artifacts include:

```text
lib_functions-v0.1.6-bld1006-record-ready.src
heartbeat-v0.1.11-bld1011-record-ready.src
mapscan3-v0.1.11-bld1011-record-ready.src
libscan1-v0.1.11-bld1011-record-ready.src
mission-mail-probe-v0.1.0-bld1000.src
mail_credentials-v0.1.0-bld1000-generic-template.src
```

## 4.4 Current implemented versions

| Component | Status | Version | Build | Current versioned artifact |
|---|---:|---:|---:|---|
| `lib_functions` | **IMPLEMENTED** | `0.1.6` | `1006` | `lib_functions-v0.1.6-bld1006-record-ready.src` |
| `heartbeat` | **IMPLEMENTED** | `0.1.11` | `1011` | `heartbeat-v0.1.11-bld1011-record-ready.src` |
| `mapscan3` | **IMPLEMENTED** | `0.1.11` | `1011` | `mapscan3-v0.1.11-bld1011-record-ready.src` |
| `libscan1` | **IMPLEMENTED** | `0.1.11` | `1011` | `libscan1-v0.1.11-bld1011-record-ready.src` |
| mission-mail probe | **IMPLEMENTED / RETIRED** | `0.1.0` | `1000` | `mission-mail-probe-v0.1.0-bld1000.src` |
| mail credential template | **IMPLEMENTED TEMPLATE** | `0.1.0` | `1000` | `mail_credentials-v0.1.0-bld1000-generic-template.src` |
| `expscan2` | **PLANNED** | — | — | not implemented |
| `mission-organizer` | **PLANNED** | — | — | not implemented |
| `mission-runner` | **PLANNED** | — | — | not implemented |

## 4.5 Version/build calculation

**LOCKED / IMPLEMENTED**

Versions are tuples, not decimal values.

```text
version_score =
    major * 1,000,000
  + minor * 1,000
  + patch
```

```text
current_bld =
    last_bld
  + version_score(current_version)
  - version_score(last_version)
```

Each distinct intentional and testable change increments the patch version once. Multiple source-line changes implementing one logical change count as one patch.

## 4.6 State-file format

**LOCKED / IMPLEMENTED**

```text
scanner|online_state|activity|wait_seconds|last_beat|expires_at|script_version|script_bld|lib_version|lib_bld|detail
```

Lease expiration allows Heartbeat to detect a stopped or crashed worker even when Ctrl+C prevents it from writing `OFFLINE`.

## 4.7 Manifest format

**LOCKED / IMPLEMENTED**

```text
type|name|version|bld|path|last_checked
```

The manifest represents source versions currently available in the ANE3 source tree. Worker state reports the versions/builds embedded into the deployed command.

## 4.8 Private mail configuration

**LOCKED / IMPLEMENTED BY USER**

Generic external/on-camera template:

```text
mail_credentials-v0.1.0-bld1000-generic-template.src
```

Generic content:

```greyscript
ANE3_MAIL_CONFIG_VERSION = "0.1.0"
ANE3_MAIL_CONFIG_BLD = 1000

ANE3_MAIL_USER = "your@email"
ANE3_MAIL_PASSWORD = "your_password"
```

Private compiled deployment path:

```text
/home/Archeagus/ANE3/CONFIG/PRIVATE/mail_credentials
```

The actual private plaintext source stays outside the in-game ANE3 tree and is not shown or published.

**OBSERVED / IMPLEMENTED**

The private credential binary successfully imported into the deployed probe and authenticated to mail.

**VERIFY AGAINST SOURCE**

Whether owner-only `chmod` restrictions were applied to the deployed binary was discussed as a recommendation but not confirmed in this conversation.

## 4.9 Credential security boundary

**LOCKED UNDERSTANDING**

The compiled private module:

- keeps credentials out of visible ANE3 source;
- keeps them out of the recording and generic published template;
- provides concealment and access control rather than absolute secrecy;
- may be embedded into a dependent deployed binary at compile time;
- requires redeployment of the dependent command when credentials change.

Variable names avoid shadowing the built-in `mail_login()` function.

---

# 5. Locked Architectural and Accountability Decisions

## 5.1 Ownership table

| Resource / Behavior | Owner | Locked accountability |
|---|---|---|
| `/ANE3/MANIFEST/ane3.manifest` | Heartbeat | Heartbeat creates and refreshes it. |
| `/ANE3/LOG/run.log` | Heartbeat | Heartbeat creates it; workers may append only when it exists. |
| `state_heartbeat.txt` | Heartbeat | Heartbeat writes its own state. |
| `state_mapscan.txt` | MapScan 3 | MapScan creates/writes its own state. Heartbeat must not create it. |
| `state_libscan.txt` | LibScan 1 | LibScan creates/writes its own state. Heartbeat must not create it. |
| `state_expscan.txt` | ExpScan 2 | ExpScan will create/write its own state. |
| `state_mission_organizer.txt` | Mission Organizer | Mission Organizer will create/write its own state. |
| Target/service mapping | MapScan 3 | No other component absorbs this responsibility. |
| Library vulnerability research | LibScan 1 | No other component absorbs this responsibility. |
| Exploit execution/classification | ExpScan 2 | Operator-attended and cautious. |
| Mission intake and normalized outcomes | Mission Organizer | Source of truth for mission records and observed lifecycle. |
| Mission workflow/dashboard | Mission Runner | Coordinates components and calculates operational totals. |
| Worker-health display | Heartbeat | Observes; does not control workers. |
| Heartbeat health | Mission Runner | Watcher-for-the-watcher behavior. |
| Shared runtime primitives | `lib_functions` | No component-specific policy or schema ownership. |

## 5.2 No false initialization

**LOCKED**

Missing state is meaningful:

```text
missing state file = NOT_INITIALIZED
empty state file   = NO_REPORT
expired lease      = STALE
```

Heartbeat must not create a worker state file merely to make the dashboard look complete.

## 5.3 Mission Organizer as mission source of truth

**LOCKED**

Mission Organizer is authoritative for:

- mission thread identity;
- normalized mission facts;
- current client-confirmed lifecycle;
- submission attempt count;
- unsuccessful submission count;
- raw source thread;
- future mission-type outcome analytics.

Mission Runner consumes these facts and coordinates action.

## 5.4 Financial facts versus calculation

**LOCKED**

Store:

```text
mission_level
mission_payout
status
```

Do not store:

```text
realized_earnings
realized_payout
```

Mission payout is determined by mission level on the HackShop website, not by the mail thread. There are no partial payouts.

## 5.5 Mail process accountability

**LOCKED**

- one new process per synchronization;
- one mail login;
- one fetch;
- required thread reads;
- no automatic deletion;
- no persistent/reused mail object;
- exit after persistence.

## 5.6 Mission data must remain recoverable

**LOCKED**

Raw full threads are preserved before or alongside normalization. A parser failure must not destroy the original source material.

## 5.7 Operator agency in ExpScan 2

**LOCKED**

ExpScan 2 is intentionally cautious and interactive. The operator classifies ambiguous native output rather than ANE3 pretending to understand output it cannot reliably capture.

---

# 6. Current Mail-Thread and Mission-Lifecycle Model

## 6.1 Meaning of `mail_id`

**OBSERVED / LOCKED MODEL**

Although the API/output labels the value `MailID`, operationally the returned ID identifies an entire conversation thread, not one immutable individual message.

A single `mail_id` read can contain:

```text
original Mission Contract
→ player completion reply
→ client unresolved/rejection response
→ later player reply
→ final client satisfaction response
```

Therefore, ANE3 terminology should use:

```text
mail_thread_id
```

in normalized mission records, even though GreyScript returns it as `MailID`.

## 6.2 Preview format

**OBSERVED**

The successful probe returned a list of previews. Each observed preview contained:

```text
MailID: <uuid>
From: <address>
Subject: <subject>
<truncated body preview>
```

The observed mailbox returned:

```text
fetch_type=list
fetch_count=14
```

Eight observed messages used:

```text
Subject: Mission Contract
```

Other observed subjects included:

```text
something strange
Login issues
I'm watching you
```

## 6.3 Mission identification rule

**LOCKED FOR CURRENT IMPLEMENTATION**

Use exact subject matching:

```text
Subject == "Mission Contract"
```

Do not require the currently observed sender as the primary rule.

**UNRESOLVED**

How future subject variants or alternate mission senders should be configured has not been locked.

## 6.4 Known threads must be reread

**LOCKED**

Incorrect behavior:

```text
known mail_id
→ skip
```

Required behavior:

```text
unknown thread
→ create mission record

known unchanged thread
→ retain record

known changed thread
→ increment revision
→ replace latest raw snapshot
→ reparse lifecycle and mission facts
```

Every relevant known thread is reread during a new one-shot scan because new replies modify the same thread.

## 6.5 Target addressing

**OBSERVED**

Full mission bodies can contain:

```text
public IP
local IP
```

Example observed values:

```text
public IP = 51.100.113.146
local IP  = 10.0.1.7
```

Another observed mission:

```text
public IP = 78.128.108.156
local IP  = 192.168.1.2
```

Normalized fields:

```text
target_ip
target_lan_ip
```

LAN IP is optional.

**PLANNED**

Extract using explicit body anchors where possible, then validate with GreyScript IP validation functions.

**UNRESOLVED**

A robust generic fallback for missions with multiple public/LAN candidates or materially different wording has not yet been implemented.

## 6.6 Observed mission types

**OBSERVED / PLANNED CLASSIFICATION**

Observed preview/body wording supports at least:

```text
ACADEMIC_RECORD
POLICE_RECORD
FILE_RETRIEVAL
```

Observed patterns:

```text
"change some grades in his academic record"
→ ACADEMIC_RECORD

"modify the information of a police record"
→ POLICE_RECORD

"get a file that contains important information"
→ FILE_RETRIEVAL
```

Unknown mission wording should still be preserved and marked for review rather than discarded.

## 6.7 Observed academic-record fields

**OBSERVED**

Example one:

```text
target_name   = Dalila Demutt
target_detail = Mathematics
objective     = change Mathematics to approved at least
```

Example two:

```text
target_name   = Kathleen Blyth
target_detail = Economy
objective     = increase Economy qualification by at least one point
```

**UNRESOLVED**

The exact normalized objective wording and subtype-specific schema for every mission type have not been locked in code.

## 6.8 Mission status model

**LOCKED CURRENT VOCABULARY**

Current primary statuses:

```text
QUEUED
ATTEMPTED
COMPLETED
FAILED
PAUSED
```

Earlier planning also included:

```text
NEW
ARCHIVED
PARSE_FAILED
```

**UNRESOLVED**

The final distinction between `NEW` and `QUEUED`, and whether `PARSE_FAILED` is a lifecycle status or a separate `parse_status`, must be locked before production serialization.

Current preferred model separates:

```text
status
queue_status
parse_status
```

## 6.9 Observed event patterns

### Contract received

**OBSERVED**

A thread containing only the contract can be treated as not yet submitted.

Preferred current state:

```text
status = QUEUED
attempts = 0
failed_attempts = 0
last_thread_event = CONTRACT_RECEIVED
```

### Player submission

**OBSERVED**

Player replies appeared as lines beginning with:

```text
>
```

Examples:

```text
> Done.
> done?
> Done now?
```

Each observed player submission represents an attempt.

### Requirements unmet but mission still active

**OBSERVED**

Exact observed response:

```text
You have not completed the order requirements, the mission is still active.
```

Interpretation:

```text
failed_attempts += 1
status = ATTEMPTED
last_thread_event = REQUIREMENTS_UNMET
```

This is not a permanent mission failure.

### Client satisfied

**OBSERVED**

Exact observed response:

```text
The customer is satisfied with the job. There has been an income in your account.
```

Interpretation:

```text
status = COMPLETED
last_thread_event = CLIENT_SATISFIED
```

The final authoritative client event determines the current mission state. Earlier unsuccessful attempts remain recorded without overriding a later successful resolution.

## 6.10 Attempt-count example

**OBSERVED**

Kathleen Blyth academic mission:

```text
attempts = 2
failed_attempts = 1
status = COMPLETED
```

Sequence:

```text
contract
→ "done?"
→ requirements unmet / mission active
→ "Done now?"
→ client satisfied
```

## 6.11 Permanent failure

**UNRESOLVED**

No permanent-failure client phrase has been captured in the conversation.

Mission Organizer must not classify a mission as `FAILED` merely because it contains an unsuccessful attempt. The permanent-failure rule remains **VERIFY AGAINST SOURCE** until an actual thread is observed.

## 6.12 Mission level and payout

**OBSERVED / LOCKED**

Mission payout is determined by mission level on the HackShop website.

The mail object/thread does not expose the mission level or payout in the observed output.

Store future fields:

```text
mission_level
mission_payout
```

Initial imported value:

```text
UNKNOWN
```

Do not infer payout from mail text.

## 6.13 Proposed normalized mission record

**LOCKED DIRECTION / NOT YET IMPLEMENTED**

Key-value records were accepted as the preferred future-ready direction because they are easier to inspect, extend, and avoid delimiter collisions.

Example field set:

```text
mission_id
mail_thread_id
thread_revision

status
queue_status
mission_type

subject
sender

objective
target_ip
target_lan_ip
target_name
target_detail

mission_level
mission_payout

attempts
failed_attempts
last_thread_event

parse_status
created
updated
source_file

expscan_status
expscan_session
selected_exploit
access_result
privilege_result
completion_submitted

notes
```

`realized_earnings` is explicitly excluded.

**VERIFY AGAINST SOURCE**

The exact final field order, empty-value representation, escaping rules, and record parser/writer functions are not yet implemented.

---

# 7. Monitoring, Ownership, Failure Detection, and Watcher-for-the-Watcher

## 7.1 Lease-based failure detection

**IMPLEMENTED**

Workers report:

```text
activity
wait_seconds
last_beat
expires_at
```

Heartbeat treats an online report whose lease has expired as:

```text
STALE
```

This handles:

- Ctrl+C termination without an `OFFLINE` write;
- crashes;
- scripts blocked longer than their declared lease;
- stopped terminals.

## 7.2 Current activity vocabulary

**IMPLEMENTED / LOCKED**

Known dashboard/activity states include:

```text
ACTIVE
IDLE
STALE
UPDATE_AVAILABLE
UPDATE_REQUIRED
OFFLINE
ERROR
UNKNOWN
NOT_INITIALIZED
NO_REPORT
```

## 7.3 Current color semantics

**IMPLEMENTED**

```text
ACTIVE             green
IDLE               blue
STALE              yellow
UPDATE_AVAILABLE   yellow
UPDATE_REQUIRED    red
OFFLINE            red
ERROR              red
UNKNOWN             grey
NOT_INITIALIZED    grey
NO_REPORT           grey
```

Build comparison:

```text
current            green
one build behind   yellow
two+ behind        red
unknown             grey
```

## 7.4 Current Heartbeat limitation

**IMPLEMENTED LIMITATION**

Heartbeat v0.1.11 currently renders only MapScan, LibScan, and ExpScan rows.

It does not currently:

- display Mission Organizer;
- display other future worker modules;
- show a Mission Runner row;
- evaluate itself in its own dashboard.

## 7.5 Target architecture

**LOCKED / PLANNED**

```text
Mission Runner
    watches Heartbeat

Heartbeat
    watches MapScan 3
    watches LibScan 1
    watches ExpScan 2
    watches Mission Organizer
    watches other operational workers
```

Mission Runner’s watcher-for-the-watcher function is independent of Heartbeat’s own display.

## 7.6 Mail failure policy

**LOCKED DIRECTION / NOT YET IMPLEMENTED**

- login or fetch error: record `ERROR`, exit;
- individual read failure: record it and continue where safe;
- parser failure: preserve raw thread, mark parse state, continue;
- database write failure: preserve raw material and report error;
- no automatic deletion of mail.

## 7.7 Mission Organizer phases

**PLANNED**

State detail may expose phases such as:

```text
STARTING
CONNECTING
FETCHING
READING
PARSING
WRITING
COMPLETE
ERROR
OFFLINE
```

The shared state activity field may remain within the established activity vocabulary while the detailed phase is placed in the `detail` field.

---

# 8. Completed Episode 8 Work and Current Position

## 8.1 Recorded sections

**IMPLEMENTED / RECORDED**

Sections:

```text
8.0
8.1
8.2
```

were successfully recorded after revisions and clean resets.

**VERIFY AGAINST SOURCE**

The exact final on-camera titles and detailed contents of 8.0–8.2 should be verified against the episode outline or recorded footage. This checkpoint establishes their completion, not a reconstructed scene-by-scene script.

## 8.2 Working foundation at the completion of 8.2

**IMPLEMENTED / RECORDED**

```text
lib_functions  v0.1.6  bld 1006
heartbeat      v0.1.11 bld 1011
mapscan3       v0.1.11 bld 1011
libscan1       v0.1.11 bld 1011
```

The delimiter defects were corrected and the foundation was successfully recorded.

## 8.3 Mission-mail probe

**IMPLEMENTED / OBSERVED / RECORDED**

Episode 8.3 is in the can.

On-camera setup:

- user-created private mail credentials binary was already deployed before recording;
- generic `your@email` source was retained for safe on-camera explanation;
- disposable probe was deployed as `test`;
- probe connected successfully;
- fetch returned 14 previews;
- one full mission thread was read;
- raw data was saved in `mail-fetch-last.txt`;
- later a second full thread fixture with two attempts was supplied;
- no mail was modified.

Confirmed discoveries:

- exact `Mission Contract` subject is a usable current mission filter;
- `MailID` identifies an evolving thread;
- full thread includes contract, player replies, and client responses;
- public and LAN targets can both be present;
- attempt count and unsuccessful submissions can be observed;
- client satisfaction can confirm completion;
- payout/mission level is absent from mail.

## 8.4 Current position

**PLANNED**

Episode 8.4 will implement a single-pass, Mission Runner-ready Mission Organizer parser with future-ready fields for later ExpScan 2 and mission-outcome processing.

The desired Episode 8 progression is:

```text
8.3 — discover and validate mail data
8.4 — normalize it through Mission Organizer
8.5 — build the first Mission Runner draft
8.6 — demonstrate the first ExpScan 2 pass and close the episode
```

**UNRESOLVED BLOCKER BEFORE CODE**

The operational mission-queue schema/path collision with current MapScan 3 must be resolved before Mission Organizer’s queue writer is locked.

---

# 9. Episode 9+ Backlog

## 9.1 Mission analytics and prioritization

**BACKLOG — LOCKED INTENT**

Mission Organizer becomes the authoritative source for outcome analytics by mission type.

Track:

```text
mission_type
status
attempts
failed_attempts
created
updated
last_thread_event
eventual completion
unresolved/permanent failure
```

Calculate:

```text
success rate by mission type
average attempts by mission type
unsuccessful submissions before success
completion rate
unresolved rate
failure rate
sample size / confidence
```

Provide Mission Runner prioritization signals:

```text
RECOMMENDED
NORMAL
CAUTION
MANUAL_REVIEW
UNRATED
```

Conceptual interpretation:

```text
high success + low attempts
→ recommended

high success + repeated attempts
→ viable but costly

low success
→ caution/review

insufficient history
→ unrated
```

## 9.2 HackShop level/payout intake

**BACKLOG**

Capture or correlate mission level and payout from the HackShop website before or around mission acceptance.

Possible approaches discussed:

- manual level entry during mission review;
- automated/assisted HackShop listing capture and correlation.

No approach is locked.

## 9.3 Thread revision history

**BACKLOG**

Current first-pass direction stores the latest exact thread snapshot.

Future history may preserve:

```text
/ANE3/MISSIONS/HISTORY/M-000001-r001.txt
/ANE3/MISSIONS/HISTORY/M-000001-r002.txt
```

## 9.4 Greybel/VS deployment improvements

**BACKLOG**

Evaluate Greybel VS direct import through a message hook/plugin while preserving:

- external source authority;
- intentional deployment boundary;
- version/build traceability.

## 9.5 In-game APT distribution

**BACKLOG**

Research hosting ANE3 in Grey Hack multiplayer through an in-game APT repository:

- package/install;
- update;
- migration;
- same-world distribution.

Universal single-player distribution still requires external GitHub/release installer/manual import because one player’s single-player repository cannot serve another player’s world.

## 9.6 Safe installer/migration framework

**BACKLOG**

Desired future migration flow:

```text
backup old tree
→ detect layout
→ copy data
→ transform schemas
→ verify counts/content
→ preserve backup
→ commit new layout
```

Include:

- dry-run;
- migration log;
- recovery path;
- no silent destructive conversion.

## 9.7 Monitoring/UI refinement

**BACKLOG**

- explicit white standard report text;
- preserve teal for native Grey Hack output;
- restrained state/freshness background emphasis;
- expand Heartbeat monitoring after component tests;
- dedicated Mission Runner Heartbeat monitor.

## 9.8 Mission performance guidance

**BACKLOG**

Use historical performance to:

- prioritize reliable mission categories;
- flag weak mission categories;
- expose average effort;
- support future operator strategy rather than only raw automation.

---

# 10. Known GreyScript Limitations, Resolved Defects, Unresolved Defects, and Risks

## 10.1 Source importer restrictions

**OBSERVED / LOCKED TECHNICAL RULE**

The GreyScript source importer:

- accepts only a literal absolute string path;
- does not accept variables;
- does not accept concatenation or expressions;
- is triggered wherever its keyword appears, including comments and disabled examples.

Therefore:

- the importer keyword must appear only for actual imports;
- it must not appear in comments, strings, examples, or documentation in a compiled source;
- imports currently use literal `/home/Archeagus/...` paths.

## 10.2 Embedded dependency behavior

**OBSERVED / IMPLEMENTED**

Imported source is embedded/reconciled at deployment. A deployed command does not dynamically pick up later library edits.

Risk:

- running commands may report older embedded library builds;
- source-manifest build and running embedded build can differ;
- redeployment is required.

Heartbeat’s build comparison is designed to expose this.

## 10.3 Regex delimiter behavior

**OBSERVED / RESOLVED**

GreyScript `String.split()` uses regex delimiters.

Live-confirmed literal delimiters:

```greyscript
"A.B.C".split("[.]")
"A|B|C".split("[|]")
```

Correct ANE3 usage:

```greyscript
split("[.]")
split("[|]")
```

Previous broken forms include:

```text
split("/.")
split("/|")
split(".")
split("|")
```

Current `lib_functions v0.1.6` uses the confirmed forms.

## 10.4 Screen clearing

**RESOLVED**

Use:

```greyscript
clear_screen()
```

not bare:

```text
clear_screen
```

## 10.5 Home path construction

**RESOLVED FOR RUNTIME PATHS**

ANE3 runtime path helpers use:

```text
home_dir
```

rather than constructing a path from `active_user`.

This resolved viewer/root execution problems in earlier work.

**RISK / UNRESOLVED FOR IMPORTS**

Compile-time import paths remain literal and account-specific:

```text
/home/Archeagus/ANE3/...
```

Cross-user compilation/root redirection remains deferred.

## 10.6 Mail object reuse

**OBSERVED RISK / LOCKED MITIGATION**

Repeated updates through the same mail object appeared unreliable.

Mitigation:

```text
one process
one login
one fetch
required reads
exit
```

No explicit logout/disconnect API has been established; process termination discards the object.

## 10.7 Mail preview truncation

**OBSERVED**

Preview bodies are truncated. Full target and lifecycle data require reading the full thread by `MailID`.

## 10.8 `mail_id` naming mismatch

**OBSERVED**

Grey Hack labels it `MailID`, but its behavior is thread-scoped. ANE3 normalized data should use `mail_thread_id`.

## 10.9 Full-thread parser risk

**UNRESOLVED**

The parser must distinguish:

- original contract text;
- player replies;
- client responses;
- event order.

A simplistic “contains success/failure keyword” parser can misclassify a thread with earlier rejection and later success.

## 10.10 Attempt-line ambiguity

**UNRESOLVED**

Observed player replies begin with `>`, but the robustness of this rule across all mission thread formats has not been verified.

Mark exact message-boundary parsing **VERIFY AGAINST SOURCE** during Mission Organizer implementation.

## 10.11 Payout unavailable in mail

**OBSERVED / LOCKED**

Mission payout and level are not available in the observed mail object/thread. They come from HackShop.

Risk:

- Mission Runner cannot calculate full historic earnings until level/payout data is supplied or correlated;
- unknown payout must remain unknown, not zero.

## 10.12 Native overflow output

**OBSERVED DESIGN CONSTRAINT**

Some exploit/overflow output is meaningful but terminal-only and not programmatically capturable.

Mitigation:

- dedicated ExpScan 2 terminal;
- no redraw during active overflow;
- operator classification.

## 10.13 Independent terminal object sharing

**OBSERVED / ARCHITECTURAL RISK**

Independently running workers cannot reliably share `get_custom_object()` unless they are in an appropriate parent-child shell launch chain.

Mitigation direction:

- file-backed queues;
- file-backed activity/state feeds;
- Mission Runner exclusively owns its dashboard terminal.

## 10.14 Terminal naming

**UNRESOLVED**

Whether GreyScript can reliably assign names to terminals remains unconfirmed.

## 10.15 Heartbeat component coverage

**UNRESOLVED DEFECT / PLANNED CHANGE**

Heartbeat v0.1.11:

- includes `mission-runner` in the source manifest;
- does not include `mission-organizer` in the source manifest;
- displays only MapScan, LibScan, ExpScan state rows.

The current architecture requires broader worker coverage later.

## 10.16 Mission queue schema collision

**UNRESOLVED HIGH-PRIORITY CONTRACT DEFECT**

Current MapScan:

```text
/ANE3/DB/mission-queue.db
# ip|source|status|notes
```

Proposed Mission Organizer/Mission Runner queue:

```text
mission_id|priority|status|mission_type|target_ip|target_lan_ip
```

A production Mission Organizer must not overwrite the current IP-first file with the mission-ID-first format without a coordinated MapScan change.

## 10.17 Mission record encoding

**UNRESOLVED**

Key-value records are the accepted direction, but the following are not yet locked:

- escaping `=`;
- newline handling inside values;
- reserved delimiters;
- field order;
- required versus optional fields;
- forward-compatible schema version field;
- atomic update behavior.

## 10.18 Stable mission ID allocation

**UNRESOLVED IMPLEMENTATION DETAIL**

Required behavior is stable IDs independent of mailbox index:

```text
M-000001
M-000002
...
```

The exact allocation algorithm and first-import ordering have not been implemented.

## 10.19 Raw thread update policy

**LOCKED FIRST-PASS DIRECTION / BACKLOG HISTORY**

First pass may replace the latest raw snapshot after detecting a changed thread and increment `thread_revision`.

Revision history is deferred.

## 10.20 Credential secrecy

**KNOWN RISK**

Compiled credentials are not absolute protection against:

- root access;
- import reuse;
- runtime inspection;
- future decompilation.

They are a reasonable single-player concealment/access-control tradeoff, not an unbreakable vault.

---

# 11. Open Decisions Not Yet Locked

## 11.1 Mission queue contract

**UNRESOLVED — FIRST DECISION REQUIRED**

Choose one:

### Option A: Separate mission and target queues

Example:

```text
/ANE3/DB/mission-runner-queue.db
# mission_id|priority|status|mission_type|target_ip|target_lan_ip

/ANE3/DB/mission-queue.db
# ip|source|status|notes
```

This preserves MapScan v0.1.11 compatibility.

### Option B: Rename MapScan’s target input queue

Example:

```text
/ANE3/DB/mapscan-target-queue.db
```

Then reserve `mission-queue.db` for mission-centric records. This requires a coordinated MapScan version bump and migration.

### Option C: Make one queue schema serve both

Not recommended without a carefully versioned contract. Current code cannot consume a mission-ID-first record.

No option is locked.

## 11.2 Mission record schema versioning

**UNRESOLVED**

Consider adding:

```text
schema_version=
```

to every key-value mission record. Not yet accepted or implemented.

## 11.3 Exact status model

**UNRESOLVED**

Lock the final relationship between:

```text
status
queue_status
parse_status
expscan_status
```

Specific questions:

- Is `NEW` distinct from `QUEUED`?
- Is `PARSE_FAILED` a status or parse status?
- When does `ATTEMPTED` become actionable again?
- How is an explicitly abandoned mission represented?
- What exact observed phrase establishes permanent `FAILED`?

## 11.4 Mission-level/payout acquisition

**UNRESOLVED / BACKLOG**

Manual entry versus HackShop intake/correlation.

## 11.5 Mission priority in Episode 8

**UNRESOLVED**

Initial queue priority may be a constant such as `NORMAL`, but the exact field and sort behavior are not locked.

## 11.6 Objective normalization

**UNRESOLVED**

Decide how much subtype-specific parsing belongs in Mission Organizer v0.1.0:

- raw descriptive objective only;
- normalized fields such as record owner/subject/action;
- both.

## 11.7 Mission thread parser boundaries

**UNRESOLVED**

Need to verify whether:

- all player replies begin with `>`;
- blank-line boundaries are stable;
- response phrases vary;
- markup is present in `read()` output under other contexts;
- client replies can contain quoted earlier messages.

## 11.8 Mission Runner’s state file

**UNRESOLVED**

No canonical `state_mission_runner.txt` contract is established in implemented source.

Mission Runner’s watcher-for-the-watcher behavior may require one, but that decision is not locked.

## 11.9 Mission Organizer manifest inclusion

**PLANNED / UNRESOLVED VERSIONING**

Heartbeat must eventually include Mission Organizer in the source manifest, but whether that change occurs during Episode 8.4 or a later Heartbeat update is not locked.

## 11.10 ExpScan 2 invocation contract

**UNRESOLVED**

Potential direct commands were previously conceptualized, but the final Mission Runner-to-ExpScan 2 handoff has not been locked.

Required mission fields are expected to include:

```text
mission_id
mission_type
objective
target_ip
target_lan_ip
```

Exact CLI arguments versus file-backed handoff remain open.

## 11.11 Financial calculation owner

**UNRESOLVED**

Locked: not stored as mutable mission data.

Open:

- calculate directly in Mission Runner;
- calculate through a future accounting/helper layer.

## 11.12 Current wallet API

**VERIFY AGAINST SOURCE**

Whether GreyScript exposes a reliable current-wallet value for Mission Runner has not been established in this conversation.

---

# 12. Next Recommended Implementation Step

## 12.1 Resolve the queue contract before writing Mission Organizer

**RECOMMENDED FIRST ACTION**

Lock separate contracts for:

1. Mission Runner’s mission-level queue/index.
2. MapScan’s IP-level target queue.

The lowest-risk current recommendation is:

```text
/ANE3/DB/mission-runner-queue.db
# mission_id|priority|status|mission_type|target_ip|target_lan_ip
```

while preserving:

```text
/ANE3/DB/mission-queue.db
# ip|source|status|notes
```

for MapScan v0.1.11.

This is a recommendation, not yet a locked decision.

## 12.2 Draft Mission Organizer v0.1.0 bld 1000

After the queue contract is locked, implement one production-style pass:

```text
import shared library
import private credentials
bootstrap mission folders/files
login once
fetch once
filter exact Mission Contract subjects
read every matching current thread
match existing mail_thread_id
allocate stable mission ID when new
compare latest raw thread
parse addresses/type/objective/lifecycle
write normalized key-value record
update mission manifest
rebuild Mission Runner queue
feed MapScan target queue under the locked contract
report summary/state
exit
```

## 12.3 Use the two supplied academic fixtures as parser tests

Fixture A:

```text
one player submission
client satisfied
public IP + LAN IP
```

Fixture B:

```text
two player submissions
one requirements-unmet response
later client satisfied
public IP + LAN IP
```

Minimum expected assertions:

```text
attempts = 1 / 2
failed_attempts = 0 / 1
status = COMPLETED
last_thread_event = CLIENT_SATISFIED
```

## 12.4 Validate before calling it record-ready

Required Grey Hack tests:

1. First run imports all `Mission Contract` threads.
2. Stable mission IDs are created.
3. Completed and unresolved missions classify correctly.
4. Public and LAN IP fields are correct.
5. Raw full threads are preserved.
6. Second run creates no duplicates.
7. Known unchanged threads remain unchanged.
8. A changed known thread increments revision and updates status.
9. Mission Runner queue output follows the locked schema.
10. MapScan continues to receive valid IP-first target input.
11. No mail is deleted or modified.
12. Credential values never appear in visible output or stored mission data.

## 12.5 Episode 8 continuation after 8.4

```text
8.5
→ first Mission Runner dashboard and actionable queue

8.6
→ first ExpScan 2 attended execution/classification pass
→ Episode 8 conclusion
```

---

# Appendix A — Current Exact Source Imports

## Implemented core scripts

```greyscript
import_code("/home/Archeagus/ANE3/SRC/LIB/lib_functions.src")
```

## Mission mail probe

```greyscript
import_code("/home/Archeagus/ANE3/SRC/LIB/lib_functions.src")
import_code("/home/Archeagus/ANE3/CONFIG/PRIVATE/mail_credentials")
```

No importer examples or keyword mentions may be left inside compiled-source comments or strings.

---

# Appendix B — Confirmed Split Syntax

```greyscript
versionText.split("[.]")
line.split("[|]")
content.split(char(10))
```

---

# Appendix C — Current Implemented Database Headers

```text
/ANE3/DB/mission-queue.db
# ip|source|status|notes

/ANE3/DB/target-seeds.db
# ip|source|status|notes

/ANE3/DB/targets.db
# ip|source|status|last_scan

/ANE3/DB/services.db
# ip|port|state|service|lan_ip|lib_key|last_seen

/ANE3/DB/libraries.db
# lib_key|lib_name|version|sample_ip|sample_port|status|last_seen

/ANE3/DB/exploits.db
# lib_key|zone|unsafe_value|result_type|status|last_tested|notes
```

---

# Appendix D — Current Checkpoint Summary

```text
IMPLEMENTED
    lib_functions v0.1.6 bld 1006
    heartbeat v0.1.11 bld 1011
    mapscan3 v0.1.11 bld 1011
    libscan1 v0.1.11 bld 1011
    private mail credential binary
    disposable mail probe v0.1.0 bld 1000
    Episode 8.0–8.3 recorded

OBSERVED
    exact Mission Contract subject
    MailID is an evolving thread identifier
    public and local targets in full body
    multiple attempts in one mission
    requirements-unmet response keeps mission active
    client-satisfied response confirms completion
    payout/level absent from mail

PLANNED
    Mission Organizer v0.1.0
    Mission Runner first draft
    ExpScan 2 first pass
    normalized mission records
    raw mission thread storage
    mission manifest and runner queue

UNRESOLVED
    mission queue path/schema collision
    final mission status/parse/queue vocabulary
    exact record encoding and schema version
    stable ID allocation details
    permanent failure phrase
    final ExpScan handoff
    Heartbeat Mission Organizer coverage
    current-wallet API
```
