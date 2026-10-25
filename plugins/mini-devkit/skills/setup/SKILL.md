---
name: setup
description: Set up a software project for the mini-devkit development cycle (grilling, document and build skills, tester agent) by making it a git repository if needed, writing or updating its CLAUDE.md, allowing its test and build commands in the project's permissions, creating the product documents, and pointing out what adopting the cycle left orphaned or redundant. Run only when the user invokes it.
disable-model-invocation: true
---

# Setup

Prepares a project so it works with the development cycle of the `mini-devkit` plugin: the `grilling`, `document` and `build` skills and the `tester` agent, which come with this plugin. The cycle is the same in every project; what changes is the project's own facts (commands, conventions, how tests are written), so the CLAUDE.md has a fixed part, copied from the template below, and a discovered part, found in the repository or asked.

Talk to the user in the language of the conversation. Write the CLAUDE.md in English.

Personal working rules (reply language, how the user likes code, when to use the browser…) are not this skill's business: don't add them, and leave any that an existing CLAUDE.md already has untouched.

Nothing is written before the user's OK. Collect the proposals of steps 1, 5, 6 and 7 and show them together in step 8, so the user approves once.

## 1. Check the project

If the folder isn't a software project, say that this skill is for software projects and stop.

**Git.** The cycle commits after each feature, bug fix and document change, so the project must be a git repository. If it isn't one, propose `git init` and, when there's no `.gitignore`, a short one for the project's stack (dependencies, build output, test reports, `.env`, OS files like `.DS_Store`, and `.claude/settings.local.json`). If it's a repository with no commits yet, note that the first commit will come at the end (step 9).

**Tests.** The cycle needs automated tests: `build` writes them first and `tester` runs them. Find the test framework and command (package.json scripts, Makefile, pyproject, Cargo.toml, go.mod, CI config…). If there's none, say so and ask whether the user wants to set one up first. Don't generate a cycle that can't run.

## 2. Discover

Read the repository to fill the project-specific sections. Look, don't guess:

- **Project line:** what the project is, in one sentence (README, package description, manifest).
- **Commands:** install, develop/watch, build, tests, lint, run or load the app, package/release. Take them from the real scripts; note side effects worth knowing (e.g. install also enables a hook, build also type-checks).
- **Tests:** where test files live and how they're named, shared fakes, mocks or helpers, and how rules that change data or state are tested (find one existing test that does it well, to cite as the example). Also what only shows in the real app (visuals, focus, browser or device behavior): that's what goes to "To check by hand".
- **Conventions:** only rules with clear evidence in the code: a module that centralizes access to something, strict compiler settings, a test that checks consistency between files, a lint rule. Don't invent conventions and don't list what any developer would do anyway.
- **Release:** whether the project publishes versions (a store, a registry, deploys) and where the steps are written (e.g. a `STORE.md`, `RELEASING.md`, a CI workflow).
- **Product documents:** which of `SPEC.md`, `BACKLOG.md`, `CHANGELOG.md` exist, and the language of the project's existing docs.

## 3. Ask

In one short round with a suggested answer for each, ask only what you couldn't find. Always include one question: "What rule would someone new to this project break without knowing it?" Its answers become Conventions. If there are none, the Conventions section is left out.

## 4. Write the draft

Build the file from the template below. Fill every `<…>`; the final file has no placeholders. Leave out a line or section the project has nothing for, as the template notes say.

### Template

````markdown
# <Project name>

<What the project is, in one sentence.>
- How it works today: `SPEC.md`. Ideas and the feature in progress: `BACKLOG.md`. What changed in each version: `CHANGELOG.md`.

## Commands
- Install: `<command>` <(side effects worth knowing)>
- Develop: `<command>`
- Build: `<command>` <(what it also checks, where the output goes)>
- Tests: `<command>` (<framework>)
- Lint: `<command>`
- <Run / Load / Package>: <how>

## Conventions
- <rule with evidence or given by the user>

## Tests
- Tests live <where, naming pattern>.
- <Shared fakes, mocks or helpers, and what they replace.>
- <Rules that change data or state> are tested <how>, as in `<example test file>`. Only what needs <the real app, named for this project: e.g. a real browser, a device> (<what: e.g. focus, dragging, visuals>) goes into "To check by hand".

## Development cycle
Uses the `grilling`, `document` and `build` skills and the `tester` agent from the `mini-devkit` plugin. When a step needs one that isn't available, tell the user and stop; don't imitate it from memory.

1. **Idea:** if the idea is clear, record it with `document`. If it leaves behavior open (decisions that would change what gets built), start a `grilling` interview; before its final summary, load `document` and write the summary as the backlog entry, so my OK approves the text that will be recorded. "Just note it" (<the same phrases in the user's language>) always skips the interview.
2. **Build:** `build`.
3. **Bugs:** no backlog entry or tester. If the bug can be reproduced in a test (see Tests), write the test and show it fails, then fix it. If it needs <the same real app as in Tests>, put the steps to reproduce it in the commit message's `To check by hand:` block. Record it with `document` (CHANGELOG under Fixed; SPEC.md if a rule changed) and commit. If the fix needs a decision about behavior, it's a feature: go to step 1.
4. **Release:** follow <where the release steps live>.
````

Template notes:
- The **Development cycle** section is copied as is. Only the "Just note it" phrases, the name of the real app and the Release target change.
- **Release:** if the project doesn't publish versions, drop step 4. If it does but the steps aren't written anywhere, use "cut the version with `document`" and offer to write the steps down later.
- **Tests:** if no existing test shows how state-changing rules are tested, drop the "as in" part rather than citing a weak example.
- **Conventions:** drop the section when there's nothing to put in it.
- Keep the project's existing specifics that still hold (an extra section, a rule about permissions, a doc pointer): they belong to the project, not to this template.

## 5. Existing CLAUDE.md

If the project already has a `CLAUDE.md` (or `AGENTS.md`, `.claude/CLAUDE.md`), don't overwrite it. Show what changes, section by section: what's added, what the new text replaces, what stays. Keep everything project-specific that's still true.

## 6. Permissions

The cycle runs the same commands many times (tests before and after the code, the build, the tester's rounds, commits), and a permission prompt on each one breaks the flow. Propose `.claude/settings.json` entries under `permissions.allow` for:

- the test, build, lint and type-check commands from step 2, each as a prefix rule so its arguments are covered (e.g. `Bash(npm test:*)`, `Bash(npm run build:*)`, `Bash(npx vitest:*)`);
- the git commands the cycle uses locally: `git status`, `git diff`, `git log`, `git add`, `git commit`.

Never add install, publish, deploy or `git push` commands, or anything that deletes files: those keep asking. If the file exists, merge: keep every existing entry and setting, add only what's missing. Show the resulting list.

## 7. Orphans and redundancies

Check only what adopting the cycle leaves behind. This is not a general cleanup: anything unrelated to the cycle stays as it is, even if it looks unnecessary.

- **Old documents** that now overlap BACKLOG, SPEC or CHANGELOG (`TODO.md`, `FEATURES.md`, `ROADMAP.md`, `HISTORY.md`…). Don't convert them: point them out; aligning them is the `document` skill's job (step 9).
- **Sections of the old CLAUDE.md** that the new text replaces: an older workflow, backlog or test rules that the skills now own, or a line saying the cycle is installed globally in `~/.claude/`.
- **Local skills, agents and commands** (`.claude/skills/`, `.claude/agents/`, `.claude/commands/`) that overlap the cycle: a local build, backlog or tester, or an old copy of one of this plugin's skills. If one has something project-specific the plugin's doesn't cover, propose moving that part into the CLAUDE.md before removing it. Other local skills are just listed.
- **Global copies of the cycle** in `~/.claude/skills/` (`grilling`, `document`, `build`, `bootstrap`) or `~/.claude/agents/tester.md`: they duplicate this plugin and may be older versions. Point them out and suggest removing them; they're outside the project, so this always needs its own explicit OK.
- **Broken references:** the CLAUDE.md (old or new) pointing to a file, script or command that doesn't exist.

For each item, say what it is, why it's orphaned or redundant, and what you suggest (remove, move a part to the CLAUDE.md, leave).

## 8. Review

Show everything in one message: git setup (step 1), the CLAUDE.md draft or its changes (steps 4–5), the permissions (step 6) and the orphans (step 7). Wait for the OK; the user may approve parts.

## 9. Finish

1. Apply what was approved: `git init` and `.gitignore`, the CLAUDE.md, `.claude/settings.json`, the removals.
2. If any of `SPEC.md`, `BACKLOG.md`, `CHANGELOG.md` is missing, or an old document was pointed out in step 7, load the `document` skill and create or align them with it. Don't write them yourself: their formats belong to that skill.
3. Offer a commit with the changes (the first commit, in a new repository). Don't commit without the OK.
