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
<p align="center">
  <img src="img/c1.png" alt="Mapa C1" width="600">
</p>


** a. ¿Cuál es la capacidad de generación por estado?**

``` sql
CREATE TABLE resultados.prod_energia AS
SELECT
   e.cve_ent,
   e.nomgeo AS nombre_entidad,
   SUM(pe.gener_gwh) AS prod_energia_edo,
   e.geom
FROM
   public.entidades e
JOIN
   public.plantas_generadoras_tecnologia pe
ON
   ST_Intersects(pe.geom, e.geom)
GROUP BY
   e.cve_ent, e.nomgeo, e.geom
ORDER BY
   prod_energia_edo DESC;
```








 


``` sql

```

