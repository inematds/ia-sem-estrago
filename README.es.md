# 🛡️ IA sin daños

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

[![IA sin daños](guia/assets/banner-es.jpg)](https://inematds.github.io/ia-sem-estrago/guia/es/)

**La IA trabaja sola y no estropea lo que no se puede deshacer.** Basado en la documentación oficial de Claude Code y en las recomendaciones de los ingenieros de Anthropic. Le pides una tarea y te ocupas de otra cosa. Cuando vuelves, está hecha y probada. No se ha borrado ninguna carpeta, no se ha leído ninguna contraseña y Claude no ha dicho «listo» antes de tiempo.

## 📖 Guía

Para entender qué hace el kit y cómo cambia tu día a día (sin tecnicismos): **https://inematds.github.io/ia-sem-estrago/guia/es/**

## Qué es

El kit añade dos protecciones a Claude Code:

- **El bloqueo:** impide hacer lo que no se puede deshacer (borrar una carpeta entera, leer contraseñas, borrar el historial del proyecto), en cualquier modo, aunque la IA lo intente. La regla para decidir se basa en una sola pregunta: *«Si la IA se equivoca una vez, ¿se puede deshacer?»*
- **La prueba antes de decir «listo»:** defines una vez qué significa «listo» y una prueba lo comprueba cada vez que la IA intenta terminar. Si falla, vuelve y lo corrige.

Con las dos protecciones, puedes desactivar «permitir» y alejarte. Todo lo que se puede deshacer sigue adelante automáticamente, en el modo automático de Claude Code.

## Basado en Anthropic

Nada de esto es invención nuestra. El kit reúne y deja listo para usar lo que Anthropic (la empresa que desarrolla Claude) publica y recomienda:

- **Los ingenieros de Claude Code:** en la charla «Rethinking Claude Code Best Practices», Lydia Hallie, del equipo de Claude Code, explica cómo trabajan: dejan de vigilar, bloquean lo que no se puede deshacer y comprueban con pruebas que esté «listo». Así pasaron de 2.500 a más de 30.000 cambios de código al día. [Ver](https://www.youtube.com/watch?v=vlfCjWuv4es)
- **El cansancio de aprobar:** quienes aprueban todo manualmente terminan aceptando el 93 % de las solicitudes. El registro interno de Anthropic incluye casos reales de IA que borró ramas de git, envió una clave de acceso al exterior y modificó la base de datos de producción. [Artículo: How we built Claude Code auto mode](https://www.anthropic.com/research/claude-code-auto-mode)
- **El entorno aislado (sandbox):** redujo las solicitudes de permiso un 84 % y mejoró la seguridad. [Artículo: Making Claude Code safer with sandboxing](https://www.anthropic.com/research/claude-code-sandboxing)
- **Cada pieza está en la documentación oficial:** el bloqueo = [Permisos](https://code.claude.com/docs/en/permissions); el entorno aislado = [Sandbox](https://code.claude.com/docs/en/sandboxing); la prueba antes de decir «listo» = [Hooks](https://code.claude.com/docs/en/hooks); cómo hacer las solicitudes = [Buenas prácticas](https://code.claude.com/docs/en/best-practices). Verificado el 17/11/2026.

Las cifras del 14 % (personas que aprueban cada solicitud) y el 89 % (modo automático) se presentaron en la charla; las del 93 % y el 84 % aparecen en los artículos oficiales.

## Cómo empezar (sin conocimientos técnicos)

Abre Claude Code o Codex en la carpeta de tu proyecto y pega esto:

```text
Lee el kit "IA sin daños" en https://github.com/inematds/ia-sem-estrago (el README lo explica todo, incluida la instalación).
Después, revisa este proyecto y respóndeme en español sencillo:
1. ¿Qué cosas no se podrían deshacer si te equivocaras (borrar, contraseñas, enviar algo al exterior)?
2. ¿Qué prueba o comprobación demuestra que una tarea quedó lista en este proyecto?
3. ¿Qué partes del kit me convienen y cuáles no?
No cambies nada todavía. Muéstrame el plan y espera a que te diga que sí.
```

La IA te muestra el plan, tú apruebas lo que quieras y ella instala solo eso. El kit está preparado para Claude Code; Codex también puede leerlo y decirte si sirve para tu proyecto.

---

## Instalación técnica

Para quienes prefieren instalarlo a mano. Todo es texto simple para copiar; nada se instala automáticamente.

### Requisitos previos

- **Claude Code** con tu proyecto abierto (`claude --version`). Probado en la versión 2.1.326; `/auto-mode-setup`, `/sandbox` y `/doctor prompt-audit` se usaron en esa versión.
- **Python 3 y git**, para el comprobador, las pruebas y para descargar el kit.
- **Sandbox en Linux/WSL2:** necesitas `bubblewrap` y `socat` (`sudo apt install bubblewrap socat`). En macOS no hace falta instalar nada.

### Los archivos

| Archivo | Para qué sirve |
|---|---|
| `kit/.claude/settings.json` | reglas de bloqueo (`.env`, `~/.ssh`, push forzado, `rm -rf`, `reset --hard`), sandbox activado, hooks `Stop` y `TaskCompleted` |
| `kit/.claude/hooks/pronto.sh` | ejecuta la comprobación; si falla → salida 2 con el motivo; como máximo 3 intentos seguidos |
| `kit/teste-do-desfazer.md` | la pregunta que determina qué se incluye en la lista de bloqueo |
| `kit/como-e-pronto.md` | plantilla de solicitud con «listo significa…» y comprobación por tarea |
| `kit/CLAUDE.md.exemplo` | CLAUDE.md breve, sin IMPORTANTE en mayúsculas |
| `ferramentas/conferir.py` | comprueba un proyecto (CLAUDE.md, bloqueo + sandbox, hook de «listo»); solo lee |
| `exemplo/` | un inicio de sesión en tres versiones (olvidó la API antigua, cerró la sesión de todos, correcto) + `PROVA.md` con el hook ejecutándose de verdad |
| `guia/` | la página de la guía en PT, EN y ES (GitHub Pages) |

### Cómo funciona

`Solicitud con "listo significa…"` → `Claude trabaja en auto mode` → `bloqueo + sandbox impiden lo que no se puede deshacer` → `Claude intenta parar` → `el hook ejecuta la comprobación` → `falló: sale con 2 y vuelve · pasó: listo`

- **Orden de los permisos:** `deny` (bloquea), `ask` (pregunta), `allow` (permite). Claude Code comprueba `deny` primero, antes que el verificador del auto mode, en todos los modos, incluido `bypassPermissions`.
- **La regla revisa el texto del comando**, no sus efectos. Por eso el kit también activa el **sandbox**, donde el comando solo puede leer carpetas y comunicarse con los sitios permitidos (`github.com`, `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`). Si tu proyecto se comunica con otro sitio, añade el dominio a `sandbox.network.allowedDomains`.
- **Por qué `Stop` y `TaskCompleted`:** son los momentos en que Claude intenta parar o cerrar una tarea. `PostToolUse` no bloquea, ni siquiera con 2, porque la herramienta ya se ejecutó.
- **Salida 2, no 1:** con 2, Claude Code devuelve el error a Claude, que vuelve y lo corrige. Con 1, solo muestra el error y Claude sigue como si la tarea estuviera lista.
- **Protección contra bucles:** `pronto.sh` bloquea como máximo 3 veces seguidas (`PRONTO_MAX`) y después permite parar para que lo revises.
- **De dónde sale la comprobación**, en este orden: la variable `PRONTO_CMD`, el archivo `.claude/pronto.cmd` (una línea) o la detección de `npm test` / `python3 -m unittest`. Si no hay nada que comprobar, no bloquea. Si añades un `grep`, comprueba que la carpeta exista: `! grep -rn "x" src/` siempre pasa cuando no hay `src/`.

## El proceso, de principio a fin

Sustituye `~/meu-projeto` por la carpeta de tu proyecto. ⚙️ = script · 🧠 = tú (a veces con Claude).

| # | Paso | Comando | Quién |
|---|---|---|---|
| 0 | Descargar el kit | `git clone https://github.com/inematds/ia-sem-estrago.git && cd ia-sem-estrago` | 🧠 tú |
| 1 | Comprobar tu proyecto (es normal que falle la primera vez) | `python3 ferramentas/conferir.py ~/meu-projeto` | ⚙️ script |
| 2 | Copiar la configuración y el hook | `cp -rn kit/.claude ~/meu-projeto/` y `echo ".claude/.pronto-bloqueios" >> ~/meu-projeto/.gitignore` (la opción `-n` no sobrescribe; si ya existe `settings.json`, añade las claves a mano) | 🧠 tú |
| 3 | Indicar cuál es la comprobación de «listo» | `echo "python3 -m unittest discover -s tests -q" > ~/meu-projeto/.claude/pronto.cmd` (cámbialo por la prueba de tu proyecto) | 🧠 tú |
| 4 | Activar el auto mode y describir tu entorno | `claude --permission-mode auto` (o pulsa Shift+Tab hasta «⏵⏵ auto mode on»; es el valor predeterminado desde la 2.1.283), después `/auto-mode-setup` | 🧠 tú + Claude |
| 5 | Activar y probar el sandbox | `/sandbox` | 🧠 tú |
| 6 | Simplificar el CLAUDE.md | `/doctor prompt-audit` y, en cada línea, pregúntate: «¿Lo descubriría leyendo el código?» | 🧠 tú + Claude |
| 7 | Comprobar que el hook exige la prueba | `bash exemplo/rodar.sh` (olvidó: 2 · cerró la sesión: 2 · correcto: 0) | ⚙️ script |
| 8 | Volver a comprobar | paso 1 → `ia sem estrago: ok` | ⚙️ script |

El comprobador lee los archivos; no sabe si ejecutaste los pasos 4 y 5 dentro de Claude Code.

## Pruebas

```bash
python3 -m unittest discover -s tests -v   # 11 pruebas, solo Python 3
bash exemplo/rodar.sh                       # olvidó: 2 · cerró la sesión: 2 · correcto: 0
```

## La prueba

El hook se ejecutó de verdad en Claude Code 2.1.326, con la suscripción, el 17/11/2026. Ante un error oculto (la suma restaba), Claude dijo «listo»; el hook ejecutó la prueba, detectó el fallo y salió con 2; Claude volvió, encontró el error, lo corrigió y solo entonces paró. Registro: `exemplo/PROVA.md`.

## Fuentes

Charla «Rethinking Claude Code Best Practices», de Lydia Hallie (equipo de Claude Code, Anthropic, AI Engineer): https://www.youtube.com/watch?v=vlfCjWuv4es. Sintaxis verificada en la documentación oficial de Claude Code (permissions, sandboxing, hooks, auto mode) el 17/11/2026. El estudio del 14 % y el 89 % solo se presentó en la charla.

Gratis en [inema.club](https://inema.club), directamente en internet. Licencia MIT.