# 03 · Commits e Historial

## ¿Qué es un commit?

Un commit es un **snapshot completo del proyecto** en un momento dado. No es un diff (conjunto de cambios) — es una foto entera de todos los archivos trackeados.

Cada commit contiene:
- El árbol de archivos completo (tree)
- Referencia al commit anterior (parent)
- Autor, email, fecha y mensaje
- Un SHA único de 40 caracteres

> 💡 Git no guarda "qué cambió" — guarda "cómo estaba todo". La eficiencia viene de que los archivos que no cambiaron entre commits apuntan al mismo blob, sin duplicar datos.

---

## git commit

```bash
git commit -m "mensaje"           # commit con mensaje inline
git commit                        # abre el editor para escribir el mensaje
git commit -am "mensaje"          # stagea archivos tracked Y commitea en un paso
git commit --amend                # modifica el último commit (mensaje o contenido)
git commit --amend --no-edit      # modifica el último commit sin cambiar el mensaje
```

> ⚠️ `git commit -am` no agrega archivos **untracked**, solo cambios en archivos que Git ya conoce. Para archivos nuevos siempre necesitás `git add` primero.

> ⚠️ `git commit --amend` reescribe la historia. Nunca lo hagas en commits que ya pusheaste a un remote compartido — va a generar conflictos para todos los demás.

### Buenas prácticas para mensajes de commit

Un buen mensaje explica el **por qué**, no el qué (el qué ya lo muestra el diff):

```
feat: add user authentication
fix: prevent crash when input is empty
refactor: extract validation logic to separate function
docs: update README with setup instructions
chore: update dependencies
```

Convención popular: [Conventional Commits](https://www.conventionalcommits.org/)

---

## git log

Ver el historial de commits:

```bash
git log                               # historial completo
git log --oneline                     # una línea por commit
git log --graph                       # muestra el grafo de branches
git log --decorate                    # muestra a dónde apuntan HEAD y las ramas
git log --graph --decorate --oneline  # la combinación más útil del día a día
git log -n 5                          # últimos 5 commits
git log --author="Joaquin"            # filtrar por autor
git log --since="2 weeks ago"         # commits de las últimas 2 semanas
git log -- archivo.js                 # historial de un archivo específico
git log --all                         # incluye ramas remotas y todas las refs
```

### Ejemplo de salida con --graph --decorate --oneline

```
* a3f2c1e (HEAD -> trunk) feat: add login page
* 8b1d4f2 fix: correct email validation
| * c9e3a1f (feature-dashboard) feat: add dashboard skeleton
|/
* 33d6d96 (origin/trunk) docs: initial commit
```

- `*` = nodo en el grafo (un commit)
- `HEAD -> trunk` = estás parado en trunk, y trunk apunta a este commit
- `origin/trunk` = dónde está el repo remoto (la última vez que hiciste fetch)

---

## El SHA de un commit

Cada commit tiene un identificador único de 40 caracteres:

```
a3f2c1e8d9b4c7f2e1a5d8c3b6f9e2a4d7c1b5f8
```

No necesitás escribirlo completo — Git acepta los **primeros 7 caracteres**:

```bash
git show a3f2c1e
git cat-file -p a3f2c1e
```

### ¿Por qué tu SHA es diferente al de otra persona?

El SHA se calcula en base a: contenido del cambio + autor + email + timestamp + SHA del commit padre. Cualquier diferencia en alguno de esos datos produce un hash completamente distinto.

---

## git show

Ver el contenido y el diff de un commit específico:

```bash
git show                  # muestra el último commit
git show a3f2c1e          # muestra un commit específico
git show HEAD             # equivalente al primero
git show HEAD~1           # el commit anterior al último
git show HEAD~2           # dos commits atrás
```

---

## Referencias relativas

Para navegar la historia sin copiar SHAs:

```bash
HEAD        # el commit donde estás parado ahora
HEAD~1      # un commit atrás (o HEAD^)
HEAD~3      # tres commits atrás
HEAD^2      # el segundo padre (útil en merge commits)
```

```bash
git show HEAD~1           # ver el commit anterior
git diff HEAD~3 HEAD      # qué cambió en los últimos 3 commits
git log HEAD~5..HEAD      # historial de los últimos 5 commits
```

---

## git diff entre commits

```bash
git diff HEAD~1 HEAD          # qué cambió entre el último y el anterior
git diff a3f2c1e 8b1d4f2      # entre dos commits específicos
git diff main..feature        # entre dos ramas
git diff HEAD~3..HEAD         # todos los cambios de los últimos 3 commits
```

---

## git stash

Guardado temporal de cambios sin commitear. Útil cuando necesitás cambiar de contexto con trabajo a medias:

```bash
git stash                     # guarda los cambios y deja el working tree limpio
git stash pop                 # recupera los cambios guardados y borra el stash
git stash apply               # recupera los cambios pero mantiene el stash
git stash list                # ver todos los stashes guardados
git stash drop                # eliminar el stash más reciente
git stash drop stash@{2}      # eliminar un stash específico
git stash apply stash@{1}     # aplicar un stash específico sin eliminarlo
git stash -u                  # incluir archivos untracked en el stash
```

### stash vs commit: ¿cuándo usar cada uno?

| Situación | Solución |
|-----------|----------|
| Trabajo terminado, listo para guardar | `git commit` |
| Trabajo a medias, necesito cambiar de rama | `git stash` |
| Quiero pausar y no perder los cambios | `git stash` |
| Quiero un punto de control aunque esté incompleto | `git commit` (con mensaje "WIP") |

### Flujo típico con stash

```bash
# Estás trabajando en un feature a medias
git stash                  # guardás el trabajo sin commitear

git checkout otra-rama     # cambiás de rama (o hacés un pull, fix urgente, etc.)
# ... hacés lo que necesitabas ...

git checkout mi-rama       # volvés a tu rama
git stash pop              # recuperás tu trabajo donde lo dejaste
```

> 💡 `git stash pop` = `git stash apply` + `git stash drop`. Si el apply genera conflictos, el stash **no** se borra automáticamente para que no pierdas nada.

---

## git blame

Ver quién escribió cada línea de un archivo y en qué commit:

```bash
git blame archivo.js
git blame -L 10,20 archivo.js    # solo líneas 10 a 20
git blame -w archivo.js          # ignora cambios de espaciado
```

Muy útil para entender el contexto de una línea de código y en qué PR o tarea se introdujo.

---

## Resumen de comandos

| Comando | Para qué sirve |
|---------|----------------|
| `git commit -m "msg"` | Crear un commit |
| `git commit --amend` | Modificar el último commit |
| `git log --graph --decorate --oneline` | Historial visual con ramas |
| `git show <sha>` | Ver contenido y diff de un commit |
| `git diff HEAD~1` | Qué cambió en el último commit |
| `git stash` | Guardar cambios temporalmente |
| `git stash pop` | Recuperar cambios guardados |
| `git blame` | Ver autoría línea por línea |
