# *Trabajo Final Capstone: Google Data Analytics* 

## **Paso 1: Selección de Datasets**
Elegimos los datasets con los que vamos a trabajar. Basándonos en el tipo de información útil para el análisis, utilizaremos los siguientes:

1. `dailyActivity`
2. `hourlySteps`
3. `dailyIntensities`

---

## **Paso 2: Análisis y Carga de Datasets**
### **Identificación de registros extra**
Nos encontramos con que en `dailyActivity` hay 2 registros más en el dataset del `03/` en comparación con el `04/`, por lo que utilizamos `EXCEPT ALL` con el siguiente comando:

```sql
SELECT Id FROM Tabla_1
EXCEPT DISTINCT
SELECT Id FROM Tabla_2;
```

Encontramos que los siguientes IDs están de más:
```
2891001357,
6391747486
```
En total había 17 filas con estos IDs innecesarios, por lo que procedimos a eliminarlas con el siguiente comando:

```sql
DELETE FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.03_dailyActivity
WHERE Id IN (6391747486, 2891001357);
```

Realizamos el mismo procedimiento con `hourlySteps` y encontramos 2 IDs que no figuraban en `dailyActivity`:

```sql
SELECT Id
FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.03_hourlySteps
EXCEPT DISTINCT
SELECT Id FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.03_dailyActivity;
```

Esto nos dejó con los IDs:
```
2891001357,
6391747486
```
Sin embargo, el dataset con estas bajas quedaría con 32 usuarios en vez de 33. Aplicando la fórmula inversa, encontramos que también debíamos eliminar el siguiente ID:
```
4388161847
```
Esto lo replicamos en los datasets `03` y `04`.

Para eliminar las filas usamos:

```sql
DELETE FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.04_hourlySteps
WHERE Id IN (ID_A_BORRAR);
```

---

## **Paso 3: Unión de Datasets**
Ahora que los cuatro datasets tienen los mismos IDs, procedemos a unirlos con `UNION ALL`:

```sql
CREATE TABLE mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined AS
SELECT * FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.03_dailyActivity
UNION ALL
SELECT * FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.04_dailyActivity;
```

```sql
CREATE TABLE mi-primer-proyecto-436821.Trabajo_Final_Coursera.hourlySteps_combined AS
SELECT * FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.03_hourlySteps
UNION ALL
SELECT * FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.04_hourlySteps;
```

---

## **Paso 4: Limpieza de Duplicados**
Para eliminar registros duplicados, utilizamos `SELECT DISTINCT`:

```sql
CREATE TABLE mi_dataset.dailyActivity_clean AS
SELECT DISTINCT *
FROM mi_dataset.dailyActivity_combined;
```

Para verificar la cantidad de registros antes y después:

```sql
SELECT COUNT(*) AS total_registros
FROM mi_dataset.dailyActivity_combined;
```

También revisamos si hay registros nulos con:

```sql
SELECT
    column_name,
    COUNT(*) AS total_registros,
    COUNTIF(column_value IS NOT NULL) AS registros_no_nulos,
    COUNTIF(column_value IS NULL) AS registros_nulos
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyIntensities`,
UNNEST([
    STRUCT(CAST(Id AS STRING) AS column_value, "Id" AS column_name),
    STRUCT(CAST(ActivityDay AS STRING) AS column_value, "ActivityDay" AS column_name),
    STRUCT(CAST(SedentaryMinutes AS STRING) AS column_value, "SedentaryMinutes" AS column_name),
    STRUCT(CAST(LightlyActiveMinutes AS STRING) AS column_value, "LightlyActiveMinutes" AS column_name)
])
GROUP BY column_name;
```

Aplicamos esta fórmula a los tres datasets y confirmamos que no hay valores nulos, utilizamos la funcion CAST porque necesitabamos que todos esten en el mismo formato para visualizar si hay valores nulos. 

---

## **Paso 5: Revisión de Formato de Columnas**
Verificamos que los datos tengan el formato correcto (`STRING`, `INT64`, `DATE`, etc.). En caso de errores, podríamos utilizar `CAST`:

```sql
SELECT CAST(column_name AS INT64) FROM tabla;
```
No hubo problemas de formato en este caso.

---

## **Paso 6: Detección de Valores Imposibles**
Revisamos si hay valores negativos o extremadamente grandes con análisis de rangos:

```sql
SELECT
  MIN(TotalSteps) AS Min_Steps,
  MAX(TotalSteps) AS Max_Steps,
  AVG(TotalSteps) AS Avg_Steps,
  
  MIN(TotalDistance) AS Min_Distance,
  MAX(TotalDistance) AS Max_Distance,
  AVG(TotalDistance) AS Avg_Distance
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`;
```

Confirmamos que no hay valores ilógicos en los datasets.

---

## **Paso 7: Análisis de Datos**

*DATASET daily_activity* 

El primer paso será calcular los maximos, minimos, promedios y desviacion estandar del total de pasos y distancia del dataset daily_activity

```sql
SELECT  
  MIN(TotalSteps) AS Min_Steps,  
  MAX(TotalSteps) AS Max_Steps,  
  AVG(TotalSteps) AS Avg_Steps,  
  STDDEV(TotalSteps) AS StdDev_Steps,  

  MIN(TotalDistance) AS Min_Distance,  
  MAX(TotalDistance) AS Max_Distance,  
  AVG(TotalDistance) AS Avg_Distance,  
  STDDEV(TotalDistance) AS StdDev_Distance  

FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`;
```
Nos da los siguientes valores: 

*Tabla 1*

| Min_Steps | Max_Steps | Avg_Steps | StdDev_Steps | Min_Distance | Max_Distance | Avg_Distance | StdDev_Distance |
|-----------|-----------|-----------|--------------|--------------|--------------|--------------|-----------------|
| 0,000     | 36019,000 | 7321,374  | 5186,429     | 0,000        | 28,030       | 5,233        | 3,972           |

El valor de la desviacion estandar no agrega valor al trabajo actual por lo que lo desestimaremos. 

---

Luego calculamos el promedio de pasos, distancia recorrida y calorias quemadas por cada uno de los usuarios (Id): 

```sql
SELECT  
  Id,  
  AVG(TotalSteps) AS Promedio_Pasos,  
  AVG(TotalDistance) AS Promedio_Distancia,  
  AVG(Calories) AS Promedio_Calorias_Quemadas 
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`  
GROUP BY Id  
ORDER BY Promedio_Pasos DESC;
```
---

Obtuvimos los siguientes resultados: 

*Tabla 2* 

| Id         | Promedio_Pasos     | Promedio_Distancia  | Promedio_Calorias_Quemadas |
|------------|--------------------|---------------------|----------------------------|
| 8877689391 | 16424.325581395347 | 13.457906878271768  | 3428.8837209302324         |
| 8053475328 | 14784.523809523809 | 11.501666590216606  | 2932.0238095238096         |
| 1503960366 | 11935.779999999997 | 7.7327999904751765  | 1808.7399999999998         |
| 7007744171 | 11619.28947368421  | 8.2810526207873725  | 2570.2368421052633         |
| 2022484408 | 11595.093023255815 | 8.2767441716305044  | 2500.302325581396          |
| 6962181067 | 10679.888888888889 | 7.2273333549499528  | 2015.377777777778          |
| 3977333714 | 10321.523809523807 | 7.0283332793485558  | 1480.6428571428569         |
| 2347167796 | 9647.12121212121   | 6.4263636424121549  | 2033.3939393939393         |
| 8378563200 | 8555.1627906976737 | 6.7846512073694267  | 3414.1395348837218         |
| 5553957443 | 8540.6279069767425 | 5.5890697975491372  | 1855.2558139534885         |
| 7086361926 | 8459.8139534883721 | 5.7479069592907672  | 2457.6976744186049         |
| 5577150313 | 8385.9268292682955 | 6.27634143247837    | 3343.7073170731705         |
| 4702921684 | 8367.065217391304  | 6.7891304207884726  | 2918.5652173913049         |
| 1644430081 | 7780.925           | 5.65875001549721    | 2837.5749999999989         |
| 4319703577 | 7422.8139534883712 | 4.9937209216228045  | 2025.5581395348838         |
| 6117666160 | 7363.0000000000009 | 5.5750000351353695  | 2218.5526315789471         |
| 2873212765 | 7299.2558139534885 | 4.9251162645428685  | 1855.2325581395351         |
| 4558609924 | 7154.9302325581411 | 4.7297674459080365  | 1976.5813953488368         |
| 3372868164 | 6616.9333333333334 | 4.5433333555857338  | 1908.8333333333335         |
| 8583815059 | 6346.6153846153838 | 4.9512820152136     | 2662.1282051282051         |
| 1624580081 | 5167.2             | 3.4710000014305105  | 1433.7799999999997         |
| 2026352035 | 4960.13953488372   | 3.0776744173016657  | 1488.9767441860467         |
| 8253242879 | 4898.0645161290313 | 3.5106451780565324  | 1662.1935483870968         |
| 4445114986 | 4632.3695652173919 | 3.13521738803905    | 2160.6304347826085         |
| 6290855005 | 4615.8461538461543 | 3.4905128295605     | 2488.3333333333335         |
| 2320127002 | 4276.3720930232548 | 2.8904650862826857  | 1670.558139534884          |
| 4020332650 | 4049.7619047619046 | 2.9049206349466528  | 2736.0634920634907         |
| 6775888955 | 3301.2285714285713 | 2.3719999735908854  | 2284.2571428571432         |
| 1844505072 | 2876.0232558139533 | 1.9016279157164488  | 1585.3255813953492         |
| 8792009665 | 2217.0             | 1.419024387374521   | 1994.9024390243903         |
| 4057192912 | 2103.9722222222222 | 1.5527777807373144  | 1911.333333333333          |
| 1927972279 | 1269.0697674418604 | 0.87906976486014754 | 2195.4651162790697         |

---

Podemos observar cierta correlacion entre que a mayor cantidad de pasos mayor es el numero de calorias que queman, aun asi vamos a realizar una nueva consulta observando de forma particular si esta correlacion se observa cuando hay cambios en el numero de pasos de cada Id.

**¡Observacion!** 

Identifique que hay muchas filas en las cuales el numero de pasos o distancia recorrida a nivel diario es 0, esto es imposible (a nivel diario) por lo que eliminaremos estas filas del analisis utilizando el siguiente comando:

```sql
SELECT * 
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`  
WHERE TotalSteps = 0 OR TotalDistance = 0;
```
---

Una vez identificadas las filas que contienen el valor "0" en una o ambas categorias procedimos con la eliminacion de las mismas a traves del siguiente comando: 

```sql
DELETE FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`  
WHERE TotalSteps = 0 OR TotalDistance = 0;
```
---

Se eliminaron 121 registros de la tabla, una vez realizado esto ejecutamos nuevamente la consulta previa para observar si existen diferencias notables en las correlaciones observables y obtuvimos los siguientes resultados: 

*Tabla 2 Modificada* 

| Id         | Promedio_Pasos     | Promedio_Distancia | Promedio_Calorias_Quemadas |
|------------|--------------------|--------------------|----------------------------|
| 8877689391 | 16424.325581395347 | 13.457906878271768 | 3428.8837209302324         |
| 8053475328 | 14784.523809523811 | 11.501666590216606 | 2932.0238095238096         |
| 7007744171 | 12264.805555555551 | 8.7411110997200048 | 2666.4444444444453         |
| 1503960366 | 12179.367346938774 | 7.8906122351787529 | 1845.6530612244896         |
| 2022484408 | 11595.093023255815 | 8.2767441716305044 | 2500.302325581396          |
| 6962181067 | 10679.888888888887 | 7.2273333549499528 | 2015.3777777777782         |
| 3977333714 | 10321.523809523807 | 7.0283332793485576 | 1480.6428571428571         |
| 2347167796 | 9948.59375         | 6.6271875062375356 | 2084.46875                 |
| 6117666160 | 9025.612903225805  | 6.833871010811098  | 2429.9354838709678         |
| 5577150313 | 8815.97435897436   | 6.5982050956823874 | 3421.8974358974351         |
| 4702921684 | 8747.3863636363621 | 7.0977272580970423 | 3005.3863636363644         |
| 7086361926 | 8661.2380952380954 | 5.88476188689293   | 2477.4285714285711         |
| 8378563200 | 8555.1627906976737 | 6.7846512073694276 | 3414.1395348837218         |
| 5553957443 | 8540.6279069767443 | 5.5890697975491364 | 1855.2558139534885         |
| 1644430081 | 7780.925           | 5.65875001549721   | 2837.5750000000003         |
| 4319703577 | 7599.54761904762   | 5.1126190388042989 | 2073.7857142857147         |
| 2873212765 | 7473.0476190476184 | 5.0423809375081738 | 1899.4047619047619         |
| 4558609924 | 7154.9302325581411 | 4.7297674459080365 | 1976.5813953488371         |
| 6290855005 | 6923.7692307692323 | 5.2357692443407506 | 2786.2307692307695         |
| 3372868164 | 6616.9333333333334 | 4.5433333555857338 | 1908.8333333333335         |
| 8583815059 | 6513.6315789473665 | 5.0815789103508    | 2732.1842105263163         |
| 8253242879 | 6326.666666666667  | 4.534583354989687  | 1849.2916666666667         |
| 4020332650 | 5206.8367346938758 | 3.7348979592171254 | 2952.0612244897966         |
| 1624580081 | 5167.2             | 3.4710000014305114 | 1433.7799999999995         |
| 2026352035 | 4960.1395348837223 | 3.0776744173016652 | 1488.9767441860465         |
| 2320127002 | 4714.97435897436   | 3.1869230438501406 | 1706.102564102564          |
| 4445114986 | 4632.369565217391  | 3.1352173880390506 | 2160.6304347826085         |
| 6775888955 | 4621.72            | 3.3207999630272402 | 2461.5599999999995         |
| 1844505072 | 4416.3571428571431 | 2.9203571562788326 | 1771.0714285714284         |
| 4057192912 | 3606.6190476190468 | 2.6619047669782532 | 2007.7619047619046         |
| 8792009665 | 2932.161290322581  | 1.8767741897533987 | 2146.5161290322585         |
| 1927972279 | 1881.7241379310344 | 1.3034482720340119 | 2282.7586206896553         |
---

En este nuevo analisis si se observa mas claramente la correlacion, igualmente despues cuando pasemos los analisis a visualizacion en R con ggplot lo vamos a ver mas claramente. 

Lo siguiente que hice fue un analisis de correlacion entre pasos totales y calorias quemada utilizando la siguiente formula: 

```sql
SELECT 
  CORR(Promedio_Pasos, Promedio_Calorias_Quemadas) AS Correlacion
FROM (
  SELECT  
    Id,  
    AVG(TotalSteps) AS Promedio_Pasos,  
    AVG(Calories) AS Promedio_Calorias_Quemadas  
  FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`  
  WHERE TotalSteps > 0 AND TotalDistance > 0  -- Filtrar datos inválidos
  GROUP BY Id  
);
```
---

El resultado nos dio que existe una correlacion del tipo 0,386 lo cual indica que es una correlacion positiva pero de tipo debil-moderada. 

Lo siguiente que realice fue diferencia los tipos de usuarios segun el numero de pasos promedio que realizaban cada dia entre los niveles muy bajo, bajo, medio, alto o muy alto con el siguiente comando:

```sql
SELECT  
  Id,  
  AVG(TotalSteps) AS Promedio_Pasos,  
  CASE  
    WHEN AVG(TotalSteps) < 2000 THEN 'Muy Bajo'  
    WHEN AVG(TotalSteps) BETWEEN 2000 AND 4999 THEN 'Bajo'  
    WHEN AVG(TotalSteps) BETWEEN 5000 AND 7999 THEN 'Medio'  
    WHEN AVG(TotalSteps) BETWEEN 8000 AND 10999 THEN 'Alto'  
    ELSE 'Muy Alta'  
  END AS Nivel_de_Actividad  
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`  
GROUP BY Id  
ORDER BY Promedio_Pasos DESC;
```
---

Nos dio el siguiente resultado: 

*Tabla 3* 

| Id         | Promedio_Pasos     | Nivel_de_Actividad |
|------------|--------------------|--------------------|
| 8877689391 | 16424.325581395347 | Muy Alta           |
| 8053475328 | 14784.523809523811 | Muy Alta           |
| 7007744171 | 12264.805555555551 | Muy Alta           |
| 1503960366 | 12179.367346938774 | Muy Alta           |
| 2022484408 | 11595.093023255815 | Muy Alta           |
| 6962181067 | 10679.888888888887 | Alto               |
| 3977333714 | 10321.523809523807 | Alto               |
| 2347167796 | 9948.59375         | Alto               |
| 6117666160 | 9025.612903225805  | Alto               |
| 5577150313 | 8815.97435897436   | Alto               |
| 4702921684 | 8747.3863636363621 | Alto               |
| 7086361926 | 8661.2380952380954 | Alto               |
| 8378563200 | 8555.1627906976737 | Alto               |
| 5553957443 | 8540.6279069767443 | Alto               |
| 1644430081 | 7780.925           | Medio              |
| 4319703577 | 7599.54761904762   | Medio              |
| 2873212765 | 7473.0476190476184 | Medio              |
| 4558609924 | 7154.9302325581411 | Medio              |
| 6290855005 | 6923.7692307692323 | Medio              |
| 3372868164 | 6616.9333333333334 | Medio              |
| 8583815059 | 6513.6315789473665 | Medio              |
| 8253242879 | 6326.666666666667  | Medio              |
| 4020332650 | 5206.8367346938758 | Medio              |
| 1624580081 | 5167.2             | Medio              |
| 2026352035 | 4960.1395348837223 | Bajo               |
| 2320127002 | 4714.97435897436   | Bajo               |
| 4445114986 | 4632.369565217391  | Bajo               |
| 6775888955 | 4621.72            | Bajo               |
| 1844505072 | 4416.3571428571431 | Bajo               |
| 4057192912 | 3606.6190476190468 | Bajo               |
| 8792009665 | 2932.161290322581  | Bajo               |
| 1927972279 | 1881.7241379310344 | Muy Bajo           |

Para entender donde se situan los niveles de cada usuario calcule tambien la media y mediana de todos los usuarios con la siguiente formula (Aplique IA para generar la funcion ya que desconocia el comando necesario):

```sql
WITH Promedios AS (
  SELECT Id, AVG(TotalSteps) AS Promedio_Pasos
  FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`
  GROUP BY Id
)

SELECT 
  AVG(Promedio_Pasos) AS Promedio_General,
  APPROX_QUANTILES(Promedio_Pasos, 100)[OFFSET(50)] AS Mediana_General
FROM Promedios;
```
---

La misma nos dio que la *media* tiene un valor = 7783.55
y la *mediana* tiene un valor = 7473.04 

Con esto finalizamos los analisis del dataset `dailyActivity_combined` proseguiremos con el analisis de los otros dos dataset y finalmente correlacionaremos los resultados de los 3 datasets

---

*DATASET daily_intensities* 

Vamos a calcular el promedio de tiempo y segmentarlo luego segun el nivel de actividad en 4 categorias. 

```sql
SELECT
Id,
AVG(SedentaryMinutes) AS Muy_Sedentario,
AVG (LightlyActiveMinutes) AS Usual_Sedentario,
AVG(FairlyActiveMinutes) AS Algo_Activo,
AVG(VeryActiveMinutes) AS Muy_Activo,
FROM mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyIntensities 
GROUP BY Id
ORDER BY Muy_Sedentario
```
---

Mas alla de esta informacion el dataset solo por si mismo no brinda muchos mas elementos de analisis por lo que procederemos a analizar el siguiente dataset:

---

*DATASET hourlySteps_final* 

En primer lugar ordenamos el dataset por Id, y luego por fecha (Dia y Hora) a traves del siguiente comando:

```sql
SELECT *
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.hourlySteps_final`
ORDER BY id, activity_hour
```
---

Luego realizamos la siguiente consulta:

```sql
SELECT
  id,
  _0  AS `00:00`,
  _1  AS `01:00`,
  _2  AS `02:00`,
  _3  AS `03:00`,
  _4  AS `04:00`,
  _5  AS `05:00`,
  _6  AS `06:00`,
  _7  AS `07:00`,
  _8  AS `08:00`,
  _9  AS `09:00`,
  _10 AS `10:00`,
  _11 AS `11:00`,
  _12 AS `12:00`,
  _13 AS `13:00`,
  _14 AS `14:00`,
  _15 AS `15:00`,
  _16 AS `16:00`,
  _17 AS `17:00`,
  _18 AS `18:00`,
  _19 AS `19:00`,
  _20 AS `20:00`,
  _21 AS `21:00`,
  _22 AS `22:00`,
  _23 AS `23:00`,
  (
    _0 + _1 + _2 + _3 + _4 + _5 + _6 + _7 +
    _8 + _9 + _10 + _11 + _12 + _13 + _14 + _15 +
    _16 + _17 + _18 + _19 + _20 + _21 + _22 + _23
  ) / 24 AS promedio_total
FROM (
  SELECT *
  FROM (
    SELECT
      id,
      EXTRACT(HOUR FROM activity_hour) AS hour_of_day,
      step_total
    FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.hourlySteps_final`
  )
  PIVOT (
    AVG(step_total) FOR hour_of_day IN (0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
                                        12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23)
  )
)
ORDER BY id
```
---

Esta consulta en primer lugar selecciona el id, crea una columna por cada hora del dia y una ultima que es el promedio de pasos diarios de cada Id, ademas por cada Id calculamos la cantidad de pasos promedio de cada hora (a traves de EXTRACT FROM) por ultimo para que la tabla sea visualmente mas clara pivoteamos las filas de hora para transformarlas en columnas, pero eso para ver en formato markdown queda visualmente mal por lo que exclusivamente para este markdown hemos aplicado la siguiente consulta:

```sql
SELECT
  id,
  FORMAT_TIMESTAMP('%H:00', activity_hour) AS hora,
  AVG(step_total) AS promedio_pasos
FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.hourlySteps_final`
GROUP BY id, hora
ORDER BY id, hora
```

---

La consulta nos da el siguiente resultado:

*Tabla 4* 

| id         | hora  | promedio_pasos       |
|------------|-------|----------------------|
| 1503960366 | 00:00 | 131.62295081967213   |
| 1503960366 | 01:00 | 44.114754098360649   |
| 1503960366 | 02:00 | 34.622950819672127   |
| 1503960366 | 03:00 | 15.262295081967213   |
| 1503960366 | 04:00 | 2.0                  |
| 1503960366 | 05:00 | 2.0327868852459017   |
| 1503960366 | 06:00 | 8.7377049180327884   |
| 1503960366 | 07:00 | 30.83606557377049    |
| 1503960366 | 08:00 | 211.26229508196721   |
| 1503960366 | 09:00 | 1176.4918032786888   |
| 1503960366 | 10:00 | 645.80327868852453   |
| 1503960366 | 11:00 | 552.55737704918033   |
| 1503960366 | 12:00 | 621.7540983606558    |
| 1503960366 | 13:00 | 770.88524590163934   |
| 1503960366 | 14:00 | 852.11475409836066   |
| 1503960366 | 15:00 | 519.50819672131138   |
| 1503960366 | 16:00 | 578.11475409836055   |
| 1503960366 | 17:00 | 658.36065573770486   |
| 1503960366 | 18:00 | 1249.1639344262298   |
| 1503960366 | 19:00 | 1103.9672131147543   |
| 1503960366 | 20:00 | 1120.1803278688526   |
| 1503960366 | 21:00 | 1004.6499999999997   |
| 1503960366 | 22:00 | 623.38333333333333   |
| 1503960366 | 23:00 | 904.48333333333335   |

(Continua Dataset..) 

---

Por ultimo antes de arrancar a realizar correlaciones entre las tablas voy a eliminar los decimales indicando un maximo de dos decimales en cada una de las tablas con las que realizare los graficos, para eso aplicaremos la funcion ROUND, un ejemplo de la funcion aplicada con uno de los datasets trabajados seria el siguiente:

```sql
SELECT  
  ROUND(MIN(TotalSteps), 2) AS Min_Steps,  
  ROUND(MAX(TotalSteps), 2) AS Max_Steps,  
  ROUND(AVG(TotalSteps), 2) AS Avg_Steps,  
  ROUND(STDDEV(TotalSteps), 2) AS StdDev_Steps,  

  ROUND(MIN(TotalDistance), 2) AS Min_Distance,  
  ROUND(MAX(TotalDistance), 2) AS Max_Distance,  
  ROUND(AVG(TotalDistance), 2) AS Avg_Distance,  
  ROUND(STDDEV(TotalDistance), 2) AS StdDev_Distance  

FROM `mi-primer-proyecto-436821.Trabajo_Final_Coursera.dailyActivity_combined`;
```
--- 

La primer tabla actualizada quedaria asi:

*Tabla 1* 

| Min_Steps | Max_Steps | Avg_Steps | StdDev_Steps | Min_Distance | Max_Distance | Avg_Distance | StdDev_Distance |
|-----------|-----------|-----------|--------------|--------------|--------------|--------------|-----------------|
| 7.0       | 36019.0   | 8047.5    | 4870.41      | 0.01         | 28.03        | 5.75         | 3.79            |

---

Realizare el mismo trabajo con todas las tablas.

**Tablas Finales**

*Tabla 2* 

| Id         | Promedio_Pasos | Promedio_Distancia | Promedio_Calorias_Quemadas |
|------------|----------------|--------------------|----------------------------|
| 8877689391 | 16424.33       | 13.46              | 3428.88                    |
| 8053475328 | 14784.52       | 11.5               | 2932.02                    |
| 7007744171 | 12264.81       | 8.74               | 2666.44                    |
| 1503960366 | 12179.37       | 7.89               | 1845.65                    |
| 2022484408 | 11595.09       | 8.28               | 2500.3                     |
| 6962181067 | 10679.89       | 7.23               | 2015.38                    |
| 3977333714 | 10321.52       | 7.03               | 1480.64                    |
| 2347167796 | 9948.59        | 6.63               | 2084.47                    |
| 6117666160 | 9025.61        | 6.83               | 2429.94                    |
| 5577150313 | 8815.97        | 6.6                | 3421.9                     |
| 4702921684 | 8747.39        | 7.1                | 3005.39                    |
| 7086361926 | 8661.24        | 5.88               | 2477.43                    |
| 8378563200 | 8555.16        | 6.78               | 3414.14                    |
| 5553957443 | 8540.63        | 5.59               | 1855.26                    |
| 1644430081 | 7780.93        | 5.66               | 2837.58                    |
| 4319703577 | 7599.55        | 5.11               | 2073.79                    |
| 2873212765 | 7473.05        | 5.04               | 1899.4                     |
| 4558609924 | 7154.93        | 4.73               | 1976.58                    |
| 6290855005 | 6923.77        | 5.24               | 2786.23                    |
| 3372868164 | 6616.93        | 4.54               | 1908.83                    |
| 8583815059 | 6513.63        | 5.08               | 2732.18                    |
| 8253242879 | 6326.67        | 4.53               | 1849.29                    |
| 4020332650 | 5206.84        | 3.73               | 2952.06                    |
| 1624580081 | 5167.2         | 3.47               | 1433.78                    |
| 2026352035 | 4960.14        | 3.08               | 1488.98                    |
| 2320127002 | 4714.97        | 3.19               | 1706.1                     |
| 4445114986 | 4632.37        | 3.14               | 2160.63                    |
| 6775888955 | 4621.72        | 3.32               | 2461.56                    |
| 1844505072 | 4416.36        | 2.92               | 1771.07                    |
| 4057192912 | 3606.62        | 2.66               | 2007.76                    |
| 8792009665 | 2932.16        | 1.88               | 2146.52                    |
| 1927972279 | 1881.72        | 1.3                | 2282.76                    |

*Tabla 3*

| Id         | Promedio_Pasos | Nivel_de_Actividad |
|------------|----------------|--------------------|
| 8877689391 | 16424.33       | Muy Alto           |
| 8053475328 | 14784.52       | Muy Alto           |
| 7007744171 | 12264.81       | Muy Alto           |
| 1503960366 | 12179.37       | Muy Alto           |
| 2022484408 | 11595.09       | Muy Alto           |
| 6962181067 | 10679.89       | Alto               |
| 3977333714 | 10321.52       | Alto               |
| 2347167796 | 9948.59        | Alto               |
| 6117666160 | 9025.61        | Alto               |
| 5577150313 | 8815.97        | Alto               |
| 4702921684 | 8747.39        | Alto               |
| 7086361926 | 8661.24        | Alto               |
| 8378563200 | 8555.16        | Alto               |
| 5553957443 | 8540.63        | Alto               |
| 1644430081 | 7780.93        | Medio              |
| 4319703577 | 7599.55        | Medio              |
| 2873212765 | 7473.05        | Medio              |
| 4558609924 | 7154.93        | Medio              |
| 6290855005 | 6923.77        | Medio              |
| 3372868164 | 6616.93        | Medio              |
| 8583815059 | 6513.63        | Medio              |
| 8253242879 | 6326.67        | Medio              |
| 4020332650 | 5206.84        | Medio              |
| 1624580081 | 5167.2         | Medio              |
| 2026352035 | 4960.14        | Bajo               |
| 2320127002 | 4714.97        | Bajo               |
| 4445114986 | 4632.37        | Bajo               |
| 6775888955 | 4621.72        | Bajo               |
| 1844505072 | 4416.36        | Bajo               |
| 4057192912 | 3606.62        | Bajo               |
| 8792009665 | 2932.16        | Bajo               |
| 1927972279 | 1881.72        | Muy Bajo           |

*Tabla 4* 

| id         | hora  | promedio_pasos |
|------------|-------|----------------|
| 1503960366 | 00:00 | 131.62         |
| 1503960366 | 01:00 | 44.11          |
| 1503960366 | 02:00 | 34.62          |
| 1503960366 | 03:00 | 15.26          |
| 1503960366 | 04:00 | 2.0            |
| 1503960366 | 05:00 | 2.03           |
| 1503960366 | 06:00 | 8.74           |
| 1503960366 | 07:00 | 30.84          |
| 1503960366 | 08:00 | 211.26         |
| 1503960366 | 09:00 | 1176.49        |
| 1503960366 | 10:00 | 645.8          |
| 1503960366 | 11:00 | 552.56         |
| 1503960366 | 12:00 | 621.75         |
| 1503960366 | 13:00 | 770.89         |
| 1503960366 | 14:00 | 852.11         |
| 1503960366 | 15:00 | 519.51         |
| 1503960366 | 16:00 | 578.11         |
| 1503960366 | 17:00 | 658.36         |
| 1503960366 | 18:00 | 1249.16        |
| 1503960366 | 19:00 | 1103.97        |
| 1503960366 | 20:00 | 1120.18        |
| 1503960366 | 21:00 | 1004.65        |
| 1503960366 | 22:00 | 623.38         |
| 1503960366 | 23:00 | 904.48         |

(Continua Dataset) 

---

## ***Paso 8: Visualizacion de Datos***

### **Analisis de Correlación**

#### *ACLARACION* Todo el analisis teorico se encuentra en el informe en formato PDF que se encuentra adjunto en: 

Realizamos la primer visualizacion de las correlaciones planteadas en las preguntas de investigacion, para eso utilizaremos RStudio (Cloud Version): 

```r
ggplot(Tabla_2, aes(x = Promedio_Pasos, y = Promedio_Calorias_Quemadas)) +
  geom_point(color = "#F8766D", size = 3, alpha = 0.7) +
  geom_smooth(method = "lm", se = FALSE, color = "#00BFC4", linewidth = 1.2) +
  labs(
    title = "Pasos Promedio vs Calorías Quemadas",
    subtitle = paste("Coeficiente de correlación:", cor1),
    x = "Pasos Promedio por Día",
    y = "Calorías Promedio por Día"
  ) +
  theme_minimal(base_size = 11)

ggsave("grafico_1_pasos_calorias.png", width = 7, height = 5, dpi = 300, bg = "white")

``` 
--- 

Ademas calculamos las correlaciones de este grafico y los siguientes dos con la siguiente funcion:

```r
cor1 <- round(cor(Tabla_2$Promedio_Pasos, Tabla_2$Promedio_Calorias_Quemadas), 2)
cor2 <- round(cor(Tabla_2$Promedio_Pasos, Tabla_2$Promedio_Distancia), 2)
cor3 <- round(cor(Tabla_2$Promedio_Distancia, Tabla_2$Promedio_Calorias_Quemadas), 2)
```
--- 

Aplicando formulas del mismo orden obtenemos los siguientes graficos con sus respectivas visualizaciones:

**Grafico 1**
[![grafico-pasos-calorias.png](https://i.postimg.cc/RZ7xbWm6/grafico-pasos-calorias.png)](https://postimg.cc/qNvFNMz4)

**Grafico 2** 
[![Grafico-2-Pasos-Distancia.png](https://i.postimg.cc/3NBXtgXg/Grafico-2-Pasos-Distancia.png)](https://postimg.cc/q6N6R6ZR)

**Grafico 3** 
[![Grafico-3-Calorias-Distancia.png](https://i.postimg.cc/mrp2yh1t/Grafico-3-Calorias-Distancia.png)](https://postimg.cc/4KtgJf5g) 

---

Posteriormente realizamos un grafico de barra para representar los distintos nivel de actividad con la siguiente formula:

```r
library(ggplot2)

ggplot(Tabla_3, aes(x = Nivel_de_Actividad, fill = Nivel_de_Actividad)) +  #Fill lo utilizamos para rellenar automaticamente cada barra con un color en particular
  geom_bar() +
  labs(
    title = "Distribución de Niveles de Actividad",
    x = "Nivel de Actividad",
    y = "Cantidad de Usuarios"
  ) +
  theme_minimal(base_size = 14) +
  theme(legend.position = "none")

```
Obtenemos la siguiente distribucion: 

### **Graficos de Barra** 

[![Grafico-4-Barra-Actividad.png](https://i.postimg.cc/4yHqLDzT/Grafico-4-Barra-Actividad.png)](https://postimg.cc/sGyT106n) 

---

Como RStudio no cuenta con una funcion para generar graficos de torta (geom_pie) el mismo fue obtenido a traves del siguiente comando:

```r
library(ggplot2)
library(dplyr)

# Agrupamos por nivel de actividad y calculamos porcentaje
actividad_resumen <- Tabla_3 %>%
  group_by(Nivel_de_Actividad) %>%
  summarise(Cantidad = n()) %>%
  mutate(Porcentaje = round((Cantidad / sum(Cantidad)) * 100, 1),
         Etiqueta = paste0(Porcentaje, "%"))

# Gráfico de torta con porcentajes
ggplot(actividad_resumen, aes(x = "", y = Cantidad, fill = Nivel_de_Actividad)) +
  geom_bar(stat = "identity", width = 1) +
  coord_polar("y") +
  geom_text(aes(label = Etiqueta), 
            position = position_stack(vjust = 0.5), 
            color = "white", size = 5) +
  labs(
    title = "Distribución de Niveles de Actividad (Gráfico de Torta)",
    fill = "Nivel de Actividad"
  ) +
  theme_minimal(base_size = 14) +
  theme(
    axis.title = element_blank(),
    axis.text = element_blank(),
    axis.ticks = element_blank(),
    panel.grid = element_blank()
  )

```
Esta funcion convierte un grafico de barra en circular usando el eje "Y" como angulo de rotación obteniendo lo siguiente: 

### **Graficos de Torta** 

[![Grafico-5-Torta-Nivel-de-Actividad.png](https://i.postimg.cc/TPNvwq0h/Grafico-5-Torta-Nivel-de-Actividad.png)](https://postimg.cc/HJMhvMfC) 

---

### **Graficos de Linea** 

**Grafico 1: Lineal General**

[![Grafico-6-Lineal-Pasos-Por-Hora.png](https://i.postimg.cc/jSg8mzbh/Grafico-6-Lineal-Pasos-Por-Hora.png)](https://postimg.cc/LJPBZghq) 

---
Para la realizacion de los siguientes graficos aplicamos la funcion facet_wrap para poder realizar la comparativa de dos segmentos en una misma visualizacion a traves de la siguiente funcion: 

```r
library(tidyverse)

tabla_4_larga <- Tabla_4 %>%
  pivot_longer(
    cols = matches("^\\d{2}:\\d{2}$"),
    names_to = "Hora",
    values_to = "Pasos_Promedio"
  )

tabla_4_larga$Hora <- factor(
  tabla_4_larga$Hora,
  levels = formatC(0:23, width = 2, flag = "0") %>% paste0(":00")
)

tabla_4_larga <- tabla_4_larga %>%
  left_join(Tabla_3 %>% select(Id, Nivel_de_Actividad), by = c("id" = "Id"))

datos_alto <- tabla_4_larga %>%
  filter(Nivel_de_Actividad %in% c("Muy Alto", "Alto"))

ggplot(datos_alto, aes(x = Hora, y = Pasos_Promedio, group = id)) +
  geom_line(alpha = 0.3, color = "#00BFC4") +
  stat_summary(fun = mean, geom = "line", color = "#F8766D", size = 1.2, aes(group = 1)) +
  facet_wrap(~ Nivel_de_Actividad) +
  labs(
    title = "Promedio de Pasos por Hora - Niveles Muy Alto y Alto",
    x = "Hora",
    y = "Promedio de Pasos"
  ) +
  theme_minimal(base_size = 13) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

**Grafico 2: Lineal Segmento Alto y Muy Alto**

[![Grafico-7-Lineal-Segmentos-Pasos-Alto-y-Muy-Alto.png](https://i.postimg.cc/5yLSnR5X/Grafico-7-Lineal-Segmentos-Pasos-Alto-y-Muy-Alto.png)](https://postimg.cc/D4fGz5Mh) 

Una funcion muy parecida fue aplicada para generar el grafico de los segmentos medio y bajo.

**Grafico 3: Lineal Segmento Medio y Bajo** 

[![Grafico-8-Lineal-Segmentos-Pasos-Medio-Bajo.png](https://i.postimg.cc/R0Pj1NGW/Grafico-8-Lineal-Segmentos-Pasos-Medio-Bajo.png)](https://postimg.cc/sBh6sfkz) 


