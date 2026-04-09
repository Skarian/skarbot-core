# reference-module

Use this skill when the user asks to research another git repository by installing it as a submodule in the current repo.
Default behavior pins the submodule to the latest available commit at `HEAD` (or branch `HEAD`) and requires explicit user confirmation each run.

Install (Codex):

```bash
npx skills add https://github.com/Skarian/codex-skills/tree/main/project --skill reference-module -a codex -y
```

Install (Claude Code):

```bash
npx skills add https://github.com/Skarian/codex-skills/tree/main/project --skill reference-module -a claude-code -y
```
