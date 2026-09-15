# Buscador semántico de ofertas de empleo

Buscar **«resolver problemas de clientes por teléfono y chat»** y que aparezcan
ofertas tituladas **«Customer Support Legend»**, sin compartir una sola palabra.

La búsqueda por palabras clave no encuentra lo que no está escrito literalmente.
La semántica compara **significados**. Este proyecto mide **cuánto** aporta de
verdad esa diferencia, en lugar de darla por supuesta.

## Resultado

Todo sobre 10 145 ofertas y 11 consultas, comparado contra una verdad de
referencia construida de forma reproducible.

| Método | P@10 | MRR | nDCG@10 | R@100 |
|---|---|---|---|---|
| Aleatorio (control) | 0,007 | 0,037 | 0,008 | 0,010 |
| BM25 (palabras clave) | 0,182 | 0,325 | 0,181 | 0,213 |
| RRF (k=60, sin ajustar) | 0,300 | 0,530 | 0,310 | 0,297 |
| Embeddings (troceado) | 0,318 | 0,482 | 0,312 | 0,259 |
| **Híbrido ponderado** | **0,318** | **0,557** | **0,344** | **0,314** |

**Frente a BM25: P@10 ×1,75 y R@100 ×1,47. Frente al azar: P@10 ×45.**

> ⚠️ **Las cifras absolutas están subestimadas a propósito, y no deben sacarse de
> contexto.** Miden «cuántas ofertas del equipo correcto de Booking recupera», no
> «cómo de bueno es el buscador». Ver [Lo que estos números no
> dicen](#lo-que-estos-números-no-dicen).

## Lo que este proyecto no es

No es un cuaderno que calcula embeddings y enseña tres resultados escogidos a
mano. Eso no demuestra nada.

Aquí hay **una verdad de referencia construida de forma reproducible**, **una
línea base honesta** y **un control aleatorio**. Cuando un método no gana, el
README lo dice — y hay dos casos donde eso pasa.

## Los datos

Cuatro fuentes de Kaggle, **una descartada**:

| Fuente | Descargado | Útil | |
|---|---|---|---|
| [LinkedIn data analyst jobs](https://www.kaggle.com/datasets/cedricaubin/linkedin-data-analyst-jobs-listings) | 8 490 | **480** | ❌ descartada |
| [Seek Australia](https://www.kaggle.com/datasets/promptcloud/job-listings-from-seek-australia) | 5 847 | 5 602 | Distractores realistas |
| [Booking.com](https://www.kaggle.com/datasets/niekvanderzwaag/bookingcom-job-listings) | 1 985 | **819** | **La verdad de referencia** |
| [Business Analyst jobs](https://www.kaggle.com/datasets/andrewmvd/business-analyst-jobs) | 4 092 | 3 724 | Masa adicional |

**Corpus final: 10 145 ofertas únicas.** Se dejan en `datos/`, que está en
`.gitignore`: el dataset no se sube, solo el código.

**Licencia:** comprobar la de cada fuente antes de publicar nada derivado.

## Puesta en marcha

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt

jupyter lab notebooks/
```

Los cuadernos se ejecutan en orden. El 04 tarda unos 20 minutos la primera vez
(codifica 32 629 trozos); después reutiliza los vectores guardados.

| Cuaderno | Qué hace |
|---|---|
| `01-exploracion-del-corpus` | Qué hay dentro, qué se descarta y por qué |
| `02-verdad-de-referencia` | Consultas, métricas y control aleatorio |
| `03-linea-base-bm25` | La búsqueda clásica, bien configurada |
| `04-busqueda-con-embeddings` | Búsqueda semántica y sus variantes |
| `05-busqueda-hibrida` | Combinar los dos, y medirlo sin hacerse trampa |
| `06-el-buscador-en-uso` | La función de búsqueda y el experimento en español |

```python
buscar("protecting customer data and making sure we follow privacy law", k=6)
#  1. Sr Legal Counsel - Privacy          Booking.com
#  2. Junior Privacy Analyst              Booking.com
#  3. Privacy Compliance Officer          Commonwealth Bank of Australia
```

42 ms por consulta sobre 10 145 ofertas.

## Cómo se mide

**El problema de fondo:** para decir que un buscador es bueno hace falta saber
qué respuestas son correctas. Construir eso sin puntuar a ojo es la parte difícil.

**La solución:** Booking.com etiqueta cada oferta con el **equipo** al que
pertenece —Engineering, Design & UX, Finance…— y esa etiqueta **la asignó una
persona, no se deduce del título**. Once consultas describen lo que hace cada
equipo **sin nombrarlo**, y una oferta es relevante si pertenece a ese equipo.

Las otras 9 326 ofertas están en el índice como **distractores**: pueden
recuperarse por error, que es lo que hace realista la evaluación.

**Las métricas:** `P@10` como principal —de los 10 devueltos, cuántos aciertan—
más `MRR`, `nDCG@10` y `R@100`. Se descartó `Recall@10` porque con 216 ofertas
relevantes su techo es 0,046: un buscador perfecto parecería pésimo.

## Lo que ha aparecido por el camino

Las complicaciones reales son el contenido del proyecto, no obstáculos que
esconder.

### 1. Una fuente de cuatro era un 94 % repetición

8 490 filas de LinkedIn que contenían **480 descripciones distintas**. África un
4,1 % de contenido único, Canadá un 6,0 %, EE. UU. un 7,0 %. Y encima todo el
mismo puesto: `data analyst`.

**Se descartó.** El número de filas no dice nada hasta que se cuentan las
distintas.

### 2. Normalizar antes de deduplicar cambia el resultado un 40 %

En Booking, comparando descripciones **tal cual** quedan 1 352 únicas.
Normalizando espacios y mayúsculas quedan **819**. Son 533 ofertas que solo se
diferenciaban en el formato y habrían entrado tres veces al índice.

### 3. Una quinta parte del texto era plantilla corporativa

Más de la mitad de las ofertas de Booking empieza igual que otra. Los párrafos
más repetidos resultaron ser `b.skilled` (en 388 de 819), `b.responsible` (374) y
`b.offered` (319): **los títulos de sección de la plantilla interna de Booking**.

Detectarlos por frecuencia de documento y quitarlos elimina el **20,7 %** del
texto — y mejora los resultados de **los dos** métodos: **+18 % en BM25** y
**+17 % en embeddings**. Dos técnicas que no se parecen en nada coinciden.

### 4. El 83 % de las ofertas no cabe en el modelo

`all-MiniLM-L6-v2` corta a **256 tokens ≈ 192 palabras**. La mediana de las
ofertas de Booking es de 920.

Truncar se quedaría con la presentación genérica y tiraría los requisitos
concretos, que están al final. **Trocear en ventanas de 180 palabras y puntuar
cada oferta por su mejor trozo da un 25 % más** (0,2545 → 0,3182).

### 5. En Windows, importar pandas antes que torch rompe torch

```
OSError: [WinError 1114] Error en una rutina de inicialización de
biblioteca de vínculos dinámicos (DLL). Error loading ...\torch\lib\c10.dll
```

Las dos librerías traen su copia de OpenMP y la primera en cargar deja a la otra
sin inicializar. **`sentence-transformers` tiene que importarse primero**, antes
que pandas. Va contra el orden alfabético habitual: reordenar esas líneas rompe
los cuadernos.

### 6. RRF, el método más recomendado, empeora el resultado

Reciprocal Rank Fusion trata a los dos buscadores como iguales. Aquí uno casi
duplica al otro (0,318 contra 0,182), así que promediar posiciones a partes
iguales arrastra al bueno: **0,300, por debajo de los embeddings solos**.

RRF sirve cuando los ingredientes tienen calidad parecida. No era el caso.

### 7. La mejora del híbrido en P@10 era exactamente cero

El error más instructivo del proyecto, y casi lo publico.

Barriendo 21 valores de α para `(1−α)·BM25 + α·embeddings`, el mejor daba
**P@10 = 0,3545** — un 11 % sobre los embeddings solos. Pero elegí α mirando **las
mismas once consultas** con las que después lo evaluaba.

Medido dejando una consulta fuera —α elegido con las otras diez— el resultado es
**0,3182: exactamente lo mismo que los embeddings solos.**

| Métrica | Embeddings | Híbrido aparente | **Híbrido honesto** |
|---|---|---|---|
| **P@10** | 0,318 | 0,355 | **0,318** |
| MRR | 0,482 | 0,648 | **0,557** |
| nDCG@10 | 0,312 | 0,371 | **0,344** |
| R@100 | 0,259 | 0,325 | **0,314** |

**Toda la ganancia aparente en P@10 era sobreajuste.** Pero las otras tres
métricas sí mejoran de verdad: el híbrido **no mete más aciertos en el top 10, los
coloca más arriba y encuentra más en el top 100**.

Con una sola métrica la conclusión habría sido falsa en las dos direcciones: con
P@10, «no sirve»; con MRR, «mejora un 15 %».

### 8. Buscar en español funciona, y cuesta la mitad de la calidad

La promesa vistosa del proyecto era escribir en español y recuperar ofertas en
inglés. Se dejó **fuera de la evaluación a propósito**, y el cuaderno 06 explica
por qué con números:

| Escenario | P@10 | MRR | R@100 |
|---|---|---|---|
| BM25 · consulta en inglés | 0,182 | 0,325 | 0,213 |
| **BM25 · consulta en español** | **0,000** | 0,002 | **0,000** |
| Multilingüe · consulta en inglés | 0,109 | 0,162 | 0,115 |
| **Multilingüe · consulta en español** | **0,100** | 0,238 | 0,112 |

**BM25 con consultas en español saca cero absoluto.** Ni un acierto en once
consultas sobre todo el índice. Si la evaluación se hubiera hecho así, la
conclusión habría sido «los embeddings son infinitamente mejores», que es cierto
y no significa nada — y habría tapado que BM25 gana en tres de las once consultas.

**La búsqueda entre idiomas funciona:** con el mismo modelo, el español pierde
solo un 8 % frente al inglés, y el MRR incluso mejora.

**Pero el modelo multilingüe saca 0,109 donde el modelo en inglés sacaba 0,255.**
Más de la mitad de la calidad. Su ventana es de 128 tokens, la mitad. Por eso el
sistema principal se queda en inglés y esto se reporta como **capacidad aparte**,
con su compromiso al lado.

### 9. Un fallo que ninguna métrica capta

Probando el buscador con consultas libres, «enseñar y guiar a compañeros junior»
devuelve `Senior Agile Coach`, `Solutions Architect`, `Senior Software
Developer`…

La palabra `coaching` enganchó con `Agile Coach` —un rol de metodología, no de
mentoría— y el resultado derivó hacia puestos senior de tecnología en general. **El
buscador entendió «alguien con experiencia» en vez de «alguien que enseña».**

Ninguna de las métricas lo detecta, porque ninguna de las once consultas de
evaluación provoca esa confusión. Es el recordatorio de que **once consultas
miden once cosas**.

## Lo que estos números no dicen

Mirando **qué** devuelve el buscador en `Customer Service`, una consulta con
P@10 = 0,0:

| | Título | Fuente |
|---|---|---|
| · | Customer Support Legend | seek_au |
| · | Customer/Tech Support Agent — SaaS | seek_au |
| · | Help Desk Support Agent | seek_au |
| · | Online Customer Support | seek_au |

**Cinco de los seis primeros son puestos de atención al cliente.** Cualquiera
diría que la búsqueda funcionó. La métrica marca cero porque solo cuenta ofertas
del equipo correspondiente **de Booking**.

Y en `Data Science & Analytics`, el segundo resultado es **una oferta de Booking
titulada `Data Analyst`** que no está etiquetada en ese equipo: ni dentro de
Booking la etiqueta es perfecta.

> **Las cifras absolutas están subestimadas.** Miden «cuántas ofertas del equipo
> correcto de Booking recupera», no «cómo de bueno es el buscador».
>
> **La comparación entre métodos sí es válida**, porque todos se juzgan con la
> misma regla injusta.

## Limitaciones declaradas

- **Todas las ofertas relevantes son de una misma empresa.** Comparten tono y
  vocabulario. Se controla reportando también el resultado con el índice
  reducido a Booking, pero no se elimina.
- **Las consultas las escribí yo.** Seguí reglas —no usar el nombre del equipo,
  no leer las ofertas antes de redactar— y audité la fuga léxica, pero no puedo
  demostrar que no elegí palabras *porque* sé que aparecen. La única defensa real
  sería que otra persona las redactara sin ver los datos.
- **Un equipo no es un puesto.** Una oferta de `Product` puede ser legítimamente
  relevante para la consulta de `Engineering` y se cuenta como fallo.
- **El detector de plantillas no funciona en Seek**, cuyas descripciones vienen
  sin saltos de línea. Se acepta porque Seek solo aporta distractores.
- **11 consultas son pocas.** Es la razón por la que el sobreajuste del apartado 7
  fue tan grande, y por la que la validación se hizo dejando una fuera en lugar de
  con un conjunto reservado.

## Decisiones de diseño

**pgvector se dejó fuera, con el dato delante.** Buscar tarda **28 ms** por
consulta comparando 32 629 vectores, y el índice ocupa 48 MB en memoria. Una base
de datos vectorial no resolvería ningún problema que exista hoy. Tendrá sentido al
desplegar esto, cuando haya que persistir el índice y compartirlo entre procesos.

**Sin LLM.** Esto es solo recuperación. La generación de respuestas con citas es
el siguiente proyecto, y mezclarlas haría imposible saber cuál de las dos partes
falla.

## Stack

Python · pandas · **sentence-transformers** · **rank-bm25** · scikit-learn ·
matplotlib. Sin nube y sin GPU: todo corre en CPU.
