# Buscador semántico de ofertas de empleo

Buscar **«trabajo con modelos en producción»** y que aparezcan ofertas que dicen
**«MLOps Engineer»**, sin compartir una sola palabra.

La búsqueda por palabras clave no encuentra lo que no está escrito literalmente.
La búsqueda semántica compara **significados**. Este proyecto mide **cuánto** de
verdad aporta esa diferencia, en lugar de darla por supuesta.

## Lo que este proyecto no es

No es un cuaderno que calcula embeddings y enseña tres resultados escogidos a
mano. Eso no demuestra nada.

Aquí hay **una verdad de referencia construida de forma reproducible** y **una
línea base honesta (BM25)**, que es difícil de batir. Si la búsqueda semántica no
le gana, el README lo dirá.

## Estado

- [ ] Paso 0 — descargar el dataset
- [ ] Paso 1 — exploración
- [ ] Paso 2 — verdad de referencia
- [ ] Paso 3 — línea base BM25 y control aleatorio
- [ ] Paso 4 — embeddings
- [ ] Paso 5 — comparación honesta
- [ ] Paso 6 — búsqueda híbrida
- [ ] Paso 7 — demo y cierre

## Los datos

**Pendiente de descargar.** Hace falta un dataset de ofertas de empleo tech con,
como mínimo, **título del puesto** y **descripción en texto largo**.

Candidatos en Kaggle — buscar por `linkedin job postings`,
`data science job postings skills` o `job description dataset`.

Criterios: descripciones largas (no solo el título), idioma consistente, y
**entre 10 000 y 100 000 ofertas**. Más no aporta nada y ralentiza el cálculo de
embeddings.

Los archivos se dejan **tal cual, sin renombrar**, en `datos/`. Esa carpeta está
en `.gitignore`: **el dataset no se sube al repositorio**, solo el código.

## Puesta en marcha

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt

jupyter lab notebooks/        # o abrir el cuaderno desde VS Code
```

> **Aviso de tamaño.** `sentence-transformers` arrastra PyTorch: en Windows son
> del orden de 2 GB. Si es un problema, `fastembed` hace lo mismo con ONNX y
> ocupa una fracción.

## Cómo se mide

Tres métricas estándar de recuperación, todas a k=10:

| Métrica | Qué mide |
|---|---|
| **Recall@10** | De las ofertas relevantes, cuántas salen en el top 10 |
| **MRR** | Cómo de arriba aparece el primer acierto |
| **nDCG@10** | Calidad del orden completo |

Y cuatro métodos, incluyendo **un control aleatorio** — el equivalente al
`DummyClassifier` del proyecto anterior. Sin él no se sabe si un 0,30 es bueno.

| Método | Recall@10 | MRR | nDCG@10 |
|---|---|---|---|
| Aleatorio (control) | | | |
| BM25 (palabras clave) | | | |
| Embeddings | | | |
| Híbrido | | | |

## Estructura

```
datos/
  procesado/    conjuntos ya preparados que generan los cuadernos
notebooks/
resultados/     gráficos que generan los cuadernos
```

Las rutas se calculan a partir de dónde se ejecute el cuaderno: **no hay ninguna
ruta absoluta en el código**, así que el proyecto funciona clonado en cualquier
carpeta y en cualquier sistema operativo.

## Lo que ha aparecido por el camino

Se irá rellenando. Las complicaciones reales son el contenido del proyecto, no
obstáculos que esconder.
