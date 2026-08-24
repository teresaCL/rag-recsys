# Content-Based RAG Recommender

## Descripción

Este proyecto implementa una arquitectura RAG (Retrieval-Augmented Generation) completa para un sistema de recomendación basado en contenido (SRC). En lugar de depender únicamente de un modelo de lenguaje o de un ranking puramente vectorial, el sistema recupera primero los ítems más relevantes de un catálogo mediante búsqueda semántica y usa esos resultados como contexto para que un LLM genere recomendaciones finales, con justificaciones basadas en los metadatos reales de cada ítem.

Este enfoque reduce las alucinaciones al fundamentar la generación en datos reales del catálogo, permite trabajar con colecciones grandes sin reentrenar modelos, y produce recomendaciones explicables.

## Dataset

Se utiliza el [Anime Dataset 2023](https://www.kaggle.com/datasets/dbdmobile/myanimelist-dataset) de Kaggle, compuesto por:

- **anime-dataset-2023.csv**: catálogo de ~25.000 animes con sinopsis, géneros, estudios, formato, etc.
- **users-score-2023.csv**: más de 24 millones de valoraciones explícitas (escala 1-10) de 270.033 usuarios.

## Arquitectura del sistema

1. **Procesado de ítems**: limpieza de valores faltantes, transformación de variables (episodios y año de emisión a categorías textuales) y construcción de un documento unificado por ítem.
2. **Embeddings e indexación**: generación de embeddings con `all-MiniLM-L6-v2` (sentence-transformers) e indexación con FAISS (`IndexFlatIP` sobre vectores normalizados L2, equivalente a similitud coseno).
3. **Perfiles de usuario**: construcción del perfil como media ponderada por rating de los embeddings de los ítems mejor valorados por cada usuario, y recuperación de candidatos similares vía FAISS.
4. **Generación aumentada**: uso de la API de Google Gemini (`gemini-2.5-flash-lite`) para re-rankear los candidatos y generar justificaciones en formato JSON.
5. **Filtro de alucinaciones**: verificación de que cada ítem recomendado por el LLM pertenece a la lista de candidatos recuperados.

## Resumen resultados
- Las recomendaciones generadas presentan una calidad general buena: son temáticamente coherentes con los perfiles de usuario y las justificaciones hacen referencia a metadatos concretos de los ítems.

- La tasa de alucinaciones fue baja, del 4.8%, filtrada automáticamente antes de mostrar resultados.

- El LLM modificó el orden propuesto por FAISS en el 80% de los ítems evaluados, con un desplazamiento medio de 2.20 posiciones, aportando reordenamiento semántico real más allá de la similitud vectorial.

- La principal debilidad observada es la de diversidad en las recomendaciones: el sistema tiende a recomendar contenido de las mismas franquicias, lo que limita la serendipia del sistema, aunque esta es una característica típica de los enfoques basados en contenido.

Para más información sobre el trabajo realizado, consultar `Memoria_RAG.pdf`.

## Configuración

Crea un archivo `.env` en la raíz del proyecto con tu API key de Google Gemini:
```
GEMINI_API_KEY=[tu-api-key-de-aqui]
```