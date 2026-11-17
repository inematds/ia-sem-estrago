# 🛞 Fora do volante

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Você pede uma tarefa para a IA e fica vigiando, clicando em permitir, e descobre o erro só no fim. O time do Claude Code, na Anthropic, parou de trabalhar assim: em maio eles faziam 2.500 mudanças de código por dia; em setembro, mais de 30.000. **Fora do volante** é o jeito deles em três trocas, num kit para copiar no seu projeto.

## O que é

Três trocas, todas no mesmo exemplo (mudar o login para uma API nova):

1. **Pare de vigiar** — CLAUDE.md só com o que o Claude não descobre lendo o código, sem IMPORTANTE em maiúscula (`kit/CLAUDE.md.exemplo`, `/doctor prompt-audit`).
2. **Pare de aprovar tudo** — auto mode no dia a dia e, para o que não tem volta, o **teste do desfazer**: vira regra de bloqueio na configuração, que vale antes do auto mode e em todos os modos. O sandbox fecha o resto (`kit/teste-do-desfazer.md`).
3. **Confira no caminho** — diga como é pronto, um teste por tarefa, e o hook que **sai com 2** quando a checagem falha (com 1, o Claude segue como se estivesse feito) (`kit/como-e-pronto.md`).

## O kit

| Arquivo | Para quê |
|---|---|
| `kit/.claude/settings.json` | regras de bloqueio (.env, ~/.ssh, push forçado, `rm -rf`, `reset --hard`), sandbox ligado, hooks `Stop` e `TaskCompleted` |
| `kit/.claude/hooks/pronto.sh` | roda a checagem; falhou → saída 2 com o motivo; no máximo 3 voltas seguidas |
| `kit/teste-do-desfazer.md` | a pergunta que decide o que vai para a lista de bloqueio |
| `kit/como-e-pronto.md` | modelo de pedido com "pronto é…" e checagem por tarefa |
| `kit/CLAUDE.md.exemplo` | CLAUDE.md enxuto |
| `ferramentas/conferir_volante.py` | confere um projeto nas três trocas (só lê) |
| `exemplo/` | o login nas três versões (esqueceu a API velha, deslogou todo mundo, certo) + `PROVA.md` com o hook rodando de verdade no Claude Code |

## O processo, do início ao fim

| # | Passo | Comando | Quem |
|---|---|---|---|
| 1 | Conferir o seu projeto | `python3 ferramentas/conferir_volante.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar a configuração e o hook | `cp -r kit/.claude ~/meu-projeto/` (se já tiver `settings.json`, junte as chaves à mão) | 🧠 você |
| 3 | Dizer qual é a checagem de "pronto" | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` | 🧠 você |
| 4 | Ajustar o auto mode à sua casa | no Claude Code: `/auto-mode-setup` (rascunha `autoMode.environment`) | 🧠 você + Claude |
| 5 | Ligar e testar o sandbox | `/sandbox` (Linux/WSL2: `bubblewrap` e `socat`) | 🧠 você |
| 6 | Enxugar o CLAUDE.md | `/doctor prompt-audit` e o teste "ele descobriria isso lendo o código?" | 🧠 você + Claude |
| 7 | Ver o hook cobrar | `bash exemplo/rodar.sh` | ⚙️ script |
| 8 | Conferir de novo | passo 1 → `fora do volante: ok` | ⚙️ script |

## Testes

```bash
python3 -m unittest discover -s tests -v   # 11 testes, só Python 3
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## De onde vem

Da palestra "Rethinking Claude Code Best Practices", de Lydia Hallie (time do Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es — sintaxe conferida na documentação oficial do Claude Code (permissions, sandboxing, hooks, auto mode), 17/11/2026.

Grátis no [inema.club](https://inema.club). Licença MIT.
