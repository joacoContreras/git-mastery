# 03 · Git Reset

## ¿Qué es git reset?

`git reset` mueve el puntero de la rama actual hacia un commit anterior, **reescribiendo la historia**. A diferencia de `git revert`, no crea un nuevo commit — directamente cambia a dónde apunta la rama.

```
Antes:  A - B - C - D (HEAD -> trunk)

git reset HEAD~2

Después: A - B (HEAD -> trunk)
             ↑
       C y D ya no están en la rama
```

Tiene tres modos que determinan qué pasa con los cambios de los commits "removidos".

---

## Los tres modos

| Modo | Historia | Staging area (index) | Working tree |
|------|----------|----------------------|--------------|
| `--soft` | ← se mueve | Conserva los cambios | Conserva los cambios |
| `--mixed` (default) | ← se mueve | Se limpia | Conserva los cambios |
| `--hard` | ← se mueve | Se limpia | Se limpia (⚠️ destructivo) |

---

## git reset --soft

Mueve la rama hacia atrás pero **deja los cambios en el staging area**, listos para un nuevo commit. Útil para reescribir el mensaje de un commit o combinar varios commits en uno:

```bash
git reset --soft HEAD~1    # deshacer el último commit, mantener cambios staged

git status
# Changes to be committed:
#   modified: README.md    ← los cambios del commit deshecho están acá

# Podés commitear de nuevo con un mejor mensaje
git commit -m "mejor mensaje"
```

### Cuándo usarlo

- Querés cambiar el mensaje del último commit (alternativa a `git commit --amend`)
- Querés combinar manualmente varios commits en uno
- Hiciste un commit "a medias" y necesitás seguir editando antes de volver a commitear

---

## git reset --mixed (default)

Mueve la rama hacia atrás y limpia el staging area, pero **los cambios quedan en el working tree** como modificaciones sin stagear:

```bash
git reset HEAD~1           # equivalente a git reset --mixed HEAD~1
git reset --mixed HEAD~1

git status
# Changes not staged for commit:
#   modified: README.md    ← los cambios están acá, sin stagear
```

### Cuándo usarlo

- Querés "deshacer" un `git add` (unstage sin perder los cambios)
- Querés reorganizar qué va en cada commit antes de volver a stagear

```bash
# Equivalente a git rm --cached para todo lo staged
git reset HEAD
```

---

## git reset --hard

Mueve la rama hacia atrás y **destruye todos los cambios** en staging y working tree. Lo que no está commiteado o trackeado se pierde.

```bash
git reset --hard HEAD      # descarta TODO lo que no está commiteado
git reset --hard HEAD~1    # deshace el último commit Y descarta los cambios
```

### ⚠️ Advertencias importantes

- Los archivos **tracked** con cambios sin commitear se pierden
- Los archivos **untracked** (nunca hicieron `git add`) NO se borran — para eso hay que agregarlos primero
- Una vez ejecutado, el trabajo perdido es casi irrecuperable (salvo `git reflog`)

```bash
# Para destruir también archivos untracked
git add .                  # trackear todo
git reset --hard           # ahora sí los destruye también
```

### Cuándo usarlo

- Querés volver a un estado limpio y descartar todos los cambios locales
- Necesitás mover la rama a un punto anterior y no te importa perder los commits

---

## reset vs revert: cuándo usar cada uno

| Situación | Usar |
|-----------|------|
| Rama pública, commits ya pusheados | `git revert` |
| Trabajo local que no pusheaste | `git reset` |
| Querés mantener la historia completa | `git revert` |
| Querés limpiar la historia antes de pushear | `git reset` |

---

## Recuperar commits "perdidos" con reflog

Si hiciste un `reset --hard` y te arrepentiste, `git reflog` puede salvarte:

```bash
git reflog
# HEAD@{0}: reset: moving to HEAD~2
# HEAD@{1}: commit: feat: add login       ← acá estaba antes
# HEAD@{2}: commit: fix: email validation

# Recuperar el estado anterior
git checkout HEAD@{1}
git branch recovery HEAD@{1}

# O directamente mover la rama de vuelta
git reset --hard HEAD@{1}
```

> 💡 El reflog guarda todos los movimientos de HEAD por 90 días. Es tu última red de seguridad cuando `reset --hard` fue un error.

---

## git restore: el comando más específico

Para casos más puntuales, `git restore` es más explícito y menos peligroso que reset:

```bash
git restore archivo.js              # descartar cambios en working tree
git restore --staged archivo.js     # sacar un archivo del staging (unstage)
git restore --source=HEAD~2 archivo.js  # restaurar un archivo a una versión anterior
```

---

## Resumen

| Comando | Efecto |
|---------|--------|
| `git reset --soft HEAD~1` | Deshace el commit, cambios quedan staged |
| `git reset HEAD~1` | Deshace el commit, cambios quedan en working tree |
| `git reset --hard HEAD~1` | Deshace el commit y destruye los cambios |
| `git reset --hard HEAD` | Descarta todo lo que no está commiteado |
| `git restore --staged <file>` | Unstage un archivo sin perder los cambios |
| `git restore <file>` | Descarta cambios en working tree de un archivo |
