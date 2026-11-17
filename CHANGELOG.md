# Changelog

## 1.1.3 — 17/11/2026
- Novo nome: **IA sem estrago** (antes "Fora do volante"). Repositório `inematds/ia-sem-estrago`; o endereço antigo do GitHub redireciona.
- Guia reescrito para leigos: o ganho (antes e depois), como muda o dia, como funciona sem jargão, como começar pedindo à IA para analisar o kit, a prova e perguntas comuns. A parte técnica saiu do guia e foi para o README ("Instalação técnica").
- `ferramentas/conferir_volante.py` virou `ferramentas/conferir.py`; a saída final é `ia sem estrago: ok`.
- Banners novos (PT/EN/ES) com o novo nome.

## 1.0.3 — 17/11/2026
- Seção "Explicado em 1 minuto" no guia (PT/EN/ES) e "Em 1 minuto" nos 3 READMEs: problema, de onde veio, as três trocas, o kit, a prova e o que é grátis.

## 1.0.2 — 17/11/2026
- `kit/CLAUDE.md.exemplo`: título sem "NÃO" em maiúscula (o modelo seguia a própria regra só pela metade).

## 1.0.1 — 17/11/2026
- `kit/CLAUDE.md.exemplo`: a linha que pedia "sem IMPORTANTE/NUNCA" escrevia as próprias palavras em maiúscula e o conferidor acusava o modelo; agora diz "palavras de alarme em caixa alta".
- Guia publicado (landing + guia de uso em PT, EN e ES) em `guia/`, GitHub Pages por Actions; READMEs em EN e ES.
- Passo 2 com `cp -rn` (não sobrescreve um `settings.json` que já exista).

## 1.0.0 — 17/11/2026
- Kit inicial: settings (deny, sandbox, Stop/TaskCompleted), pronto.sh (saída 2, até 3 voltas), conferir_volante.py, exemplo do login, prova real no Claude Code 2.1.326, 11 testes.
