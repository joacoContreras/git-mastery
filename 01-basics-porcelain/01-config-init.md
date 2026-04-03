# 01 · Configuración e Inicialización

## ¿Qué es Git?

Git es un **sistema de control de versiones distribuido (VCS)**. A diferencia de los sistemas centralizados clásicos donde hasta hacer checkout requería permisos de admin, Git permite trabajar completamente en local y divergir tanto como se necesite.

Los comandos de Git se dividen en dos categorías:

| Categoría | Nombre | Descripción |
|-----------|--------|-------------|
| Alto nivel | **Porcelain** | Comandos del día a día: `add`, `commit`, `push`, `pull` |
| Bajo nivel | **Plumbing** | Comandos internos: `cat-file`, `hash-object`, `ls-tree` |

> La mayoría del tiempo usás porcelain. Pero entender plumbing es lo que separa a quien *usa* Git de quien *entiende* Git.

---

## git config

Git tiene un sistema de configuración en capas. Cada nivel más específico sobreescribe al anterior:

```
system  →  global  →  local  →  worktree
(menos específico)              (más específico)
```

| Flag | Alcance | Archivo |
|------|---------|---------|
| `--system` | Todos los usuarios del sistema | `/etc/gitconfig` |
| `--global` | Tu usuario, todos los repos | `~/.gitconfig` |
| `--local` | Solo el repo actual | `.git/config` |

### Configuración esencial

```bash
# Agregar nombre y email (necesario para que los commits queden a tu nombre)
git config --add --global user.name "tu nombre"
git config --add --global user.email "tu@email.com"

# Verificar un valor
git config --get user.name

# Listar toda la configuración
git config --list
git config --list --global   # solo global
git config --list --local    # solo local del repo actual

# Buscar con regex
git config --get-regexp user
# user.name     Joaquin
# user.email    cjoaquin835@gmail.com
```

### Keys duplicadas: comportamiento importante

Las keys **no son únicas**. Podés agregar la misma key varias veces:

```bash
git config --add fem.dev "is great"
git config --add fem.dev "is amazing"

git config --get-regexp fem
# fem.dev is great
# fem.dev is amazing

git config --get fem.dev
# is amazing  ← devuelve el valor más reciente de la fuente más específica
```

Para eliminar:

```bash
git config --unset fem.dev        # falla si hay múltiples valores con esa key
git config --unset-all fem.dev    # elimina todos los duplicados
git config --remove-section fem   # elimina toda la sección
```

### Prioridad entre niveles

```bash
git config --global --add fem.dev "global-value"
git config --local  --add fem.dev "local-value"

git config --get fem.dev
# local-value  ← local gana sobre global
```

La regla es: **más específico gana, y dentro del mismo nivel, el último valor agregado**.

> 💡 La configuración no es magia — es un archivo de texto. Podés ver la config local con `cat .git/config` y la global con `cat ~/.gitconfig`.

---

## git init

Crea un repositorio Git nuevo en el directorio actual. Todo el estado de Git vive dentro de la carpeta `.git`.

```bash
mkdir mi-proyecto
cd mi-proyecto
git init
# Initialized empty Git repository in .../mi-proyecto/.git/
```

### Cambiar la rama por defecto

```bash
# Cambiar de "master" a "trunk" (o "main") para todos los repos nuevos
git config --global init.defaultBranch trunk

# Renombrar la rama en un repo ya existente
git branch -m master trunk
```

---

## La carpeta .git

```bash
find .git
```

```
.git/
├── HEAD          ← apunta a la rama o commit actual
├── config        ← configuración local del repo
├── objects/      ← acá vive TODO: commits, trees, blobs
├── refs/
│   ├── heads/    ← ramas locales
│   └── tags/     ← tags
└── hooks/        ← scripts que se ejecutan en ciertos eventos de Git
```

> ⚠️ Si borrás la carpeta `.git`, borrás el repositorio entero y toda su historia. Los archivos del proyecto quedan en disco, pero Git no sabe nada de ellos.

---

## Navegar el manual desde la terminal

```bash
man git-config    # manual de git config
man git-log       # manual de git log
man git           # manual general de Git
```

Una vez dentro del manual (que usa `less`):

| Tecla | Acción |
|-------|--------|
| `j` / `k` | Bajar / subir una línea |
| `d` / `u` | Bajar / subir media página |
| `/<término>` | Buscar un término |
| `n` / `N` | Siguiente / anterior resultado |
| `q` | Salir |

---

## Flujo de trabajo básico

```bash
git status                            # ver estado del repo
git add <archivo>                     # agregar al staging area
git add .                             # agregar todo
git commit -m "mensaje"               # commitear
git log --graph --decorate --oneline  # ver historial visual
```

---

## Términos clave

| Término | Descripción |
|---------|-------------|
| **repo** | Proyecto trackeado por Git |
| **commit** | Snapshot del proyecto en un momento dado |
| **SHA** | Hash único de 40 caracteres que identifica cada objeto de Git |
| **index / staging area** | Zona intermedia entre tus archivos y el commit |
| **working tree** | Los archivos de tu proyecto tal como están en disco |
| **untracked** | Archivo que Git no conoce todavía |
| **staged** | Cambio listo para ser commiteado |
| **tracked** | Archivo que Git ya conoce (fue commiteado al menos una vez) |
| **squash** | Combinar varios commits en uno solo |
| **remote** | El mismo repo en otro lugar (ej: GitHub) |

> 💡 Si borrás un archivo **untracked**, Git no puede recuperarlo porque nunca lo conoció. Hacé `git add` seguido de `git commit` temprano y seguido — siempre podés reorganizar la historia después con squash.
