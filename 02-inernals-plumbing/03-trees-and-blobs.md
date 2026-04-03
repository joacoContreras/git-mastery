# 03 · Trees y Blobs

## Los dos tipos de objetos de contenido

En el modelo de datos de Git, todo el contenido del proyecto está representado por dos tipos de objetos:

| Tipo | Analogía | Qué contiene |
|------|----------|--------------|
| **blob** | archivo | El contenido puro de un archivo (sin nombre, sin permisos) |
| **tree** | directorio | Lista de entradas: blobs y otros trees, con nombre y permisos |

> Un **blob** no sabe cómo se llama. El nombre del archivo lo conoce el **tree** que lo contiene. Esto tiene una implicación importante: si dos archivos tienen el mismo contenido pero distinto nombre, Git usa **el mismo blob** para ambos.

---

## Estructura de un blob

Un blob es simplemente el contenido binario del archivo, sin ningún metadato:

```bash
git cat-file -t 9a71f81
# blob

git cat-file -p 9a71f81
# hello world
```

El blob no sabe que se llama `first.md`, ni dónde está, ni cuándo fue creado. Solo es bytes.

---

## Estructura de un tree

Un tree contiene una lista de entradas con el formato:

```
<modo> <tipo> <sha>                                        <nombre>
100644 blob   9a71f81a4b4754b686fd37cbb3c72d0250d344aa    first.md
100644 blob   7f112b196b963ff72675febdbb97da5204f9497e    second.md
040000 tree   a1b2c3d4e5f6...                             src/
```

### Modos de archivo

| Modo | Tipo | Significado |
|------|------|-------------|
| `100644` | blob | Archivo normal |
| `100755` | blob | Archivo ejecutable |
| `120000` | blob | Symlink |
| `040000` | tree | Directorio (subdirectorio) |
| `160000` | commit | Submódulo (gitlink) |

---

## Trees anidados: cómo Git representa directorios

Los subdirectorios son simplemente trees dentro de otros trees:

```bash
# Tree raíz del proyecto
git cat-file -p <sha-tree-raiz>
# 100644 blob abc123...    README.md
# 040000 tree def456...    src/

# Tree del subdirectorio src/
git cat-file -p def456
# 100644 blob ghi789...    app.js
# 100644 blob jkl012...    utils.js
# 040000 tree mno345...    components/
```

No hay un concepto especial de "directorio" en Git — es solo un tree apuntando a otro tree.

---

## Cómo Git aprovecha los objetos compartidos

El modelo de objetos de Git es brillante en eficiencia:

```
Commit A:
  tree → { README.md → blob:abc,  app.js → blob:def }

Commit B (solo cambió app.js):
  tree → { README.md → blob:abc,  ← mismo blob, sin duplicar
            app.js → blob:xyz }   ← nuevo blob
```

Git no vuelve a guardar `README.md` en el commit B porque el contenido no cambió y el SHA es el mismo. Solo crea un nuevo tree y un nuevo blob para `app.js`.

> Esto también aplica entre ramas: si dos ramas comparten archivos con el mismo contenido, comparten los mismos blobs.

---

## git ls-tree: explorar trees sin cat-file

```bash
git ls-tree HEAD                   # tree del commit actual
git ls-tree HEAD src/              # tree de un subdirectorio
git ls-tree -r HEAD                # todos los archivos recursivamente
git ls-tree -r --name-only HEAD    # solo los nombres (útil para listar archivos)
git ls-tree <sha>                  # tree de cualquier commit o SHA
```

---

## Recorrer el árbol completo manualmente

```bash
# 1. Obtener el SHA del commit
git log --oneline -1
# a3f2c1e feat: add second file

# 2. Ver el commit
git cat-file -p a3f2c1e
# tree 4e507fd...
# parent 33d6d96...

# 3. Ver el tree raíz
git cat-file -p 4e507fd
# 100644 blob 9a71f81...    first.md
# 100644 blob 7f112b1...    second.md

# 4. Ver un archivo específico
git cat-file -p 9a71f81
# hello world
```

---

## git cat-file vs git ls-tree vs git show

| Comando | Nivel | Para qué sirve |
|---------|-------|----------------|
| `git show <sha>` | Porcelain | Ver un commit formateado con su diff |
| `git cat-file -p <sha>` | Plumbing | Ver el contenido raw de cualquier objeto |
| `git cat-file -t <sha>` | Plumbing | Ver el tipo de un objeto |
| `git ls-tree <sha>` | Plumbing | Ver el contenido de un tree de forma legible |

---

## Crear objetos manualmente (plumbing puro)

Podés crear blobs directamente sin pasar por el staging area:

```bash
# Crear un blob a partir de un string
echo "hola mundo" | git hash-object -w --stdin
# devuelve el SHA del blob creado y lo guarda en .git/objects/

# Verificar que existe
git cat-file -t <sha-devuelto>
# blob
```

Esto es lo que Git hace internamente cuando hacés `git add` — crea un blob para cada archivo.

---

## El grafo completo

```
HEAD
 │
 ▼
commit (a3f2c1e)
 ├── parent → commit (33d6d96)
 │              ├── parent → (ninguno, es el root commit)
 │              └── tree → { first.md → blob }
 └── tree (4e507fd)
      ├── first.md   → blob (9a71f81)  ← mismo blob que el commit anterior
      ├── second.md  → blob (7f112b1)  ← nuevo blob
      └── src/       → tree (a1b2c3)
               └── app.js → blob (...)
```

Cada commit es un nodo en un **grafo acíclico dirigido (DAG)**. Los punteros siempre van de hijo a padre (hacia atrás en el tiempo), nunca al revés — por eso es "acíclico", no podés crear ciclos en la historia de Git.

---

## Takeaways

- **Blob** = contenido puro de un archivo, sin nombre ni metadatos
- **Tree** = directorio: contiene nombres + SHAs de blobs y otros trees
- **Commit** = snapshot con metadata + puntero al tree raíz + puntero al padre
- Los objetos son **inmutables**: una vez creados, nunca se modifican
- Mismo contenido = mismo SHA = mismo objeto (sin duplicación)
- Toda la historia de Git es este grafo de objetos enlazados por SHAs
