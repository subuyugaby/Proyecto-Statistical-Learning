# Resumen Ejecutivo — Predicción del Precio Horario de la Energía en Guatemala

**Curso:** Statistical Learning I · **Objetivo:** predecir el precio horario de oportunidad de la
energía (USD/MWh) en **2025** usando solo datos históricos de **2023–2024**.

---

## 1. Datos y limpieza
Tres reportes anuales del AMM (formato largo: una fila por fecha+hora). Correcciones documentadas:

- **Horas mal digitadas** (3 casos en 2023): `220→22`, `230→23`, `2300→23`.
- **Precios con caracteres basura** (9 casos): p. ej. `167.9951l`, `69.038.0`, `2.1302.wq` → se
  extrae el número válido; el valor irrecuperable (`d`) se trata como faltante.
- **Valores extremos imposibles** (~11 casos, hasta **1.3 millones** USD/MWh): errores de captura
  marcados como faltantes. Los **picos altos plausibles** (~300–450) se conservan (son escasez real).

Tras la limpieza: cada día tiene 24 registros, sin duplicados, años completos.

## 2. Comportamiento del mercado
- Precio **volátil** y sesgado a la derecha. Promedios anuales **similares**: 2023 ≈ **105**,
  2024 ≈ **116**, 2025 ≈ **85** USD/MWh; 2025 es algo más bajo y **menos volátil**.
- **Patrón horario marcado**: barato de madrugada (mín. ~3 h ≈ 77) y caro al final de la tarde
  (máx. ~18 h ≈ 136). La **hora** es el factor más influyente; el **mes** aporta estacionalidad; el
  **día de la semana** influye poco.
- Horas clasificadas en **valle / intermedia / pico** por terciles del precio medio (solo train).

## 3. Modelado
- `Pipeline` + `ColumnTransformer` con escalado, one-hot, variables cíclicas y **medias históricas
  sin fuga de información** (calculadas con funciones solo sobre el train). Validación cruzada
  **temporal** (`TimeSeriesSplit`) y optimización con Grid/Randomized Search.
- Se compararon 9 modelos: baseline, lineal, polinomial, Ridge, Lasso, ElasticNet, polinomial+Ridge,
  **Random Forest** y **Gradient Boosting**.

| Modelo | RMSE (CV) | MAE 2025 | RMSE 2025 | R² 2025 |
|--------|----------:|---------:|----------:|--------:|
| **Lasso (ganador, α≈3.16)** | **49.9** | 35.8 | 44.5 | −0.45 |
| ElasticNet | 50.8 | 35.5 | 43.4 | −0.38 |
| Ridge | 52.0 | 36.9 | 45.8 | −0.53 |
| Gradient Boosting | 53.1 | 36.4 | 46.2 | −0.56 |
| Random Forest | 54.1 | 37.0 | 47.4 | −0.65 |
| Baseline (media) | 66.0 | 37.5 | 45.3 | −0.51 |
| Polinomial puro | 35 285 | 132.8 | 213.5 | −32.4 |

El **polinomio sin regularización sobreajusta de forma extrema** (RMSE CV ≈ 35 000) mientras que su
versión regularizada lo controla: ejemplo directo del compromiso **sesgo–varianza**.

## 4. Hallazgo principal
Sobre 2025 (datos no vistos), los modelos **apenas mejoran al baseline ingenuo**: el ganador (Lasso,
RMSE 44.5) supera solo levemente a predecir la media (45.3) y el **R² sigue negativo** (−0.45), es
decir, peor que predecir el promedio del propio 2025. El error absoluto medio (~36 USD/MWh ≈ 43 % del
precio medio) es alto.

**Interpretación:** las variables de **calendario e histórico tienen poco poder predictivo** sobre el
precio horario. El precio lo determinan fundamentales del mercado ausentes de los datos (demanda,
hidrología, combustibles, disponibilidad de plantas). Los modelos reproducen el *perfil* horario pero
no el *nivel* ni la *volatilidad* específicos de 2025.

## 5. Conclusiones y recomendaciones
- **Modelo elegido:** Lasso regularizado — empata o supera a los ensembles en CV pero es más simple e
  interpretable (y selecciona variables al anular coeficientes).
- **Variables más útiles:** medias históricas por hora y hora×día de la semana, y las variables horarias.
- **¿Confiar para decisiones del AMM?** **No** para decisiones operativas: solo sirve como referencia
  cualitativa del patrón horario relativo.
- **Mejoras:** incorporar **demanda, generación, hidrología, precios de combustibles, indisponibilidad
  de plantas e intercambios**; reentrenar con datos recientes; explorar modelos de series temporales
  con regresores exógenos.

*Detalle completo, código y gráficas en `Proyecto1_Precios_Energia.ipynb`. Predicciones horarias en
`outputs/predicciones_2025.csv`.*
