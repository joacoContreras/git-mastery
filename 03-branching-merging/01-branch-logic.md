# 01 · Ramas y Remotos

## ¿Qué es un remote?

Un **remote** es simplemente una copia del mismo repositorio en otro lugar. No tiene que ser GitHub o GitLab — puede ser otra carpeta en tu misma máquina. Lo que importa es que es el mismo proyecto con sus propios commits y estado.

```bash
# Ver los remotes configurados
git remote -v
# origin  https://github.com/usuario/repo.git (fetch)
# origin  https://github.com/usuario/repo.git (push)
```

### Convenciones de nombres

| Nombre | Cuándo se usa |
|--------|---------------|
| `origin` | El repo principal (tu fuente de verdad en GitHub/GitLab) |
| `upstream` | Cuando tenés un fork: `origin` es tu fork, `upstream` es el repo original |

---

## git remote

```bash
git remote add <nombre> <uri>         # agregar un remote
git remote -v                         # listar remotes con sus URLs
git remote remove <nombre>            # eliminar un remote
git remote rename <viejo> <nuevo>     # renombrar un remote
```

### Ejemplo

```bash
# Agregar un remote local (útil para practicar sin GitHub)
git remote add origin ../otro-repo

# Agregar el repo original de un proyecto que forkaste
git remote add upstream https://github.com/original/repo.git
```

---

## git fetch

Descarga el estado del remote **sin modificar tus ramas locales**. Solo actualiza las referencias `origin/*`:

```bash
git fetch                  # fetch de origin (el remote por defecto)
git fetch origin           # explícito
git fetch --all            # fetch de todos los remotes configurados
```

Después de un fetch:

```bash
git log origin/trunk       # ver los commits del remote sin estar en esa rama
git diff HEAD origin/trunk # ver qué diferencia hay entre tu rama y el remote
```

> 💡 `git fetch` es siempre seguro — nunca toca tu working tree ni tus ramas locales. Es una buena práctica hacerlo frecuentemente para estar al tanto de los cambios del equipo.

---

## git pull

Combina `git fetch` + `git merge` (o `git fetch` + `git rebase`) en un solo comando:

```bash
git pull                          # fetch + merge de la rama trackeada
git pull origin trunk             # explícito: remote y rama
git pull --rebase                 # fetch + rebase en vez de merge
git pull --rebase origin trunk    # explícito con rebase
```

### fetch vs pull: ¿cuál usar?

| Comando | Qué hace | Cuándo usarlo |
|---------|----------|---------------|
| `git fetch` | Solo descarga, no modifica nada local | Cuando querés ver qué hay antes de integrar |
| `git pull` | Descarga e integra automáticamente | Cuando confiás en los cambios entrantes |

> 💡 Muchos desarrolladores prefieren `git fetch` + revisar + `git merge` o `git rebase` manualmente, para tener más control sobre qué están integrando y cómo.

---

## Tracking: vincular ramas locales con remotas

Git no asume automáticamente que dos ramas con el mismo nombre están relacionadas. Tenés que configurar el **tracking** explícitamente para poder usar `git pull` y `git push` sin argumentos:

```bash
# Configurar tracking en una rama existente
git branch --set-upstream-to=origin/trunk trunk

# Al crear una rama nueva que trackea una remota
git checkout -b mi-rama origin/mi-rama

# Al hacer push por primera vez, configurar tracking al mismo tiempo
git push -u origin mi-rama
```

Verificar el tracking:

```bash
git branch -vv
# * trunk  a3f2c1e [origin/trunk] feat: add login
#   feature  8b1d4f2 [origin/feature: ahead 2] wip
```

- `ahead 2` = tenés 2 commits locales que el remote no tiene
- `behind 3` = el remote tiene 3 commits que vos no tenés
- `ahead 2, behind 1` = divergieron (necesitás hacer pull y merge/rebase)

---

## git push

Subir tus commits locales al remote:

```bash
git push                           # push a la rama trackeada
git push origin trunk              # explícito
git push -u origin mi-rama         # push + configurar tracking
git push origin local:remota       # push con nombre diferente en el remote
git push origin :rama-a-borrar     # borrar una rama en el remote
git push --force-with-lease        # force push seguro (ver nota abajo)
```

> ⚠️ `git push --force` sobreescribe la historia del remote. Puede destruir commits de tus compañeros. Usá siempre `--force-with-lease` si necesitás forzar un push — verifica que nadie más pusheó desde tu último fetch antes de sobreescribir.

### ¿Por qué falla el push a la rama actual de un repo?

```
! [remote rejected] trunk -> trunk (branch is currently checked out)
```

No podés pushear a la rama que está actualmente checkeada en el repo destino — cambiaría el working tree por debajo del usuario que está trabajando ahí. La solución es que el destino cambie de rama antes del push.

---

## git branch

```bash
git branch                    # listar ramas locales
git branch -a                 # listar todas (locales + remotas)
git branch -vv                # listar con info de tracking y último commit
git branch nueva-rama         # crear rama (sin cambiarte a ella)
git branch -d rama            # borrar rama (solo si está mergeada)
git branch -D rama            # borrar rama forzado
git branch -m viejo nuevo     # renombrar rama
```

### Ramas remotas (origin/*)

Cuando hacés `git fetch`, Git actualiza referencias del tipo `origin/trunk`, `origin/feature`, etc. Estas representan el estado del remote **la última vez que hiciste fetch** — no el estado actual en tiempo real.

```bash
git branch -a
# * trunk
#   remotes/origin/trunk
#   remotes/origin/feature-login
#   remotes/origin/bar
```

---

## git checkout y git switch

Para moverte entre ramas:

```bash
# Forma clásica (git checkout)
git checkout trunk              # cambiar a una rama existente
git checkout -b nueva-rama      # crear y cambiar a la vez

# Forma moderna (git switch, desde Git 2.23)
git switch trunk
git switch -c nueva-rama        # equivalente a checkout -b
```

Cuando hacés checkout a una rama que existe en el remote pero no localmente, Git la crea automáticamente y configura el tracking:

```bash
git checkout feature-login
# Branch 'feature-login' set up to track remote branch 'feature-login' from 'origin'.
# Switched to a new branch 'feature-login'
```

---

## pull.rebase: configurar el comportamiento de pull

Por defecto, `git pull` hace un merge. Para que siempre haga rebase:

```bash
# Solo para este repo
git config pull.rebase true

# Para todos los repos
git config --global pull.rebase true
```

> 💡 Muchos equipos prefieren `pull --rebase` para mantener una historia lineal sin commits de merge innecesarios. Es una preferencia de equipo, no hay una respuesta correcta universal.

---

## Resumen del flujo remote

```bash
# Flujo típico del día a día
git fetch                          # ver qué hay en el remote
git log HEAD..origin/trunk         # qué commits nuevos hay
git pull --rebase                  # integrar con rebase

# Hacer tu trabajo
git add -p
git commit -m "feat: ..."

# Compartir
git push
```

| Comando | Qué hace |
|---------|----------|
| `git remote add` | Registrar un remote |
| `git fetch` | Descargar estado del remote (sin modificar local) |
| `git pull` | fetch + merge/rebase |
| `git push` | Subir commits locales al remote |
| `git branch -vv` | Ver estado de tracking de todas las ramas |
