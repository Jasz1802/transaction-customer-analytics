# Prueba Técnica - Analista Junior IRIS (Jennyfer Arias Sánchez)

## Descripción

Este proyecto corresponde a la solución de la prueba técnica para el cargo de **Analista Junior**.

El objetivo fue realizar un proceso de **exploración, limpieza, estandarización e integración de datos** a partir de los archivos `transacciones.csv` y `clientes.json` utilizando **PySpark**, obteniendo un único conjunto de datos limpio para posteriormente construir un dashboard en **Power BI**.

---

# Tecnologías utilizadas

- Python 3.11
- PySpark 3.5.6
- Java JDK 17
- Hadoop (WinUtils para Windows)
- Power BI
- Git

---

# Estructura del proyecto

```
prueba_tecnica_analista_junior_iris/
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

# 1. ¿Qué se necesita para ejecutar el código?

Para ejecutar correctamente el proyecto es necesario contar con el siguiente entorno:

- Python 3.11
- Java JDK 17
- PySpark 3.5.6
- Hadoop (WinUtils) configurado para Windows
- Jupyter Notebook o Visual Studio Code con la extensión de Jupyter
- Las dependencias especificadas en el archivo `requirements.txt`

## Instalación

Crear el entorno virtual:

```bash
python -m venv .venv
```

Activar el entorno virtual (PowerShell):

```powershell
.\.venv\Scripts\Activate.ps1
```

Instalar las dependencias:

```bash
pip install -r requirements.txt
```

Finalmente, ejecutar el notebook ubicado en:

```
notebooks/limpieza_datos.ipynb
```

siguiendo el orden de las celdas.

---

# 2. ¿Qué se encontró?

Durante la exploración de los archivos **Transacciones** y **Clientes** se identificaron diferentes problemas de calidad de datos que requerían limpieza y estandarización antes de realizar la unión de la información.

## Archivo Transacciones

### Sucursal

Se encontraron nombres de ciudades escritos de diferentes formas (por ejemplo: **BOGOTA**, **Bogotá**, **bogota**), además de espacios al inicio y al final de algunos registros.

Se eliminaron los espacios y se unificó el formato utilizando mayúscula inicial y la acentuación correspondiente.

### Fecha

Se identificaron múltiples formatos de fecha, como:

- dd-mm-yyyy
- yyyy-mm-dd
- dd/mm/yyyy
- dd.mm.yyyy

entre otros.

Primero se unificó el separador y posteriormente todas las fechas se transformaron al formato estándar **YYYY-MM-DD**, facilitando su tratamiento como tipo fecha.

### Monto

Los valores presentaban distintos formatos según la moneda, incluyendo:

- símbolos ($)
- texto (COP)
- separadores de miles
- decimales diferentes entre COP y USD
- valores como **N/A** o **sin dato**

Se eliminaron los caracteres no numéricos y se estandarizó el formato para convertir la columna a un tipo numérico.

### Tasa de interés

Se encontraron valores nulos, porcentajes escritos con `%` y otros en formato decimal.

Se eliminaron los caracteres innecesarios y se normalizó el formato numérico.

### Moneda

Existían diferentes representaciones para una misma moneda (por ejemplo: **COP**, **cop**, **Pesos**, **$**, **USD**, **Dólares**).

Se unificaron todas las categorías en dos valores estándar:

- COP
- USD

### Estado

Se encontraron diferentes formas de representar el mismo estado, como:

- APROBADA
- aprobado
- pend
- En proceso
- rechazada

Se estandarizaron en tres únicos valores:

- Aprobada
- Pendiente
- Rechazada

### Canal

Se identificaron diferencias en mayúsculas, minúsculas, abreviaturas y nombres completos (por ejemplo: **web**, **WEB**, **ATM**, **Cajero automático**, **App móvil**).

Se normalizaron los nombres para mantener una única representación por canal.

### Tipo de producto

Un mismo producto aparecía con diferentes nombres o abreviaturas, como:

- CTA_AHORROS
- Cuenta Ahorros
- Ahorros
- Certificado de Depósito

Se realizó una estandarización para conservar un único nombre por tipo de producto.

---

## Archivo Clientes

### Ciudad

Se encontraron diferencias en mayúsculas, tildes y espacios adicionales.

Se eliminaron los espacios y se unificó el formato de escritura.

### Fecha de alta

Al igual que en transacciones, se identificaron múltiples formatos de fecha e incluso meses escritos con abreviaturas (ene, mar, ago, etc.).

Todas las fechas se transformaron al formato **YYYY-MM-DD**.

### Segmento

Existían diferencias únicamente en el uso de mayúsculas y minúsculas (premium, Premium, EMPRESARIAL, pyme).

Se estandarizaron los valores manteniendo un único formato.

### Tipo de documento

Se encontraron diferentes representaciones del mismo documento, como:

- CC
- cc
- C.C.
- NIT
- nit

Se unificaron en los valores:

- CC
- NIT

### Activo

La columna contenía diferentes formas de representar valores booleanos:

- SI
- NO
- true
- false
- 1
- 0

Todos los registros se transformaron al tipo de dato booleano (`true` y `false`).

### Contacto

La información de contacto se encontraba almacenada como una estructura con correo electrónico y teléfono.

Se separó en dos columnas independientes (`email` y `telefono`) y el número telefónico fue normalizado eliminando caracteres especiales como espacios, paréntesis y guiones.

---

# 3. ¿Qué se decidió hacer con las fechas distintas a 2024, duplicados, clientes no identificados, datos faltantes y valores negativos?

## Fechas distintas a 2024

Se decidió conservar los registros con fechas diferentes a 2024, ya que hacen parte de la información original de la base de datos y no existía un requerimiento que indicara eliminarlos.

Además, como el objetivo final era construir un dashboard para analizar el comportamiento del negocio, resulta más útil mantener el histórico de la información.

En caso de requerir un análisis para un año específico, como 2024, basta con utilizar un segmentador o filtro en el dashboard, sin necesidad de descartar datos durante el proceso de limpieza.

---

## Duplicados

Se revisó cuidadosamente la existencia de registros duplicados, teniendo en cuenta que un mismo cliente puede realizar varias transacciones en un mismo día o adquirir diferentes productos, por lo que no era correcto eliminar registros únicamente por compartir el mismo `id_cliente`.

Para identificar duplicados reales se verificó que todas las columnas del registro fueran exactamente iguales, prestando especial atención al `id_transaccion`, ya que este identifica de manera única cada operación.

En el caso del archivo **clientes.json**, sí se eliminaron los registros duplicados con el mismo `id_cliente`, debido a que este archivo almacena la información única de cada cliente y no debería contener más de un registro para un mismo identificador.

---

## Clientes no identificados

Se decidió conservar las transacciones cuyos clientes no se encontraban en la tabla de clientes.

El motivo fue evitar sesgar los análisis, ya que estas transacciones siguen representando movimientos financieros válidos y contienen información importante, como el `id_transaccion`, el monto, la fecha y el producto.

Al realizar el **LEFT JOIN**, la información del cliente simplemente permanece como **NULL**, permitiendo identificar posteriormente qué transacciones no tienen un cliente asociado.

---

## Datos faltantes

Se decidió conservar los valores faltantes como **NULL** en el archivo CSV limpio para no alterar la información original.

Posteriormente, durante la construcción del dashboard, se utilizaron las herramientas de **Power Query** para mejorar la visualización:

- "Desconocido" para la columna **segmento**.
- "Sin información" para las demás columnas de texto.
- Valores **Blank** para las columnas numéricas.

Se evitó reemplazarlos por **0**, ya que esto podría alterar cálculos como promedios, sumas o indicadores financieros.

---

## Valores negativos

Se decidió conservar los valores negativos, ya que representan información que puede ser válida dentro del contexto del negocio, como devoluciones, retiros, ajustes contables o saldos pendientes.

Eliminarlos o convertirlos en valores positivos podría ocultar el estado real de las finanzas y generar análisis incorrectos.

---

# 4. Uso de IA: dónde, por qué, para qué y qué se verificó

Hice uso de Inteligencia Artificial como herramienta de apoyo durante el desarrollo de la prueba técnica, principalmente para resolver dudas puntuales, evaluar diferentes alternativas de implementación y solucionar inconvenientes técnicos.

### Limpieza de fechas del archivo clientes.json

Utilicé IA para definir una estrategia de estandarización de las fechas, ya que este archivo presentaba múltiples formatos y meses escritos como texto (por ejemplo: ene, feb, mar, ago), lo que ocasionaba que algunas fechas se convirtieran en valores **NULL** al intentar transformarlas directamente.

Como una posible solución, la IA me propuso utilizar la configuración `spark.sql.legacy.timeParserPolicy = LEGACY`; sin embargo, decidí no implementarla, ya que únicamente ocultaba el problema de compatibilidad de los formatos y funcionaba como un parche.

En su lugar, opté por normalizar previamente las fechas y traducir las abreviaturas de los meses antes de convertirlas al tipo `date`, obteniendo una solución más robusta y controlada.

### Configuración del entorno de PySpark

También utilicé IA para resolver inconvenientes relacionados con la configuración del entorno de desarrollo.

Durante la prueba se presentaron problemas de compatibilidad entre la versión de Python instalada y algunas dependencias de PySpark, además de la configuración de Hadoop para la exportación de archivos.

Como alternativa, la IA me sugirió finalizar el proceso utilizando Pandas para generar el archivo CSV; sin embargo, decidí mantener el desarrollo en PySpark, ya que era la herramienta recomendada para la prueba técnica y consideré que mantener todo el flujo en una sola tecnología hacía el proceso más consistente.

### Resolución de errores

También utilicé IA para comprender el origen de algunos errores generados por PySpark, como:

- AnalysisException
- Py4JJavaError

Esto me permitió identificar la causa de los problemas y aplicar una solución adecuada.

### Documentación

Finalmente, utilicé IA como apoyo para redactar la documentación del proceso y las respuestas de este informe.

La información fue proporcionada por mí y posteriormente verifiqué que la redacción reflejara correctamente el trabajo realizado.

### Verificación

En todos los casos utilicé la IA como una herramienta de apoyo y no como un reemplazo del proceso de análisis.

Antes de implementar cualquier sugerencia, verifiqué que la solución resolviera el problema planteado, que los datos conservaran su consistencia y que las transformaciones produjeran el resultado esperado.

---

# Resultado final

Como resultado del proceso se obtuvo:

- Un notebook con todo el proceso de exploración, limpieza y transformación de datos utilizando PySpark.
- Un archivo CSV limpio (`union_transacciones_clientes_limpia.csv`) obtenido a partir de la integración de las tablas de transacciones y clientes.
- Un dashboard desarrollado en Power BI utilizando el conjunto de datos limpio como fuente de información.