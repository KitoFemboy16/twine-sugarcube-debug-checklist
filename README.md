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

## License

The checklist is available under CC BY 4.0. See `LICENSE`.

— Lantern Loop Studio
