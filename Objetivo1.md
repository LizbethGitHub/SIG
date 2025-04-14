## Objetivo 1

Identificar zonas de alto y bajo consumo energético a nivel municipal, estatal y regional. 

#### 

``` sql
SELECT public.consumo_elect_sector_mun_2022.agr, public.consumo_elect_sector_mun_2022.ind
   (residencial + industrial + comercial) AS consumo_tot
FROM public.consumo_elect_sector_mun_2022;
UPDATE public.consumo_elect_sector_mun_2022
SET consumo_tot = agr + res + pub + com + ind + mind;
----------
UPDATE public.consumo_elect_sector_mun_2022
SET
 agr  = COALESCE(agr, 0),
 res  = COALESCE(res, 0),
 pub  = COALESCE(pub, 0),
 com  = COALESCE(com, 0),
 ind  = COALESCE(ind, 0),
 mind = COALESCE(mind, 0);
```
1. **¿Cuáles son los municipios que mayor energía eléctrica consumen a nivel nacional? El 1% de los municipios que más consumen** 
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

