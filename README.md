# mini-devkit

A Claude Code plugin with a fixed development cycle, the same in every project:

| Piece | What it does |
|---|---|
| `setup` skill | Prepares a project for the cycle: git, CLAUDE.md, permissions, product documents. Run `/mini-devkit:setup`. |
| `grilling` skill | Interviews you about an idea, a round of questions at a time, until nothing is left open. |
| `document` skill | Owns `BACKLOG.md` (ideas), `SPEC.md` (how it works today) and `CHANGELOG.md` (what changed). |
| `build` skill | Builds a backlog feature test-first, then has it checked by the `tester` agent before committing. |
| `tester` agent | Independent tester: runs the suite, checks every rule is covered, tries to break the feature. |

## Install

```
/plugin marketplace add ~/Desktop/_mini-devkit
/plugin install mini-devkit@mini-devkit
```

From GitHub: `/plugin marketplace add ramonk2o/mini-devkit`, then the same install command.

After changing the skills here, run `/plugin marketplace update mini-devkit` and bump `version` in `plugins/mini-devkit/.claude-plugin/plugin.json` so installed copies pick it up.

## Use in a project

1. Open Claude Code in the project and run `/mini-devkit:setup`.
2. Then just talk: "note this idea…", "let's build X", "X has a bug…", "release 1.2.0". The CLAUDE.md written by setup tells Claude which skill each step uses.

## Credits

`grilling` is from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see `plugins/mini-devkit/skills/grilling/LICENSE`).

## License

MIT, see `LICENSE`. `grilling` keeps its own MIT license from Matt Pocock.
