EDA y reducción dimensional de la estructura transaccional cripto

Este notebook realiza un análisis exploratorio de la estructura probabilística del mercado cripto a partir de variables de participación transaccional (*_yp) y precios spot (*USDT) con frecuencia original de 5 minutos.

El objetivo del commit es doble:

caracterizar la concentración, desigualdad y estabilidad temporal de las participaciones yp;

usar esa evidencia para diseñar una reducción dimensional jerárquica y dinámica antes del modelado posterior con GAT y otros modelos.

Notebook: EDA y Split 11/2024-12/2025.ipynb

Período de datos analizado: noviembre de 2024 a diciembre de 2025.

Dataset de entrada

El notebook carga: 112024-122025_clean_add.parquet

Características observadas al abrir el dataset:
Métrica
Valor
Filas
131,616
Columnas
20,918
float32
19,897
int8
1,020
datetime64[ns, UTC]
1
Memoria aproximada 9.9 GB

La columna temporal utilizada es bar_end.

1. Agregación macro a 6 horas
Se seleccionan únicamente las columnas que terminan en:
_yp
USDT
A partir de las velas originales de 5 minutos se construye un dataframe de frecuencia 6H, calculando por variable:
media;
máximo;
mínimo.

El resultado es dfmacro:
Métrica
Valor
Filas
1,829
Columnas
3,061
Tipo numérico
float32
Memoria aproximada
21.4 MB
El dataframe se guarda como:
112024-122025_macro_add1.parquet
2. Análisis descriptivo de yp
Para cada una de las variables *_yp_mean se calcula su media temporal y se estudia la distribución transversal resultante.
El universo contiene 510 participaciones.

[Uploading 2025.ipynb…]()

Estadísticos principales

Estadístico

Valor

Media

0.001961

Mediana

0.000348

Varianza

0.000122

Desviación estándar

0.011041

Sesgo

11.380363

Curtosis

142.161713

Q1

0.000191

Q3

0.000791

IQR

0.000600

Máximo

0.164137

La distribución presenta una asimetría positiva extrema y una cola derecha muy pesada: una fracción reducida de activos concentra gran parte de la masa probabilística.

También se calculan percentiles P1, P5, P10, P25, P50, P75, P90, P95 y P99 y se grafica el histograma correspondiente.

3. Concentración: Lorenz y Gini

Se construye la curva de Lorenz de las participaciones medias y se obtiene:

Gini = 0.830928

El valor confirma una estructura fuertemente concentrada.

Concentración acumulada

Grupo

Nº variables

Masa acumulada

Top 1%

6

53.29%

Top 5%

26

71.64%

Top 10%

51

78.97%

Top 25%

128

88.47%

Top 50%

255

95.15%

La mitad inferior de las variables concentra, por lo tanto, apenas 4.85% de la masa media total.

Variables necesarias para alcanzar una masa determinada

Masa objetivo

Variables necesarias

% del universo

50%

6

1.18%

80%

56

10.98%

90%

150

29.41%

95%

251

49.22%

99%

406

79.61%

4. Entropía, HHI y número efectivo de participantes

La distribución se normaliza y se calculan medidas de diversidad y concentración.

Métrica

Resultado

Variables nominales

510

Entropía de Shannon

4.008575

Entropía máxima

6.234411

Entropía normalizada

0.642976

Nº efectivo Shannon

55.068367

HHI

0.064010

Nº efectivo Simpson/HHI

15.622677

Aunque existen 510 variables nominales, la diversidad observada equivale aproximadamente a 55 participantes igualmente ponderados. Cuando se enfatizan los participantes dominantes mediante HHI/Simpson, el sistema equivale a apenas 16 participantes efectivos.

5. Concentración dentro del núcleo informativo

Se estudian las 55 variables de mayor yp_mean, aproximando el número efectivo de Shannon global.

Estas 55 variables capturan aproximadamente:

79.82% de la masa original

Dentro del propio núcleo la concentración vuelve a aparecer:

Grupo dentro de las 55

Nº variables

Masa del núcleo

Top 1%

1

20.56%

Top 5%

3

50.32%

Top 10%

6

66.76%

Top 25%

14

81.73%

Top 50%

28

90.73%

Además:

N efectivo Shannon ≈ 18.69
N efectivo Simpson ≈ 9.98

Esto motiva una lectura jerárquica de la estructura:

510 → 55 → 19 → 10 → 3

Interpretación:

510: universo nominal;

55: número efectivo Shannon del mercado completo;

19: número efectivo Shannon dentro del núcleo de 55;

10: número efectivo Simpson dentro del núcleo;

3: variables necesarias para superar aproximadamente el 50% de la masa del núcleo.

Nota sobre el Gini del subconjunto de 55

La celda actual que calcula gini55 contiene un denominador incorrecto y devuelve un valor fuera del rango válido [0, 1]. Ese resultado no debe interpretarse. El análisis general de concentración y las demás métricas no dependen de ese valor.

6. Concentración de la dispersión transversal

Para los subconjuntos jerárquicos se compara la suma de cuadrados respecto de la media global con el resto del universo.

Subconjunto

n

% de dispersión transversal

Top Shannon global

55

98.20%

Top Shannon del núcleo

19

98.07%

Top Simpson del núcleo

10

97.47%

Top 50% del núcleo

3

86.53%

Estos porcentajes se interpretan como concentración de la dispersión transversal respecto de la media global, no como “varianza explicada” de un modelo o de un PCA.

7. Validación temporal de la estructura

Para evitar que las conclusiones dependan únicamente del promedio de toda la muestra, las métricas se recalculan sobre ventanas de 30 días.

En cada ventana se obtienen:

Gini;

Shannon y Shannon normalizado;

número efectivo Shannon;

HHI;

número efectivo Simpson;

masa de los Top 1%, 5%, 10%, 25% y 50%;

número de variables necesarias para alcanzar 50%, 80%, 90%, 95% y 99% de la masa.

La estructura jerárquica se mantiene durante todo el período. Entre las ventanas observadas:

Gini se mantiene aproximadamente entre 0.842 y 0.913;

Top 5% concentra aproximadamente 65.8%–82.7%;

Top 10% concentra aproximadamente 76.6%–88.8%;

Top 25% concentra aproximadamente 90.4%–95.1%;

Top 50% concentra aproximadamente 97.3%–98.5%;

Nº efectivo Shannon varía aproximadamente entre 30 y 74;

Nº efectivo Simpson varía aproximadamente entre 11 y 26.

La concentración es, por lo tanto, una propiedad temporalmente persistente del sistema, aunque su intensidad cambia por régimen.

8. Retención y Jaccard del Top 5%

El notebook analiza si los 26 activos que forman el Top 5% permanecen estables entre ventanas consecutivas.

Resultados:

Métrica

Resultado

Retención media

72.05%

Jaccard medio

56.72%

Retención mínima

57.69%

Retención máxima

80.77%

Coincidencias medias

18.73 de 26

Entradas/salidas medias

7.27 por período

La conclusión central es:

La estructura de concentración es persistente, pero la identidad de los líderes es parcialmente dinámica.

Existe un núcleo con elevada recurrencia, pero aproximadamente una cuarta parte del Top 5% puede rotar entre períodos consecutivos.

Conclusiones principales

El mercado está fuertemente concentrado. Una fracción pequeña de las variables acumula la mayor parte de la masa probabilística.

La dimensión nominal sobreestima enormemente la dimensión efectiva. El sistema pasa de 510 variables nominales a ~55 participantes efectivos por Shannon y ~16 por Simpson/HHI.

La concentración es anidada. Incluso dentro del núcleo de 55 variables vuelve a aparecer una estructura altamente jerárquica.

La dispersión transversal también está concentrada. Los 55 principales concentran ~98.2% de la suma de cuadrados respecto de la media global.

La estructura es temporalmente estable. Los cuantiles superiores mantienen masas elevadas a través de ventanas de 30 días.

Los líderes no son completamente permanentes. El Top 5% tiene una retención media de 72.05%, por lo que una arquitectura basada únicamente en nombres fijos perdería parte de la dinámica del mercado.

La reducción dimensional debería preservar posiciones estructurales, no identidades estáticas.

Estrategia propuesta de reducción dimensional

La siguiente etapa consiste en construir una representación jerárquica, dinámica y causal del mercado.

La idea no es seleccionar permanentemente BTC, ETH, SOL, etc., sino ordenar los activos según su yp usando únicamente información disponible hasta cada instante y representar roles estadísticos dinámicos.

1. División dinámica por ranking

Una primera partición propuesta es:

Bottom 50%
50% – 75%
75% – 90%
Núcleo superior

Los límites se recalculan de manera causal y la identidad de los activos puede cambiar en el tiempo.

2. Compresión progresiva según relevancia

Bottom 50%

La mitad inferior concentra muy poca masa y presenta baja relevancia individual. Se comprime en una representación agregada del bloque.

Mediana – Q3

Mantener:

2 líderes dinámicos + residual agregado del resto del bloque

Q3 – D9

Mantener:

4 líderes dinámicos + residual agregado del resto del bloque

Núcleo superior

La cabeza se vuelve a tratar como una distribución anidada. El esquema discutido para un núcleo de aproximadamente 55 variables es:

28 inferiores  → 1 agregado
14 siguientes → 2 líderes dinámicos + 1 residual
8 siguientes  → 4 líderes dinámicos + 1 residual
5 superiores  → variables individuales dinámicas

Importante: 28 + 14 + 8 + 5 = 55. Por lo tanto, esta partición corresponde naturalmente al núcleo Shannon aproximado de 55 variables, no al Top 10% exacto de 510, que contiene aproximadamente 51 variables. Si se desea usar estrictamente el Top 10%, los cortes deberán ajustarse.

3. Información estructural de los residuales

La suma del residual conserva la masa del grupo, pero no describe su estructura interna. Por ello se propone complementar cada residual con variables como:

volatilidad interna;

entropía de Shannon interna;

HHI/concentración interna;

componentes principales obtenidos mediante PCA.

4. PCA causal sobre los residuales

El PCA se aplicará solo a las variables residuales, no a los líderes que ya se mantienen explícitos.

Objetivo:

capturar factores comunes de variación/volatilidad dentro de grupos grandes;

evitar duplicar información de los líderes explícitos;

reducir dimensionalidad adicional sin destruir completamente la dinámica interna de la cola.

El PCA deberá ser rolling/expanding y entrenado exclusivamente con información pasada para evitar leakage.

5. Representación conceptual final

La arquitectura buscada puede resumirse como:

Ranking dinámico causal
        ↓
Bloques jerárquicos
        ↓
Líderes dinámicos explícitos
        +
Masa residual agregada
        +
Entropía / concentración / volatilidad residual
        +
PCA causal de los residuales
        ↓
Representación comprimida para GAT / modelos posteriores

La hipótesis de trabajo es que esta representación preserva la geometría y concentración de la distribución probabilística mejor que una selección fija de activos por nombre.

Consideraciones metodológicas

Todos los rankings, cuantiles y selecciones deben calcularse usando solo información pasada respecto del instante que se desea predecir.

El PCA debe ajustarse de manera rolling o expanding; nunca sobre toda la muestra.

Las variables agregadas deben construirse de forma que no dupliquen exactamente información ya mantenida como variables individuales.

La estabilidad de la estrategia deberá validarse fuera de muestra comparando distintos esquemas de compresión.

La justificación de la agregación no debe basarse directamente en la Ley de los Grandes Números: las participaciones crypto no son observaciones iid. La evidencia relevante proviene de la concentración empírica, la estabilidad temporal y la reducción de heterogeneidad observada en los estratos inferiores.

Dependencias principales

pandas
numpy
matplotlib
pyarrow

El notebook está diseñado para ejecutarse en Google Colab con Google Drive montado.

Estado actual

Implementado en este commit:

carga e inspección del dataset;

agregación 6H;

EDA de las probabilidades yp;

percentiles y estadísticos descriptivos;

Lorenz y Gini;

concentración por Top-k;

umbrales de masa;

Shannon, HHI y números efectivos;

análisis recursivo del núcleo informativo;

concentración de la dispersión transversal;

estabilidad temporal a 30 días;

retención y Jaccard del Top 5%.

Siguiente etapa:

implementar la compresión dinámica por bloques;

construir features de entropía, volatilidad y concentración interna;

incorporar PCA causal de residuales;

comparar esquemas de compresión antes de alimentar el GAT.
