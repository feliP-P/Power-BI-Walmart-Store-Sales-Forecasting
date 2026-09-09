# 🛒 Walmart Store Sales Forecasting & Performance Analysis

## 📌 Contexto del Proyecto
Como parte de mi desarrollo continuo en el stack de datos, decidí poner a prueba mis habilidades de modelado y visualización utilizando el dataset de **Walmart Recruiting - Store Sales Forecasting** (Kaggle). En lugar de aplicar modelos de Machine Learning tradicionales en Python, el objetivo de este proyecto fue resolver el problema analítico íntegramente dentro del ecosistema de **Power BI**, emulando un entorno corporativo real donde las decisiones de negocio requieren visualizaciones interactivas e insights rápidos.
(En una siguiente etapa planeo utilizar este mismo análisis para saltar a la creación de modelos de predicción de machine learning o modelos de estadísticos/series temporales)

## 🛠️ Stack Tecnológico y Flujo de Trabajo
*   **Power Query (ETL):** Limpieza de datos, estandarización de formatos regionales (conversión de puntos decimales) y tratamiento de valores nulos (discretización de variables económicas y climáticas).
*   **Modelado de Datos:** Diseño de una base de datos analítica bajo un **Esquema en Estrella (Star Schema)**, separando tablas transaccionales de dimensiones descriptivas para optimizar el filtrado cruzado.
*   **DAX (Data Analysis Expressions):** Creación de medidas dinámicas e inteligencia de tiempo (Time Intelligence) para evaluar rendimiento interanual y rentabilidad.
*   **Visualización & Forecasting:** Diseño de dashboards interactivos e implementación del algoritmo nativo de Suavizado Exponencial de Power BI para proyectar ventas futuras.

## Dashboards: Gráficos
En el primer Dashboard agregué como gráfico principal un gráfico de líneas en el que podemos ver las ventas totales de los negocios de walmart a los largo del tiempo (desde febrero de 2010 hasta octubre de 2012) y luego extendemos el gráfico con la previsión para los siguientes 8 meses. Podemos observar en este gráfico que hay grandes picos de ventas en noviembre y diciembre correspondientes al black friday y navidad respectivamente y nuestra previsión logra capturarlo perfectamente, además de la depresión y recuperación en los meses posteriores, osea que logramos capturar bien el comportamiento de esta serie temporal.
También agregué tablas con los datos especificos de las ventas de cada año, graficos comparativos de las cantidad de ventas que se produjeron en días festivos y en días normales, y el promedio de ventas en un día festivo y un día normal, gráficos que comparan las ventas que se produjeron en cada tipo de tienda y por último grafiqué la relación entre el tamaño de las tiendas y las ventas que producen (en donde observamos que, con mucho sentido, hay una correlación muy fuerte).
![Dashboard 1 serie completa](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard1.jpg)

Luego utilizando los filtros interactivos que agregamos, podemos "eliminar las ventas" de los días festivos y su efecto en las ventas, logrando observar el comportamiento de las ventas durante el resto de "días normales". Se observa mejor que enero es el mes con menos ventas y otras dinamicas estacionales que quedaban obscurecidas por el impacto mayor de las fechas festivas.

![Dashboard 1 sin holidays](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard1SinFeriadosNiNavidad.jpg)

En el segundo dashboard dispuse los scatter plots sobre los que realicé una regresión para poder ver mejor la relación y la dinámica entre las ventas y otros factores externos, como el indice de desempleo (con una pequeña correlación negativa, es decir que cuando crece el indice decrecen las ventas), el indice CPI (con una pequeña correlación positiva) y la temperatura (en la que se ve una correlación positiva), aunque la mayoría de ventas se producen en torno a los 57° como se puede ver en el otro gráfico de Ventas Totales según la Temperatura media.
También integré al igual que en el dashboard anterior algunos filtros interactivos para poder analizar como cambian estas relaciones y efectos en distintos momentos, distintas tiendas y si incluímos o no los días festivos en el análisis (en general los días festivos nos dificultan ver estas relaciones ya que siempre hay altas ventas sin importar los factores externos, por eso en esta imagen no los tomé en cuenta).

![Dashboard 2 scatter plots y regresiones](https://github.com/feliP-P/Power-BI-Walmart-Store-Sales-Forecasting/blob/main/dashboard2.jpg)




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
