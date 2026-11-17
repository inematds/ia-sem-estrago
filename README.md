# 🛡️ IA sem estrago

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![IA sem estrago](guia/assets/banner.jpg)](https://inematds.github.io/ia-sem-estrago/guia/)

**A IA trabalha sozinha e não estraga o que não tem volta.** Você pede a tarefa e vai fazer outra coisa. Quando volta, ela está feita e testada. Nenhuma pasta foi apagada, nenhuma senha foi lida, e o Claude não disse "pronto" antes da hora.

## 📖 Guia

Para entender o que o kit faz e como ele muda o seu dia (sem parte técnica): **https://inematds.github.io/ia-sem-estrago/guia/**

## O que é

O kit põe duas proteções no Claude Code:

- **A trava:** barra o que não tem volta (apagar uma pasta inteira, ler senhas, apagar o histórico do projeto), em qualquer modo, mesmo que a IA tente. A regra para decidir é uma pergunta só: *"se a IA errar isso uma vez, dá para voltar atrás?"*
- **O teste antes do "pronto":** você diz uma vez o que é "pronto", e um teste confere isso toda vez que a IA tenta encerrar. Se falhar, ela volta e conserta.

Com as duas, você pode desligar o "permitir" e sair de perto. Tudo o que tem volta segue sozinho, no modo automático do Claude Code.

**De onde vem:** de uma palestra do time do Claude Code, na Anthropic, que trabalha assim. Eles passaram de 2.500 para mais de 30.000 mudanças de código por dia. No estudo mostrado na palestra, quem aprovava comando por comando pegou só 14% dos comandos perigosos; o modo automático barrou 89%.

## Como começar (sem ser técnico)

Abra o Claude Code, ou o Codex, na pasta do seu projeto e cole:

```text
Leia o kit "IA sem estrago" em https://github.com/inematds/ia-sem-estrago (o README explica tudo, inclusive a instalação).
Depois olhe este projeto e me responda em português simples:
1. O que aqui não teria volta se você errasse (apagar, senhas, enviar algo para fora)?
2. Qual teste ou conferência prova que uma tarefa ficou pronta neste projeto?
3. Quais partes do kit valem a pena para mim, e quais não?
Não mude nada ainda. Mostre o plano e espere eu dizer sim.
```

A IA mostra o plano, você aprova o que quiser e ela instala só isso. O kit é montado para o Claude Code; o Codex também consegue ler e dizer se ele serve para o seu projeto.

---

## Instalação técnica

Para quem prefere instalar à mão. Tudo é texto simples para copiar; nada instala sozinho.

### Pré-requisitos

- **Claude Code** com o seu projeto aberto (`claude --version`). Provado na versão 2.1.326; `/auto-mode-setup`, `/sandbox` e `/doctor prompt-audit` foram usados nessa versão.
- **Python 3 e git**, para o conferidor, os testes e para baixar o kit.
- **Sandbox no Linux/WSL2:** precisa de `bubblewrap` e `socat` (`sudo apt install bubblewrap socat`). No macOS não precisa instalar nada.

### Os arquivos

| Arquivo | Para quê |
|---|---|
| `kit/.claude/settings.json` | regras de bloqueio (`.env`, `~/.ssh`, push forçado, `rm -rf`, `reset --hard`), sandbox ligado, hooks `Stop` e `TaskCompleted` |
| `kit/.claude/hooks/pronto.sh` | roda a checagem; falhou → saída 2 com o motivo; no máximo 3 voltas seguidas |
| `kit/teste-do-desfazer.md` | a pergunta que decide o que vai para a lista de bloqueio |
| `kit/como-e-pronto.md` | modelo de pedido com "pronto é…" e checagem por tarefa |
| `kit/CLAUDE.md.exemplo` | CLAUDE.md enxuto, sem IMPORTANTE em maiúscula |
| `ferramentas/conferir.py` | confere um projeto (CLAUDE.md, bloqueio + sandbox, hook do pronto); só lê |
| `exemplo/` | um login nas três versões (esqueceu a API velha, deslogou todo mundo, certo) + `PROVA.md` com o hook rodando de verdade |
| `guia/` | a página do guia em PT, EN e ES (GitHub Pages) |

### Como funciona

`Pedido com "pronto é…"` → `Claude trabalha no auto mode` → `bloqueio + sandbox barram o que não tem volta` → `Claude tenta parar` → `hook roda a checagem` → `falhou: sai com 2 e volta · passou: pronto`

- **Ordem das permissões:** `deny` (bloqueia), `ask` (pergunta), `allow` (libera). O Claude Code confere `deny` primeiro, antes do verificador do auto mode, em todos os modos, inclusive `bypassPermissions`.
- **A regra olha o texto do comando**, não o efeito. Por isso o kit liga também o **sandbox**, em que o comando só lê as pastas e só fala com os sites liberados (`github.com`, `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`). Se o seu projeto fala com outro site, acrescente o domínio em `sandbox.network.allowedDomains`.
- **Por que `Stop` e `TaskCompleted`:** são os momentos em que o Claude tenta parar ou fechar uma tarefa. `PostToolUse` não bloqueia nem com 2, porque a ferramenta já rodou.
- **Saída 2, não 1:** com 2, o Claude Code devolve o erro ao Claude, que volta e conserta. Com 1, só mostra o erro e o Claude segue como se estivesse feito.
- **Proteção contra laço:** o `pronto.sh` bloqueia no máximo 3 vezes seguidas (`PRONTO_MAX`) e depois deixa parar para você olhar.
- **De onde vem a checagem**, nesta ordem: a variável `PRONTO_CMD`, o arquivo `.claude/pronto.cmd` (uma linha) ou a detecção de `npm test` / `python3 -m unittest`. Sem nada para checar, não bloqueia. Se juntar um `grep`, confira que a pasta existe: `! grep -rn "x" src/` passa sempre quando não há `src/`.

## O processo, do início ao fim

Troque `~/meu-projeto` pela pasta do seu projeto. ⚙️ = script · 🧠 = você (às vezes com o Claude).

| # | Passo | Comando | Quem |
|---|---|---|---|
| 0 | Baixar o kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 você |
| 1 | Conferir o seu projeto (falhas na 1ª vez são normais) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar a configuração e o hook | `cp -rn kit/.claude ~/meu-projeto/` e `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (o `-n` não sobrescreve; se já houver `settings.json`, junte as chaves à mão) | 🧠 você |
| 3 | Dizer qual é a checagem de "pronto" | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (troque pelo teste do seu projeto) | 🧠 você |
| 4 | Ligar o auto mode e descrever a sua casa | `claude --permission-mode auto` (ou Shift+Tab até "⏵⏵ auto mode on"; padrão desde a 2.1.283), depois `/auto-mode-setup` | 🧠 você + Claude |
| 5 | Ligar e testar o sandbox | `/sandbox` | 🧠 você |
| 6 | Enxugar o CLAUDE.md | `/doctor prompt-audit` e, em cada linha, "ele descobriria isso lendo o código?" | 🧠 você + Claude |
| 7 | Ver o hook cobrar | `bash exemplo/rodar.sh` (esqueceu: 2 · deslogou: 2 · certo: 0) | ⚙️ script |
| 8 | Conferir de novo | passo 1 → `ia sem estrago: ok` | ⚙️ script |

O conferidor lê os arquivos; ele não sabe se você rodou os passos 4 e 5 dentro do Claude Code.

## Testes

```bash
python3 -m unittest discover -s tests -v   # 11 testes, só Python 3
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## A prova

O hook rodou de verdade no Claude Code 2.1.326, pela assinatura, em 17/11/2026. Com um erro escondido (a soma subtraía), o Claude disse "pronto"; o hook rodou o teste, viu a falha e saiu com 2; o Claude voltou, achou o erro, consertou e só então parou. Registro: `exemplo/PROVA.md`.

## De onde vem

Palestra "Rethinking Claude Code Best Practices", de Lydia Hallie (time do Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Sintaxe conferida na documentação oficial do Claude Code (permissions, sandboxing, hooks, auto mode) em 17/11/2026. O estudo dos 14% e 89% foi só mostrado na palestra.

Grátis no [inema.club](https://inema.club), direto na internet. Licença MIT.
