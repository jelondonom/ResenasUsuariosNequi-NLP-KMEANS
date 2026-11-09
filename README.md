# Lo que 34.328 usuarios le dicen a Nequi: análisis de sentimiento y temas en reseñas de Google Play

Análisis de texto de las reseñas públicas de la app **Nequi Colombia** (noviembre de 2025 a noviembre de 2026) para identificar qué daña la experiencia de los usuarios, cómo cambia en el tiempo y qué acciones debería priorizar la compañía.

Proyecto final del curso de Analítica No Supervisada — Negocios Internacionales, Universidad EAFIT.

---

## Resultados clave

| Indicador | Valor |
|---|---|
| Reseñas analizadas | **34.328** (12 meses, ~94 por día) |
| Sentimiento del texto (validado) | **57,1 %** positivo · 14,3 % neutral · **28,6 %** negativo |
| Tema más negativo | **Acceso a la cuenta**: 90,1 % negativas, y en aumento (3,3 % → 9,8 % de las reseñas entre ene y nov de 2026) |
| Pico de crisis | **Diciembre de 2025**: las quejas por caídas fueron el 27,6 % de las reseñas del mes |
| Respuesta de Nequi en la tienda | **~95 %** de reseñas respondidas hasta julio de 2026 → **~2 %** desde septiembre de 2026 |
| Fortaleza principal | **Facilidad de uso**: 78,8 % de menciones positivas y principal motivo de recomendación |

**Seis prioridades para Nequi:** (1) actualizaciones que no rompan el acceso ni las notificaciones, (2) plan de capacidad para quincenas y diciembre, (3) soporte que resuelva, con foco en fraude, (4) registro sin fricción, (5) proteger la facilidad de uso y (6) monitorear el texto de las reseñas, no solo las estrellas.

---

## Pregunta de negocio

> ¿Qué problemas dañan la experiencia de los usuarios de Nequi, cómo cambian en el tiempo y qué acciones debe priorizar la compañía para proteger su reputación en su nueva etapa independiente?

Nequi alcanzó 28 millones de usuarios en 2026 y se separó de Bancolombia para operar como compañía de financiamiento independiente. Su reputación ahora depende de su propia experiencia de usuario.

---

## Dataset original

No existe una base pública de reseñas de Nequi, así que se construyó con *web scraping* (notebook `01`).

| Atributo | Detalle |
|---|---|
| Fuente | Reseñas públicas de *Nequi Colombia* (`com.nequi.MobileApp`) en Google Play |
| Herramienta | `google-play-scraper` 1.2.7 · tienda de Colombia · idioma español |
| Periodo | 04/11/2025 a 03/11/2026 |
| Tamaño | 34.328 reseñas × 9 columnas · sin IDs duplicados |
| Huella MD5 | `b9705088dc50ef3281929065dd051213` |
| Privacidad | No incluye nombres ni fotos de usuarios |

| Columna | Tipo | Descripción | Nulos |
|---|---|---|---|
| `fuente` | texto | Origen de la reseña (siempre "Google Play") | 0 |
| `id_comentario` | texto | Identificador único de la reseña | 0 |
| `texto` | texto | Reseña tal como la escribió el usuario | 0 |
| `calificacion` | entero (1–5) | Estrellas otorgadas | 0 |
| `me_gusta` | entero | Votos de "útil" de otros usuarios | 0 |
| `fecha` | fecha y hora | Fecha de la **última edición** de la reseña | 0 |
| `version_app` | texto | Versión de la app al reseñar | 3.473 (10,1 %) |
| `respuesta_nequi` | texto | Respuesta oficial de Nequi (vacío = no respondió) | 8.296 (24,2 %) |
| `fecha_respuesta` | fecha y hora | Fecha de la respuesta | 8.296 (24,2 %) |

---

## Metodología

| # | Módulo | Qué se hizo |
|---|---|---|
| 0 | Configuración | Semilla, rutas con `pathlib`, paletas aptas para daltonismo, huella MD5 del archivo original |
| 1 | Caso de negocio y carga | Pregunta de negocio, hipótesis iniciales, diagnóstico de calidad |
| 2 | Limpieza de texto | Bitácora de 10 decisiones; **ninguna reseña eliminada**. Dos versiones del texto: mínima (para modelos) y profunda (para conteos). Normalización Unicode, glosario ortográfico detectado con similitud de palabras, stopwords que conservan negaciones |
| 3 | EDA | Longitud, evolución mensual, palabras y bigramas frecuentes, nubes de palabras, emojis, reseñas extremas |
| 4 | Clustering | Embeddings multilingües (`paraphrase-multilingual-MiniLM-L12-v2`) + K-Means (K = 6), nombres automáticos con TF-IDF |
| 5 | Sentimiento | `pysentimiento` (RoBERTuito, español) + capa de emojis con pesos de −3 a +3 + etiquetas adicionales (pregunta, riesgo reputacional, intención de abandono, recomendación, afinidad de marca) |
| 6 | Validación | Muestra manual ciega y estratificada (210) + muestra de confirmación (50). Exactitud ponderada por estrato: **61,4 % → 77,3 %** tras una regla de desempate, confirmada al **92 %** en datos nuevos |
| 7 | Temas | Diccionario multi-etiqueta de 12 temas, contrastado con los clusters |
| 8 | Cruce sentimiento × tema | Mapa de prioridad, evolución mensual por tema, tasa y tiempo de respuesta de Nequi |
| 9 | Insights | 6 insights: hallazgo → causa → evidencia → implicación |
| 10 | Estrategias y cierre | 6 estrategias con acción, beneficio y KPI; exportación y verificación MD5 |

**Decisión metodológica destacada:** la validación manual mostró que el modelo no reconocía elogios de una palabra ("excelente", "bueno") y los marcaba como neutrales. Sin esa validación, el proyecto habría reportado 38,8 % de reseñas positivas en lugar de 57,1 %.

---

## Estructura del repositorio

```
├── README.md
├── resumen_ejecutivo.md               # una página con indicadores, figuras y recomendaciones
├──presentacion_nequi.pptx        # presentación del caso
├── requirements.txt
├── notebooks/
│   ├── 01_recoleccion_datos.ipynb     # web scraping de Google Play
│   └── 02_analisis_nequi.ipynb        # análisis completo (ejecutado, con salidas)
├── data/
│   ├── raw/                           # Base original: 34.328 reseñas × 9 columnas, tal como se descargaron. Archivo de solo lectura (MD5 `b9705088dc50ef3281929065dd051213`)
│   └── procesados/                     # Base limpia del módulo 2: texto normalizado, emojis extraídos, tokens y flags de calidad. No se eliminó ninguna reseña
├── modelos/ 
│   ├── 04_embeddings.npy                   # Evita recalcular los embeddings (1–2 minutos con GPU)
│   └── 04_kmeans_comentarios.joblib        # Modelo K-Means entrenado (K = 6), junto con el nombre del modelo de embeddings, los nombres de los 6 grupos y el mínimo de palabras (4) 
├── salidas/                           # bitácora, perfiles, validación y resultados en .xlsx
├── reportes/
│   └── figuras/                       # 19 figuras con prefijo de módulo (01_ … 08_)
```

---

## Cómo ejecutar

1. Abrir los notebooks en Google Colab y activar una GPU (*Entorno de ejecución → Cambiar tipo de entorno → T4 GPU*).
2. Ajustar `RUTA_PROYECTO` (celda 0.3) a la carpeta de Drive del proyecto.
3. Ejecutar `01_recoleccion_datos.ipynb` para descargar las reseñas. **Nota:** Google Play muestra siempre las reseñas más recientes, así que una nueva descarga cubrirá otro periodo.
4. Ejecutar `02_analisis_nequi.ipynb` de principio a fin. Los embeddings y el sentimiento se guardan en caché la primera vez (2–5 minutos con GPU).
5. La validación (módulo 6) requiere etiquetar a mano las muestras exportadas a `salidas/`.

Instalación local: `pip install -r requirements.txt` (Python 3.13).

---

## Cómo usar el modelo de clustering

El modelo asigna una reseña nueva (4 palabras o más) a uno de los 6 grupos temáticos:

```python
import joblib
from sentence_transformers import SentenceTransformer

modelo = joblib.load("modelos/04_kmeans_comentarios.joblib")
codificador = SentenceTransformer(modelo["modelo_embeddings"])

resenas = ["no me deja entrar a mi cuenta desde ayer"]
vectores = codificador.encode(resenas, normalize_embeddings=True)
grupos = [modelo["nombres_cluster"][c] for c in modelo["kmeans"].predict(vectores)]
print(grupos)   # ej.: ['deja · cuenta · crear']
```

---

## Limitaciones

- Solo usuarios de Android; las reseñas son autoseleccionadas (escribe más quien está muy satisfecho o muy molesto).
- `fecha` es la última edición de la reseña, no la original; los tiempos de respuesta solo se calculan en reseñas no editadas.
- El modelo de sentimiento no detecta el sarcasmo y es conservador con textos cortos (corregido en parte con la regla de desempate). La clase Neutral sigue siendo poco confiable (11,1 % de precisión).
- La etiqueta manual la hizo un solo revisor; algunas etiquetas contradicen los criterios escritos, por lo que la precisión de Negativo (71,4 %) es probablemente un piso.
- La intención de abandono explícita es poco frecuente (47 reseñas): sus porcentajes son indicativos.
- El diccionario de temas no capta sinónimos no previstos ni el contexto.

---

## Datos y uso responsable

Las reseñas son públicas, pero el texto completo no se publica en este repositorio por respeto a los términos de Google Play y a los usuarios. El notebook `01` permite reconstruir la base. El análisis no identifica a ninguna persona.

## Autores

- Juan Esteban Londoño-Marín — Negocios Internacionales, Universidad EAFIT
- [Nombre del integrante]
