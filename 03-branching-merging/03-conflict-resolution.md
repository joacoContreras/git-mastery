# 03 · Resolución de Conflictos

## ¿Cuándo ocurre un conflicto?

Un conflicto ocurre cuando Git no puede resolver automáticamente diferencias entre dos ramas. La causa más común: **dos personas editaron la misma línea del mismo archivo** en commits distintos.

Git puede resolver automáticamente cambios en líneas distintas. Lo que no puede resolver solo es cuando la misma línea fue modificada de dos maneras diferentes — ahí te pide que decidas vos.

---

## Cómo se ve un conflicto en el archivo

```
<<<<<<< HEAD
A + 2
=======
A + 1
>>>>>>> 9648be0ae764528ac63759d7e49fc623ae0af373
D
E
remote-change
```

| Marcador | Significado |
|----------|-------------|
| `<<<<<<< HEAD` | Inicio de tu versión (rama actual) |
| `=======` | Separador entre las dos versiones |
| `>>>>>>> sha` | Fin de la versión entrante (con su SHA) |

Para resolver: **editás el archivo** dejando solo el contenido que querés (sin los marcadores), luego `git add` y continuás.

---

## Resolver un conflicto paso a paso

### Con merge

```bash
# 1. Identificar qué archivos están en conflicto
git status
# Unmerged paths:
#   both modified:   README.md

# 2. Abrir el archivo y resolver (editar, dejar solo lo que querés)
vim README.md

# 3. Marcar como resuelto
git add README.md

# 4. Commitear el merge
git commit
# Git sugiere un mensaje automático tipo "Merge branch '...' into ..."
```

### Con rebase

```bash
# 1. Identificar el conflicto
git status

# 2. Resolver el archivo
vim README.md

# 3. Marcar como resuelto
git add README.md

# 4. Continuar el rebase (NO hacer git commit)
git rebase --continue
```

> ⚠️ La diferencia clave: con **merge** hacés `git commit` para cerrar el conflicto. Con **rebase** hacés `git rebase --continue` — si hacés commit, rompés el proceso de rebase.

---

## Comandos de emergencia

```bash
git merge --abort     # cancelar el merge y volver al estado anterior
git rebase --abort    # cancelar el rebase y volver al estado anterior
git rebase --skip     # saltear el commit que genera conflicto (lo descarta)
```

---

## Elegir un lado completo: --ours y --theirs

Cuando no necesitás mezclar — solo querés una versión completa del archivo:

```bash
git checkout --ours README.md      # quedarse con la versión de HEAD
git checkout --theirs README.md    # quedarse con la versión entrante
git add README.md                  # marcar como resuelto
```

### La inversión de ours/theirs en rebase

Este es el punto más confuso de Git con conflictos:

| Operación | `ours` | `theirs` |
|-----------|--------|----------|
| **merge** | Tu rama (HEAD actual) | La rama que mergeás |
| **rebase** | La rama base (sobre la que rebaseás) | Tus commits (los que se replayan) |

Durante `git pull --rebase origin trunk`:
- Git internamente checkoutea `origin/trunk` primero
- Luego replaya tus commits encima uno a uno
- Por eso `ours` = `origin/trunk` y `theirs` = tus commits locales

> 💡 Regla práctica para no confundirse: **en rebase, `theirs` son tus cambios**. Si querés quedarte con lo tuyo, usá `--theirs`.

---

## Conflictos con rebase: el problema del replay

El rebase replaya commits **uno por uno**. Esto significa que si tenés 5 commits y el primero genera un conflicto con la base, vas a tener que resolverlo cada vez que rebaseés sobre esa misma base.

```
Situación:
      E - F - G    feature   (G modifica la misma línea que C)
     /
A - B - C - D      trunk

Primer rebase:
→ replay E: ok
→ replay F: ok
→ replay G: CONFLICTO con C → resolvés

Trunk avanza a D2:
A - B - C - D - D2   trunk

Segundo rebase:
→ replay E: ok
→ replay F: ok
→ replay G: CONFLICTO con C de nuevo → tenés que resolver otra vez
```

La solución a esto es **rerere**.

---

## rerere: nunca resolver el mismo conflicto dos veces

`rerere` (REuse REcorded REsolution) hace que Git recuerde cómo resolviste un conflicto y lo aplique automáticamente la próxima vez:

```bash
# Activar rerere
git config rerere.enabled true          # para este repo
git config --global rerere.enabled true # para todos los repos
```

Una vez activado, la próxima vez que aparezca el mismo conflicto Git lo resuelve solo:

```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Resolved 'README.md' using previous resolution.  ← rerere en acción
```

### Gestionar resoluciones de rerere

```bash
# Ver qué resoluciones tiene guardadas
ls .git/rr-cache/

# Olvidar una resolución (si la resolviste mal)
git rerere forget README.md

# Ver el estado de rerere
git rerere status
```

> 💡 Si activás `rerere` y resolvés un conflicto incorrectamente, rerere va a repetir ese error. Usá `git rerere forget` para borrarlo y volver a resolverlo bien.

---

## git log -p: entender el origen de un conflicto

Antes de resolver, puede ser útil ver el historial del archivo para entender qué cambió y por qué:

```bash
git log -p README.md          # historial con diffs de un archivo
git log -p -1                 # diff del último commit
git log --oneline -1          # solo ver a qué commit pertenece un SHA
```

Para validar a qué rama pertenece el SHA del conflicto:

```bash
# El SHA que aparece en >>>>>>> pertenece a la rama entrante
git log --oneline | grep 9648be0
# O directamente
git show 9648be0
```

---

## Conflictos comunes y cómo evitarlos

| Situación | Prevención |
|-----------|-----------|
| Editar la misma línea en dos ramas | Comunicación con el equipo, PRs chicos y frecuentes |
| Long-lived feature branch | Rebasear seguido sobre `trunk` para ir integrando cambios |
| Conflictos repetidos en rebase | Activar `rerere` |
| No saber qué lado elegir | `git log -p` para entender el contexto antes de decidir |

---

## Flujo completo de conflicto con merge

```bash
git pull origin trunk
# CONFLICT (content): Merge conflict in README.md

git status
# both modified: README.md

vim README.md
# editar: dejar solo el contenido correcto, eliminar marcadores

git add README.md
git commit
# [trunk d8a2f95] Merge branch 'trunk' of ../hello-git into trunk
```

## Flujo completo de conflicto con rebase

```bash
git pull --rebase origin trunk
# CONFLICT (content): Merge conflict in README.md

git status
# interactive rebase in progress; onto 958f33f
# both modified: README.md

vim README.md
# editar: dejar solo el contenido correcto, eliminar marcadores

git add README.md
git rebase --continue
# Successfully rebased and updated refs/heads/trunk.
```

---

## Resumen de comandos

| Comando | Para qué |
|---------|----------|
| `git status` | Ver qué archivos están en conflicto |
| `git merge --abort` | Cancelar el merge |
| `git rebase --abort` | Cancelar el rebase |
| `git rebase --continue` | Continuar el rebase después de resolver |
| `git rebase --skip` | Saltear un commit conflictivo (lo descarta) |
| `git checkout --ours <file>` | Elegir la versión de la rama base |
| `git checkout --theirs <file>` | Elegir la versión de la rama entrante |
| `git rerere forget <file>` | Borrar una resolución guardada incorrectamente |
| `git log -p -1` | Ver el diff del último commit |
