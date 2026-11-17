# 🛡️ IA sin daños

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![IA sin daños](guia/assets/banner-es.jpg)](https://inematds.github.io/ia-sem-estrago/guia/es/)

**La IA trabaja sola y no estropea lo que no tiene vuelta atrás.** Le pides la tarea y te ocupas de otra cosa. Cuando vuelves, está hecha y probada. No se ha borrado ninguna carpeta, no se ha leído ninguna contraseña y Claude no ha dicho «listo» antes de tiempo.

## 📖 Guía

Para entender qué hace el kit y cómo puede cambiar tu día (sin parte técnica): **https://inematds.github.io/ia-sem-estrago/guia/es/**

## Qué es

El kit pone dos protecciones en Claude Code:

- **El bloqueo:** impide lo que no tiene vuelta atrás (borrar una carpeta entera, leer contraseñas, borrar el historial del proyecto), en cualquier modo, aunque la IA lo intente. Para decidir, basta con una pregunta: *«Si la IA se equivoca al hacerlo una vez, ¿se puede deshacer?»*
- **La prueba antes de decir «listo»:** dices una vez qué significa «listo» y una prueba lo comprueba cada vez que la IA intenta terminar. Si falla, vuelve y lo corrige.

Con las dos protecciones, puedes desactivar «permitir» y alejarte. Todo lo que se puede deshacer sigue adelante por su cuenta, en el modo automático de Claude Code.

**De dónde viene:** de una charla del equipo de Claude Code, en Anthropic, que trabaja de esta manera. Pasaron de 2.500 a más de 30.000 cambios de código al día. En el estudio que se mostró en la charla, quienes aprobaban los comandos uno por uno detectaron solo el 14 % de los comandos peligrosos; el modo automático bloqueó el 89 %.

## Cómo empezar (sin conocimientos técnicos)

Abre Claude Code o Codex en la carpeta de tu proyecto y pega esto:

```text
Lee el kit "IA sin daños" en https://github.com/inematds/ia-sem-estrago (el README lo explica todo, incluida la instalación).
Después revisa este proyecto y respóndeme en español sencillo:
1. ¿Qué cosas no se podrían deshacer si te equivocaras (borrar, contraseñas, enviar algo fuera)?
2. ¿Qué prueba o comprobación demuestra que una tarea está terminada en este proyecto?
3. ¿Qué partes del kit me convienen y cuáles no?
Todavía no cambies nada. Muéstrame el plan y espera a que te diga que sí.
```

La IA muestra el plan, tú apruebas lo que quieras y ella instala solo eso. El kit está pensado para Claude Code; Codex también puede leerlo y decirte si sirve para tu proyecto.

---

## Instalación técnica

Para quienes prefieren instalarlo a mano. Todo es texto para copiar; nada se instala por sí solo.

### Requisitos previos

- **Claude Code** con tu proyecto abierto (`claude --version`). Probado en la versión 2.1.326; `/auto-mode-setup`, `/sandbox` y `/doctor prompt-audit` se usaron en esa versión.
- **Python 3 y git**, para el verificador, las pruebas y descargar el kit.
- **Sandbox en Linux/WSL2:** necesita `bubblewrap` y `socat` (`sudo apt install bubblewrap socat`). En macOS no hace falta instalar nada.

### Los archivos

| Archivo | Para qué sirve |
|---|---|
| `kit/.claude/settings.json` | reglas de bloqueo (`.env`, `~/.ssh`, push forzado, `rm -rf`, `reset --hard`), sandbox activado, hooks `Stop` y `TaskCompleted` |
| `kit/.claude/hooks/pronto.sh` | ejecuta la comprobación; si falla → salida 2 con el motivo; como máximo 3 intentos seguidos |
| `kit/teste-do-desfazer.md` | la pregunta que decide qué va a la lista de bloqueo |
| `kit/como-e-pronto.md` | modelo de solicitud con «listo es…» y comprobación por tarea |
| `kit/CLAUDE.md.exemplo` | CLAUDE.md breve, sin IMPORTANTES en mayúsculas |
| `ferramentas/conferir.py` | comprueba un proyecto (CLAUDE.md, bloqueo + sandbox, hook de «listo»); solo lee |
| `exemplo/` | un inicio de sesión en tres versiones (olvidó la API antigua, cerró la sesión de todo el mundo, correcto) + `PROVA.md` con el hook ejecutándose de verdad |
| `guia/` | la página de la guía en PT, EN y ES (GitHub Pages) |

### Cómo funciona

`Solicitud con «listo es…»` → `Claude trabaja en auto mode` → `el bloqueo + sandbox impiden lo que no tiene vuelta atrás` → `Claude intenta parar` → `el hook ejecuta la comprobación` → `falló: sale con 2 y vuelve · pasó: listo`

- **Orden de los permisos:** `deny` (bloquea), `ask` (pregunta), `allow` (permite). Claude Code comprueba `deny` primero, antes que el verificador del auto mode, en todos los modos, incluido `bypassPermissions`.
- **La regla revisa el texto del comando**, no su efecto. Por eso el kit también activa el **sandbox**, donde el comando solo puede leer carpetas y comunicarse con los sitios permitidos (`github.com`, `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`). Si tu proyecto se comunica con otro sitio, añade el dominio en `sandbox.network.allowedDomains`.
- **Por qué `Stop` y `TaskCompleted`:** son los momentos en que Claude intenta parar o cerrar una tarea. `PostToolUse` no puede bloquear, ni siquiera con 2, porque la herramienta ya se ejecutó.
- **Salida 2, no 1:** con 2, Claude Code devuelve el error a Claude, que vuelve y lo corrige. Con 1, solo muestra el error y Claude sigue como si estuviera hecho.
- **Protección contra bucles:** `pronto.sh` bloquea como máximo 3 veces seguidas (`PRONTO_MAX`) y después permite parar para que lo revises.
- **De dónde obtiene la comprobación**, en este orden: la variable `PRONTO_CMD`, el archivo `.claude/pronto.cmd` (una línea) o la detección de `npm test` / `python3 -m unittest`. Si no hay nada que comprobar, no bloquea. Si añades un `grep`, comprueba que la carpeta existe: `! grep -rn "x" src/` siempre pasa cuando no existe `src/`.

## El proceso, de principio a fin

Cambia `~/meu-projeto` por la carpeta de tu proyecto. ⚙️ = script · 🧠 = tú (a veces con Claude).

| # | Paso | Comando | Quién |
|---|---|---|---|
| 0 | Descargar el kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 tú |
| 1 | Comprobar tu proyecto (es normal que falle la primera vez) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar la configuración y el hook | `cp -rn kit/.claude ~/meu-projeto/` y `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (`-n` no sobrescribe; si ya hay un `settings.json`, añade las claves a mano) | 🧠 tú |
| 3 | Indicar cuál es la comprobación de «listo» | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (cámbialo por la prueba de tu proyecto) | 🧠 tú |
| 4 | Activar el auto mode y describir tu entorno | `claude --permission-mode auto` (o pulsa Shift+Tab hasta que aparezca "⏵⏵ auto mode on"; es el modo predeterminado desde la 2.1.283), después `/auto-mode-setup` | 🧠 tú + Claude |
| 5 | Activar y probar el sandbox | `/sandbox` | 🧠 tú |
| 6 | Simplificar el CLAUDE.md | `/doctor prompt-audit` y, en cada línea, «¿lo descubriría leyendo el código?» | 🧠 tú + Claude |
| 7 | Comprobar que el hook lo exige | `bash exemplo/rodar.sh` (lo olvidó: 2 · cerró la sesión: 2 · correcto: 0) | ⚙️ script |
| 8 | Volver a comprobar | paso 1 → `ia sem estrago: ok` | ⚙️ script |

El verificador lee los archivos; no sabe si ejecutaste los pasos 4 y 5 dentro de Claude Code.

## Pruebas

```bash
python3 -m unittest discover -s tests -v   # 11 pruebas, solo Python 3
bash exemplo/rodar.sh                       # lo olvidó: 2 · cerró la sesión: 2 · correcto: 0
```

## La prueba

El hook se ejecutó de verdad en Claude Code 2.1.326, con la suscripción, el 17/11/2026. Había un error oculto (la suma restaba) y Claude dijo «listo»; el hook ejecutó la prueba, detectó el fallo y salió con 2; Claude volvió, encontró el error, lo corrigió y solo entonces terminó. Registro: `exemplo/PROVA.md`.

## De dónde viene

Charla «Rethinking Claude Code Best Practices», de Lydia Hallie (equipo de Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Sintaxis comprobada en la documentación oficial de Claude Code (permissions, sandboxing, hooks, auto mode) el 17/11/2026. El estudio del 14 % y el 89 % solo se mostró en la charla.

Gratis en [inema.club](https://inema.club), directamente en internet. Licencia MIT.