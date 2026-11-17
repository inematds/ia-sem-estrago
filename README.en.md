# 🛡️ No-Damage AI

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![No-Damage AI](guia/assets/banner-en.jpg)](https://inematds.github.io/ia-sem-estrago/guia/en/)

**AI works on its own without damaging anything you can’t undo.** You assign a task and go do something else. When you come back, it’s done and tested. No folders have been deleted, no passwords have been read, and Claude hasn’t said “done” before it really was.

## 📖 Guide

To understand what the kit does and how it can change your day, without the technical details: **https://inematds.github.io/ia-sem-estrago/guia/en/**

## What it is

The kit adds two protections to Claude Code:

- **The block:** stops actions you can’t undo, such as deleting an entire folder, reading passwords, or deleting the project history. It works in any mode, even if the AI tries. It uses one question to decide: *“If the AI gets this wrong once, can I undo it?”*
- **The check before “done”:** you define what “done” means once, and a test checks it every time the AI tries to finish. If the check fails, it goes back and fixes the problem.

With both protections in place, you can turn off “allow” and walk away. Anything you can undo continues on its own in Claude Code’s auto mode.

**Where the idea comes from:** a talk by the Claude Code team at Anthropic, who work this way. They went from 2,500 to more than 30,000 code changes a day. In the study shown in the talk, people who approved commands one by one caught only 14% of dangerous commands; auto mode blocked 89%.

## How to get started (no technical skills needed)

Open Claude Code or Codex in your project folder and paste this:

```text
Read the “No-Damage AI” kit at https://github.com/inematds/ia-sem-estrago (the README explains everything, including installation).
Then look at this project and answer in simple English:
1. What could not be undone if you made a mistake here (deleting something, passwords, sending something outside)?
2. What test or check proves that a task is complete in this project?
3. Which parts of the kit are useful for me, and which are not?
Don’t change anything yet. Show me the plan and wait for me to say yes.
```

The AI shows you the plan. You approve what you want, and it installs only those parts. The kit is designed for Claude Code; Codex can also read it and tell you whether it suits your project.

---

## Technical installation

For those who prefer to install it manually. Everything is plain text to copy; nothing installs itself.

### Prerequisites

- **Claude Code** with your project open (`claude --version`). Tested with version 2.1.326; `/auto-mode-setup`, `/sandbox`, and `/doctor prompt-audit` were used in that version.
- **Python 3 and git**, for the checker, tests, and downloading the kit.
- **Sandbox on Linux/WSL2:** requires `bubblewrap` and `socat` (`sudo apt install bubblewrap socat`). Nothing needs to be installed on macOS.

### Files

| File | Purpose |
|---|---|
| `kit/.claude/settings.json` | blocking rules (`.env`, `~/.ssh`, forced push, `rm -rf`, `reset --hard`), sandbox enabled, `Stop` and `TaskCompleted` hooks |
| `kit/.claude/hooks/pronto.sh` | runs the check; on failure, exits with code 2 and the reason; at most 3 consecutive retries |
| `kit/teste-do-desfazer.md` | the question that determines what goes on the block list |
| `kit/como-e-pronto.md` | request template with “done means…” and a task check |
| `kit/CLAUDE.md.exemplo` | concise CLAUDE.md, without uppercase IMPORTANT warnings |
| `ferramentas/conferir.py` | checks a project (CLAUDE.md, blocking rules + sandbox, done hook); read-only |
| `exemplo/` | one login in three versions (forgot the old API, logged everyone out, correct) + `PROVA.md` with the hook running for real |
| `guia/` | the guide page in PT, EN, and ES (GitHub Pages) |

### How it works

`Request with “done means…”` → `Claude works in auto mode` → `block + sandbox stop anything that can’t be undone` → `Claude tries to stop` → `hook runs the check` → `failed: exits with 2 and tries again · passed: done`

- **Permission order:** `deny` (blocks), `ask` (asks), `allow` (allows). Claude Code checks `deny` first, before the auto mode checker, in every mode, including `bypassPermissions`.
- **The rule checks the command text**, not its effects. That’s why the kit also enables the **sandbox**, where commands can only read folders and connect to approved sites (`github.com`, `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`). If your project connects to another site, add its domain to `sandbox.network.allowedDomains`.
- **Why `Stop` and `TaskCompleted`:** these are the points when Claude tries to stop or close a task. `PostToolUse` cannot block a command, even with exit code 2, because the tool has already run.
- **Exit code 2, not 1:** with 2, Claude Code returns the error to Claude, which goes back and fixes it. With 1, it only shows the error, and Claude continues as if the task were done.
- **Loop protection:** `pronto.sh` blocks at most 3 consecutive times (`PRONTO_MAX`), then lets Claude stop so you can take a look.
- **Where the check comes from**, in this order: the `PRONTO_CMD` variable, the `.claude/pronto.cmd` file (one line), or automatic detection of `npm test` / `python3 -m unittest`. If there is nothing to check, it does not block. If you combine it with a `grep`, check that the folder exists: `! grep -rn "x" src/` always passes when there is no `src/` folder.

## The process, from start to finish

Replace `~/meu-projeto` with your project folder. ⚙️ = script · 🧠 = you (sometimes with Claude).

| # | Step | Command | Who |
|---|---|---|---|
| 0 | Download the kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 you |
| 1 | Check your project (failures the first time are normal) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copy the configuration and hook | `cp -rn kit/.claude ~/meu-projeto/` and `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (`-n` does not overwrite; if `settings.json` already exists, merge the keys manually) | 🧠 you |
| 3 | Set the “done” check | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (replace it with your project’s test) | 🧠 you |
| 4 | Turn on auto mode and describe your environment | `claude --permission-mode auto` (or press Shift+Tab until “⏵⏵ auto mode on”; the default since 2.1.283), then `/auto-mode-setup` | 🧠 you + Claude |
| 5 | Turn on and test the sandbox | `/sandbox` | 🧠 you |
| 6 | Trim down CLAUDE.md | `/doctor prompt-audit` and, for each line, ask “Could it figure this out by reading the code?” | 🧠 you + Claude |
| 7 | Check that the hook catches the problem | `bash exemplo/rodar.sh` (forgot: 2 · logged out: 2 · correct: 0) | ⚙️ script |
| 8 | Check again | step 1 → `ia sem estrago: ok` | ⚙️ script |

The checker reads the files; it cannot tell whether you ran steps 4 and 5 inside Claude Code.

## Tests

```bash
python3 -m unittest discover -s tests -v   # 11 tests, Python 3 only
bash exemplo/rodar.sh                       # forgot: 2 · logged out: 2 · correct: 0
```

## The proof

The hook ran for real in Claude Code 2.1.326, using the subscription, on 17/11/2026. With a hidden error (the sum was subtracting), Claude said “done”; the hook ran the test, found the failure, and exited with code 2. Claude went back, found the error, fixed it, and only then stopped. Record: `exemplo/PROVA.md`.

## Where the idea comes from

Talk: “Rethinking Claude Code Best Practices” by Lydia Hallie (Claude Code team, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Syntax checked against the official Claude Code documentation (permissions, sandboxing, hooks, auto mode) on 17/11/2026. The 14% and 89% study was only shown in the talk.

Free at [inema.club](https://inema.club), right on the internet. MIT License.