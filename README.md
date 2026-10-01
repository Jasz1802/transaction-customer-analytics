# Data Cleaning & Integration with PySpark

## Descripción

Proyecto de **exploración, limpieza, estandarización e integración de datos** desarrollado con **PySpark**, a partir de información de transacciones y clientes almacenada en diferentes formatos.

El proyecto aborda un proceso completo de preparación de datos, desde la identificación de problemas de calidad hasta la generación de un conjunto de datos limpio y estructurado para su posterior análisis y visualización en **Power BI**.

El flujo incluye transformación de fechas, normalización de categorías, tratamiento de valores faltantes, validación de duplicados, transformación de tipos de datos y unión de diferentes fuentes de información.

---

## Tecnologías utilizadas

* Python 3.11
* PySpark 3.5.6
* Java JDK 17
* Hadoop / WinUtils
* Power BI
* Jupyter Notebook
* Git

---

## Estructura del proyecto

```text
data_cleaning_integration/

│
├── data/
│   ├── clientes.json
│   └── transacciones.csv
│
├── dashboard/
│   └── dashboard.pbix
│
├── notebooks/
│   └── limpieza_datos.ipynb
│
├── output/
│   └── union_transacciones_clientes_limpia.csv
│
├── README.md
├── requirements.txt
└── .gitignore
```

---

# 1. Entorno y ejecución

Para ejecutar el proyecto se requiere:

* Python 3.11
* Java JDK 17
* PySpark 3.5.6
* Hadoop / WinUtils configurado para Windows
* Jupyter Notebook o Visual Studio Code con extensión de Jupyter
* Dependencias especificadas en `requirements.txt`

### Instalación

Crear el entorno virtual:

```bash
python -m venv .venv
```

Activar el entorno virtual en PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Instalar las dependencias:

```bash
pip install -r requirements.txt
```

Ejecutar el notebook:

```text
notebooks/limpieza_datos.ipynb
```

Las celdas deben ejecutarse en orden para reproducir el proceso completo de transformación.

---

# 2. Exploración y calidad de los datos

Durante la exploración de las fuentes se identificaron diferentes problemas de calidad que requerían tratamiento antes de integrar la información.

## Transacciones

### Sucursal

Se encontraron nombres de ciudades escritos con diferentes combinaciones de mayúsculas, minúsculas, tildes y espacios.

Se realizó una normalización del texto para conservar una única representación de cada ciudad.

### Fecha

Se identificaron múltiples formatos de fecha, incluyendo:

* `dd-mm-yyyy`
* `yyyy-mm-dd`
* `dd/mm/yyyy`
* `dd.mm.yyyy`

Las fechas fueron normalizadas y convertidas al formato estándar:

```text
YYYY-MM-DD
```

permitiendo su posterior tratamiento como tipo `date`.

### Monto

Los valores presentaban diferentes representaciones dependiendo de la moneda, incluyendo:

* Símbolos monetarios
* Texto como `COP`
* Separadores de miles
* Diferentes formatos decimales
* Valores como `N/A` o `sin dato`

Se realizó una limpieza de caracteres y una transformación a formato numérico.

### Tasa de interés

Se encontraron valores nulos, porcentajes representados con `%` y valores expresados directamente en formato decimal.

Se normalizó la información para obtener una representación numérica consistente.

### Moneda

Se identificaron diferentes representaciones para una misma moneda, por ejemplo:

* `COP`
* `cop`
* `Pesos`
* `$`
* `USD`
* `Dólares`

Los valores fueron estandarizados en las categorías:

* `COP`
* `USD`

### Estado

Se encontraron diferentes formas de representar los estados de las transacciones, como:

* `APROBADA`
* `aprobado`
* `pend`
* `En proceso`
* `rechazada`

Se normalizaron en tres categorías:

* `Aprobada`
* `Pendiente`
* `Rechazada`

### Canal

Se identificaron diferencias entre mayúsculas, minúsculas, abreviaturas y nombres completos, por ejemplo:

* `web`
* `WEB`
* `ATM`
* `Cajero automático`
* `App móvil`

Se realizó una estandarización para mantener una representación única por canal.

### Tipo de producto

Un mismo producto aparecía con diferentes nombres o abreviaturas, por ejemplo:

* `CTA_AHORROS`
* `Cuenta Ahorros`
* `Ahorros`
* `Certificado de Depósito`

Se normalizaron las categorías para conservar una única representación por tipo de producto.

---

## Clientes

### Ciudad

Se encontraron diferencias en mayúsculas, tildes y espacios adicionales.

Se realizó una limpieza y normalización de los valores.

### Fecha de alta

Se identificaron múltiples formatos de fecha, incluyendo meses representados mediante abreviaturas como:

* `ene`
* `mar`
* `ago`

Las fechas fueron normalizadas y convertidas al formato:

```text
YYYY-MM-DD
```

### Segmento

Se encontraron diferencias únicamente en el uso de mayúsculas y minúsculas, por ejemplo:

* `premium`
* `Premium`
* `EMPRESARIAL`
* `pyme`

Los valores fueron estandarizados para mantener una representación consistente.

### Tipo de documento

Se encontraron diferentes representaciones para los documentos, como:

* `CC`
* `cc`
* `C.C.`
* `NIT`
* `nit`

Se unificaron en:

* `CC`
* `NIT`

### Activo

La columna contenía diferentes representaciones de valores booleanos:

* `SI`
* `NO`
* `true`
* `false`
* `1`
* `0`

Los registros fueron transformados al tipo booleano:

```text
true / false
```

### Información de contacto

La información de contacto se encontraba almacenada como una estructura que contenía correo electrónico y teléfono.

Se separó en dos columnas independientes:

* `email`
* `telefono`

El número telefónico fue normalizado eliminando caracteres especiales como espacios, paréntesis y guiones.

---

# 3. Criterios de tratamiento de los datos

## Fechas fuera de 2024

Se conservaron los registros con fechas diferentes a 2024 debido a que hacen parte de la información disponible y no existía un criterio de negocio que justificara su eliminación.

Mantener el histórico permite realizar análisis sobre diferentes periodos y aplicar posteriormente filtros específicos desde la capa de visualización.

---

## Duplicados

Se realizó una validación de registros duplicados considerando la estructura de cada fuente.

En las transacciones no se eliminaron registros únicamente por compartir un mismo `id_cliente`, ya que un cliente puede realizar múltiples operaciones.

Para identificar duplicados reales se consideró la totalidad del registro y, especialmente, el `id_transaccion` como identificador de cada operación.

En la información de clientes se eliminaron registros duplicados asociados al mismo `id_cliente`, debido a que este identificador representa de manera única a cada cliente.

---

## Clientes no identificados

Se conservaron las transacciones cuyo `id_cliente` no tenía correspondencia en la información de clientes.

La integración se realizó mediante un `LEFT JOIN`, permitiendo conservar las transacciones y representar como `NULL` la información del cliente que no pudo ser asociada.

De esta forma, los registros no identificados pueden ser analizados posteriormente como parte de la calidad de los datos.

---

## Datos faltantes

Los valores faltantes se conservaron como `NULL` durante el proceso de transformación para evitar modificar artificialmente la información original.

En la etapa de visualización se utilizaron representaciones más amigables para el usuario, por ejemplo:

* `Desconocido` para segmentos sin información.
* `Sin información` para otros campos de texto.
* `Blank` para valores numéricos sin información.

Se evitó reemplazar valores faltantes por `0`, ya que hacerlo podría afectar cálculos como promedios, sumas e indicadores financieros.

---

## Valores negativos

Los valores negativos fueron conservados debido a que pueden representar operaciones o situaciones válidas dentro de un contexto financiero, como devoluciones, ajustes, retiros o movimientos contables.

Por esta razón, no fueron transformados automáticamente a valores positivos ni eliminados durante la limpieza.

---

# 4. Integración de las fuentes

Una vez finalizada la limpieza y estandarización, se integraron las fuentes de transacciones y clientes mediante el identificador:

```text
id_cliente
```

El resultado fue un conjunto de datos consolidado que combina la información de las transacciones con los atributos disponibles de cada cliente.

El proceso permite mantener las transacciones sin correspondencia en la fuente de clientes, conservando la trazabilidad de los registros originales.

---

# 5. Visualización

A partir del conjunto de datos limpio se desarrolló un dashboard en **Power BI**.

El dashboard permite explorar la información mediante diferentes dimensiones y métricas, facilitando el análisis de:

* Comportamiento de las transacciones
* Montos
* Estados
* Canales
* Productos
* Monedas
* Segmentos de clientes
* Distribución temporal
* Calidad y disponibilidad de la información

El archivo `.pbix` se encuentra disponible dentro de la carpeta:

```text
dashboard/
```

---

# 6. Resultado final

Como resultado del proyecto se obtuvo:

* Un notebook desarrollado en **PySpark** con el proceso de exploración, limpieza y transformación.
* Un conjunto de datos consolidado a partir de las fuentes de clientes y transacciones.
* Un archivo CSV limpio y estructurado:

```text
union_transacciones_clientes_limpia.csv
```

* Un dashboard desarrollado en **Power BI** para la exploración y análisis de los datos.

El proyecto representa un flujo completo de **Data Cleaning → Data Transformation → Data Integration → Data Visualization**, utilizando herramientas orientadas al procesamiento y análisis de datos.
