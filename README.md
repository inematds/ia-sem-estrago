# 🛞 Fora do volante

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![Fora do volante](guia/assets/banner.jpg)](https://inematds.github.io/fora-do-volante/guia/)

Você pode parar de vigiar o Claude, de clicar em permitir e de descobrir o erro só no fim: o time do Claude Code já parou. Em maio eles faziam 2.500 mudanças de código por dia; em setembro, mais de 30.000. O método é sair do volante, em três trocas.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/fora-do-volante/guia/**

## O que é

Quem usa o Claude Code costuma ficar olhando cada passo, clica em "permitir" a toda hora e só descobre no fim que a tarefa não estava pronta. **Fora do volante** é um método em três trocas, tirado de uma palestra do time do Claude Code, na Anthropic, e um kit grátis para copiar no seu projeto. Serve para quem usa o Claude Code no dia a dia, com código ou com texto. Para entender, basta ler o guia; para usar o kit, você precisa do Claude Code e do Python 3.

## O método

Três trocas, todas no mesmo exemplo (mudar o login para uma API nova):

1. **Pare de vigiar** — CLAUDE.md só com o que o Claude não descobre lendo o código, sem IMPORTANTE em maiúscula (`kit/CLAUDE.md.exemplo`, `/doctor prompt-audit`).
2. **Pare de aprovar tudo** — auto mode no dia a dia e, para o que não tem volta, o **teste do desfazer**: vira regra de bloqueio na configuração, que vale antes do auto mode e em todos os modos. O sandbox fecha o resto (`kit/teste-do-desfazer.md`).
3. **Confira no caminho** — diga como é pronto, um teste por tarefa, e o hook que **sai com 2** quando a checagem falha (com 1, o Claude segue como se estivesse feito) (`kit/como-e-pronto.md`).

No estudo mostrado na palestra, as pessoas que aprovavam comando por comando pegaram 14% dos comandos perigosos; o auto mode barrou 89%. Esse estudo foi só mostrado na palestra; não está na página oficial da Anthropic.

**A prova:** o kit rodou de verdade no Claude Code 2.1.326. O Claude disse "pronto", o hook rodou o teste, viu a falha e saiu com 2; o Claude voltou, achou o erro, consertou e só então parou (`exemplo/PROVA.md`).

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
| `guia/` | a página do guia em PT, EN e ES (GitHub Pages) |

## O processo, do início ao fim

| # | Passo | Comando | Quem |
|---|---|---|---|
| 0 | Baixar o kit | `git clone https://github.com/inematds/fora-do-volante.git && cd fora-do-volante` | 🧠 você |
| 1 | Conferir o seu projeto | `python3 ferramentas/conferir_volante.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar a configuração e o hook | `cp -rn kit/.claude ~/meu-projeto/` (não sobrescreve; se já tiver `settings.json`, junte as chaves à mão; ponha `.claude/.pronto-bloqueios` no `.gitignore`) | 🧠 você |
| 3 | Dizer qual é a checagem de "pronto" | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (troque pelo comando de teste do seu projeto) | 🧠 você |
| 4 | Ligar o auto mode e ajustar à sua casa | `claude --permission-mode auto` (ou Shift+Tab até "⏵⏵ auto mode on"; padrão desde a 2.1.283), depois `/auto-mode-setup` (rascunha `autoMode.environment`) | 🧠 você + Claude |
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

Grátis no [inema.club](https://inema.club), direto na internet. Licença MIT.
