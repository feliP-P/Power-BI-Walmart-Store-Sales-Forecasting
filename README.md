# 🛒 Walmart Store Sales Forecasting & Performance Analysis

## 📌 Contexto del Proyecto
Como parte de mi desarrollo continuo en el stack de datos, decidí poner a prueba mis habilidades de modelado y visualización utilizando el dataset de **Walmart Recruiting - Store Sales Forecasting** (Kaggle). En lugar de aplicar modelos de Machine Learning tradicionales en Python, el objetivo de este proyecto fue resolver el problema analítico íntegramente dentro del ecosistema de **Power BI**, emulando un entorno corporativo real donde las decisiones de negocio requieren visualizaciones interactivas e insights rápidos.

## 🛠️ Stack Tecnológico y Flujo de Trabajo
*   **Power Query (ETL):** Limpieza de datos, estandarización de formatos regionales (conversión de puntos decimales) y tratamiento de valores nulos (discretización de variables económicas y climáticas).
*   **Modelado de Datos:** Diseño de una base de datos analítica bajo un **Esquema en Estrella (Star Schema)**, separando tablas transaccionales de dimensiones descriptivas para optimizar el filtrado cruzado.
*   **DAX (Data Analysis Expressions):** Creación de medidas dinámicas e inteligencia de tiempo (Time Intelligence) para evaluar rendimiento interanual y rentabilidad.
*   **Visualización & Forecasting:** Diseño de dashboards interactivos e implementación del algoritmo nativo de Suavizado Exponencial de Power BI para proyectar ventas futuras.

## 🗄️ Modelado Relacional (Vista de Modelo)
Estructuré el modelo para asegurar que los filtros fluyan correctamente desde las dimensiones (`stores`, `Calendario`) hacia los hechos (`train`, `features`), aislando los cálculos en una tabla dedicada (`_Medidas`).

![Esquema en Estrella de Walmart](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/esquema.jpg)

## 📊 Muestra de Código DAX
Para evaluar el crecimiento real del negocio frente a la estacionalidad del retail, implementé métricas de inteligencia de tiempo y diseñé nuevos atributos para poder capturar correctamente la estacionalidad de navidad (la temporada de más ventas de todo el año). Un ejemplo clave para aislar el rendimiento durante feriados:

```dax
Ventas en Feriados = 
CALCULATE(
    [Ventas Totales], 
    'train'[IsHoliday] = TRUE
)

Weeks_Until_Christmas = 
VAR CurrentDate = 'features'[Date]
VAR CurrentYear = YEAR(CurrentDate)
VAR ChristmasWeekDate = 
    SWITCH(
        CurrentYear,
        2010, DATE(2010, 12, 31),
        2011, DATE(2011, 12, 30),
        2012, DATE(2012, 12, 28),
        BLANK()
    )
RETURN
IF(
    NOT(ISBLANK(ChristmasWeekDate)) && CurrentDate <= ChristmasWeekDate && MONTH(CurrentDate) = 12,
    DATEDIFF(CurrentDate, ChristmasWeekDate, WEEK),
    0
)
