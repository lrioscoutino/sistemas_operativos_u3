# Unidad 3 — Administración de memoria

**Materia:** Sistemas Operativos (AEC-1061)
**Grupo:** S5C — Aula E10
**Horario:** Martes 18:00–20:00 · Jueves 17:00–19:00
**Enfoque:** teórico-conceptual, con simulaciones ejecutables en Python

Continuación directa de la [Unidad 2](../sistemas_operativos_u2/README.md): ahí viste cómo el núcleo administra procesos y el CPU. Esta unidad entra a otro recurso que el núcleo administra: la memoria — cómo acomoda procesos en la RAM física, cómo la paginación resuelve la fragmentación, y cómo la memoria virtual permite ejecutar procesos más grandes que la RAM disponible, decidiendo con algoritmos de reemplazo qué sacrificar cuando no alcanza el espacio.

## Contenido

- [02_practica_administracion_memoria.md](02_practica_administracion_memoria.md) — Práctica guiada paso a paso: 3.1-3.2 asignación por particiones (first-fit/best-fit, fragmentación interna), 3.3 paginación (traducción de direcciones, fallo de página), 3.4 memoria virtual (algoritmos de reemplazo FIFO/LRU/Óptimo, thrashing)

## Nota sobre el código

Las tres simulaciones de la práctica son Python **ejecutado y verificado** antes de publicarse — las salidas que aparecen en el documento son exactamente las que produce el código al correrlo, no transcripciones de un libro.

## Fuentes

- Programa sinóptico oficial AEC-1061 — TecNM
- Silberschatz, A. — *Fundamentos de Sistemas Operativos* (9na ed.) — ejemplo de first-fit/best-fit y la secuencia de referencias de reemplazo de páginas
- Tanenbaum, A. — *Sistemas Operativos Modernos* (3ra ed.)
- Stallings, W. — *Sistemas Operativos* (5ta ed.)
