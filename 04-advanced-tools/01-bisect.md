# 01 · Git Bisect

## El problema

Algo se rompió en algún punto de los últimos 500 commits. Reproducir el bug toma varios minutos. ¿Cómo encontrás el commit culpable sin revisar uno por uno?

---

## Estrategia 1: buscar en los logs

Antes de recurrir a bisect, hay dos formas de buscar en el historial que pueden ahorrarte tiempo si conocés alguna pista.

### Buscar por palabra clave en mensajes de commit

```bash
git log --grep "foo"              # commits cuyo mensaje contiene "foo"
git log -p --grep "foo"           # idem + muestra el diff de cada uno
```

### Buscar por cambios en un archivo específico

```bash
git log -p -- src/index.js        # historial con diffs de ese archivo
```

### Buscar por texto dentro del código modificado

```bash
git log -S "nombre_funcion"       # commits que agregaron o eliminaron ese texto
git log -S "nombre_funcion" -p    # idem + muestra el diff
```

### Cuándo usar cada uno

| Método | Cuándo conviene |
|--------|-----------------|
| `--grep` | Sabés palabras clave en los mensajes de commit |
| `-- archivo` | Sabés en qué archivo está el bug |
| `-S "texto"` | Sabés el nombre de la función o variable afectada |

> 💡 Estos métodos dependen de buenos mensajes de commit y de saber dónde buscar. Cuando no tenés esas pistas, usá bisect.

---

## Estrategia 2: git bisect (búsqueda binaria)

`git bisect` implementa **búsqueda binaria** sobre la historia de commits. En lugar de revisar commit por commit, divide el espacio a la mitad en cada paso.

### El principio

```
a -------- desconocido -------- b
(funciona)                    (roto)
```

Git selecciona el commit del medio `c` y te pide que lo testees:

- Si `c` **funciona** → el bug está entre `c` y `b`
- Si `c` **falla** → el bug está entre `a` y `c`

Repetís esto hasta aislar el commit culpable.

### Complejidad: O(log n)

```
  1 commit  → 1 paso
  8 commits → 3 pasos
 16 commits → 4 pasos
 64 commits → 6 pasos
500 commits → 9 pasos
```

---

## Bisect manual: paso a paso

```bash
# 1. Iniciar la sesión
git bisect start

# 2. Marcar el commit actual como malo
git bisect bad

# 3. Marcar el último commit conocido como bueno
git bisect good b56ed57

# 4. Git checkoutea el commit del medio → correr el test
npm run test

# 5. Según el resultado, informar a Git
git bisect good    # si el test pasó
git bisect bad     # si el test falló

# Repetir paso 4-5 hasta que Git diga:
# "3798398f... is the first bad commit"

# 6. Terminar y volver a HEAD
git bisect reset
```

> 💡 Git te va mostrando cuántos pasos faltan: `Bisecting: 5 revisions left to test after this (roughly 3 steps)`.

---

## Bisect automatizado: git bisect run

Si tenés un comando que devuelve exit code 0 cuando todo está bien y distinto de 0 cuando falla, podés automatizar todo el proceso:

```bash
git bisect start
git bisect bad
git bisect good b56ed57

# Git corre el comando automáticamente en cada paso
git bisect run npm test
git bisect run ./node_modules/.bin/vitest --run
git bisect run ./scripts/check.sh
```

Git va testeando solo y al final te da el commit culpable sin intervención manual.

> 💡 `git bisect run` es especialmente poderoso cuando cada test tarda varios minutos — Git hace todo el trabajo mientras vos tomás un café.

---

## Otros usos de bisect

Bisect no es solo para bugs — sirve para encontrar **cualquier cambio** en la historia:

- ¿Cuándo empezó a pasar un test que antes fallaba? (`good` y `bad` son relativos)
- ¿En qué commit se introdujo una regresión de performance?
- ¿Cuándo cambió el comportamiento de una función?

---

## Resumen de comandos

| Comando | Para qué |
|---------|----------|
| `git bisect start` | Iniciar una sesión de bisect |
| `git bisect bad` | Marcar el commit actual (o uno específico) como malo |
| `git bisect good <sha>` | Marcar un commit como bueno |
| `git bisect run <cmd>` | Automatizar la búsqueda con un comando |
| `git bisect reset` | Terminar la sesión y volver a HEAD |
| `git log --grep "term"` | Buscar commits por palabra en el mensaje |
| `git log -p -- archivo` | Ver historial con diffs de un archivo |
| `git log -S "texto"` | Buscar commits que cambiaron ese texto en el código |
