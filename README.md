# Proyecto 1 — Predicción de Precios de Oportunidad de la Energía en Guatemala

Modelos de regresión, ingeniería de variables y explicabilidad para predecir el **precio horario de
oportunidad de la energía** (USD/MWh) en Guatemala durante **2025**, entrenando únicamente con datos
históricos de **2023 y 2024** (Administrador del Mercado Mayorista, AMM).

## Contenido

| Archivo | Descripción |
|---------|-------------|
| `Proyecto1_Precios_Energia.ipynb` | **Notebook principal**: único archivo donde se ejecuta código. Contiene todo el desarrollo (carga, limpieza, EDA, ingeniería de variables, pipelines, modelado, optimización, evaluación 2025 y respuestas a las preguntas). |
| `Precios2023.xls`, `Precios2024.xls`, `Precios2025.xls` | Datos crudos del AMM (entrada). |
| `outputs/figures/` | Gráficas generadas por el notebook. |
| `outputs/predicciones_2025.csv` | Predicciones para 2025 (fecha, hora, precio predicho, precio real, error absoluto). |
| `Resumen_Ejecutivo.md` | Resumen ejecutivo con hallazgos, mejor modelo y conclusiones. |
| `requirements.txt` | Dependencias para reproducir el proyecto. |

## Cómo reproducir

```bash
# 1) Instalar dependencias (Python 3.9+)
pip install -r requirements.txt

# 2) Abrir y ejecutar el notebook de principio a fin
jupyter notebook Proyecto1_Precios_Energia.ipynb
#   o, sin interfaz:
python3 -m nbconvert --to notebook --execute --inplace Proyecto1_Precios_Energia.ipynb
```

El notebook crea automáticamente la carpeta `outputs/` con las figuras y el CSV de predicciones.

## Notas de diseño

- **Sin fuga de información (*data leakage*)**: los datos de 2025 nunca se usan para entrenar ni
  ajustar transformaciones. Las variables derivadas del precio (medias históricas, clasificación de
  horas) se calculan con funciones **únicamente sobre el train (2023–2024)** y luego se aplican a 2025.
- **Validación cruzada temporal** (`TimeSeriesSplit`): cada fold entrena con el pasado y valida con el
  futuro inmediato.
- **Modelos comparados**: baseline, regresión lineal, polinomial, regularización (Ridge/Lasso/
  ElasticNet), polinomial+regularización y ensembles (Random Forest, Gradient Boosting).
