# 🛞 Hands off the wheel

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![Hands off the wheel](guia/assets/banner-en.jpg)](https://inematds.github.io/fora-do-volante/guia/en/)

You can stop watching Claude, clicking to allow things, and discovering errors only at the end: the Claude Code team already has. In May, they made 2,500 code changes a day; by September, more than 30,000. The method is to take your hands off the wheel in three shifts.

## 📖 Guide

Complete guide (landing page + walkthrough): **https://inematds.github.io/fora-do-volante/guia/en/**

## What it is

People who use Claude Code often watch every step, click “allow” all the time, and only discover at the end that the task wasn’t ready. **Hands off the wheel** is a method in three shifts, drawn from a talk by the Claude Code team at Anthropic, plus a free kit to copy into your project. It’s for people who use Claude Code day to day, with code or text. To understand it, just read the guide; to use the kit, you need Claude Code and Python 3.

## The method

Three shifts, all shown in the same example (changing the login to use a new API):

1. **Stop watching** — CLAUDE.md with only what Claude can’t discover by reading the code, with no uppercase IMPORTANT (`kit/CLAUDE.md.exemplo` (in Portuguese), `/doctor prompt-audit`).
2. **Stop approving everything** — auto mode for day-to-day work and, for anything irreversible, the **undo test**: it becomes a blocking rule in the configuration, which applies before auto mode and in every mode. The sandbox closes off the rest (`kit/teste-do-desfazer.md` (in Portuguese)).
3. **Check along the way** — say what “done” looks like, one test per task, and the hook that **exits with 2** when the check fails (with 1, Claude continues as if it were done) (`kit/como-e-pronto.md` (in Portuguese)).

In the study shown in the talk, people who approved commands one by one caught 14% of dangerous commands; auto mode blocked 89%. This study was only shown in the talk; it isn’t on Anthropic’s official page.

**The proof:** the kit ran for real in Claude Code 2.1.326. Claude said “done,” the hook ran the test, detected the failure, and exited with 2; Claude came back, found the error, fixed it, and only then stopped (`exemplo/PROVA.md` (in Portuguese)).

## The kit

| File | What it’s for |
|---|---|
| `kit/.claude/settings.json` | blocking rules (.env, ~/.ssh, forced push, `rm -rf`, `reset --hard`), sandbox enabled, `Stop` and `TaskCompleted` hooks |
| `kit/.claude/hooks/pronto.sh` | runs the check; failure → exit 2 with the reason; at most 3 consecutive rounds |
| `kit/teste-do-desfazer.md` | the question that decides what goes on the block list |
| `kit/como-e-pronto.md` | request template with “done means…” and a check for each task |
| `kit/CLAUDE.md.exemplo` | lean CLAUDE.md |
| `ferramentas/conferir_volante.py` | checks a project across the three shifts (read-only) |
| `exemplo/` | the login in three versions (forgot the old API, logged everyone out, correct) + `PROVA.md` (in Portuguese) with the hook running for real in Claude Code |
| `guia/` | the guide page in PT, EN, and ES (GitHub Pages) |

## The process, from start to finish

| # | Step | Command | Who |
|---|---|---|---|
| 0 | Download the kit | `git clone https://github.com/inematds/fora-do-volante.git && cd fora-do-volante` | 🧠 you |
| 1 | Check your project | `python3 ferramentas/conferir_volante.py ~/meu-projeto` | ⚙️ script |
| 2 | Copy the configuration and hook | `cp -rn kit/.claude ~/meu-projeto/` (does not overwrite; if `settings.json` already exists, merge the keys manually; add `.claude/.pronto-bloqueios` to `.gitignore`) | 🧠 you |
| 3 | Define the “done” check | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` | 🧠 you |
| 4 | Adapt auto mode to your setup | in Claude Code: `/auto-mode-setup` (drafts `autoMode.environment`) | 🧠 you + Claude |
| 5 | Enable and test the sandbox | `/sandbox` (Linux/WSL2: `bubblewrap` and `socat`) | 🧠 you |
| 6 | Trim CLAUDE.md | `/doctor prompt-audit` and the test “would it figure this out by reading the code?” | 🧠 you + Claude |
| 7 | See the hook enforce the check | `bash exemplo/rodar.sh` | ⚙️ script |
| 8 | Check again | step 1 → `fora do volante: ok` | ⚙️ script |

## Tests

```bash
python3 -m unittest discover -s tests -v   # 11 tests, Python 3 only
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## Where it comes from

From the talk “Rethinking Claude Code Best Practices,” by Lydia Hallie (Claude Code team, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es — syntax checked against the official Claude Code documentation (permissions, sandboxing, hooks, auto mode), 17/11/2026.

Free at [inema.club](https://inema.club), directly on the internet. MIT License.