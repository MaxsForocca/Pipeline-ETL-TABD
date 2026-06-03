# Pipeline ETL para Integración de Datos en Data Warehouse 🚀

Este proyecto implementa un pipeline de extracción, transformación y carga (ETL) utilizando Python y la biblioteca **Pandas** para integrar fuentes de datos heterogéneas procedentes de dos canales de venta (Catálogo y Web) en un almacén de datos centralizado (**Data Warehouse**) estructurado bajo un modelo en estrella en **PostgreSQL**.

## 👥 Integrantes
* **MAXS SEBASTIAN JOAQUIN FOROCCA MAMANI**
* **VLADIMIR JAWARD HANCCO SONCCO**

---

## 🎯 Objetivos del Proyecto
* Diseñar e implementar un pipeline ETL robusto para la consolidación de transacciones comerciales distribuidas.
* Realizar un Análisis Exploratorio de Datos (EDA) exhaustivo para detectar anomalías estructurales y lógicas en archivos crudos.
* Aplicar técnicas quirúrgicas de curación de datos (reubicación de columnas desfasadas, unificación de filas truncadas por saltos de línea y homologación de formatos).
* Modelar y poblar una base de datos relacional PostgreSQL con un esquema en estrella (`DimProduct`, `DimCustomer`, `DimTime` y `FactOrders`).

---

## 📊 Arquitectura del Pipeline ETL

```text
[ Fuentes Crudas ] ──> [ Auditoría y Limpieza ] ──> [ Homologación ] ──> [ Modelo Estrella ] ──> [ PostgreSQL ]
 - Catalog_Orders       - Reubicación de columnas    - Tipos de datos     - DimProduct            (Colab Local)
 - Web_Orders           - Remoción de " e hilos      - Datetime parsed    - DimCustomer
 - Products             - Unificación por \n         - Cast Enteros       - FactOrders
```
## 🛠️ Diagnóstico y Curación de Calidad de Datos (Data Cleansing)
Durante la fase de auditoría estructural fila por fila, se detectaron y corrigieron anomalías críticas en las fuentes que impedían la carga directa en el SGBD:

1.  Desplazamiento de Columnas (ID 124 - Catalog): Presencia de comas y comillas huérfanas (," ,") que causaron un desfase consecutivo en los atributos (CATALOG, PCODE, QTY y custnum). Se solucionó mediante reubicación posicional dirigida.

2.  Truncamiento por Salto de Línea (ID 4158 - Catalog): Un salto de línea físico (\n) fragmentó la transacción en dos filas distintas de Pandas, rompiendo el número de factura (INV). Se unificaron las propiedades y se eliminó la fila huérfana.

3.  Inconsistencia en Tipos de Datos y Fechas: * Las fechas de Catálogos venían en formato M/Y/D con año a 2 dígitos (%m/%y/%d), mientras que Web utilizaba D/M/Y con año a 4 dígitos (%d/%m/%Y). Ambas se transformaron con éxito al tipo nativo datetime64[ns].

  *  Se estandarizaron los identificadores y cantidades (ID, INV, QTY) al formato de enteros de alta precisión compatible con nulos (Int64 de Pandas).

4.  Duplicidad Analítica en Proveedores (supplier): El maestro de productos presentaba múltiples variantes de nombres para un mismo proveedor (ej. Software America, Inc., Software America, Software America Incorporated). Fueron normalizados masivamente mediante expresiones regulares (Regex).

##🗄️ Diseño Físico del Data Warehouse (Modelo Estrella)
Una vez integrados y limpios los conjuntos de datos, el pipeline genera de forma automatizada las tablas relacionales con sus respectivas claves primarias, foráneas y restricciones de integridad:

*  DimProduct: Contiene los atributos consolidados del catálogo maestro de productos (PCODE, TYPE, DESCRIP, PRICE, COST, supplier).

*  DimCustomer: Normaliza los identificadores únicos y nombres/códigos de los clientes transaccionales.

*  FactOrders: Tabla de hechos que unifica los canales de venta guardando las métricas de negocio (QTY, ingresos estimados, márgenes) y asociando las dimensiones de análisis.

## 🚀 Instalación y Ejecución
Prerrequisitos
El entorno está preparado para ejecutarse de forma directa en Google Colab o de manera local con las siguientes librerías instaladas:

```Bash
pip install pandas numpy sqlalchemy psycopg2-binary matplotlib seaborn
```
**Inicialización de PostgreSQL en el Entorno**
El notebook aprovisiona automáticamente el clúster de base de datos e inicializa los roles de seguridad necesarios utilizando identificadores válidos para evitar conflictos con palabras reservadas de SQL:

```Bash
# Iniciar servicio local
!service postgresql start

# Crear usuario y base de datos con privilegios administrativos
!sudo -u postgres psql -c "CREATE USER user_205 WITH PASSWORD 'password';"
!sudo -u postgres psql -c "CREATE DATABASE db_205 OWNER user_205;"
!sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE db_205 TO user_205;"
```
**Ejecución del Pipeline**
1.  Sube los archivos origen (Catalog_Orders.txt, Web_Orders.txt y products.txt) al entorno de ejecución.

2.  Ejecuta secuencialmente las celdas del archivo tabd_205.ipynb.

3.  El script imprimirá los histogramas automatizados de distribución estadística y confirmará la inyección masiva de registros mediante el método .to_sql() de Pandas hacia PostgreSQL utilizando sentencias ejecutables formateadas de manera segura con text().

## 📈 Resultados e Impacto Analítico
*  0% Registros Huérfanos: La corrección óptica de códigos internos (como caracteres 'O' por ceros numéricos) garantizó un cruce de datos (merge) perfecto sin pérdida de integridad referencial.

*  Simetría Estructural: Las transacciones web y por catálogo ahora comparten el mismo esquema de tipos y formatos de tiempo, listas para alimentar herramientas BI (PowerBI, Tableau) o consultas OLAP directas en SQL.
