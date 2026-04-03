# 05 · Git Tags

## ¿Qué es un tag?

Un tag es una **referencia con nombre a un punto específico de la historia**. La forma más fácil de pensarlo: es como una rama que no se puede mover. Mientras que las ramas avanzan con cada nuevo commit, un tag siempre apunta al mismo commit.

Se usan principalmente para marcar **versiones de releases**:

```
A - B - C - D - E - F (HEAD -> main)
        ↑           ↑
      v1.0.0      v1.1.0
```

---

## Tipos de tags

| Tipo | Cómo se crea | Qué guarda |
|------|-------------|------------|
| **Lightweight** | `git tag <nombre>` | Solo un puntero al commit (como un alias) |
| **Annotated** | `git tag -a <nombre> -m "mensaje"` | Nombre, fecha, tagger, mensaje — objeto completo en Git |

Para releases públicos se recomienda usar **annotated tags** porque contienen más metadata y son objetos de primera clase en Git (tienen su propio SHA).

---

## Operaciones básicas

```bash
# Crear un lightweight tag en el commit actual
git tag v1.0.0

# Crear un annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Crear un tag en un commit específico (no en HEAD)
git tag v0.9.0 a3f2c1e

# Listar todos los tags
git tag
git tag -l "v1.*"    # filtrar con patrón

# Ver información de un tag annotated
git show v1.0.0

# Eliminar un tag local
git tag -d v1.0.0
```

---

## Tags en git log

Los tags aparecen en el log igual que las ramas:

```bash
git log --oneline -3
# d0ba67f (HEAD -> trunk, tag: v1.0.0) feat: add login
# d53a122 (tag: v0.9.0) A + 10
# ec6930d merged
```

---

## Checkout de un tag: detached HEAD

Cuando hackeás `git checkout` sobre un tag, entrás en estado **detached HEAD** — igual que si hicieras checkout de un SHA directamente. Esto es esperado: los tags son inmutables, no podés "estar en un tag" como estás en una rama.

```bash
git checkout v1.0.0
# Note: switching to 'v1.0.0'.
# You are in 'detached HEAD' state.
# HEAD is now at d0ba67f feat: add login
```

Si necesitás hacer cambios desde ese punto, creá una rama nueva:

```bash
git switch -c hotfix/desde-v1.0.0
```

---

## Tags y remotes

A diferencia de las ramas, **los tags no se pushean automáticamente** con `git push`:

```bash
# Pushear un tag específico
git push origin v1.0.0

# Pushear todos los tags locales
git push --tags

# Pushear ramas Y tags a la vez
git pull --tags    # al hacer pull también trae los tags

# Eliminar un tag en el remote
git push origin --delete v1.0.0
git push origin :v1.0.0    # sintaxis alternativa
```

---

## Tags vs Ramas: diferencias clave

| | Rama | Tag |
|---|------|-----|
| **Se mueve** | Sí, con cada commit | No, siempre apunta al mismo commit |
| **Checkout** | Estado normal | Detached HEAD |
| **Push automático** | Sí (si está configurado) | No, requiere `--tags` |
| **Uso típico** | Trabajo en progreso | Marcar versiones o hitos |

---

## Convenciones de nombres

La convención más extendida es [Semantic Versioning](https://semver.org/):

```
v<major>.<minor>.<patch>

v1.0.0    ← primera versión estable
v1.1.0    ← nueva funcionalidad, compatible hacia atrás
v1.1.1    ← bugfix, sin cambios de API
v2.0.0    ← cambios que rompen compatibilidad (breaking changes)
```

---

## Resumen de comandos

| Comando | Para qué |
|---------|----------|
| `git tag <nombre>` | Crear tag lightweight en HEAD |
| `git tag -a <nombre> -m "msg"` | Crear tag annotated |
| `git tag` | Listar todos los tags |
| `git show <tag>` | Ver info del tag y el commit |
| `git tag -d <nombre>` | Eliminar tag local |
| `git push --tags` | Subir todos los tags al remote |
| `git push origin --delete <tag>` | Eliminar tag en el remote |
| `git checkout <tag>` | Checkout de un tag (detached HEAD) |
