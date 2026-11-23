# FORK.md — cambios propios sobre syalia-srl/aegis

Este fork (`UDLDavid/aegis`) lleva los cambios de udl sobre `syalia-srl/aegis`.
La rama `udl/deploy` es la base de upstream más **todas** las ramas de cambio, y
lo que se instala en los servidores es una etiqueta suya: `udl-AAAA.MM.DD`.

- Remotos: `origin` = `syalia-srl/aegis` (upstream), `fork` = `UDLDavid/aegis`.
- Base de esta serie: `origin/main` = `a2c8d5ed` (349 commits por delante de
  `v2.22.0`; upstream todavía no etiqueta 3.x, así que la «3.x» es `main`).
- Tamaño: 27 commits, 64 ficheros, +3.725 / −188.
- PRs: **#1114** `pool-attribution`, abierto — «A session's pool, not its
  harness, decides whose account it spends». **#1115** `pool-connection`,
  draft — «A claude-code agent on an account of its own: the connection».
- Los mensajes de los commits citan también #1113, que es el PR de origen de la
  serie; no es de este fork.

## Reglas de este fork

1. Un cambio = una rama = un PR. Si dos cambios dependen entre sí, se apilan
   (`pool-connection` va sobre `pool-attribution`).
2. El fragmento `changelog.d/<issue>-<slug>.<categoría>.md` va en la rama del
   PR, nunca en `udl/deploy`.
3. `udl/deploy` = punta de la serie + este `FORK.md`. No se edita para arreglar
   código: se regenera desde la rama de arriba y se vuelve a etiquetar.
4. La etiqueta de despliegue es anotada y apunta al commit exacto que debe
   correr en los dos servidores.
5. **No subir `version` de `pyproject.toml`.** El enlace entre servidores compara
   versiones (`links.py`, `BOTH_WAYS_SINCE = "3.0.0"`) y la versión sigue siendo
   `2.22.0` en todos los builds. El build real se identifica por el `commit_id`
   de `direct_url.json`.

## Los cambios

Cada fila es un commit de la serie; «prueba» es el selector de pytest que lo
cubre (además de `uv run pytest -q` completo y `uv run aegis doctor --probe`).

### A. El pool de una sesión — rama `pool-attribution`, PR #1114

Qué se rompe sin esto: la cuota se cobra al *harness* y no a la cuenta del
*pool*, así que un agente `go-claude` no descuenta de OpenCode Go y el enfriamiento
de modelos se comparte entre cuentas que no son la misma.

| commit | qué hace | prueba |
| --- | --- | --- |
| `bb40595d` | `pool_of` y `pool_of_record`: resolver el pool de una sesión | `tests/test_quota_gate.py` |
| `bd2f30fc` | el registro del spawn lleva el pool (opcional) | `tests/test_session.py` |
| `2881b4d7` | guards, avisos y líneas de schedule leen el pool de la sesión | `tests/test_guards.py tests/test_notices.py` |
| `a2296f69` | spec: diseño del pool de sesión, su revisión y el plan | — |
| `918e2557` | el enfriamiento se indexa por dueño y pool de la sesión | `tests/test_cascade.py tests/test_cascade_app.py` |
| `b6b5aa94` | el catálogo de modelos lee el enfriamiento por claves de agente | `tests/test_cascade.py` |
| `7acf7355` | el turno se factura a la cuenta del pool de la sesión | `tests/usage/test_turns.py` |
| `c3447983` | el pool resuelto aparece en la ficha de la sesión | `tests/test_quota_wire.py` |
| `220c496a` | el cliente dibuja el medidor del pool que nombra la ficha | `tests/test_pool_browser.py` |
| `3a51b02e` | una caída de pool se queda en su propio espacio | `tests/test_cascade.py tests/usage/test_turns.py` |
| `35d294e4` | formato de los tests de cascade | `tests/test_cascade.py tests/test_cascade_app.py` |
| `0d4b0fe2` | plan: A1–A3 hechos y sus desvíos | — |
| `b4b9fe61` | observe, refresco del turno y tarjeta de login se quedan en el pool de Claude | `tests/test_quota_wire.py tests/test_session.py` |
| `9685af4d` | el pool en la página de uso, fragmento de changelog y DESIGN | — |

### B. La conexión de un agente claude-code — rama `pool-connection`, PR #1115

Qué se rompe sin esto: no se puede declarar un agente claude-code con
`base_url`, `pool`, `api_key_file` o `env`; no existe `go-claude` y Claude Code
solo puede gastar la cuenta de Claude.

| commit | qué hace | prueba |
| --- | --- | --- |
| `5d4a23fc` | `agents`: parsear y rechazar una conexión (base_url, pool, api_key_file, env) | `tests/test_agent_config.py` |
| `8167f2d7` | Settings: la conexión se ve en la ficha y sobrevive a guardar | `tests/test_config_coverage.py tests/test_config_write.py` |
| `32312e5f` | el spawn lleva la conexión del agente y la guarda | `tests/test_agent_config.py tests/test_session.py` |
| `34cba04f` | `doctor`: informa de un agente de conexión sin el catálogo de Claude | `tests/test_doctor.py` |
| `e057d6c1` | los cuatro campos, nombrados, para que `env` no tome un `str` | — |
| `a80a42cf` | si el host cambia de harness, se sueltan conexión y pool | `tests/test_policy_spawn.py` |
| `a11d1024` | claude: la sesión corre contra su endpoint con su clave | `tests/fake_claude.py tests/test_btw.py` |
| `b3f65873` | un turno de conexión queda sin precio y su ficha sin dólares | `tests/test_meta.py tests/test_session.py` |
| `b01a7a4d` | `doctor --probe`: probar un agente de conexión | `tests/test_doctor.py` |
| `99bca06b` | docs: las cuatro claves, la sonda, fragmento de changelog y la excepción del login | — |
| `cdd14827` | el meta de una sesión viva se responde antes de que exista su fichero | `tests/test_session.py` |
| `4b9eca6d` | el transcript no pone dólares a un turno de conexión | `tests/test_fold.py` |
| `69a882fa` | el journal reconstruye el uso de una sesión desde su spawn | `tests/test_usage_e2e.py` |

## Desplegar

```bash
# 1. En el repo: etiquetar (solo si cambió la serie)
git switch udl/deploy && git merge --ff-only pool-connection
git tag -a udl-2026.11.23 -m "build desplegado en udl-HP-Laptop-14-fq1xxx y vps"
git push fork udl/deploy && git push fork udl-2026.11.23

# 2. En cada extremo, el mismo ref
uv tool install --force "aegis-harness @ git+ssh://git@github.com/UDLDavid/aegis.git@udl-2026.11.23"

# 3. Comprobar que los dos tienen el mismo commit
python3 - <<'PY'
import glob, json
for p in glob.glob('/home/udl/.local/share/uv/tools/aegis-harness/lib/python*/site-packages/aegis_harness-*.dist-info/direct_url.json'):
    print(json.load(open(p)).get('vcs_info', {}).get('commit_id'))
PY

# 4. Reiniciar solo cuando los dos coinciden, en la misma ventana
ssh vps 'sudo systemctl restart aegis && systemctl is-active aegis'
# portátil: Ctrl-C en la terminal donde corre aegis y relanzar `aegis serve --detach`
```

Un `uv tool upgrade aegis-harness` sin referencia instala la **2.22.0 de PyPI** y
rompe el enlace: siempre por etiqueta.

## Actualizar cuando salga una versión

```bash
git fetch origin --tags && git fetch fork
git log --oneline a2c8d5ed..origin/main          # qué trae upstream
gh pr view 1114 -R syalia-srl/aegis --json state,mergedAt
gh pr view 1115 -R syalia-srl/aegis --json state,mergedAt
git cherry -v origin/main pool-connection        # "-" = ya está en upstream
# (un squash no se detecta con git cherry: míralo en el PR)

git switch pool-attribution && git rebase origin/main
git switch pool-connection && git rebase --update-refs origin/main
# si el #1114 ya entró:
git rebase --onto origin/main pool-attribution pool-connection

git range-diff a2c8d5ed..69a882fa origin/main..pool-connection
uv run pytest -q
# regenerar udl/deploy, nueva etiqueta y desplegar como arriba
```

Conflictos esperables: `src/aegis/session.py`, `app.py`, `config.py`,
`quota/*`, `cascade.py`, `claude/connection.py`, `client/js/*` y `help/*.md`.
`changelog.d/` casi nunca: cada fragmento es un fichero nuevo.

## Config y conocimiento

- Un solo `.aegis.yaml` en git con `hosts:` indexado por **nombre de servidor**
  (`udl-HP-Laptop-14-fq1xxx`, `vps`), desplegado con `cp` (un symlink se rompe
  al guardar desde Settings). `go-claude` va en la parte común.
- El conocimiento (skills y memorias) **no** se sincroniza: queda por servidor.
  Si algún día se comparte, repo privado y un `pull --allow-unrelated-histories`
  una sola vez. No publicar lo que lleve credenciales.
- `aegis link` federa sesiones, agentes, colas y schedules; no sincroniza
  configuración, conocimiento ni versiones. El enlace exige el mismo protocolo:
  si un extremo queda en otro build, pasa a `mismatch`/`too_old` hasta igualarlo.
