# 04 · Git Worktree

## El problema que resuelve

Estás en flow state trabajando en `feature/login`. Llega un mensaje urgente: hay que investigar un bug en `main` ahora mismo.

Tus opciones sin worktrees:
1. `git stash` → cambiar de rama → investigar → volver → `git stash pop`
2. Hacer un commit "WIP" para poder cambiar de rama
3. Clonar el repo en otra carpeta (pesado y duplica todo)

Con worktrees: **crear un directorio separado con otra rama, sin tocar tu trabajo actual**.

---

## ¿Qué es un worktree?

Cuando hacés `git init`, Git crea el **main working tree** — el directorio del proyecto con su `.git/`. Un **linked working tree** es un directorio adicional conectado al mismo `.git`, que puede tener una rama distinta checkeada.

```
hello-git/          ← main working tree (.git/ completo)
  ├── .git/
  ├── src/
  └── README.md

foo-bar/            ← linked working tree (.git es solo un archivo puntero)
  ├── .git          ← archivo: "gitdir: ../hello-git/.git/worktrees/foo-bar"
  ├── src/
  └── README.md     ← mismos archivos, rama diferente
```

El linked worktree **no duplica la historia de Git** — solo apunta al `.git` del repo principal. Por eso es liviano.

---

## Operaciones básicas

```bash
# Crear un linked worktree (la rama se llama igual que el basename del path)
git worktree add ../foo-bar

# Crear un worktree con una rama existente
git worktree add ../hotfix-branch hotfix/login-bug

# Crear un worktree con una rama nueva desde un punto específico
git worktree add -b nueva-rama ../nueva-carpeta origin/trunk

# Listar todos los worktrees
git worktree list

# Eliminar un worktree (borra el directorio y limpia .git)
git worktree remove ../foo-bar

# Limpiar referencias a worktrees que ya no existen en disco
git worktree prune
```

---

## El .git en un linked worktree

Una de las cosas más llamativas: en el linked worktree, `.git` no es una carpeta sino un **archivo de texto**:

```bash
cat foo-bar/.git
# gitdir: /home/usuario/hello-git/.git/worktrees/foo-bar
```

Y en el repo principal, Git guarda el estado del worktree:

```
hello-git/.git/worktrees/
└── foo-bar/
    ├── HEAD        ← a qué commit/rama está el worktree
    ├── gitdir      ← path al .git file del worktree
    ├── index       ← staging area del worktree
    └── logs/
```

---

## Worktrees vs Stash: comparación

| | Stash | Worktree |
|---|-------|----------|
| **Complejidad** | Simple | Un poco más de setup |
| **Estado conservado** | Solo cambios (no el contexto del editor) | Directorio completo con su propio estado |
| **Múltiples contextos** | Una pila de stashes | Un directorio por contexto, siempre disponibles |
| **npm install** | Una sola vez | Una vez por worktree |
| **Cambio de contexto** | `git stash` + `git checkout` + `git stash pop` | `cd ../otro-worktree` |
| **Ideal para** | Interrupciones cortas | Trabajo paralelo frecuente o largo |

> 💡 Con worktrees nunca necesitás hacer stash para cambiar de contexto. Solo abrís otra carpeta en tu editor y listo.

---

## Flujo típico con worktrees

```bash
# Estás trabajando en feature/login
cd ~/projects/mi-repo    # main worktree, rama feature/login

# Llega el bug urgente en main
git worktree add ../mi-repo-hotfix main

# Abrís otra ventana del editor en ../mi-repo-hotfix
cd ../mi-repo-hotfix
# ... investigás, fixeás, commiteás ...
git commit -m "fix: corregir bug de login en main"
git push origin main

# Eliminás el worktree cuando terminaste
git worktree remove ../mi-repo-hotfix

# Volvés a tu trabajo original sin haber tocado nada
cd ~/projects/mi-repo
# todo sigue exactamente como lo dejaste
```

---

## Consideraciones prácticas

### Ventajas
- Cambio de contexto instantáneo sin stash ni commits WIP
- Cada worktree tiene su propio staging area e index
- Los worktrees son livianos (no duplican la base de datos de objetos)

### Desventajas
- Si cada rama requiere `npm install`, tenés que hacerlo en cada worktree
- Lenguajes como Rust con compilación pesada duplican el tiempo de build por rama
- Los archivos `.env` locales no se comparten — hay que configurarlos en cada worktree

### ¿Cuándo conviene stash de todas formas?
Para interrupciones muy cortas (menos de 5 minutos) donde crear y eliminar un worktree sería más overhead que simplemente hacer stash.

---

## Resumen de comandos

| Comando | Para qué |
|---------|----------|
| `git worktree add <path>` | Crear un nuevo worktree |
| `git worktree add <path> <rama>` | Crear worktree con rama existente |
| `git worktree list` | Ver todos los worktrees activos |
| `git worktree remove <path>` | Eliminar un worktree limpiamente |
| `git worktree prune` | Limpiar referencias a worktrees borrados manualmente |
