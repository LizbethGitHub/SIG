## Antes de empezar

Puedes descargar los datos desde [Google Drive](https://drive.google.com/file/d/1vdl6zHGmNvYA9OYH43GkE1GEpkIwDQte/view).

#### Generar extensión postgis, base de datos, esquema 

1. **Habilitación de PostGIS** 
   Para trabajar con datos espaciales en PostgreSQL, fue necesario establecer la conexión utilizando la extensión `PostGIS`.
2. **Creación de la base de datos**
   Se creó una base de datos a la cual se añadieron las capas de trabajo.
3. **Definición del esquema de resultados** 
   Se creó un esquema destinado a almacenar los resultados de las consultas.

``` sql
create database energia;
create extension postgis;
create schema resultados;
```
#### Añadir las capas y tablas en QGis
A través del software QGis, se adjuntaron las tablas y capas necesarias para rezalizar este proyecto


#### Asignar sistema de coordenadas o proyección
Algunas de las capas utilizadas en este trabajo no contaban con un sistema de referencia espacial. Por ello, antes de iniciar las consultas, se les asignó uno, para posteriormente ser reproyectadas.

``` sql
UPDATE public.regiones_cenace 
SET geom = ST_SetSRID(geom, 4326);
UPDATE public."loc_5_viv_sin_elect_2020."
SET geom = ST_SetSRID(geom, 4326);
```

#### Reproyectar las capas
Las capas utilizadas tienen una escala estatal, municipal y regional. Dado que era necesario realizar diversas mediciones, se optó por emplear una proyección cartográfica que proporcionara resultados en unidades métricas. En el caso de México, la proyección recomendada para trabajos a nivel nacional es la Cónica Conforme de Lambert, Datum ITRF2008, correspondiente al EPSG:6372.

Esta proyección es especialmente adecuada para representar países con gran extensión en el sentido este-oeste, como México, ya que mantiene la conformidad (preserva las formas locales de los objetos geográficos) y minimiza la distorsión angular.

``` sql
ALTER TABLE public.consumo_elect_sector_mun_2022
ALTER COLUMN geom
TYPE Geometry(MultiPolygon, 6372)
USING ST_Transform(geom, 6372);
```

#### Asignar claves estatales y municipales
A las tablas que no contaban con información de clave estatal y municipal se les añadió una nueva columna, la cual fue actualizada mediante una intersección espacial con otra capa que sí contenía dichos atributos. 

*Estatal*

``` sql
alter table public.plantas_bioenergia add column cve_ent varchar(254);
update public.plantas_bioenergia pe --tabla que se quiere actualizar
set cve_ent = cesm.cve_ent	--cve_ent recibe los valores de dc.cve_ent
from public.consumo_elect_sector_mun_2022 cesm   --tabla de la que se toman los datos
where ST_Intersects(pe.geom,cesm.geom);

```
*Municipal*

``` sql
alter table public.plantas_bioenergia add column cve_mun varchar(254);
update public.plantas_bioenergia pe --tabla que se quiere actualizar
set cve_mun = cesm.cvegeomun --cve_mun recibe los valores de dc.cve_mun
from public.consumo_elect_sector_mun_2022 cesm -- tabla de la que se toman los datos
where ST_Intersects(pe.geom,cesm.geom);

```

#### Asignación de PK
Debido a la naturaleza de los datos y a las distintas escalas de los insumos de entrada, se decidió no asignar claves primarias (PK) a estas capas. Esta decisión se tomó porque, durante algunos procesos de análisis, ciertos atributos quedaron con valores nulos, lo cual incumple con la regla base de una PK (no permitir valores nulos). Sin embargo, eliminar esos registros habría afectado los resultados del análisis, por lo que se optó por conservarlos.


























