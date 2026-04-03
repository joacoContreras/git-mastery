# 01 · Hash Objects y el Modelo de Datos de Git

## Todo en Git es un objeto

Git almacena **todo** en forma de objetos dentro de `.git/objects/`. Hay cuatro tipos:

| Tipo | Analogía | Descripción |
|------|----------|-------------|
| **blob** | archivo | Contenido de un archivo en un momento dado |
| **tree** | directorio | Lista de blobs y otros trees con sus nombres y permisos |
| **commit** | snapshot | Apunta a un tree + metadata (autor, fecha, mensaje, parent) |
| **tag** | etiqueta anotada | Apunta a un commit con información extra |

---

## El SHA: cómo Git identifica objetos

Cada objeto tiene un identificador único de 40 caracteres generado con SHA-1:

```
5ba786fcc93e8092831c01e71444b9baa2228a4f
```

Este hash se calcula a partir del **contenido del objeto**. Eso significa:
- Dos archivos con el mismo contenido → mismo SHA (Git no duplica datos)
- Cualquier cambio mínimo en el contenido → SHA completamente diferente
- Podés verificar la integridad de cualquier objeto recalculando su hash

> 💡 Git no es solo un sistema de versiones — es una **base de datos de objetos direccionada por contenido** (content-addressable storage). El SHA no es un número de versión, es la identidad del contenido.

---

## Dónde viven los objetos

Los objetos se guardan en `.git/objects/` usando los **primeros 2 caracteres del SHA como directorio** y los **38 restantes como nombre de archivo**:

```
.git/objects/
├── 5b/
│   └── a786fcc93e8092831c01e71444b9baa2228a4f   ← commit
├── 4e/
│   └── 507fdc6d9044ccd8a4a3061324c9f711c4667d   ← tree
├── 9a/
│   └── 71f81a4b4754b686fd37cbb3c72d0250d344aa   ← blob
└── pack/    ← objetos comprimidos (Git los empaqueta para eficiencia)
```

```bash
# Ver todos los objetos sueltos en la base de datos
find .git/objects -type f
```

---

## git cat-file: el inspector de objetos

El comando de plumbing más útil para entender qué tiene Git adentro:

```bash
git cat-file -p <sha>    # muestra el contenido del objeto (pretty print)
git cat-file -t <sha>    # muestra el tipo: blob, tree, commit, tag
git cat-file -s <sha>    # muestra el tamaño en bytes
```

### Recorrer la cadena commit → tree → blob

Dado el SHA de un commit, podés llegar al contenido de cualquier archivo en 3 pasos:

```bash
# Paso 1: inspeccionar el commit
git cat-file -p 5ba786f

# tree 4e507fdc6d9044ccd8a4a3061324c9f711c4667d
# parent 33d6d9645ac045c6d99eba9d6706d0d48421e582
# author Joaquin <cjoaquin835@gmail.com> 1705891256 -0300
#
# mi primer commit

# Paso 2: inspeccionar el tree
git cat-file -p 4e507fd

# 100644 blob 9a71f81a4b4754b686fd37cbb3c72d0250d344aa    first.md

# Paso 3: inspeccionar el blob (el archivo en sí)
git cat-file -p 9a71f81

# hello world
```

---

## git hash-object

Permite calcular el SHA de cualquier contenido o crear objetos manualmente en la base de datos:

```bash
# Calcular el SHA de un archivo sin guardarlo
git hash-object archivo.txt

# Guardar el objeto en la base de datos de Git (-w = write)
git hash-object -w archivo.txt

# Calcular el SHA de un string
echo "hello world" | git hash-object --stdin
# 8ab686eafeb1f44702738c8b0f24f2567c36da6d
```

> 💡 Siempre que el contenido sea el mismo, el SHA será el mismo — sin importar el nombre del archivo, la fecha, el sistema operativo, o quién lo creó.

---

## ¿Por qué Git no guarda diffs?

Una confusión muy común es pensar que Git guarda solo "los cambios" entre versiones. La realidad:

- Git guarda el **archivo completo** como un blob cada vez que cambia
- Si un archivo **no cambió** entre commits, el nuevo commit apunta al **mismo blob** — sin duplicación
- Esto es eficiente: los archivos que no cambian no se vuelven a guardar

```
Commit 1:  tree → { first.md → blob:9a71f81 }

Commit 2:  tree → { first.md → blob:9a71f81,   ← mismo blob, sin duplicar
                    second.md → blob:7f112b1 }  ← nuevo blob
```

> Cuando Git muestra un diff con `git show` o `git log -p`, lo está **calculando al momento** comparando los blobs — no lo tenía guardado.

---

## git cat-file vs git show

Ambos sirven para inspeccionar objetos, pero con propósitos distintos:

| Comando | Nivel | Cuándo usarlo |
|---------|-------|---------------|
| `git show <sha>` | Porcelain | Ver un commit formateado con diff incluido |
| `git cat-file -p <sha>` | Plumbing | Ver la estructura interna raw de cualquier objeto |
| `git cat-file -t <sha>` | Plumbing | Saber el tipo de un objeto (blob, tree, commit) |

```bash
# git show en un commit → muestra el diff formateado
git show a3f2c1e

# git cat-file en el mismo commit → muestra la estructura interna
git cat-file -p a3f2c1e
# tree 4e507fd...
# parent 33d6d96...
# author ...
```

---

## Verificar integridad

Como los SHAs son deterministas, Git puede detectar corrupción de datos:

```bash
git fsck           # verifica la integridad de todos los objetos
git fsck --full    # verificación más exhaustiva
```

---

## Resumen visual del modelo de objetos

```
commit (5ba786f)
  ├── parent: 33d6d96    ← commit anterior
  ├── author / committer / mensaje
  └── tree (4e507fd)
        ├── first.md   → blob (9a71f81)   "hello world"
        └── src/       → tree (a1b2c3d)
                └── app.js → blob (...)   "console.log(...)"
```

Una vez que entendés este modelo, Git deja de ser magia. Todo comando de porcelain es en el fondo una operación sobre este grafo de objetos.
