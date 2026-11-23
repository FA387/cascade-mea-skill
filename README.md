# CASCADE skill for AI assistants

An Agent Skill that teaches an AI assistant how to run [CASCADE](https://github.com/FA387/cascade-mea), choose filters for noisy recordings and explain the results. The same folder works in **Claude Code**, **Cursor**, **OpenAI Codex** and **Google Antigravity**, and in other tools that read the Agent Skills format.

CASCADE (Criticality Avalanche Spike Cross-platform Analysis Detection Engine) analyses multi-electrode array (MEA) spike recordings from 3Brain, Maxwell Biosystems, Cortical Labs, Multi Channel Systems, Axion BioSystems and FinalSpark. It must be installed first ([installation](https://github.com/FA387/cascade-mea#installation)); the assistant runs the `cascade` commands for you.

## Install

Copy the `skills/cascade-mea` folder into the skills folder of your tool. Keep the folder name `cascade-mea`.

| Tool | All projects | One project only |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `<project>/.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `<project>/.cursor/skills/` or `<project>/.agents/skills/` |
| Codex | `~/.agents/skills/` | `<project>/.agents/skills/` |
| Antigravity | `~/.gemini/config/skills/` | `<project>/.agents/skills/` |

For example, on macOS, Linux or WSL, for all projects:

```bash
git clone https://github.com/FA387/cascade-mea-skill.git
cp -r cascade-mea-skill/skills/cascade-mea ~/.claude/skills/          # Claude Code
cp -r cascade-mea-skill/skills/cascade-mea ~/.cursor/skills/          # Cursor
cp -r cascade-mea-skill/skills/cascade-mea ~/.agents/skills/          # Codex (Cursor reads it too)
cp -r cascade-mea-skill/skills/cascade-mea ~/.gemini/config/skills/   # Antigravity
```

Create the target folder first if it does not exist (`mkdir -p ~/.cursor/skills`). Cursor also reads `~/.claude/skills/` and `~/.agents/skills/`. A project's `.agents/skills/` folder is read by Cursor, Codex and Antigravity.

To update: `git pull` in `cascade-mea-skill`, then copy the folder again.

## Use

Ask in plain words, for example "analyse this 3Brain recording", "is my culture critical?" or "which filters should I use for this noisy MEA file?". The assistant loads the skill when the request matches. Most tools also let you call it by name: `/cascade-mea` (in Codex: `$cascade-mea`).

The skill describes CASCADE v0.9.6. Check your version with `cascade --version`.

## What is inside

```
skills/cascade-mea/
├── SKILL.md                      when to use it, how to run and interpret CASCADE
└── references/
    ├── installation.md           step-by-step install for non-programmers
    ├── file-formats.md           what each manufacturer's file contains
    ├── noisy-recordings.md       choosing --ae, --max_rate, --max_rpv, --max_amplitude, --stim_blank
    ├── flags.md                  every flag of cascade, cascade-info, cascade-ltpltd
    ├── outputs.md                every output file and figure
    └── links.md                  CASCADE, manufacturer docs, method references
```

## Contributing

Improvements are welcome: a clearer explanation, a workflow from your lab, a device tip, or a fix where the skill and CASCADE disagree.

1. Fork this repository and edit the files in `skills/cascade-mea/`.
2. Keep facts in line with CASCADE itself: flag names and defaults as in `cascade --help`, output names as in a real run.
3. Open a pull request saying what you changed and which CASCADE version you checked it with.

Questions or problems with the skill: open an issue here. Problems with CASCADE itself: [CASCADE issues](https://github.com/FA387/cascade-mea/issues).

## Citing

If the skill helps your work, please cite CASCADE: Avila F, Kim J, Park J-C, Kang R, Lee H, Park S-H. CASCADE: criticality avalanche spike cross-platform analysis detection engine, a multi-manufacturer MEA bash analysis pipeline. *Bioinformatics*. 2026;42(8):btag570. https://doi.org/10.1093/bioinformatics/btag570

## Licence

MIT, © 2026 Forbes Avila (KIT, UST). The flag and output descriptions are adapted from the CASCADE README (Apache 2.0).
