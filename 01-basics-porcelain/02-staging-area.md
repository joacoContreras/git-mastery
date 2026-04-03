# 02 · Staging Area

## ¿Qué es el staging area?

El **staging area** (también llamado *index*) es la zona intermedia entre tus archivos en disco y un commit. Es lo que hace a Git especialmente poderoso: podés decidir con precisión **qué cambios incluir en cada commit**, sin necesidad de commitear todo junto.

```
Working Tree  →  Staging Area  →  Repositorio (.git)
  (editar)         (git add)         (git commit)
```

---

## Estados de un archivo

```
Untracked  ──git add──►  Staged  ──git commit──►  Tracked
                            ▲                         │
                            └──────git add────────────┘
                                  (si se modifica)
```

| Estado | Descripción |
|--------|-------------|
| **Untracked** | Git no sabe nada de este archivo. Si lo borrás, se pierde para siempre. |
| **Staged** | El archivo (o sus cambios) está en el index, listo para el próximo commit. |
| **Tracked** | Git ya tiene registro de este archivo en al menos un commit. |
| **Modified** | El archivo es tracked pero tiene cambios que todavía no están staged. |

---

## git add

```bash
git add <archivo>     # agrega un archivo específico
git add .             # agrega todo lo que cambió en el directorio actual
git add *.js          # agrega todos los .js (glob pattern)
git add src/          # agrega todo dentro de src/
git add -p            # modo interactivo: elegís qué partes stagear (ver abajo)
```

### git add -p (patch mode) — el más poderoso en la práctica

Permite stagear **partes de un archivo** (hunks), no el archivo entero. Muy útil cuando hiciste varios cambios distintos en el mismo archivo y querés commitearlos por separado:

```bash
git add -p archivo.js
```

Te va mostrando cada bloque de cambios y te pregunta:

| Tecla | Acción |
|-------|--------|
| `y` | Stagear este hunk |
| `n` | No stagear |
| `s` | Dividir en hunks más chicos |
| `e` | Editar el hunk manualmente |
| `q` | Salir |

> 💡 `git add -p` es una de las herramientas más subestimadas de Git. Te permite hacer commits atómicos y limpios incluso cuando mezclaste varios cambios en el mismo archivo.

---

## git status

Muestra el estado actual del repo: qué está staged, qué está modificado y qué es untracked.

```bash
git status        # versión completa
git status -s     # versión corta (compacta)
```

### Ejemplo de salida completa

```
On branch trunk

Changes to be committed:          ← staged (listo para commitear)
  (use "git rm --cached <file>..." to unstage)
        new file:   README.md

Changes not staged for commit:    ← modificado pero no staged
  (use "git add <file>..." to update what will be committed)
        modified:   app.js

Untracked files:                  ← Git no los conoce
  (use "git add <file>..." to include in what will be committed)
        notas.txt
```

### Salida corta (`-s`)

```
A  README.md     ← Added (staged)
 M app.js        ← Modified (no staged)
M  utils.js      ← Modified (staged)
?? notas.txt     ← Untracked
```

La primera columna es el estado en el staging area, la segunda es el estado en el working tree.

---

## git rm --cached

Saca un archivo del staging area **sin borrarlo del disco**:

```bash
git rm --cached archivo.txt
git rm --cached -r carpeta/    # recursivo para directorios
```

Útil cuando hiciste `git add` de algo que no querías (por ejemplo, un `.env` con credenciales).

### Diferencia con git rm

| Comando | Staging area | Disco |
|---------|-------------|-------|
| `git rm --cached` | Elimina | Intacto |
| `git rm` | Elimina | También elimina |

---

## git diff

Ver exactamente qué cambió antes de commitear:

```bash
git diff                  # cambios en working tree que NO están staged
git diff --staged         # cambios que SÍ están staged (lo que iría en el próximo commit)
git diff HEAD             # todos los cambios respecto al último commit
git diff <sha1> <sha2>    # diferencia entre dos commits
git diff main..feature    # diferencia entre dos ramas
```

> 💡 Antes de hacer `git commit`, el hábito de correr `git diff --staged` te salva de commitear cosas que no querías.

---

## Flujo completo de ejemplo

```bash
# 1. Crear un archivo
echo "# Mi proyecto" > README.md

# 2. Ver que está untracked
git status

# 3. Stagear
git add README.md

# 4. Revisar qué se va a commitear
git diff --staged

# 5. Commitear
git commit -m "docs: add README"

# 6. Verificar que el working tree está limpio
git status
# On branch trunk
# nothing to commit, working tree clean
```

---

## .gitignore

Archivo que le dice a Git qué archivos y carpetas ignorar. Los archivos ignorados nunca aparecen como untracked en `git status`:

```gitignore
node_modules/
.env
*.log
dist/
.DS_Store
```

```bash
git status --ignored    # ver qué archivos están siendo ignorados
```

> ⚠️ Si ya commiteaste un archivo y después lo agregás al `.gitignore`, Git no deja de trackearlo automáticamente. Primero tenés que hacer:
> ```bash
> git rm --cached archivo.txt
> git commit -m "chore: untrack archivo.txt"
> ```

---

## Buenas prácticas

- **Un commit = una idea.** Usá `git add -p` para separar cambios distintos en commits distintos.
- **Commitear temprano y seguido.** Podés reorganizar la historia después con squash.
- **Revisá antes de commitear.** `git diff --staged` te muestra exactamente qué vas a commitear.
- **Nunca committear credenciales.** Si ya lo hiciste, no alcanza con borrarlas en el próximo commit — la historia de Git las guarda. Hay que reescribir la historia o revocar las credenciales de inmediato.
