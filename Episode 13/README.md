# ANE3 Mission Context — v0.1.0 preview build

First implementation: a shared rule interpreter, interactive in-game editor,
and three seed rules (file retrieval, named credentials, ANY-user credentials).
This package is a separate development copy. The supplied MissionScan and
lib_functions originals have not been modified.

## Installation: three files

Copy these contents into your existing in-game workspace:

| Package file | In-game destination |
|---|---|
| SRC/LIB/lib_mission_context.src | /home/Archeagus/ANE3/SRC/LIB/lib_mission_context.src |
| SRC/mission-context.src | /home/Archeagus/ANE3/SRC/mission-context.src |
| CONFIG/mission-context-rules.db | /home/Archeagus/ANE3/CONFIG/mission-context-rules.db |

Keep the existing lib_functions.src v0.1.6. Build mission-context.src as an
executable named mission-context using the normal CodeEditor workflow.
The import paths follow the current MissionScan source supplied by Jay.
If the in-game account/root differs, adjust both import paths first.

No game mod, Greybel installation, mailbox credentials, save reset, or bulk
mission export is required to run this preview. Do not replace MissionScan.

## First test

1. Run mission-context. Select P at the main menu.
2. Enter M-000007 (or the actual existing RAW mission ID).
3. Expect PARSED, file_retrieve_named r1, original IPs, the full sales_report.bin
   path, and Coos with INFERRED_HOME_PATH provenance. user_requirement stays
   UNSTATED: this does not require logging in as Coos.
4. At the main menu, select a rule number to edit it, or N to create one.
5. Match/exclude/extract/set/require prompts append lines to the draft. Use L
   to replace an existing line or - to remove it; duplicate assignments are
   rejected. Rule ID and revision are managed by the editor.
6. P previews the whole proposed rule set against a RAW ID or absolute text
   file path. W saves only after this exact draft successfully parses an
   example as the selected rule. Type SAVE at the prompt. B discards a draft
   after an explicit YES.

The seed rules already recognize this file mission; the first preview checks
installation. Teaching additional variants starts with N. For an on-camera
file-path teaching demonstration, remove file_path extraction and requirement
from a separate demo copy of the seed rules before the session, then add both
through the editor. Do not use the production config as a disposable demo.

## Rule format

Each block starts with rule=ID and ends with end. Metadata keys rule, revision,
family, action occur once. Every match phrase must occur; any exclude phrase
vetoes the rule. Matching ignores case; extracted values retain case.

    extract=field|literal text before value|literal text after value
    extract=field|literal text before value|@WORD
    extract=field|literal text before value|@LINE
    set=field|fixed value
    require=field
    derive=target_user|HOME_OWNER

Extraction is confined to the same line after the anchor. Anchors and literal
end delimiters are not regex. @WORD stops at whitespace; @LINE takes the rest
of the line. Surrounding whitespace and one terminal sentence period are
removed. A literal filename ending in a period needs a future explicit
punctuation policy; current fixtures all use sentence punctuation.
Pipe characters are reserved separators inside set/extract/derive operations.
Alternatives with genuinely different templates currently use separate rules.
There is no any-of recognition group or alternative-anchor group in v0.1.0.
If several rules match, AMBIGUOUS is returned with their IDs and no chosen rule.
Repeated extraction anchors also require review rather than choosing a value.

Supported fields are shown by E and S in the editor. User selection uses
SPECIFIC, ANY, UNSTATED, or NOT_APPLICABLE; blank usernames are never promoted
to ANY. The explicit ANY value is violet #B39DDB. Other fields may use fixed
qualifiers where the contract supports them. Do not encode N/A as a fake IP.

## Boundaries and persistence

The original raw text remains untouched. Interpretation strips display markup
and stops at a > reply, a subsequent From/Subject header, or the two known
client feedback phrases. This covers the supplied full thread. Other layouts
require fixtures before we claim support. Family matching alone is not mailbox
classification: the current editor accepts a supplied text file, and performs
no sender/subject eligibility decisions. Police intake headers remain untested.

Saving writes only mission-context-rules.db and its .previous backup, checking
each write result and rereading the written content. It checks whether the
config changed externally since load, but is not a transactional multi-writer
system; run one editor at a time. The .previous file holds one prior complete
ruleset. Immutable rule history and mission interpretation revisions belong
to the later reprocessing integration.

MissionScan currently continues using its own parser. Saving a rule here does
not teach the deployed MissionScan yet. No mission records, completion status,
ExpScan state, target seeds, or queues are changed by this preview build.

## Tests

TESTS/context-tests.src is an optional in-game regression runner with embedded
fixtures and assertions; it reads the installed rules and writes no files.
It expects the original seed rules. Build and run it like other scripts after
the two libraries are installed. TESTS contains body-only samples plus the
full Coos RAW thread. You need not copy these separate fixture files for the
embedded test runner, or when using your existing RAW files.

See VALIDATION.md for checks actually performed and remaining in-game checks.

## Next integration increment

After in-game build/preview checks, connect the same interpreter to MissionScan
with explicit reprocessing, add rule/interpretation provenance, preserve the
thread revision and execution fields, and review Mission Runner consumers
before changing queue readiness semantics. Add the other collected mission
variants through rules, retaining constraints, hints, submission and address
roles as separate fields. Rule changes should not implicitly seed targets.
