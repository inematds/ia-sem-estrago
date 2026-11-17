# 🛞 Hands off the wheel

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![Hands off the wheel](guia/assets/banner-en.jpg)](https://inematds.github.io/fora-do-volante/guia/en/)

You can stop watching Claude, clicking allow, and finding out only at the end that something went wrong: the Claude Code team already has. In May, they were making 2,500 code changes a day; by September, more than 30,000. The method is to take your hands off the wheel in three changes.

## 📖 Usage guide

Complete guide (landing page + step-by-step): **https://inematds.github.io/fora-do-volante/guia/en/**

## What it is

People who use Claude Code often watch every step, click “allow” all the time, and only find out at the end that the task wasn’t done. **Hands off the wheel** is a method in three changes, drawn from a talk by the Claude Code team at Anthropic, and a free kit to copy into your project. It’s for people who use Claude Code every day, with code or text. To understand it, just read the guide; to use the kit, you need Claude Code and Python 3.

## The method

Three changes, all using the same example (switching the login to a new API):

1. **Stop watching** — CLAUDE.md with only what Claude can’t discover by reading the code, without IMPORTANT in uppercase (`kit/CLAUDE.md.exemplo` (in Portuguese), `/doctor prompt-audit`).
2. **Stop approving everything** — auto mode for day-to-day work and, for anything irreversible, the **undo test**: it becomes a blocking rule in the configuration, which applies before auto mode and in every mode. The sandbox handles the rest (`kit/teste-do-desfazer.md` (in Portuguese)).
3. **Check along the way** — say what done looks like, one test per task, and a hook that **exits with 2** when the check fails (with 1, Claude carries on as if the task were done) (`kit/como-e-pronto.md` (in Portuguese)).

In the study shown in the talk, people who approved commands one by one caught 14% of dangerous commands; auto mode blocked 89%. That study was only shown in the talk; it isn’t on Anthropic’s official website.

**The proof:** the kit ran for real in Claude Code 2.1.326. Claude said “done,” the hook ran the test, found the failure, and exited with 2; Claude came back, found the error, fixed it, and only then stopped (`exemplo/PROVA.md` (in Portuguese)).

## The kit

| File | What it’s for |
|---|---|
| `kit/.claude/settings.json` | blocking rules (.env, ~/.ssh, forced push, `rm -rf`, `reset --hard`), sandbox enabled, `Stop` and `TaskCompleted` hooks |
| `kit/.claude/hooks/pronto.sh` | runs the check; on failure → exit code 2 with the reason; at most 3 consecutive rounds |
| `kit/teste-do-desfazer.md` | the question that decides what goes on the blocklist |
| `kit/como-e-pronto.md` | request template with “done means…” and a check for each task |
| `kit/CLAUDE.md.exemplo` | lean CLAUDE.md |
| `ferramentas/conferir_volante.py` | checks a project against the three changes (read-only) |
| `exemplo/` | the login in three versions (forgot the old API, logged everyone out, correct) + `PROVA.md` with the hook running for real in Claude Code |
| `guia/` | the guide page in PT, EN, and ES (GitHub Pages) |

## The process, from start to finish

| # | Step | Command | Who |
|---|---|---|---|
| 0 | Download the kit | `git clone https://github.com/inematds/fora-do-volante.git && cd fora-do-volante` | 🧠 you |
| 1 | Check your project | `python3 ferramentas/conferir_volante.py ~/my-project` | ⚙️ script |
| 2 | Copy the configuration and hook | `cp -rn kit/.claude ~/my-project/` (does not overwrite; if you already have `settings.json`, merge the keys by hand; add `.claude/.pronto-bloqueios` to `.gitignore`) | 🧠 you |
| 3 | Specify the “done” check | `echo "python3 -m unittest discover -s tests -q" > ~/my-project/.claude/pronto.cmd` (replace with your project’s test command) | 🧠 you |
| 4 | Turn on auto mode and adapt it to your setup | `claude --permission-mode auto` (or press Shift+Tab until “⏵⏵ auto mode on”; default since 2.1.283), then `/auto-mode-setup` (drafts `autoMode.environment`) | 🧠 you + Claude |
| 5 | Turn on and test the sandbox | `/sandbox` (Linux/WSL2: `bubblewrap` and `socat`) | 🧠 you |
| 6 | Trim CLAUDE.md | `/doctor prompt-audit` and the test “could it discover this by reading the code?” | 🧠 you + Claude |
| 7 | See the hook enforce the check | `bash exemplo/rodar.sh` | ⚙️ script |
| 8 | Check again | step 1 → `fora do volante: ok` | ⚙️ script |

## Tests

```bash
python3 -m unittest discover -s tests -v   # 11 tests, Python 3 only
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## Where it comes from

From the talk “Rethinking Claude Code Best Practices” by Lydia Hallie (Claude Code team, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es — syntax checked against the official Claude Code documentation (permissions, sandboxing, hooks, auto mode), Nov 17, 2026.

Free at [inema.club](https://inema.club), online. MIT License.