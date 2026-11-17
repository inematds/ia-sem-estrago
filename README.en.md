# 🛡️ No-Damage AI

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![No-Damage AI](guia/assets/banner-en.jpg)](https://inematds.github.io/ia-sem-estrago/guia/en/)

**AI works on its own without damaging anything that can’t be undone.** Based on the official Claude Code documentation and recommendations from Anthropic’s engineers. You give it a task and go do something else. When you come back, the task is done and tested. No folders have been deleted, no passwords have been read, and Claude didn’t say “done” before it was time.

## 📖 Guide

To understand what the kit does and how it can change your day (without the technical details): **https://inematds.github.io/ia-sem-estrago/guia/en/**

## What it is

The kit adds two safeguards to Claude Code:

- **The blocker:** stops actions that can’t be undone (deleting an entire folder, reading passwords, deleting the project history) in any mode, even if the AI tries. It uses one question to decide: *“If the AI gets this wrong once, can I undo it?”*
- **The test before “done”:** you define what “done” means once, and a test checks it every time the AI tries to finish. If the test fails, it goes back and fixes the problem.

With both safeguards, you can turn off “allow” and step away. Anything that can be undone proceeds on its own in Claude Code’s auto mode.

## Based on Anthropic’s work

None of this is made up. The kit brings together what Anthropic (the company behind Claude) publishes and recommends, ready to use:

- **Claude Code engineers:** In the talk “Rethinking Claude Code Best Practices,” Lydia Hallie from the Claude Code team shows how they work: they stop watching, block actions that can’t be undone, and require a test before calling something “done.” That’s how they went from 2,500 to more than 30,000 code changes a day. [Watch](https://www.youtube.com/watch?v=vlfCjWuv4es)
- **Approval fatigue:** People who approve everything by hand end up accepting 93% of requests. Anthropic’s internal record includes real cases of AI deleting Git branches, sending an access key outside the organization, and changing the production database. [Article: How we built Claude Code auto mode](https://www.anthropic.com/research/claude-code-auto-mode)
- **The sandbox:** It reduced permission requests by 84% while improving security. [Article: Making Claude Code safer with sandboxing](https://www.anthropic.com/research/claude-code-sandboxing)
- **Every part is in the official documentation:** the blocker = [Permissions](https://code.claude.com/docs/en/permissions); the sandbox = [Sandbox](https://code.claude.com/docs/en/sandboxing); the test before “done” = [Hooks](https://code.claude.com/docs/en/hooks); how to make requests = [Best practices](https://code.claude.com/docs/en/best-practices). Checked on 11/17/2026.

The figures of 14% (people who approve requests one by one) and 89% (auto mode) were shown in the talk; the 93% and 84% figures are in the official articles.

## Getting started (no technical skills needed)

Open Claude Code or Codex in your project folder and paste:

```text
Read the "No-Damage AI" kit at https://github.com/inematds/ia-sem-estrago (the README explains everything, including installation).
Then look at this project and answer in simple English:
1. What could not be undone if you made a mistake here (deleting something, passwords, sending something outside the project)?
2. What test or check proves that a task is complete in this project?
3. Which parts of the kit would be useful to me, and which would not?
Don’t change anything yet. Show me the plan and wait for me to say yes.
```

The AI shows you the plan. You approve what you want, and it installs only those parts. The kit is set up for Claude Code; Codex can also read it and tell you whether it fits your project.

---

## Technical installation

For anyone who prefers to install things by hand. Everything is plain text to copy; nothing installs on its own.

### Requirements

- **Claude Code** with your project open (`claude --version`). Tested with version 2.1.326; `/auto-mode-setup`, `/sandbox`, and `/doctor prompt-audit` were used in that version.
- **Python 3 and git**, for the checker, tests, and downloading the kit.
- **Sandbox on Linux/WSL2:** requires `bubblewrap` and `socat` (`sudo apt install bubblewrap socat`). Nothing needs to be installed on macOS.

### Files

| File | Purpose |
|---|---|
| `kit/.claude/settings.json` | Blocking rules (`.env`, `~/.ssh`, forced pushes, `rm -rf`, `reset --hard`), sandbox enabled, `Stop` and `TaskCompleted` hooks |
| `kit/.claude/hooks/pronto.sh` | Runs the check; if it fails, exits with code 2 and the reason; allows at most 3 consecutive attempts |
| `kit/teste-do-desfazer.md` | The question used to decide what goes on the block list |
| `kit/como-e-pronto.md` | Request template with “done means…” and a check for each task |
| `kit/CLAUDE.md.exemplo` | A short CLAUDE.md without “IMPORTANT” in all caps |
| `ferramentas/conferir.py` | Checks a project (CLAUDE.md, blocking + sandbox, done hook); read-only |
| `exemplo/` | One login in three versions (forgot the old API, logged everyone out, correct) + `PROVA.md` with the hook running for real |
| `guia/` | The guide page in PT, EN, and ES (GitHub Pages) |

### How it works

`Request with “done means…”` → `Claude works in auto mode` → `blocker + sandbox stop actions that can’t be undone` → `Claude tries to stop` → `hook runs the check` → `failed: exits with 2 and goes back · passed: done`

- **Permission order:** `deny` (blocks), `ask` (asks), `allow` (allows). Claude Code checks `deny` first, before the auto mode checker, in every mode, including `bypassPermissions`.
- **The rule checks the command text**, not its effect. That’s why the kit also enables the **sandbox**, where commands can read only the project folders and can connect only to approved sites (`github.com`, `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`). If your project connects to another site, add its domain to `sandbox.network.allowedDomains`.
- **Why `Stop` and `TaskCompleted`:** these are the moments when Claude tries to stop or finish a task. `PostToolUse` cannot block, even with code 2, because the tool has already run.
- **Exit code 2, not 1:** with code 2, Claude Code sends the error back to Claude, which goes back and fixes it. With code 1, it only shows the error and Claude continues as if the task were done.
- **Loop protection:** `pronto.sh` blocks at most 3 times in a row (`PRONTO_MAX`), then lets Claude stop so you can take a look.
- **Where the check comes from**, in this order: the `PRONTO_CMD` variable, the `.claude/pronto.cmd` file (one line), or detection of `npm test` / `python3 -m unittest`. If there is nothing to check, it does not block. If you add a `grep`, make sure the folder exists: `! grep -rn "x" src/` always passes when there is no `src/` folder.

## The process, from start to finish

Replace `~/meu-projeto` with your project folder. ⚙️ = script · 🧠 = you (sometimes with Claude).

| # | Step | Command | Who |
|---|---|---|---|
| 0 | Download the kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 you |
| 1 | Check your project (failures the first time are normal) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copy the configuration and hook | `cp -rn kit/.claude ~/meu-projeto/` and `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (`-n` does not overwrite; if `settings.json` already exists, add the keys by hand) | 🧠 you |
| 3 | Set the “done” check | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (replace it with your project’s test) | 🧠 you |
| 4 | Turn on auto mode and describe your setup | `claude --permission-mode auto` (or press Shift+Tab until “⏵⏵ auto mode on”; default since 2.1.283), then `/auto-mode-setup` | 🧠 you + Claude |
| 5 | Turn on and test the sandbox | `/sandbox` | 🧠 you + Claude |
| 6 | Shorten CLAUDE.md | `/doctor prompt-audit` and, for each line, ask “Could it figure this out by reading the code?” | 🧠 you + Claude |
| 7 | Check that the hook enforces the rule | `bash exemplo/rodar.sh` (forgot: 2 · logged out: 2 · correct: 0) | ⚙️ script |
| 8 | Check again | Step 1 → `ia sem estrago: ok` | ⚙️ script |

The checker reads the files; it cannot tell whether you ran steps 4 and 5 inside Claude Code.

## Tests

```bash
python3 -m unittest discover -s tests -v   # 11 tests, only Python 3
bash exemplo/rodar.sh                       # forgot: 2 · logged out: 2 · correct: 0
```

## The proof

The hook ran for real in Claude Code 2.1.326, using a subscription, on 11/17/2026. With a hidden bug (the sum was subtracting), Claude said “done”; the hook ran the test, found the failure, and exited with code 2. Claude went back, found and fixed the bug, and only then stopped. Record: `exemplo/PROVA.md`.

## Sources

Talk: “Rethinking Claude Code Best Practices,” by Lydia Hallie (Claude Code team, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Syntax checked against the official Claude Code documentation (permissions, sandboxing, hooks, auto mode) on 11/17/2026. The 14% and 89% study was only shown in the talk.

Free at [inema.club](https://inema.club), directly on the internet. MIT License.