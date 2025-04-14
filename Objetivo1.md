## Objetivo 1
**Identificar zonas de alto y bajo consumo energético a nivel municipal, estatal y regional.**
  
Previo a realizar la primer consulta en donde se solicita el consumo energético por municipio. Se realizó una suma de los diferentes tipos de consumo. Esto porque la capa tiene el consumo por sector, que en este caso son seis: agrícola(agr), residencial(res), público(pub), comercial(com), industrial (ind) e industria mediana (mind)

Previo a realizar la primera consulta, en la cual se solicita el consumo energético por municipio, se llevó a cabo una suma de los distintos tipos de aprovechamiento energético. Esta operación fue necesaria debido a que la capa original contiene el consumo desagregado por sector, los cuales son seis: agrícola (agr), residencial (res), público (pub), comercial (com), industrial (ind) e industria mediana (mind).

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
   
``` sql
CREATE TABLE resultados.consumo_region_resumen AS
SELECT nom_mun, agr, res, pub, com, ind, mind, region_con, consumo_tot, consumo_per_cap, geom
FROM public.consumo_elect_sector_mun_2022
ORDER BY consumo_tot DESC
LIMIT 25;
```
*Resultado consulta*
![C1](img/c1.png)










 
2. **Creación de la base de datos**
   Se creó una base de datos a la cual se añadieron las capas de trabajo.
3. **Definición del esquema de resultados** 
   Se creó un esquema destinado a almacenar los resultados de las consultas.

``` sql

```

