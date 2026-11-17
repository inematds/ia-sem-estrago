# 🛞 Suelta el volante

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![Suelta el volante](guia/assets/banner-es.jpg)](https://inematds.github.io/fora-do-volante/guia/es/)

Puedes dejar de vigilar a Claude, de hacer clic en permitir y de descubrir el error recién al final: el equipo de Claude Code ya lo dejó. En mayo hacían 2,500 cambios de código por día; en septiembre, más de 30,000. El método consiste en soltar el volante en tres cambios.

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/fora-do-volante/guia/es/**

## Qué es

Quienes usan Claude Code suelen mirar cada paso, hacer clic en «permitir» todo el tiempo y descubrir recién al final que la tarea no estaba lista. **Suelta el volante** es un método de tres cambios, tomado de una charla del equipo de Claude Code en Anthropic, y un kit gratuito para copiar en tu proyecto. Sirve para quienes usan Claude Code a diario, con código o con texto. Para entenderlo, basta con leer la guía; para usar el kit, necesitas Claude Code y Python 3.

## El método

Tres cambios, todos con el mismo ejemplo (cambiar el inicio de sesión a una API nueva):

1. **Deja de vigilar** — CLAUDE.md solo con lo que Claude no puede descubrir al leer el código, sin IMPORTANTE en mayúsculas (`kit/CLAUDE.md.exemplo` (en portugués), `/doctor prompt-audit`).
2. **Deja de aprobar todo** — auto mode en el día a día y, para lo irreversible, la **prueba del deshacer**: se convierte en una regla de bloqueo en la configuración, que se aplica antes del auto mode y en todos los modos. El sandbox cierra el resto (`kit/teste-do-desfazer.md` (en portugués)).
3. **Verifica durante el proceso** — indica qué significa que está listo, una prueba por tarea y el hook que **sale con 2** cuando falla la verificación (con 1, Claude sigue como si ya estuviera hecho) (`kit/como-e-pronto.md` (en portugués)).

En el estudio presentado en la charla, quienes aprobaban cada comando por separado detectaron el 14% de los comandos peligrosos; auto mode bloqueó el 89%. Ese estudio solo se presentó en la charla; no está en la página oficial de Anthropic.

**La prueba:** el kit se ejecutó de verdad en Claude Code 2.1.326. Claude dijo «listo», el hook ejecutó la prueba, detectó el fallo y salió con 2; Claude volvió, encontró el error, lo corrigió y recién entonces se detuvo (`exemplo/PROVA.md` (en portugués)).

## El kit

| Archivo | Para qué sirve |
|---|---|
| `kit/.claude/settings.json` | reglas de bloqueo (.env, ~/.ssh, push forzado, `rm -rf`, `reset --hard`), sandbox activado, hooks `Stop` y `TaskCompleted` |
| `kit/.claude/hooks/pronto.sh` | ejecuta la verificación; si falla → salida 2 con el motivo; máximo 3 vueltas seguidas |
| `kit/teste-do-desfazer.md` | la pregunta que decide qué va a la lista de bloqueo |
| `kit/como-e-pronto.md` | modelo de solicitud con «listo significa…» y verificación por tarea |
| `kit/CLAUDE.md.exemplo` | CLAUDE.md breve |
| `ferramentas/conferir_volante.py` | verifica un proyecto según los tres cambios (solo lectura) |
| `exemplo/` | el inicio de sesión en tres versiones (olvidó la API anterior, cerró la sesión de todos, correcto) + `PROVA.md` con el hook ejecutándose de verdad en Claude Code |
| `guia/` | la página de la guía en PT, EN y ES (GitHub Pages) |

## El proceso, de principio a fin

| # | Paso | Comando | Quién |
|---|---|---|---|
| 0 | Descargar el kit | `git clone https://github.com/inematds/fora-do-volante.git && cd fora-do-volante` | 🧠 tú |
| 1 | Verificar tu proyecto | `python3 ferramentas/conferir_volante.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar la configuración y el hook | `cp -rn kit/.claude ~/meu-projeto/` (no sobrescribe; si ya tienes `settings.json`, combina las claves manualmente; agrega `.claude/.pronto-bloqueios` a `.gitignore`) | 🧠 tú |
| 3 | Indicar cuál es la verificación de «listo» | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` | 🧠 tú |
| 4 | Adaptar auto mode a tu entorno | en Claude Code: `/auto-mode-setup` (esboza `autoMode.environment`) | 🧠 tú + Claude |
| 5 | Activar y probar el sandbox | `/sandbox` (Linux/WSL2: `bubblewrap` y `socat`) | 🧠 tú |
| 6 | Simplificar CLAUDE.md | `/doctor prompt-audit` y la prueba «¿podría descubrirlo leyendo el código?» | 🧠 tú + Claude |
| 7 | Ver cómo el hook lo exige | `bash exemplo/rodar.sh` | ⚙️ script |
| 8 | Verificar otra vez | paso 1 → `fora do volante: ok` | ⚙️ script |

## Pruebas

```bash
python3 -m unittest discover -s tests -v   # 11 pruebas, solo Python 3
bash exemplo/rodar.sh                       # esqueceu: 2 · deslogou: 2 · certo: 0
```

## Origen

De la charla «Rethinking Claude Code Best Practices», de Lydia Hallie (equipo de Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es — sintaxis verificada en la documentación oficial de Claude Code (permissions, sandboxing, hooks, auto mode), 17/11/2026.

Gratis en [inema.club](https://inema.club), directamente en internet. Licencia MIT.