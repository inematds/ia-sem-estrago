# Prova real: o hook com saída 2 no Claude Code (17/11/2026)

Claude Code 2.1.326, pela assinatura, numa pasta descartável com `kit/.claude/` copiado.

- `calc.py` com erro de propósito: `soma(a, b)` devolvia `a - b`.
- `.claude/pronto.cmd`: `python3 -m unittest discover -s tests -q`.
- Pedido: `claude -p "Responda só: pronto." --permission-mode acceptEdits`.

O que aconteceu (5 turnos):
1. O Claude respondeu "pronto" e tentou parar.
2. O hook `Stop` rodou o teste, viu a falha e saiu com **2**, mandando a saída do teste.
3. O Claude voltou, achou o `a - b`, trocou para `a + b`, rodou a checagem de novo e só então parou.

Resposta final do Claude:

> O teste falhava porque `soma()` em `calc.py` subtraía em vez de somar (`a - b`). Troquei para `a + b` e rodei a checagem de novo: o teste passa (`OK`).

Com saída 1 no lugar de 2, o "pronto" do passo 1 teria sido a resposta final.
