# minidev

A Claude Code plugin with a fixed development cycle, the same in every project: ideas are interviewed until nothing is left open, features are built test-first and checked by an independent tester, and three product documents keep track of what may come, what exists and what changed.

## Install

In Claude Code:

```
/plugin marketplace add ramonk2o/minidev
/plugin install minidev@minidev
```

Then, in each project where you want the cycle, run:

```
/minidev:setup
```

To get new versions later: `/plugin marketplace update minidev`.

## What's inside

| Piece | What it does |
|---|---|
| `setup` skill | Prepares a project for the cycle: git, CLAUDE.md, permissions, product documents. Runs only when you call `/minidev:setup`. |
| `grilling` skill | Interviews you about an idea, a round of questions at a time, until nothing is left open. |
| `document` skill | The only writer of `BACKLOG.md`, `SPEC.md` and `CHANGELOG.md`. |
| `build` skill | Builds a backlog feature test-first, then has it checked by the `tester` agent before committing. |
| `tester` agent | Independent tester: runs the suite, checks every rule is covered, tries to break the feature. Never fixes code. |

You don't need to call `grilling`, `document` or `build` by name. Just talk ("note this idea…", "let's build X", "X has a bug…", "release 1.2.0"): the CLAUDE.md written by `setup` tells Claude which skill each step uses.

## How the cycle works

```
 idea ──► grilling (if behavior is open) ──► BACKLOG.md: Idea
                                                  │
                                                  ▼
                                   build: critical test list ──► your OK
                                                  │
                                                  ▼
                                   new branch: feature/<name>
                                                  │
                                                  ▼
                                   tests first (shown failing) ──► code
                                                  │
                                                  ▼
                                   tester agent ◄──► fixes (new tester each round)
                                                  │
                                                  ▼
                       one commit: code + tests + SPEC + CHANGELOG, entry leaves BACKLOG
                                                  │
                                                  ▼
                                   you check by hand what tests can't
                                                  │
                                                  ▼
                                   merge into main, delete branch, push
```

### 1. Setup (once per project)

`/minidev:setup` reads the repository and proposes, all in one message for a single approval:

- `git init` and a `.gitignore`, if the project isn't a repository yet;
- a `CLAUDE.md` with the project's real commands, test conventions and the development cycle (an existing CLAUDE.md is updated, not overwritten);
- `.claude/settings.json` permissions for the test, build and local git commands (including `switch` and `merge`), so the cycle doesn't stop on a prompt for each run (install, publish, push and deletes keep asking);
- what the cycle leaves orphaned: old `TODO.md`/`FEATURES.md`, local skills that overlap the plugin, broken references.

It then creates the missing product documents through `document`. The cycle needs automated tests; if the project has none, setup says so before going on.

### 2. Idea

A clear idea is recorded straight into `BACKLOG.md` as an **Idea**. If it leaves behavior open, Claude starts a `grilling` interview: rounds of numbered questions, each with a recommended answer, asking only what can be decided now and looking up facts itself instead of asking you. The final summary is the backlog entry, so approving it approves what gets recorded. Saying "just note it" skips the interview.

### 3. Build

`build` works on one feature at a time, in a fixed order:

1. **Rules check.** If the entry is missing a few rules, Claude asks them directly; if it's still a bare idea, it offers a `grilling` interview first. Tests built on guessed rules would encode assumptions nobody approved.
2. **Critical test list.** In plain language, no code, split in two: what the test suite will check, and what you'll **check by hand** in the real app (visuals, focus, dragging…). You approve the list; the entry becomes **In progress**.

All the work happens on a `feature/<name>` branch created from an up-to-date main branch, so main only receives finished, verified features.
3. **Tests first.** The automated tests are written and run, and must fail for the right reason: the feature is missing. A test that passes before the code exists isn't testing the feature.
4. **Code.** Written until the critical tests, the whole suite and the build pass. A critical test is never changed to make it pass without asking you.

### 4. Independent tester

The `tester` agent gets only the backlog entry, the diff and the test/build commands: not the conversation or the author's reasoning. Not knowing what was meant is its value. It runs everything, checks that each rule has a test that would fail if the rule broke, and writes extra tests trying to break the feature. It reports:

- **Failures:** Claude fixes the code, keeps the test that caught it and launches a *new* tester, so nothing carries over from the last round.
- **Uncovered rules:** missing tests are added.
- **Assumptions:** tests of behavior nobody decided. They're dropped and the question comes to you.

This repeats until a round has no failures, or stops after 3 rounds to show you what's left.

### 5. Finish

`document` moves the feature out of the backlog: the rules that were actually built go to `SPEC.md` and the change goes to `CHANGELOG.md` under `[Unreleased]`. Code, tests and the three documents go in **one commit**, whose message records the test results, the tester's rounds and a `To check by hand:` block (findable later with `git log --grep "To check by hand"`). You then check those items in the real app; anything that fails there is fixed on the same branch. Once you confirm everything works, the branch is merged into main, deleted and main is pushed. A merge conflict stops the cycle and is shown to you. Anything that fails after the merge is handled as a bug.

### Bugs and releases

- **Bug:** no backlog entry, no tester, on its own `fix/<name>` branch, merged into main and pushed once the fix is confirmed. If it can be reproduced in a test, the test is written and shown failing first, then the fix. The fix is recorded under Fixed in the changelog (and in SPEC.md if a rule changed). If the fix needs a decision about behavior, it's a feature and goes back to the idea step.
- **Release:** "release 1.4.0" moves the `[Unreleased]` changes into a version in `CHANGELOG.md`, after you confirm the version, date and entries. Pushing always asks first.

## The three documents

Each answers one question, so they never repeat each other:

| Document | Answers | Time |
|---|---|---|
| `BACKLOG.md` | What may come, and why you want it | future |
| `SPEC.md` | How the product works **today**: rules, data, technical decisions | present |
| `CHANGELOG.md` | What changed in each version, for the people who use it | past |

`SPEC.md` describes only what exists: it's updated when a feature is finished, from what was built, not when it's planned. Each fact lives in one place, and git history keeps the detail behind all three.

## Developing the plugin

To work on the plugin from a local clone, add it as a marketplace by path instead of the GitHub name:

```
/plugin marketplace add /path/to/minidev
```

After changing the skills, bump `version` in `plugins/minidev/.claude-plugin/plugin.json` and run `/plugin marketplace update minidev` so installed copies pick it up.

## Credits

`grilling` is from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see `plugins/minidev/skills/grilling/LICENSE`).

## License

MIT, see `LICENSE`. `grilling` keeps its own MIT license from Matt Pocock.
