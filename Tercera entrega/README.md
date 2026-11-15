# Tercera entrega: régimen de transferencia y auditoría del modelo

Esta carpeta reúne la Fase 3 del proyecto. Responde a lo que pidió la retroalimentación de la
Fase 2, en ese orden:

| Pedido del profesor | Estado | Dónde está |
|---|---|---|
| 1. Cuerpo congelado frente a ajuste fino completo, sobre la misma partición | ✅ Hecho, con un régimen parcial y una curva de aprendizaje | `comparacion_regimenes_transferencia.ipynb`, `resultados/` |
| 2. Partición temporal (entrenar con lo antiguo, evaluar con lo reciente) | ❌ Pendiente | — |
| 3. Auditoría de explicabilidad: ¿aprendió veracidad o fuente? | 🟡 Avanzada | `auditoria_interpretabilidad_modelo.ipynb` |
| 4. Video de sustentación de duración razonable, en un canal propio | ❌ Pendiente | — |

## Contenido

```
Tercera entrega/
├── README.md                                  este archivo
├── Info                                       resumen corto, igual que en las otras entregas
├── comparacion_regimenes_transferencia.ipynb  cuaderno ejecutado en Colab (GPU T4)
├── auditoria_interpretabilidad_modelo.ipynb   auditoría del modelo de la Fase 2 (no entrena)
├── informe_transferencia_fase3.pdf            informe con los hallazgos de la comparación
├── particion_fase2.csv                        partición train/val/test de la Fase 2, por id
└── resultados/
    ├── regimenes/          comparación de los tres regímenes con el corpus completo
    ├── curva_aprendizaje/  los tres regímenes con 300, 633, 1.200 y 2.358 noticias
    ├── figuras/            gráficos generados por el cuaderno
    └── corridas/           una carpeta por corrida: historial por época, predicciones y métricas
```

### `resultados/regimenes/`

| Archivo | Contenido |
|---|---|
| `tabla_regimenes.csv` | Tabla final: media ± desviación de las tres semillas de evaluación por régimen, y TF-IDF + SVM |
| `todas_las_corridas.csv` | Las 17 corridas (fase A: elección de la tasa; fase B: tres semillas), con validación y prueba |
| `corridas_validacion.csv` | Las mismas corridas, solo con métricas de validación |
| `fase_A_tasas.csv`, `fase_A_tasas_elegidas.json` | Rejilla de tasas de aprendizaje probada por régimen y la tasa elegida en validación |
| `seleccion_regimen.json` | Decisión preregistrada, tomada solo con validación y guardada con fecha antes de mirar la prueba |
| `mcnemar_prueba.csv` | Comparaciones por pares en prueba (McNemar exacto), por semilla |
| `reconciliacion_fase2.csv` | Las tres cifras de la Fase 2 con los mismos hiperparámetros (0,951, 0,964 y 0,969) junto a las de esta fase |
| `resumen_informe.txt` | Resumen en texto que imprime la §22 del cuaderno |

### `resultados/curva_aprendizaje/`

| Archivo | Contenido |
|---|---|
| `resumen_curva.csv` | Media ± desviación por régimen y tamaño, con diagnósticos de colapso |
| `corridas_curva.csv` | Las 39 corridas de la curva (36 + 3 del control con la tasa del artículo) |
| `mcnemar_curva.csv` | Comparaciones por pares en cada tamaño |
| `hechos_curva.txt` | Resumen en texto que imprime la §24 del cuaderno |

Los historiales por época de las corridas de la curva no están aquí: quedaron en la carpeta de
Drive `DL_fase3_regimenes/curva_aprendizaje/runs/`. En `corridas/` sí están los de las 17
corridas de la comparación principal.

## Resultado principal

F1-macro en prueba, media ± desviación de tres semillas (42, 2026 y 7), sobre la misma partición
de la Fase 2:

| Régimen | Parámetros entrenables | 2.358 noticias | 300 noticias |
|---|---:|---:|---:|
| Congelado (solo la cabeza) | 592.130 | 0,937 ± 0,008 | 0,897 ± 0,011 |
| Parcial (2 últimas capas) | 14.767.874 | 0,953 ± 0,005 | 0,920 ± 0,019 |
| Completo | 124.644.866 | 0,967 ± 0,007 | 0,930 ± 0,013 |
| TF-IDF + SVM | — | 0,935 | 0,871 ± 0,006 |

- **El ajuste completo supera al congelado en todos los tamaños**, incluso con 300 noticias, y
  ninguna de las 39 corridas de la curva colapsó. El colapso de la Fase 1 se explica por la receta
  de entrenamiento (tasa baja, lote 16 y una parada temprana que cortó el entrenamiento mientras
  la pérdida seguía en ln 2), no por el tamaño del corpus.
- **La regla preregistrada eligió el parcial** por validación (0,979 frente a 0,971), mientras que
  en prueba gana el completo. Entre ambos la evidencia está dividida; la elección no se cambió
  después de ver la prueba.
- **La auditoría de interpretabilidad** muestra que el modelo marca como falsas el 29 % de las
  noticias reales de otros medios reputables, frente al 0 % de El Espectador, y que ignora las
  fechas: un F1 alto puede estar midiendo la fuente y no la veracidad.

El detalle, las pruebas estadísticas y las limitaciones están en `informe_transferencia_fase3.pdf`.

## Cómo reproducir

1. Abrir `comparacion_regimenes_transferencia.ipynb` en Colab con GPU T4 (botón al inicio del
   cuaderno) y ejecutar todo. Descarga solo el corpus de `Segunda entrega/` y comprueba su MD5.
2. Tarda entre 60 y 90 minutos para la comparación, y unos 30 más para la curva de aprendizaje
   (§24). Cada corrida se guarda al terminar; si Colab se desconecta, al volver a ejecutar se
   retoma donde quedó.
3. Los números pueden diferir en los últimos decimales: el entrenamiento en GPU no es del todo
   determinista. Por eso se reportan tres semillas.

**Advertencia sobre la partición.** La partición de la Fase 2 sale de `StratifiedGroupKFold` y
cambia con la versión de scikit-learn: con la 1.6.1 (la de la Fase 2) se obtiene 2.358 / 452 / 449;
con versiones recientes, otra. El cuaderno no la recalcula: la trae incrustada, la verifica fila
por fila contra los conteos de la Fase 2 y la guarda en `particion_fase2.csv`.

**Auditoría de interpretabilidad.** Carga los pesos de `salida_bayesiana/mejor_modelo/` (la corrida
de la Fase 2 con exactitud 0,951, no la de 0,969). Los pesos no están en el repositorio: el cuaderno los
busca en disco, en el repositorio o pide subirlos en un `.zip`.

## Pendiente

- **Partición temporal.** El código existe como opción (`"estrategia_particion": "temporal"`) en los
  cuadernos de las entregas 1 y 2, pero nunca se ejecutó. Ojo con el diseño: antes de 2018 el
  corpus no tiene ninguna noticia real, así que un corte simple por fecha cambia a la vez la época,
  el balance de clases y la mezcla de fuentes.
- **Entrenar sin un medio y evaluar con él**, para medir cuánto del F1 es reconocimiento de la fuente.
- **Repetir la receta de la Fase 1 sin parada temprana**, para confirmar en una corrida que el
  colapso fue la parada prematura.
- **Video de sustentación.**
