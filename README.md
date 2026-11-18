# 🛡️ IA sem estrago

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![IA sem estrago](guia/assets/banner.jpg)](https://inematds.github.io/ia-sem-estrago/guia/)

**A IA trabalha sozinha e não estraga o que não tem volta.** Baseado na documentação oficial do Claude Code e nas recomendações dos engenheiros da Anthropic. Você pede a tarefa e vai fazer outra coisa. Quando volta, ela está feita e testada. Nenhuma pasta foi apagada, nenhuma senha foi lida, e o Claude não disse "pronto" antes da hora.

## 📖 Guia

Para entender o que o kit faz e como ele muda o seu dia (sem parte técnica): **https://inematds.github.io/ia-sem-estrago/guia/**

## O que é

O kit põe duas proteções no Claude Code:

- **A trava:** barra o que não tem volta (apagar uma pasta inteira, ler senhas, apagar o histórico do projeto), em qualquer modo, mesmo que a IA tente. A regra para decidir é uma pergunta só: *"se a IA errar isso uma vez, dá para voltar atrás?"*
- **O teste antes do "pronto":** você diz uma vez o que é "pronto", e um teste confere isso toda vez que a IA tenta encerrar. Se falhar, ela volta e conserta.

Com as duas, você pode desligar o "permitir" e sair de perto. Tudo o que tem volta segue sozinho, no modo automático do Claude Code.

## Baseado na Anthropic

Nada aqui é invenção nossa. O kit junta, pronto para usar, o que a Anthropic (a empresa que faz o Claude) publica e recomenda:

- **Os engenheiros do Claude Code:** na palestra "Rethinking Claude Code Best Practices", Lydia Hallie, do time do Claude Code, mostra como eles trabalham: param de vigiar, travam o que não tem volta e cobram o "pronto" com teste. Assim passaram de 2.500 para mais de 30.000 mudanças de código por dia. [Assistir](https://www.youtube.com/watch?v=vlfCjWuv4es)
- **O cansaço de aprovar:** quem aprova tudo à mão acaba aceitando 93% dos pedidos; o registro interno da Anthropic tem casos reais de IA apagando ramos do git, enviando uma chave de acesso para fora e mexendo no banco de produção. [Artigo: How we built Claude Code auto mode](https://www.anthropic.com/research/claude-code-auto-mode)
- **A caixa fechada (sandbox):** reduziu os pedidos de permissão em 84%, com mais segurança. [Artigo: Making Claude Code safer with sandboxing](https://www.anthropic.com/research/claude-code-sandboxing)
- **Cada peça na documentação oficial:** a trava = [Permissões](https://code.claude.com/docs/en/permissions); a caixa fechada = [Sandbox](https://code.claude.com/docs/en/sandboxing); o teste antes do "pronto" = [Hooks](https://code.claude.com/docs/en/hooks); o jeito de pedir = [Boas práticas](https://code.claude.com/docs/en/best-practices). Conferido em 17/11/2026.

Os 14% (pessoas que aprovam um por um) e 89% (modo automático) foram mostrados na palestra; os 93% e 84% estão nos artigos oficiais.

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

## E se eu abro o Claude sem restrição?

Muita gente abre o Claude Code assim:

```bash
claude --chrome --dangerously-skip-permissions --exclude-dynamic-system-prompt-sections "$@"
```

**O kit continua valendo.** Testamos em 17/11/2026, no Claude Code 2.1.327, com `--dangerously-skip-permissions`:

| O que pedimos | O que aconteceu |
|---|---|
| ler o `.env` | **negado** pela trava (`Read(./.env)`) |
| `rm -rf lixo` | **negado** pela trava; a pasta ficou lá |
| dizer "pronto" com a checagem falhando | o hook barrou 3 vezes e depois soltou (`PRONTO_MAX`) |
| usar uma ferramenta com regra `ask` | **pediu confirmação** mesmo nesse modo (no `-p`, vira recusa) |

O que muda é outra coisa: sem restrição, **o porteiro do modo automático sai**. No modo `auto`, um segundo modelo olha cada ação que não está na trava e barra o que parece estrago ou envio para fora. Com `--dangerously-skip-permissions`, tudo o que não está na trava passa direto. Por isso:

- **Prefira `claude --permission-mode auto`** (é o padrão desde a 2.1.283). Você tem o mesmo "não pede licença" e ainda o porteiro.
- **Se for usar sem restrição**, a trava do kit é a única coisa entre a IA e o estrago: confira que ela está no projeto (`python3 ferramentas/conferir.py .`). A Anthropic recomenda esse modo só em contêiner ou máquina virtual.
- **`--chrome` abre o navegador**, e o navegador fica fora da caixa fechada (o sandbox só cerca comandos do terminal). Ponha regra `ask` nos conectores e no navegador (`kit/exemplos/conectores.json`).
- `--exclude-dynamic-system-prompt-sections` só mexe no texto de sistema; não muda nenhuma proteção.

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
| `kit/exemplos/pronto-texto.json` | "pronto" para texto: hook `Stop` do tipo `agent` que confere fonte de cada afirmação e número contra `dados.csv` |
| `kit/exemplos/conectores.json` | regras `ask` para conectores e navegador (enviar, apagar, lixeira, remover, publicar) |
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
- **Conectores e navegador** (Gmail, Drive, Slack, `--chrome`): as ferramentas se chamam `mcp__<servidor>__<ação>` (veja os nomes em `/mcp`). O kit sugere `ask`, não `deny`: `ask` continua pedindo confirmação em todo modo, inclusive `--dangerously-skip-permissions`, e você ainda consegue mandar o e-mail quando quiser. Para proibir de vez, mova a regra para `deny`. O `*` vale em qualquer parte do nome (`mcp__*__*send*`).
- **A caixa fechada só cerca comandos do terminal.** As ferramentas Read/Edit e os conectores ficam fora dela; por isso a trava repete `~/.ssh`, `.env` e `secrets/`. No Linux, se aparecer `Sandbox disabled: ... socat not installed`, o sandbox **não está ligado**: instale `bubblewrap` e `socat`.
- **O modo automático deixa ler o `.env` e mandar a senha para a API dela** (é o padrão da Anthropic). Quem quer que a IA nunca veja a senha precisa da trava `Read(./.env)`, que o kit já põe.
- **Diga ao modo automático o que é seu.** O porteiro trata como "de fora" tudo o que não conhece. Liste os seus repositórios, buckets e domínios em `autoMode.environment`, **em `~/.claude/settings.json`** (o Claude Code ignora `autoMode` em `.claude/settings.json` do projeto, para um repositório não se autoliberar):
  ```json
  { "autoMode": { "environment": ["$defaults", "Source control: github.com/minha-org e todos os repos dela"] } }
  ```
  `$defaults` mantém as regras da Anthropic; `/auto-mode-setup` escreve isso para você e `claude auto-mode config` mostra o resultado.
- **Limite do próprio Claude Code:** depois de 8 bloqueios seguidos de hook `Stop`, ele encerra sozinho (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`). O `pronto.sh` para em 3, antes disso.
- **De onde vem a checagem**, nesta ordem: a variável `PRONTO_CMD`, o arquivo `.claude/pronto.cmd` (uma linha) ou a detecção de `npm test` / `python3 -m unittest`. Sem nada para checar, não bloqueia. Se juntar um `grep`, confira que a pasta existe: `! grep -rn "x" src/` passa sempre quando não há `src/`.

## O processo, do início ao fim

Troque `~/meu-projeto` pela pasta do seu projeto. ⚙️ = script · 🧠 = você (às vezes com o Claude).

| # | Passo | Comando | Quem |
|---|---|---|---|
| 0 | Baixar o kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 você |
| 1 | Conferir o seu projeto (falhas na 1ª vez são normais) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar a configuração e o hook | `cp -rn kit/.claude ~/meu-projeto/` e `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (o `-n` não sobrescreve; se já houver `settings.json`, junte as chaves à mão) | 🧠 você |
| 3 | Dizer qual é a checagem de "pronto" (texto: junte `kit/exemplos/pronto-texto.json`) | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (troque pelo teste do seu projeto) | 🧠 você |
| 4 | Ligar o auto mode e descrever a sua casa | `claude --permission-mode auto` (ou Shift+Tab até "⏵⏵ auto mode on"; padrão desde a 2.1.283), depois `/auto-mode-setup` | 🧠 você + Claude |
| 5 | Ligar e testar o sandbox; se usa conectores ou `--chrome`, juntar `kit/exemplos/conectores.json` | `/sandbox` | 🧠 você |
| 6 | Enxugar o CLAUDE.md | `/doctor prompt-audit` e, em cada linha, "ele descobriria isso lendo o código?" | 🧠 você + Claude |
| 7 | Ver o hook cobrar | `bash exemplo/rodar.sh` (esqueceu: 2 · deslogou: 2 · certo: 0) | ⚙️ script |
| 8 | Conferir de novo | passo 1 → `ia sem estrago: ok` | ⚙️ script |

O conferidor lê os arquivos; ele não sabe se você rodou os passos 4 e 5 dentro do Claude Code.

## Testes

```bash
python3 -m unittest discover -s tests -v   # 12 testes, só Python 3
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## A prova

O hook rodou de verdade no Claude Code 2.1.326, pela assinatura, em 17/11/2026. Com um erro escondido (a soma subtraía), o Claude disse "pronto"; o hook rodou o teste, viu a falha e saiu com 2; o Claude voltou, achou o erro, consertou e só então parou. Registro: `exemplo/PROVA.md`.

## De onde vem

Palestra "Rethinking Claude Code Best Practices", de Lydia Hallie (time do Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Sintaxe conferida na documentação oficial do Claude Code (permissions, sandboxing, hooks, auto mode) em 17/11/2026. O estudo dos 14% e 89% foi só mostrado na palestra. Resumo de terceiros da mesma palestra, que inspirou a trava de conectores e o "pronto" de texto: "Anthropic Engineer Explains: The New Way to Use Claude Code", de Nate Herk (PDF); conferimos cada item na documentação antes de pôr no kit.

Grátis no [inema.club](https://inema.club), direto na internet. Licença MIT.
