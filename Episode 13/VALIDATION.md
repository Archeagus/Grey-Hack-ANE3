# Validation — 2026-10-10

## Performed

- Parsed interpreter, editor, and regression runner using greyscript-core 2.6.0.
- Executed the actual interpreter source with the supplied lib_functions 0.1.6
  in Greybel 3.7.12's mock environment: **30 checks passed, zero failed**.
  Local test scaffolding inlined imports and seed-rule bytes; the interpreter
  functions were not translated to a different language or replaced by mocks.
- Exercised the actual editor control flow in that environment using scripted
  input and the mock filesystem: load existing rule, preview Coos, save revision
  2, verify exact previous-rules backup, verify mission record unchanged.
- Exercised new-rule creation through the editor: police full-record deletion,
  objective assignment, person/IP extraction, required-field declarations,
  save rejected before preview, successful Irina preview, save new rule,
  exact backup and unchanged mission record verified. This added rule exists
  only in the temporary test environment; the shipped seed file has 3 rules.

## Interpreter coverage

Actual Coos RAW markup and reply boundary; full path; inferred username;
UNSTATED login requirement; correct scoped IP pair; Uilezni and held-out Duns;
ANY without an invented username; replies cannot introduce a different family
or address; missing required addresses; unknown text; repeated anchor;
overlapping rules; duplicate IDs; missing end delimiter; malformed rules;
172.16 and 10 local addresses; invalid address rejection.

## Remaining in-game verification

This has **not** been compiled or executed inside Grey Hack. Mock execution is
not proof of the game's compiler, filesystem errors, terminal dimensions,
rich-text colors, or additional mail-thread formats.

Build the editor, preview M-000007, then preview a named and ANY-user contract.
Check violet ANY readability. The optional context-tests runner should report
30 passed and 0 failed with the original seed rules. Existing MissionScan is
not modified or integrated in this increment.

## Known limits of this first increment

- Three seeded variants, not all 12 harvested variants.
- All-of phrases and exclusions; no any-of groups or alternative extraction
  anchor lists yet. Separate rules can cover distinct templates.
- One prior config backup, not immutable revision history or transactional
  multi-writer storage.
- Known thread boundaries only; body-only previews don't establish mailbox
  eligibility. No automatic mail access.
- No MissionScan reprocessing, projection, execution, or lifecycle changes.
- A successful preview is for the chosen sample, not a whole-corpus proof.
  Use the regression runner and additional RAW previews after editing rules.

External validation tooling was installed only in the assistant's temporary
workspace. The delivered in-game files require no Greybel or game mod.
