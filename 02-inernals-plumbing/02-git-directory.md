# 02 · El Directorio .git

## Todo el estado de Git vive acá

La carpeta `.git` es el corazón del repositorio. Si la borrás, borrás toda la historia — los archivos del proyecto quedan en disco, pero Git no sabe nada de ellos.

```bash
find .git      # listar todo el contenido
```

---

## Estructura completa de .git

```
.git/
├── HEAD                    ← dónde estás parado ahora (rama o SHA)
├── config                  ← configuración local del repo
├── description             ← descripción (usada por GitWeb, ignorable)
├── COMMIT_EDITMSG          ← mensaje del último commit
├── index                   ← el staging area (archivo binario)
│
├── objects/                ← base de datos de todos los objetos Git
│   ├── ab/
│   │   └── cd1234...       ← objeto individual (blob, tree o commit)
│   ├── info/
│   └── pack/               ← objetos comprimidos en packfiles
│
├── refs/
│   ├── heads/              ← ramas locales (un archivo por rama)
│   │   └── trunk           ← contiene el SHA del último commit de trunk
│   ├── remotes/
│   │   └── origin/
│   │       └── trunk       ← dónde está origin/trunk (último fetch)
│   └── tags/               ← tags del repo
│
├── logs/
│   ├── HEAD                ← historial de todos los movimientos de HEAD
│   └── refs/heads/trunk    ← historial de movimientos de cada rama
│
└── hooks/                  ← scripts que se ejecutan en eventos de Git
    ├── pre-commit.sample
    ├── commit-msg.sample
    └── ...
```

---

## HEAD

El archivo `HEAD` apunta a la rama (o commit) donde estás parado:

```bash
cat .git/HEAD
# ref: refs/heads/trunk       ← estás en la rama trunk (estado normal)
```

Cuando hacés checkout a un commit específico (detached HEAD):

```bash
git checkout a3f2c1e

cat .git/HEAD
# a3f2c1e8d9b4c7f2e1a5d8c3b6f9e2a4d7c1b5f8  ← apunta directamente a un SHA
```

> ⚠️ En estado "detached HEAD", los commits que hagas no pertenecen a ninguna rama. Si cambiás de rama sin crear una nueva, esos commits se pierden (aunque se pueden recuperar con `git reflog`).

---

## refs/heads/ — las ramas locales

Cada rama local es simplemente un **archivo de texto con el SHA del último commit**:

```bash
cat .git/refs/heads/trunk
# a3f2c1e8d9b4c7f2e1a5d8c3b6f9e2a4d7c1b5f8
```

Cuando hacés un commit nuevo en `trunk`, Git actualiza ese archivo con el nuevo SHA. Eso es todo lo que es una rama: **un puntero a un commit que se mueve automáticamente**.

```bash
# Ver a qué commit apunta cada rama
cat .git/refs/heads/trunk
cat .git/refs/heads/feature-login
```

---

## refs/remotes/ — ramas del remote

Igual que las locales, pero representan el estado del remote **la última vez que hiciste fetch o pull**:

```bash
cat .git/refs/remotes/origin/trunk
# 33d6d9645ac045c6d99eba9d6706d0d48421e582
```

Si tu rama local está adelantada al remote, estos SHAs van a ser distintos. Eso es exactamente lo que `git status` te muestra con "Your branch is ahead of 'origin/trunk' by 2 commits."

---

## El index (staging area)

El archivo `.git/index` es la representación binaria del staging area. No está pensado para leerlo directamente, pero podés inspeccionarlo con:

```bash
git ls-files --stage    # lista los archivos en el index con sus SHAs y permisos
```

---

## logs/ — el reflog

Git guarda un historial de todos los movimientos de HEAD en `.git/logs/`. Esto es lo que permite `git reflog`:

```bash
git reflog                # ver todos los movimientos de HEAD
git reflog show trunk     # movimientos de una rama específica
```

### El reflog como red de seguridad

```bash
# Hiciste un reset --hard y "perdiste" commits
git reflog
# HEAD@{0}: reset: moving to HEAD~2
# HEAD@{1}: commit: feat: add login page    ← acá estaba antes del reset
# HEAD@{2}: commit: fix: email validation

# Recuperar el commit "perdido"
git checkout HEAD@{1}
# o directamente
git branch recovery-branch HEAD@{1}
```

> 💡 El reflog es local — no se sube al remote. Git lo guarda por **90 días** por defecto. Mientras ese tiempo no pase, casi nada es irrecuperable.

---

## hooks/ — automatización

Los hooks son scripts que Git ejecuta automáticamente en ciertos eventos. Los archivos `.sample` son ejemplos desactivados — para activar uno, borrá la extensión `.sample` y dále permisos de ejecución:

```bash
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

### Hooks más usados

| Hook | Cuándo se ejecuta | Uso típico |
|------|-------------------|------------|
| `pre-commit` | Antes de crear un commit | Correr linter o tests |
| `commit-msg` | Al escribir el mensaje | Validar formato del mensaje |
| `post-commit` | Después de crear un commit | Notificaciones |
| `pre-push` | Antes de hacer push | Correr test suite completo |
| `post-merge` | Después de un merge | Actualizar dependencias |

### Ejemplo: pre-commit que valida el código

```bash
#!/bin/sh
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit abortado."
  exit 1
fi
```

> ⚠️ Los hooks son **locales** — no se comparten con el repo remoto al hacer push. Si querés hooks compartidos en equipo, usá herramientas como [Husky](https://typicode.github.io/husky/) que los guardan en el repo.

---

## config — configuración local

```bash
cat .git/config
```

```ini
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    url = https://github.com/usuario/repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "trunk"]
    remote = origin
    merge = refs/heads/trunk
```

Todo lo que configurás con `git config --local` (o `git config` sin flag dentro de un repo) termina acá.

---

## Resumen: archivos clave

| Archivo / Directorio | Qué contiene |
|----------------------|--------------|
| `HEAD` | Rama o commit actual |
| `config` | Configuración local |
| `index` | Staging area (binario) |
| `objects/` | Todos los commits, trees y blobs |
| `refs/heads/` | Punteros de ramas locales |
| `refs/remotes/` | Estado del remote (último fetch) |
| `logs/` | Historial de movimientos (reflog) |
| `hooks/` | Scripts de automatización |

> La conclusión más importante: Git no es magia. Es una base de datos de archivos de texto y binarios con convenciones muy precisas. Entender `.git` es entender Git.
