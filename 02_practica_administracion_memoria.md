# Práctica guiada: Administración de memoria, paso a paso

Tutorial práctico para la Unidad 3 (3.1–3.4 del temario AEC-1061). La idea es que **tú escribas y corras cada simulación**, veas el resultado, y entiendas por qué el sistema operativo toma esas decisiones — no es solo leer teoría, es ver con tus propios ojos un fallo de página ocurrir y un algoritmo de reemplazo decidir a quién saca.

Requisito: Python 3 (no se necesita ninguna librería externa).

---

## Paso 1 — Memoria real: asignación por particiones (3.1–3.2)

### El problema

El sistema operativo recibe procesos de distintos tamaños y tiene que acomodarlos en "huecos" de memoria física disponibles. La estrategia que elige para decidir **en qué hueco** mete cada proceso afecta directamente cuánta memoria se desperdicia (fragmentación).

### Lo que vas a construir

Dos estrategias clásicas de asignación:
- **First-fit:** usa el primer hueco que alcance.
- **Best-fit:** usa el hueco más pequeño que alcance (para desperdiciar lo menos posible ese hueco).

```python
def first_fit(particiones, procesos):
    libres = particiones.copy()
    asignacion = {}
    for p, tam in procesos:
        for i, espacio in enumerate(libres):
            if espacio >= tam:
                asignacion[p] = (i, tam, espacio - tam)  # (particion, usado, frag interna)
                libres[i] -= tam
                break
        else:
            asignacion[p] = None
    return asignacion

def best_fit(particiones, procesos):
    libres = particiones.copy()
    asignacion = {}
    for p, tam in procesos:
        candidatos = [(i, espacio) for i, espacio in enumerate(libres) if espacio >= tam]
        if not candidatos:
            asignacion[p] = None
            continue
        i, espacio = min(candidatos, key=lambda x: x[1])
        asignacion[p] = (i, tam, espacio - tam)
        libres[i] -= tam
    return asignacion

particiones = [100, 500, 200, 300, 600]
procesos = [("P1", 212), ("P2", 417), ("P3", 112), ("P4", 426)]

print("Particiones iniciales:", particiones)
print("\n=== First-fit ===")
for p, res in first_fit(particiones, procesos).items():
    print(p, "->", res)

print("\n=== Best-fit ===")
for p, res in best_fit(particiones, procesos).items():
    print(p, "->", res)
```

### Corre el código y compara con esta salida real (verificada)

```
Particiones iniciales: [100, 500, 200, 300, 600]

=== First-fit ===
P1 -> (1, 212, 288)
P2 -> (4, 417, 183)
P3 -> (1, 112, 176)
P4 -> None

=== Best-fit ===
P1 -> (3, 212, 88)
P2 -> (1, 417, 83)
P3 -> (2, 112, 88)
P4 -> (4, 426, 174)
```

### Qué acaba de pasar — léelo con cuidado

Con **first-fit**, `P4` (426 bytes) **no entra en ningún lado** — el log dice `None`. ¿Por qué? Porque first-fit ya gastó la partición de 600 en `P2` (417) sin pensar si había una opción mejor, y la de 500 ya la ocupó `P1`+`P3` juntos. Al llegar `P4`, ya no queda ningún hueco libre de 426 o más.

Con **best-fit**, los cuatro procesos caben. Best-fit fue más cuidadoso: metió a cada proceso en el hueco más ajustado posible, dejando la partición grande (600) disponible para cuando llegara el proceso grande (`P4`).

**Esto es exactamente el caso de libro de texto** (Silberschatz) usado para enseñar que "mejor ajuste" no es automáticamente peor que "primer ajuste" — aquí, en este caso concreto, es estrictamente mejor. (En otros casos puede ser al revés: best-fit deja huecos pequeños inútiles que peor-ajuste no generaría. Es parte del ejercicio: prueba con otros tamaños de partición y procesos y observa cuándo cambia el ganador.)

El tercer número de cada tupla (ej. `288` en `P1 -> (1, 212, 288)`) es la **fragmentación interna**: memoria que quedó asignada a ese proceso pero que no usa. Súmala para los cuatro procesos en cada estrategia — ese total es el costo real de la decisión del sistema operativo.

### Ejercicio para ti

Cambia `particiones` y `procesos` por otros valores, ejecuta de nuevo, y responde: ¿en qué caso first-fit gana a best-fit en fragmentación total? (Pista: prueba con muchos procesos pequeños y pocas particiones grandes.)

---

## Paso 2 — Paginación: traducir una dirección lógica a una dirección física (3.3)

### El problema

La paginación resuelve la fragmentación externa del Paso 1 dividiendo la memoria en bloques de tamaño fijo (páginas/marcos). Pero introduce un paso nuevo: cada vez que un programa pide leer o escribir una dirección, el sistema operativo tiene que **traducirla** usando la tabla de páginas del proceso.

### Lo que vas a construir

```python
TAM_PAGINA = 1024  # bytes (1 KB por página, valor didáctico)

tabla_paginas = {0: 5, 1: 2, 2: 7, 3: 1}  # pagina logica -> marco fisico

def traducir(direccion_logica, tabla, tam_pagina):
    pagina = direccion_logica // tam_pagina
    desplazamiento = direccion_logica % tam_pagina
    if pagina not in tabla:
        return None  # fallo de pagina
    marco = tabla[pagina]
    direccion_fisica = marco * tam_pagina + desplazamiento
    return pagina, desplazamiento, marco, direccion_fisica

direcciones = [0, 1023, 1024, 2500, 4096, 5000]
for d in direcciones:
    r = traducir(d, tabla_paginas, TAM_PAGINA)
    if r is None:
        print(f"Dirección lógica {d}: FALLO DE PÁGINA (página {d // TAM_PAGINA} no está en la tabla)")
    else:
        pagina, desp, marco, fisica = r
        print(f"Dirección lógica {d}: página={pagina} desplazamiento={desp} -> marco={marco} -> dirección física={fisica}")
```

### Salida real (verificada)

```
Dirección lógica 0: página=0 desplazamiento=0 -> marco=5 -> dirección física=5120
Dirección lógica 1023: página=0 desplazamiento=1023 -> marco=5 -> dirección física=6143
Dirección lógica 1024: página=1 desplazamiento=0 -> marco=2 -> dirección física=2048
Dirección lógica 2500: página=2 desplazamiento=452 -> marco=7 -> dirección física=7620
Dirección lógica 4096: FALLO DE PÁGINA (página 4 no está en la tabla)
Dirección lógica 5000: FALLO DE PÁGINA (página 4 no está en la tabla)
```

### Qué acaba de pasar

Fíjate en las primeras dos direcciones (`0` y `1023`): **ambas caen en la misma página** (página 0, porque `1023 // 1024 == 0`), pero producen direcciones físicas distintas porque el desplazamiento dentro de la página es diferente. Esto es la idea central de paginación: la página completa se mueve como bloque, pero dentro de ella cada byte mantiene su posición relativa.

Las direcciones `4096` y `5000` caen en la página 4 (`4096 // 1024 == 4`), que **no existe en la tabla de páginas** de este proceso — eso es un **fallo de página** real: el proceso intentó acceder a memoria que el sistema operativo no le tiene mapeada en este momento. En un SO real, aquí se dispara una interrupción, el SO decide si es un acceso inválido (el proceso se cierra) o si la página sí le pertenece pero está en disco (entonces el SO la trae de swap — eso lo ves en el Paso 3).

### Ejercicio para ti

Agrega la página `4: 3` a `tabla_paginas` y vuelve a correr. ¿Qué cambia en la salida para `4096` y `5000`? Calcula a mano la dirección física esperada antes de correrlo, para comprobar que entendiste la fórmula.

---

## Paso 3 — Memoria virtual: algoritmos de reemplazo de páginas (3.4)

### El problema

En memoria virtual no todas las páginas de un proceso caben en RAM al mismo tiempo — hay más páginas que marcos físicos. Cuando el proceso pide una página que no está en RAM (fallo de página) y **ya no hay marcos libres**, el sistema operativo tiene que **decidir a cuál página sacar** para hacerle espacio a la nueva. Esa decisión es el algoritmo de reemplazo, y de qué tan buena sea depende directamente el rendimiento del sistema.

### Lo que vas a construir

Tres algoritmos clásicos, corriendo sobre la **misma secuencia de referencias** y el **mismo número de marcos**, para poder comparar sus fallos cara a cara:

- **FIFO:** saca la página que lleva más tiempo en memoria, sin importar si se usó hace un segundo.
- **LRU** (*Least Recently Used*): saca la página que no se ha usado en más tiempo.
- **Óptimo:** saca la página que **no se va a volver a usar en más tiempo** (mirando el futuro — en la práctica es imposible de implementar en un SO real porque no se puede predecir el futuro, pero sirve como "mejor caso posible" para comparar contra los otros dos).

```python
def fifo(referencias, n_marcos):
    marcos = []
    cola = []
    fallos = 0
    historial = []
    for ref in referencias:
        if ref in marcos:
            historial.append((ref, marcos.copy(), False))
            continue
        fallos += 1
        if len(marcos) < n_marcos:
            marcos.append(ref)
            cola.append(ref)
        else:
            viejo = cola.pop(0)
            marcos[marcos.index(viejo)] = ref
            cola.append(ref)
        historial.append((ref, marcos.copy(), True))
    return fallos, historial

def lru(referencias, n_marcos):
    marcos = []
    uso = {}
    fallos = 0
    historial = []
    for t, ref in enumerate(referencias):
        if ref in marcos:
            uso[ref] = t
            historial.append((ref, marcos.copy(), False))
            continue
        fallos += 1
        if len(marcos) < n_marcos:
            marcos.append(ref)
        else:
            lru_pag = min(marcos, key=lambda p: uso[p])
            marcos[marcos.index(lru_pag)] = ref
        uso[ref] = t
        historial.append((ref, marcos.copy(), True))
    return fallos, historial

def optimo(referencias, n_marcos):
    marcos = []
    fallos = 0
    historial = []
    for t, ref in enumerate(referencias):
        if ref in marcos:
            historial.append((ref, marcos.copy(), False))
            continue
        fallos += 1
        if len(marcos) < n_marcos:
            marcos.append(ref)
        else:
            def prox_uso(p):
                futuro = referencias[t+1:]
                return futuro.index(p) if p in futuro else float('inf')
            victima = max(marcos, key=prox_uso)
            marcos[marcos.index(victima)] = ref
        historial.append((ref, marcos.copy(), True))
    return fallos, historial

referencias = [7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2]
n_marcos = 3

for nombre, fn in [("FIFO", fifo), ("LRU", lru), ("Óptimo", optimo)]:
    fallos, hist = fn(referencias, n_marcos)
    print(f"=== {nombre} (marcos={n_marcos}) ===")
    for ref, marcos, fue_fallo in hist:
        marca = "FALLO" if fue_fallo else "hit"
        print(f"  ref={ref:>2}  marcos={marcos}  {marca}")
    print(f"Total de fallos: {fallos} de {len(referencias)} referencias\n")
```

### Salida real (verificada)

```
=== FIFO (marcos=3) ===
  ref= 7  marcos=[7]  FALLO
  ref= 0  marcos=[7, 0]  FALLO
  ref= 1  marcos=[7, 0, 1]  FALLO
  ref= 2  marcos=[2, 0, 1]  FALLO
  ref= 0  marcos=[2, 0, 1]  hit
  ref= 3  marcos=[2, 3, 1]  FALLO
  ref= 0  marcos=[2, 3, 0]  FALLO
  ref= 4  marcos=[4, 3, 0]  FALLO
  ref= 2  marcos=[4, 2, 0]  FALLO
  ref= 3  marcos=[4, 2, 3]  FALLO
  ref= 0  marcos=[0, 2, 3]  FALLO
  ref= 3  marcos=[0, 2, 3]  hit
  ref= 2  marcos=[0, 2, 3]  hit
Total de fallos: 10 de 13 referencias

=== LRU (marcos=3) ===
  ref= 7  marcos=[7]  FALLO
  ref= 0  marcos=[7, 0]  FALLO
  ref= 1  marcos=[7, 0, 1]  FALLO
  ref= 2  marcos=[2, 0, 1]  FALLO
  ref= 0  marcos=[2, 0, 1]  hit
  ref= 3  marcos=[2, 0, 3]  FALLO
  ref= 0  marcos=[2, 0, 3]  hit
  ref= 4  marcos=[4, 0, 3]  FALLO
  ref= 2  marcos=[4, 0, 2]  FALLO
  ref= 3  marcos=[4, 3, 2]  FALLO
  ref= 0  marcos=[0, 3, 2]  FALLO
  ref= 3  marcos=[0, 3, 2]  hit
  ref= 2  marcos=[0, 3, 2]  hit
Total de fallos: 9 de 13 referencias

=== Óptimo (marcos=3) ===
  ref= 7  marcos=[7]  FALLO
  ref= 0  marcos=[7, 0]  FALLO
  ref= 1  marcos=[7, 0, 1]  FALLO
  ref= 2  marcos=[2, 0, 1]  FALLO
  ref= 0  marcos=[2, 0, 1]  hit
  ref= 3  marcos=[2, 0, 3]  FALLO
  ref= 0  marcos=[2, 0, 3]  hit
  ref= 4  marcos=[2, 4, 3]  FALLO
  ref= 2  marcos=[2, 4, 3]  hit
  ref= 3  marcos=[2, 4, 3]  hit
  ref= 0  marcos=[2, 0, 3]  FALLO
  ref= 3  marcos=[2, 0, 3]  hit
  ref= 2  marcos=[2, 0, 3]  hit
Total de fallos: 7 de 13 referencias
```

### Qué acaba de pasar — la parte más importante de toda la práctica

Los tres algoritmos corrieron **exactamente la misma secuencia de 13 referencias**, con **los mismos 3 marcos disponibles**, y dieron resultados distintos:

| Algoritmo | Fallos de página | ¿Por qué |
|---|---|---|
| FIFO | 10 | Saca páginas por antigüedad, sin importar si se acaban de usar. Fíjate en `ref=0` (la 5ª referencia): FIFO la mantiene porque aún no le toca salir por orden de llegada — pero más adelante saca páginas que sí se iban a volver a usar pronto. |
| LRU | 9 | Un poco mejor que FIFO porque sí toma en cuenta el uso reciente — pero sigue siendo una aproximación basada en el pasado. |
| Óptimo | 7 | El mejor posible, porque "hace trampa": sabe el futuro. Ningún sistema operativo real puede implementar esto literal, pero sirve como vara de medir qué tan cerca (o lejos) está LRU del ideal. |

La relación `Óptimo (7) ≤ LRU (9) ≤ FIFO (10)` **no es casualidad de este ejemplo** — es la propiedad que se enseña en teoría: el óptimo siempre es una cota inferior, y LRU generalmente se acerca más a él que FIFO porque usa información más reciente para decidir.

Esto también es la explicación de por qué los sistemas operativos reales (Linux, Windows) usan variantes de LRU (como el algoritmo del reloj / *clock*, una aproximación barata de LRU) y casi nunca FIFO puro — la diferencia de fallos se traduce directamente en más accesos a disco, que son miles de veces más lentos que la RAM. Cuando esto se descontrola (demasiados fallos, el CPU pasa más tiempo intercambiando páginas que ejecutando procesos), se le llama **thrashing**.

### Ejercicio para ti

Cambia `n_marcos` a `4` y vuelve a correr los tres. ¿Bajan los fallos en los tres algoritmos? ¿Se mantiene el orden `Óptimo ≤ LRU ≤ FIFO`? Luego prueba con `n_marcos = 1` — ¿qué le pasa a los tres cuando casi no hay memoria?

---

## Resumen de lo que cubriste

| Tema del programa oficial | Dónde lo viste en esta práctica |
|---|---|
| 3.1 Política y filosofía | La discusión de por qué first-fit vs best-fit no tiene una respuesta única — es una política, con tradeoffs |
| 3.2 Memoria real | Paso 1 — particiones, fragmentación interna |
| 3.3 Organización de memoria virtual | Paso 2 — paginación, traducción de direcciones, fallo de página |
| 3.4 Administración de memoria virtual | Paso 3 — algoritmos de reemplazo (FIFO, LRU, Óptimo), thrashing |
