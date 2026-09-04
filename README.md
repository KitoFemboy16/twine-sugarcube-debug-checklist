# Twine / SugarCube Debugging Checklist

A checklist-first workflow for isolating story-state, passage-flow, macro, and save-compatibility bugs before changing code.

## 1. Freeze the reproduction

- Record the starting passage.
- List every click or input needed to trigger the fault.
- State the expected passage, UI, and variable result.
- Capture the actual result and console error.
- Reproduce again from a new game before touching code.

## 2. Check passage flow

- Confirm every target passage exists with exact spelling and case.
- Search for duplicate passage names.
- Inspect conditional links whose destination changes with state.
- Check whether startup or passage hooks redirect unexpectedly.

## 3. Inspect state

Before the failing action, capture only the variables involved:

```javascript
console.table({
  currentPassage: State.passage,
  exampleFlag: State.variables.exampleFlag,
  exampleCount: State.variables.exampleCount
});
```

Then confirm:

- variable names match in every passage and widget;
- values have the expected type;
- temporary variables are not mistaken for story variables;
- initialization runs before the first read;
- per-character data is not accidentally stored in one global value.

## 4. Isolate macros and widgets

- Reduce the fault to the smallest passage that still fails.
- Test each custom widget with known inputs.
- Check opening and closing macro tags.
- Treat string-built macros as suspicious; variable-name errors inside generated text often fail silently.
- Inspect browser-console errors before rewriting the widget.

## 5. Check save compatibility

- Reproduce from a new game and a pre-change save.
- Add defaults for new state fields.
- Confirm custom objects serialize and revive correctly.
- Avoid renaming persistent fields without a migration.

## 6. Prove the repair

Keep a short repair record:

1. reproduction before the change;
2. root cause;
3. smallest code change;
4. new-game result;
5. existing-save result;
6. any remaining limitation.

A repair is not complete because one passage loaded once.

## Sanitized repair inquiries

Use the repository issue form for a scoped diagnosis or quote. Do not upload private, licensed, or unreleased story source to a public issue. Provide a minimal reproduction, console output, and redacted variable snapshots instead.

The first response is a diagnosis or quote. It is not a contract and does not claim that a repair has been completed.

## Current community signal

The Interactive Fiction Community Forum remains active around Twine and SugarCube tooling: https://intfiction.org/latest

Open playtest request (observed 2026-08-31, posted 2026-08-29, author Mathael00): a choice-based SugarCube RPG is looking for testers to compare two versions of the same game — version A uses 1v1 combat, version B supports multiple enemies with target selection. The author asks testers to judge combat pacing, difficulty, target selection, and whether multi-enemy encounters justify their added complexity: https://intfiction.org/t/looking-for-playtesters-comparing-two-different-combat-systems-for-twine-sugarcube/81730

### How to report on an A/B combat playtest

If you take part, structure the comparison so the author can act on it:

1. One line per encounter: where it happens, enemies faced, turns taken, outcome.
2. Difficulty in player terms: where the first defeat happened and what you expected to happen instead.
3. Target-selection friction: how many extra clicks or decisions multi-enemy fights added, and whether any of those choices ever felt meaningless.
4. Answer the author's actual question with specifics: does multi-enemy add enough pacing variety or tactical choice to justify the extra UI complexity?
5. Separate taste from friction. "I preferred version A" is taste; "in version B, selecting a second enemy cost three clicks per round" is a finding.

That same habit — concrete, verifiable observations instead of "it's broken" — is what makes any SugarCube defect report reviewable. If a playtest surfaces a reproducible defect, this repository's issue form takes sanitized repair inquiries: a minimal reproduction, console output, and redacted variable snapshots. The first response is a diagnosis or quote; it is not a contract and does not claim that a repair has been completed.

### IFComp 2026 games by development system (observed 2026-09-02)

Forum thread (posted 2026-09-02T08:41Z by Steve Evans / ybosde): the 2026 update of an Excel spreadsheet listing all IFComp games by development system back to 1995. The author counts 30 games with some form of parser and 41 without this year, and invites corrections. Two corrections followed in-thread: the Z-Code entry "WITHOUT corporation" was corrected from Inform to ZIL after a check of its IFDB page, and "Uninteractive Fiction 3" was amended from Twine to GDevelop JS after a source-HTML check. A reply also noted that "Forensic Necromancy" and "THE RAT" are distributed as both Z-machine and Å-machine files, an unresolved categorization question. Thread: https://intfiction.org/t/ifcomp-2026-games-by-development-system/81843

### How to verify an authoring-system label

1. Read the declared label from public metadata first (comp entry page, IFDB listing) — but treat it as a claim, not a fact.
2. Verify against the artifact itself: view source for engine/version strings (for example SugarCube version markers or Twine build tags) and note file extensions (`.zil`, `.z8`, `.ulx`) that point to a different lineage than the label.
3. Keep distribution format separate from authoring system: a Z-machine file can come from Inform, ZIL, or another compiler. The "WITHOUT corporation" correction in the thread above is exactly this trap in reverse.
4. Record the source URL and the date observed for each label, and log corrections publicly the way the spreadsheet author did, so downstream users know which labels were revised.

The same claim-then-verify habit is what makes a SugarCube defect report reviewable. If a census-verified or playtest-surfaced reproducible defect appears, the issue form takes sanitized repair inquiries (minimal reproduction, console output, redacted variable snapshots); the first response is a diagnosis or quote, not a contract.

### Return-loop from menus: the `<<script>>` scope trap (observed 2026-09-03)

Forum thread (posted 2026-09-03T01:57Z by Sneaky_Proto, Twine 2.12.0 / SugarCube): moving from one menu to the next without returning to the main game caused a return-loop, and a `<<script>>` block assigning `$Return=previous();` never updated `$Return`. Reply (svlin, 2026-09-03T06:49Z): `$Return` is SugarCube markup, not JavaScript — inside a `<<script>>` block it is just text. Assign with `<<set $Return = previous()>>`, or inside the script use `State.variables.Return = previous()`. The reply also points to the "arbitrarily long return" recipe in the SugarCube documentation as the ready-made pattern for this exact problem. Thread: https://intfiction.org/t/how-to-dynamically-compare-the-tags-of-a-target-passage-and-any-future-passage/81869

#### How to keep menu returns from misfiring

1. Never assign story variables inside `<<script>>` with `$name` syntax. Markup does not execute there; use `State.variables.name` (JavaScript) or `<<set $name = ...>>` (markup). If the variable "never updates", suspect that the assignment never ran.
2. Print the value back at the moment you assign it — `console.log(State.variables.Return)` or `<<= $Return>>` — so a silent no-op is visible immediately instead of surfacing as a wrong return target several clicks later.
3. Decide which passages a return link is allowed to target before you store one: this author gated the assignment with `<<if !tags(previous()).includes("menu")>>`, which keeps menu passages from ever becoming a return destination.
4. For longer histories, prefer a documented pattern (the docs' "arbitrarily long return") over an improvised variable, and record the source URL and date of any snippet you adopt so you can re-check it against the docs version you actually use.

The same print-it-back habit is what makes a SugarCube defect report reviewable: if a menu-return or state bug reproduces, the issue form takes sanitized repair inquiries (minimal reproduction, console output, redacted variable snapshots); the first response is a diagnosis or quote, not a contract.

### A five-day jam postmortem worth reading (observed 2026-09-03)

Forum postmortem (posted 2026-09-03T14:59Z by dee_cooke): "A Bonded Postmortem" covers the making of *A Bonded Harvester*, an Iron ChIF 2026 entry built in five days. Process facts worth stealing: a first rough version went to testers at day 2.5, and those same testers then worked through several builds with the author. The post closes with a stats block — planned 29 running miles, ran 18; nightly sleep 8 → 4 hours — instead of a victory narrative. Thread: https://intfiction.org/t/a-bonded-postmortem-seasonal-spice-speed-grief-and-dice/81880

#### How to write a jam postmortem others can act on

1. Record plan vs. actual with numbers, not adjectives: the author's 29 planned / 18 run and 8 → 4 sleep lines say more than "it was intense" ever would.
2. Send a rough build to testers early and iterate with the same testers through several builds, rather than saving testing for one big round at the end.
3. Name the mechanic you deliberately tuned and the trade-off you accepted — here, dice tuned for colour rather than hindrance.
4. State the constraint you worked under (five days, the jam ingredient unknown at planning time) so readers can judge how your advice transfers to their own constraints.

The same record-plan-vs-actual, verify-at-each-step discipline is what makes a repair note reviewable. If a jam-produced or postmortem-surfaced SugarCube defect reproduces, the issue form takes sanitized repair inquiries (minimal reproduction, console output, redacted variable snapshots); the first response is a diagnosis or quote, not a contract and not a claim that a repair has been completed.

### Command feedback that proves nothing (observed 2026-09-04)

Forum thread (posted 2026-09-03T22:25Z by DeusIrae, reviewing the IFComp 2026 entry WITHOUT corporation): the player cannot tell whether play is on track because the in-game hints are not especially helpful — current goals listed as "???", `get-hint` answering "run", manually invoked routines answered with "that's a bad idea" and `RUN ADVANCE` reporting an empty routine, and movement around the map doing nothing. Post 2 (iaraya): stuck at the same place; `GIVE-HINTS` still says "Run." when the routines are off, and it is unclear whether that is an instruction to the player or a machine message that the command has run. Post 4 (RowanMercer): "Run." reads as an in-world hint for the current scene, the machinery and the fiction layer both use the same terse language, and the advice is to re-trace room descriptions and newly-changed exits/objects after the last fourth-wall event rather than forcing the ADVANCE/CHASE routines directly. Thread: https://intfiction.org/t/without-corporation-hints/81887

#### How to tell "the command ran" from "the state changed"

1. Never accept a command's reply as evidence of effect. A response like "Run." can be an in-world hint, a refusal, or a no-op; the reply itself does not distinguish the three.
2. Re-read observable state after every command: current goals, room description, exits, and any objects that changed after the last major event. If nothing moved, treat the command as a no-op regardless of what it said.
3. Compare before/after snapshots instead of memory: capture the variables, passage, and visible exits before a suspicious command and diff them against after. The `console.table` capture in section 3 of this checklist is the smallest version of that habit.
4. When a routine reports it is empty or unavailable, log which invocation produced the message and what state preceded it; re-running a failed routine and expecting a different outcome hides the precondition that was never met.

The same observe-don't-trust-the-reply habit is what makes a SugarCube defect report reviewable: if a feedback-mismatch or state bug reproduces, the issue form takes sanitized repair inquiries (minimal reproduction, console output, redacted variable snapshots); the first response is a diagnosis or quote, not a contract and not a claim that a repair has been completed.

## License

The checklist is available under CC BY 4.0. See `LICENSE`.

— Lantern Loop Studio
