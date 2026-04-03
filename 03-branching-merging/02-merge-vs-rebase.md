# 02 · Merge vs Rebase

## El problema que resuelven los dos

Estás trabajando en tu rama y el remote avanzó. Necesitás integrar esos cambios. Tenés dos opciones: **merge** o **rebase**. Ambas logran el mismo resultado final — tu código + los cambios del remote — pero con historias completamente distintas.

---

## git merge

Crea un **commit de merge** que une dos historias:

```
Antes:
      E - F       feature
     /
A - B - C - D     trunk

Después de git merge trunk (desde feature):
      E - F - M   feature
     /         \
A - B - C - D   (M = merge commit)
```

```bash
git checkout feature
git merge trunk
```

La historia **preserva exactamente lo que pasó**: cuándo divergió cada rama y cuándo se unieron.

---

## git rebase

**"Replanta"** tus commits encima de otra rama. No crea un commit de merge — reescribe los commits:

```
Antes:
      E - F         feature
     /
A - B - C - D       trunk

Después de git rebase trunk (desde feature):
              E'- F'  feature
             /
A - B - C - D         trunk
```

```bash
git checkout feature
git rebase trunk
```

Los commits E y F se reescriben como E' y F' (mismo contenido, nuevo SHA porque cambia el parent). La historia queda **lineal**, como si hubieras empezado a trabajar desde D.

---

## Merge vs Rebase: comparación directa

| | `merge` | `rebase` |
|---|---|---|
| **Historia** | Preserva todo tal cual | La lineariza y reescribe |
| **Commit extra** | Sí, crea un merge commit | No |
| **Conflictos** | Se resuelven una vez | Se resuelven commit por commit |
| **Seguridad** | Siempre seguro | Nunca en ramas compartidas |
| **Legibilidad** | Más ruidosa con muchos merges | Más limpia y fácil de seguir |
| **Reversibilidad** | Fácil de revertir el merge commit | Más complejo de revertir |

---

## La regla de oro del rebase 🚨

> **Nunca hagas rebase de commits que ya están en un remote compartido.**

Rebase reescribe el SHA de cada commit. Si otra persona ya tiene esos commits y vos los reescribís, las historias divergen y Git no puede reconciliarlas automáticamente. El resultado es un caos de commits duplicados y conflictos difíciles de resolver.

**Rebase es para trabajo local.** Una vez que pusheaste, usá merge.

---

## Rebase interactivo: squash y más

El rebase interactivo (`-i`) te permite editar, reordenar, combinar o eliminar commits antes de compartirlos:

```bash
git rebase -i HEAD~3    # editar los últimos 3 commits interactivamente
git rebase -i <sha>     # editar desde un commit específico hasta HEAD
```

Se abre un editor con algo así:

```
pick 9ebedbd Added 1 to the end
pick 8456d89 Added 2 to the end
pick f000c2e Added 3 to the end

# Comandos:
# p, pick   = usar el commit tal cual
# r, reword = usar el commit pero editar el mensaje
# e, edit   = pausar para hacer cambios
# s, squash = combinar con el commit anterior
# f, fixup  = como squash pero descarta el mensaje
# d, drop   = eliminar el commit
```

### Squash: combinar commits en uno

```
pick 9ebedbd Added 1 to the end
squash 8456d89 Added 2 to the end
squash f000c2e Added 3 to the end
```

Git te pide un nuevo mensaje para el commit combinado. El resultado:

```
Antes:  A - B - C1 - C2 - C3
Después: A - B - C (C1+C2+C3 combinados en uno)
```

> 💡 **Workflow recomendado:** hacé muchos commits chicos mientras desarrollás (con mensajes tipo `WIP:` o `SQUASHME:`), y antes de abrir un PR squasheá todo en uno o pocos commits limpios y descriptivos. Los revisores te lo agradecen.

---

## ours vs theirs durante un conflicto

Cuando hay un conflicto, Git marca las dos versiones:

```
<<<<<<< HEAD
A + 2           ← "ours" (tu rama actual)
=======
A + 1           ← "theirs" (la rama entrante)
>>>>>>> sha...
```

Para elegir un lado completo sin editar el archivo manualmente:

```bash
git checkout --ours README.md      # quedarse con la versión de HEAD
git checkout --theirs README.md    # quedarse con la versión entrante
```

### ⚠️ ours/theirs se invierte en rebase

Esto es uno de los puntos más confusos de Git:

| Operación | `ours` | `theirs` |
|-----------|--------|----------|
| **merge** | Tu rama (donde estás parado) | La rama que estás mergeando |
| **rebase** | La rama base (sobre la que rebaseás) | Tus commits (los que se están replicando) |

Durante un `git rebase trunk` estando en `feature`:
- Git primero checkoutea `trunk` internamente
- Luego replay tus commits uno a uno encima
- Por eso `ours` = `trunk` y `theirs` = tus commits de `feature`

> 💡 Para no confundirte: en rebase, pensá "ours es la base, theirs son mis cambios." O simplemente siempre inspeccioná el contenido del archivo antes de elegir.

---

## rerere: reuse recorded resolution

`rerere` ("REuse REcorded REsolution") hace que Git recuerde cómo resolviste un conflicto y lo aplique automáticamente la próxima vez que aparezca el mismo conflicto:

```bash
# Activar rerere (recomendado activarlo globalmente)
git config --global rerere.enabled true

# Para un solo repo
git config rerere.enabled true
```

Especialmente útil con rebase: como rebase replaya commits uno a uno, podés encontrarte resolviendo el mismo conflicto varias veces. Con rerere, solo lo resolvés una vez.

```bash
# Ver qué resoluciones tiene guardadas rerere
ls .git/rr-cache/

# Olvidar una resolución incorrecta
git rerere forget README.md
```

---

## pull --rebase vs pull (merge)

```bash
# Pull con merge (default): crea merge commits
git pull origin trunk

# Pull con rebase: historia lineal, sin merge commits
git pull --rebase origin trunk

# Configurar rebase como default para todos los pulls
git config --global pull.rebase true
```

### ¿Cuándo preferir cada uno?

**Usá merge cuando:**
- Integrás ramas de larga duración o features completos
- Querés preservar el contexto de cuándo y cómo se integró algo
- Trabajás en una rama pública que otros ya tienen

**Usá rebase cuando:**
- Actualizás tu feature branch con los últimos cambios de `trunk`/`main`
- Querés limpiar tu historia antes de un PR
- Trabajás solo en una rama que todavía no compartiste

---

## Workflow recomendado

```bash
# 1. Actualizás tu feature branch con los últimos cambios
git checkout feature
git pull --rebase origin trunk

# 2. Desarrollás con commits chicos
git add -p
git commit -m "SQUASHME: ajuste en validación"
git commit -m "SQUASHME: fix test"

# 3. Antes del PR, limpiás la historia
git rebase -i HEAD~5   # squash de tus commits en uno o pocos

# 4. Pusheás
git push -u origin feature

# 5. En trunk, merge limpio (fast-forward porque ya rebaseaste)
git checkout trunk
git merge feature      # fast-forward, sin merge commit extra
git push origin trunk
```
