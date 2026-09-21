# 🎾 ATP World Tour (2000–2019) | Data Analytics & Power BI Dashboard

Dashboard interactivo y análisis integral sobre más de 50.000 partidos del circuito profesional masculino ATP disputados entre las temporadas 2000 y 2019.

El proyecto abarca el ciclo analítico completo: desde la ingesta, limpieza y preparación de datos en Python hasta el modelado dimensional, cálculo de lógica avanzada mediante DAX y el diseño visual e interactivo en Microsoft Power BI.

---

## 📌 Vistas del Dashboard

### 1. Rendimiento y Estadísticas Operativas (Overview Global)

* **Visión macro del circuito:** Seguimiento de volumen de partidos, duración media por encuentro y evolución de aces por temporada.
* **Segmentación multidimensional:** Filtros interactivos por año, tipo de superficie (Clay, Hard, Grass, Carpet), categoría de torneo (Grand Slam, Masters 1000, ATP 500/250) y rondas competitivas.
* **Métricas de servicio y resiliencia:** Análisis de efectividad de primer servicio y porcentaje de puntos de rotura salvados (`% bp_saved`).

### 2. Análisis Cara a Cara (Head-to-Head & Matchups)

* **Comparador dinámico independiente:** Selección cruzada de dos tenistas cualesquiera del circuito mediante tablas desconectadas de dimensiones.
* **Marcador H2H reactivo:** Balance histórico directo de victorias y derrotas con tarjetas de títulos condicionales dinámicos.
* **Desglose por superficie:** Visualización apilada de porcentaje de victorias en arcilla, pista dura y hierba.
* **Registro de enfrentamientos detallado:** Tabla histórica con formato condicional automatizado que tiñe cada partido según el jugador vencedor.

---

## 💡 Insights & Hallazgos Analíticos

### 1. Polarización y Dominio por Superficie (Caso Big 3: Nadal vs Federer)

* **Monopolio en arcilla:** En sus enfrentamientos directos, Rafael Nadal registra una tasa de victoria del **86,67%** en tierra batida (*Clay*), consolidando la mayor ventaja táctica en su balance particular histórico (23-14).
* **Reversión en superficies rápidas:** La ventaja se invierte a favor de Roger Federer en pista dura (*Hard*), donde lidera con un **52,63%** de victorias, y en hierba (*Grass*), alcanzando un **66,67%** de efectividad.
* **Concentración competitiva en finales:** Más del 60% de los 37 encuentros disputados entre ambos ocurrieron en finales (ronda F) de Grand Slams y Masters 1000.

### 2. Eficiencia del Servicio y Dinámica de Juego

* **Umbral crítico del primer servicio:** Los ganadores de los partidos mantienen una media sostenida de puntos ganados con el primer servicio superior al **73%** en todas las superficies. Variaciones de apenas 3–4 puntos porcentuales en esta métrica determinan el desenlace de los sets en partidos disputados.
* **Evolución Aces vs Duración:** A pesar del incremento sostenido en la tasa de aces por partido a lo largo de las décadas (asociado a materiales y biomecánica), la duración media de los encuentros no ha disminuido proporcionalmente (~100-110 min), producto de un mayor desgaste en rallies de fondo.

### 3. Puntos de Quiebre y Resiliencia en Rondas Críticas

* **Gestión de situaciones de presión (`% bp_saved`):** Los jugadores que alcanzan instancias definitorias (semifinales y finales) salvan en promedio más del **65%** de las bolas de quiebre concedidas, actuando como una de las variables con mayor capacidad predictiva de victoria.

---

## 🛠️ Stack Tecnológico y Arquitectura

* **Procesamiento y ETL:** Python (Pandas) — Depuración de registros, tipado de marcadores, normalización de campos temporales y consolidación de nombres de tenistas.
* **Business Intelligence:** Microsoft Power BI Desktop.
* **Lógica de Métricas:** DAX (Data Analysis Expressions).

---

## 📐 Soluciones DAX Destacadas

### 1. Tablas Desconectadas para Selectores Independientes H2H

```dax
Selector_Jugador_A = 
DISTINCT(
    UNION(
        ALLNOBLANKROW(atp_matches_clean[winner_name_full]),
        ALLNOBLANKROW(atp_matches_clean[loser_name_full])
    )
)

Selector_Jugador_B = 
DISTINCT(
    UNION(
        ALLNOBLANKROW(atp_matches_clean[winner_name_full]),
        ALLNOBLANKROW(atp_matches_clean[loser_name_full])
    )
)


2. Balance de Victorias Cara a Cara (Head-to-Head)
H2H_Victorias_J1 = 
VAR J1 = SELECTEDVALUE(Selector_Jugador_A[winner_name_full])
VAR J2 = SELECTEDVALUE(Selector_Jugador_B[winner_name_full])
RETURN
IF(
    ISBLANK(J1) || ISBLANK(J2),
    BLANK(),
    CALCULATE(
        COUNTROWS(atp_matches_clean),
        atp_matches_clean[winner_name_full] = J1,
        atp_matches_clean[loser_name_full] = J2
    ) + 0
)

H2H_Victorias_J2 = 
VAR J1 = SELECTEDVALUE(Selector_Jugador_A[winner_name_full])
VAR J2 = SELECTEDVALUE(Selector_Jugador_B[winner_name_full])
RETURN
IF(
    ISBLANK(J1) || ISBLANK(J2),
    BLANK(),
    CALCULATE(
        COUNTROWS(atp_matches_clean),
        atp_matches_clean[winner_name_full] = J2,
        atp_matches_clean[loser_name_full] = J1
    ) + 0
)

H2H_Total_Partidos = [H2H_Victorias_J1] + [H2H_Victorias_J2]

3. Título dinamicos de tarjetas
Titulo_Tarjeta_J1 = 
VAR Nombre = SELECTEDVALUE(Selector_Jugador_A[winner_name_full], "Jugador 1")
RETURN
"Victorias " & Nombre

Titulo_Tarjeta_J2 = 
VAR Nombre = SELECTEDVALUE(Selector_Jugador_B[winner_name_full], "Jugador 2")
RETURN
"Victorias " & Nombre

4. Formato Condicional Dinámico por Ganador de Partido
Color_Ganador_H2H = 
VAR J1 = SELECTEDVALUE(Selector_Jugador_A[winner_name_full])
VAR J2 = SELECTEDVALUE(Selector_Jugador_B[winner_name_full])
VAR GanadorPartido = SELECTEDVALUE(atp_matches_clean[winner_name_full])
RETURN
SWITCH(
    TRUE(),
    GanadorPartido = J1, "#E66C7D",   -- Tono distintivo Jugador 1
    GanadorPartido = J2, "#9CAFF5",   -- Tono distintivo Jugador 2
    "#333333"
)
