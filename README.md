# 🛒 Walmart Store Sales Forecasting & Performance Analysis

## 📌 Contexto del Proyecto
Como parte de mi desarrollo continuo en el stack de datos, decidí poner a prueba mis habilidades de modelado y visualización utilizando el dataset de **Walmart Recruiting - Store Sales Forecasting** (Kaggle). En lugar de aplicar modelos de Machine Learning tradicionales en Python, el objetivo de este proyecto fue resolver el problema analítico íntegramente dentro del ecosistema de **Power BI**, emulando un entorno corporativo real donde las decisiones de negocio requieren visualizaciones interactivas e insights rápidos.

*(En una siguiente etapa planeo utilizar este mismo análisis para avanzar hacia la creación de modelos predictivos de Machine Learning o análisis estadístico avanzado de series temporales).*

## 🛠️ Stack Tecnológico y Flujo de Trabajo
*   **Power Query (ETL):** Limpieza de datos, estandarización de formatos regionales (conversión de separadores decimales) y tratamiento de valores nulos (discretización de variables económicas y climáticas).
*   **Modelado de Datos:** Diseño de una base de datos analítica bajo un **Esquema en Estrella (Star Schema)**, separando tablas transaccionales de dimensiones descriptivas para optimizar el filtrado cruzado.
*   **DAX (Data Analysis Expressions):** Creación de medidas dinámicas e inteligencia de tiempo (Time Intelligence) para evaluar rendimiento interanual y rentabilidad.
*   **Visualización & Forecasting:** Diseño de dashboards interactivos e implementación del algoritmo nativo de Suavizado Exponencial de Power BI para proyectar ventas futuras.

## 📊 Dashboards y Visualizaciones

### Dashboard 1: Serie Temporal, Estacionalidad y Proyección
*   **Tendencia y Previsión:** En el gráfico principal de líneas se observan las ventas totales semanales (febrero 2010 – octubre 2012) y una extensión con la previsión para los 8 meses posteriores. El modelo captura con precisión los picos de ventas de noviembre y diciembre (Black Friday y Navidad), así como la contracción estacional y posterior recuperación en los meses siguientes.
*   **Métricas y Rendimiento:** Incorporé tablas con datos de ventas anuales, comparativas entre semanas regulares y festivas (volumen total y promedio por día festivo vs. normal) y el desempeño por formato de tienda (Type A, B y C).
*   **Correlación por Superficie:** Se incluye la relación directa entre el tamaño de la tienda y su volumen de facturación, donde se aprecia una correlación positiva marcada.

![Dashboard 1 serie completa](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard1.jpg)

#### Aislamiento de Días Normales (Filtrado de Feriados)
A través de ingeniería de atributos (desarrollando un contador de semanas restantes hacia Navidad) y el uso de segmentadores interactivos, es posible aislar el efecto de las festividades para evaluar el comportamiento base del negocio. Al remover estos picos, se hace evidente que enero es históricamente el mes con menor facturación y se descubren dinámicas estacionales secundarias que quedaban enmascaradas por el impacto de fin de año.

![Dashboard 1 sin holidays](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard1SinFeriadosNiNavidad.jpg)

### Dashboard 2: Factores Macroeconómicos y Clima
Disposición de diagramas de dispersión (*scatter plots*) con líneas de regresión para analizar la correlación entre factores externos y las ventas semanales:
*   **Nivel de Desempleo:** Muestra una correlación negativa moderada (a mayor tasa de desempleo, menor nivel de ventas).
*   **Índice CPI:** Presenta una correlación ligeramente positiva.
*   **Temperatura:** Revela una correlación positiva general, aunque el histograma discretizado por temperatura promedio evidencia que el grueso de la facturación se concentra alrededor de los 57 °F.
*   **Filtros Interactivos:** Permiten evaluar cómo varían estas relaciones según tienda, período y presencia de feriados (se excluyeron los festivos en esta vista para evitar distorsiones causadas por la inelasticidad de la demanda en esas fechas).

![Dashboard 2 scatter plots y regresiones](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard2.jpg)

## 🗄️ Modelado Relacional (Vista de Modelo)
Estructuré el modelo relacional para asegurar que los filtros fluyan en sentido unidireccional (1:*) desde las dimensiones (`stores`, `Calendario`) hacia las tablas de hechos (`train`, `features`), aislando los cálculos en una tabla dedicada (`_Medidas`).

![Esquema en Estrella de Walmart](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/esquema.jpg)

## 📐 Muestra de Código DAX
Para evaluar el crecimiento frente a la estacionalidad del retail, implementé métricas de inteligencia de tiempo y diseñé atributos calculados para aislar la temporada navideña:

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
