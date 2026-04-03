# 02 · Git Revert

## ¿Qué es git revert?

`git revert` crea un **nuevo commit que invierte los cambios** de un commit anterior. No borra nada de la historia — agrega un commit que "deshace" el efecto del commit apuntado.

```
Antes:  A - B - C - D (HEAD)
                ↑
          queremos deshacer C

Después: A - B - C - D - C' (HEAD)
                          ↑
                    C' invierte los cambios de C
```

> 💡 Si el commit es materia, `git revert` es la antimateria. La historia queda intacta y podés ver exactamente qué se revirtió y cuándo.

---

## revert vs restore vs reset

Es fácil confundir estos tres comandos:

| Comando | Qué hace | Toca la historia |
|---------|----------|-----------------|
| `git revert <sha>` | Crea un nuevo commit que invierte otro | No (agrega un commit) |
| `git restore <archivo>` | Descarta cambios en working tree o staging | No |
| `git reset <sha>` | Mueve el puntero de la rama hacia atrás | Sí (reescribe) |

**Regla práctica:** en ramas compartidas o públicas, siempre preferí `git revert` sobre `git reset` — no reescribe la historia y es seguro para todos.

---

## Uso básico

```bash
git revert <sha>          # revertir un commit específico
git revert HEAD           # revertir el último commit
git revert HEAD~2         # revertir el commit de hace dos pasos
git revert <sha> --no-edit  # revertir sin abrir el editor de mensajes
```

Git abre el editor con un mensaje pre-generado tipo `Revert "nombre del commit original"`. Podés editarlo o dejarlo tal cual.

---

## Encontrar el commit a revertir

Antes de revertir, conviene revisar qué contiene el commit:

```bash
git log --oneline                  # ver la historia resumida
git log -p --grep "palabra"        # buscar por mensaje + ver diff
git log -p -- archivo.js           # historial de un archivo específico
git show <sha>                     # ver el diff completo de un commit
git log -p -1                      # diff del último commit
```

---

## Cuando el revert genera un conflicto

`git revert` puede generar conflictos si el archivo cambió tanto desde el commit original que Git no puede invertir los cambios limpiamente. El proceso es igual al de un merge conflict:

```bash
git revert a665b08
# CONFLICT (content): Merge conflict in README.md

git status
# You are currently reverting commit a665b08.
# both modified: README.md

# Resolver el conflicto manualmente en el archivo
vim README.md

git add README.md
git revert --continue     # NO git commit, igual que rebase
```

### Comandos de emergencia

```bash
git revert --abort    # cancelar el revert y volver al estado anterior
git revert --skip     # saltear este commit (lo ignora)
```

---

## El commit resultante

Después de un revert exitoso, el log muestra:

```bash
git log -p -1
# commit 7488b35 (HEAD -> trunk)
# Author: ...
#
#     Revert "E"
#
#     This reverts commit a665b08996994c2e6620a6367b0ab524be221cb2.
#
# diff --git a/README.md b/README.md
# --- a/README.md
# +++ b/README.md
# @@ -1,5 +1,4 @@
#  A
#  D
# -E
#  remote-change
```

El mensaje incluye automáticamente qué commit se revirtió y su SHA — muy útil para auditoría.

---

## Revertir múltiples commits

```bash
# Revertir un rango de commits (de más nuevo a más viejo)
git revert HEAD~3..HEAD

# Revertir varios commits sin crear un commit por cada uno
git revert --no-commit HEAD~3..HEAD
git commit -m "revert: deshacer últimos 3 commits"
```

> 💡 Con `--no-commit` (o `-n`) Git aplica todos los cambios invertidos al staging area pero no commitea hasta que vos lo hagas, permitiéndote crear un solo commit de revert limpio.

---

## Cuándo usar revert

- Cuando necesitás deshacer algo en una rama **pública o compartida**
- Cuando querés mantener la historia completa (auditoría, compliance)
- Cuando el commit a deshacer ya fue pusheado al remote

**No usar revert cuando:** el commit es reciente, solo está en local, y preferís limpiar la historia — en ese caso `git reset` o `git commit --amend` son más apropiados.
