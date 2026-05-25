# ConnectaTel — Análisis de Segmentación de Usuarios y Patrones de Uso
 
Proyecto de análisis exploratorio sobre el catálogo de planes de **ConnectaTel**, una empresa ficticia de telecomunicaciones que opera en México y Colombia. El objetivo es identificar segmentos de clientes por edad, patrones de uso y oportunidades para diferenciar la oferta comercial.
 
## Contexto del proyecto
 
ConnectaTel ofrece dos planes (Básico y Premium) y necesita entender si estos planes realmente están segmentando el comportamiento de sus usuarios o si los clientes se comportan igual sin importar el plan contratado.
 
Las preguntas de negocio a responder:
 
- ¿Qué segmentos de clientes muestran mayor o menor uso de llamadas y mensajes?
- ¿Qué usuarios presentan valores atípicos que puedan indicar comportamientos inusuales, fraude o errores de registro?
- ¿Cómo varía el uso según la edad y el tipo de plan contratado?
- ¿Qué patrones pueden ayudar a diseñar mejores planes, optimizar la oferta y mejorar la satisfacción del cliente?
## Hallazgos clave
 
- **No hay diferenciador real entre planes:** ambos concentran aproximadamente el 74% de uso medio, con distribuciones prácticamente iguales.
- **El único diferenciador por edad es la duración de llamada:** los adultos mayores tienden a sostener llamadas más largas (hasta 82 minutos totales al mes).
- **Cali concentra la mayor proporción de adultos mayores (31.8%) y de plan Premium**, consistente con su perfil de ciudad ideal para retiro.
- **Los heavy users (114 usuarios, 2.85% del total) son transversales geográficamente:** el rango entre la ciudad con más y menos heavy users es de apenas 13 usuarios, lo que descarta un patrón regional.
- **Problema de calidad de dato detectado:** 14.1% de usuarios sin ciudad registrada, validado como faltante aleatorio (MCAR).
## Stack utilizado
 
- **Lenguaje:** Python
- **Análisis y manipulación:** Pandas, NumPy
- **Visualización:** Matplotlib, Seaborn
- **Entorno:** Jupyter Notebook
## Estructura del repositorio
 
```
connectatel-user-segmentation/
├── README.md
├── notebooks/
│   ├── 01_exploracion_datasets.ipynb            # Identificación de problemas de calidad
│   ├── 02_limpieza_calculo_estadistico.ipynb    # Pipeline de limpieza + cálculos
│   └── 03_analisis.ipynb                        # Segmentación, hallazgos y conclusiones
├── data/
│   ├── plans.csv                # Datos crudos: planes ofrecidos
│   ├── users_latam.csv          # Datos crudos: usuarios
│   ├── usage.csv                # Datos crudos: uso de servicios
│   └── user_profile.csv         # Output del notebook 02 (entrada del notebook 03)
├── requirements.txt
└── .gitignore
```
 
## Cómo ejecutar el proyecto
 
1. Clona el repositorio:
   ```bash
   git clone https://github.com/RobertoBG1111/connectatel-user-segmentation.git
   cd connectatel-user-segmentation
   ```
 
2. Instala las dependencias:
   ```bash
   pip install -r requirements.txt
   ```
 
3. Ejecuta los notebooks **en orden**:
   - `01_exploracion_datasets.ipynb` → Exploración inicial e identificación de problemas
   - `02_limpieza_calculo_estadistico.ipynb` → Limpieza, agregación y cálculos estadísticos (genera `user_profile.csv`)
   - `03_analisis.ipynb` → Análisis, visualización y conclusiones
## Decisiones metodológicas
 
Decisiones técnicas que se tomaron durante el proyecto y su justificación:
 
### Tratamiento de valores faltantes
 
- **Edad (-999):** se identificaron 55 registros con el sentinel `-999`. Se imputaron con la mediana y se mantuvo un flag (`age_imputed`) para trazabilidad. Esto evita perder registros sin comprometer la calidad de los datos.
- **Ciudad (`?` + NaN):** se identificaron 565 registros (14.1%) sin ciudad válida (469 nulos originales + 96 con valor `'?'`). Se unificaron como categoría `'Desconocido'` en lugar de eliminarse, para mantener visibilidad en análisis agregados.
### Validación MAR del faltante en `length` y `duration`
 
El dataset `usage.csv` presenta valores faltantes en las columnas `length` (caracteres de mensaje) y `duration` (minutos de llamada). Al analizar el patrón mediante un `groupby` sobre la variable `type`, se confirmó que el faltante es **MAR** (Missing At Random):
 
- 99.93% de los registros con `type = 'call'` tienen `length` nulo
- 99.93% de los registros con `type = 'text'` tienen `duration` nulo
El faltante no es aleatorio sino determinado por la naturaleza del servicio: una llamada no tiene caracteres y un mensaje no tiene duración.
 
### Validación MCAR del faltante de ciudad
 
Antes de descartar el faltante geográfico como ruido aleatorio, se comparó la distribución del grupo `'Desconocido'` contra el resto del dataset por plan y grupo de edad. Las diferencias máximas fueron de **0.3 puntos porcentuales**, confirmando que el faltante es **MCAR** (Missing Completely At Random). Esto implica que las conclusiones sobre planes, uso y demografía no están sesgadas, aunque sí se recomienda revisar el flujo de captura en el negocio.
 
### Winsorización al percentil 99
 
Los valores extremos en `cant_llamadas`, `cant_mensajes`, `cant_min_llamada` y `cant_caracteres` se acotaron al percentil 99. Se eligió este método sobre el IQR clásico porque el IQR tradicional resultaba demasiado restrictivo dado el alto sesgo a la derecha de las distribuciones, descartando valores que sí eran realistas dentro de los límites de los planes.
 
**Validación:** se cuantificó el impacto de la winsorización por columna, confirmando que solo se modificó entre **0.75% y 1.90%** de los registros, dependiendo de la columna. Esto valida que el ajuste no distorsiona los patrones de uso del usuario típico.
 
### Segmentación de usuarios
 
- **Por edad:** Joven (18-30), Adulto (31-60), Adulto Mayor (61+)
- **Por nivel de uso:**
  - *Bajo uso:* menos de 5 llamadas y mensajes
  - *Medio uso:* entre 5 y 10 en alguno de los servicios
  - *Alto uso:* 10 o más en cualquiera de los servicios
## Limitaciones del análisis
 
- El análisis se realizó sobre datos de un único año (2024), por lo que no se pueden inferir tendencias temporales ni estacionalidad.
- 14.1% de usuarios sin ciudad registrada limitan el análisis geográfico al subconjunto de 3,434 usuarios con ubicación conocida.
- No se cuenta con datos de satisfacción, churn rate ni información de la competencia, por lo que las recomendaciones sobre retención requerirían información adicional fuera del alcance del proyecto.
- La clasificación de "heavy user" se definió a partir de los umbrales del proyecto (≥10 llamadas o ≥12 mensajes), pero no necesariamente refleja el criterio interno de ConnectaTel.
## Recomendaciones al negocio
 
1. **Redefinir el diferenciador entre planes Básico y Premium**, ya que el comportamiento de uso es prácticamente idéntico entre ambos.
2. **Diseñar planes adaptados al contexto regional y de segmento de edad**. Por ejemplo, un plan orientado a adultos mayores en Cali con menor cantidad de llamadas permitidas pero sin restricción en duración por llamada individual.
3. **Revisar el flujo de captura de datos de ciudad**, ya que el 14.1% sin registro geográfico representa una pérdida de visibilidad operativa relevante.
## Sobre este proyecto
 
Proyecto desarrollado como parte del Sprint 7 del **Bootcamp de Data Analytics de TripleTen**. Las conclusiones y recomendaciones presentadas son ejercicios académicos basados en un dataset simulado y no representan análisis sobre una empresa real.
 
---
 
**Autor:** Roberto Barrera García  
**Contacto:** [LinkedIn](https://www.linkedin.com/in/robertobarreragarcia/) · robertobg1111@outlook.com
 

















