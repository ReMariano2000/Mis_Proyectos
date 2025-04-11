
# 📊 Análisis de Datos de Fitbit con R - Bellabeat Case Study  

Este proyecto forma parte del Certificado Profesional de Análisis de Datos de Google. Se analizan datos recolectados por dispositivos Fitbit para generar insights aplicables al producto **Leaf** de Bellabeat, orientado al bienestar femenino.

---

## 📌 1. Business Task  

**Objetivo**:  
Analizar patrones de uso de dispositivos inteligentes (Fitbit) para entender los comportamientos de actividad física y ofrecer recomendaciones estratégicas que Bellabeat pueda aplicar a su producto Leaf.  

**Preguntas clave**:
- ¿Qué tendencias de uso de smart devices se observan?
- ¿Cómo aplican a los clientes de Bellabeat?
- ¿Cómo pueden guiar la estrategia de marketing de la empresa?

---

## 📂 2. Fuente de Datos  

Los datos provienen del siguiente dataset público de Kaggle:  
[📎 FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit)

Contiene datos de 30 usuarios durante abril 2016. Archivos utilizados:
- `dailySteps_merged.csv`
- `hourlySteps_merged.csv`
- `dailyCalories_merged.csv`
- `dailyIntensities_merged.csv`

---

## 🧹 3. Limpieza y Transformación  

Las tareas de limpieza se realizaron íntegramente en **RStudio** utilizando `tidyverse`.

### Funciones principales usadas:
```r
library(tidyverse)
library(lubridate)

# Conversión de fechas
daily$Date <- mdy(daily$ActivityDate)

# Agrupamiento y promedio por hora
hourly_avg <- hourly %>%
  mutate(Hour = hour(ActivityHour)) %>%
  group_by(Id, Hour) %>%
  summarise(PasosPromedio = mean(StepTotal))

# Clasificación de nivel de actividad
actividad_nivel <- daily %>%
  mutate(nivel = case_when(
    TotalSteps < 5000 ~ "Muy Bajo",
    TotalSteps < 7500 ~ "Bajo",
    TotalSteps < 10000 ~ "Medio",
    TRUE ~ "Alto"
  ))
```

---

## 📈 4. Análisis Exploratorio  

### Correlación entre pasos y calorías
```r
cor(daily$TotalSteps, daily$Calories)
```

### Patrón horario de actividad
```r
ggplot(hourly_avg, aes(x = Hour, y = PasosPromedio)) +
  geom_line() +
  labs(title = "Promedio de pasos por hora", x = "Hora", y = "Pasos")
```

### Comparativa de usuarios según su nivel de actividad
```r
ggplot(hourly_avg_segmented, aes(x = Hour, y = PasosPromedio)) +
  geom_line() +
  facet_wrap(~nivel)
```

---

## 📌 5. Conclusiones

- El pico de actividad física diaria se da entre las **18 y 20 hs**.
- Se observaron distintos patrones de uso según nivel de usuario.
- La relación entre pasos, calorías y minutos activos permite modelar el comportamiento del usuario.

---

## 💡 6. Recomendaciones

### Marketing
- Enviar notificaciones en horarios de mayor actividad.
- Campañas personalizadas según nivel de usuario.

### Producto
- Crear objetivos dinámicos de pasos.
- Agregar un sistema de recompensas para incentivar a usuarios inactivos.
- Visualizaciones claras para seguimiento personal.

---

## 🛠️ Herramientas

- **Lenguaje**: R
- **IDE**: RStudio
- **Librerías**: tidyverse, lubridate, ggplot2

---
