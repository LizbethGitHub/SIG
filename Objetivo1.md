## Objetivo 1

### Identificar zonas de alto y bajo consumo energético a nivel municipal, estatal y regional.

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
#### 1. ¿Cuáles son los municipios que tienen mayor consumo de energía eléctrica a nivel nacional?
   
``` sql
CREATE TABLE resultados.consumo_region_resumen AS
SELECT nom_mun, agr, res, pub, com, ind, mind, region_con, consumo_tot, consumo_per_cap, geom
FROM public.consumo_elect_sector_mun_2022
ORDER BY consumo_tot DESC
LIMIT 25;
```
<p align="center">
  <img src="mapas/Municipios con mayor consumo de energia electrica en MWh 2022.jpg" alt="Mapa C1" width="70%">
</p>


#### 2. ¿Cuál es la capacidad de generación por estado?
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

#### 3. ¿Cuál es el estado que más consume energía?
``` sql
WITH resumen AS (
 SELECT
   cve_ent,
   SUM(consumo_tot) AS consumo_tot_ent,
   SUM(agr) AS agr_ent,
   SUM(ind) AS ind_ent,
   SUM(pub) AS pub_ent,
   SUM(com) AS com_ent,
   SUM(mind) AS mind_ent,
   SUM(res) AS res_ent
 FROM consumo_elect_sector_mun_2022
 GROUP BY cve_ent
)
UPDATE entidades AS e
SET
 consumo_tot_ent = r.consumo_tot_ent,
 agr_ent = r.agr_ent,
 ind_ent = r.ind_ent,
 pub_ent = r.pub_ent,
 com_ent = r.com_ent,
 res_ent = r.res_ent,
 mind_ent = r.mind_ent
FROM resumen AS r
WHERE e.cve_ent = r.cve_ent;
```
<p align="center">
  <img src="mapas/Entidades con mayor consumo de energía eléctica en 2022.jpg" alt="Mapa C1" width="70%">
</p>

#### 4. ¿Entidad con mayor consumo?
``` sql
CREATE TABLE resultados.prod_energia_sector AS
SELECT nomgeo,
      SUM(consumo_tot_ent) AS cons_tot_ent
FROM public.entidades e
GROUP BY nomgeo
ORDER BY cons_tot_ent DESC;
```

#### 4. ¿Cuál es el consumo total por sector de cada región CENACE?
``` sql
CREATE TABLE resultados.consumo_region_resumen AS
SELECT region_con,
 SUM(agr) AS agricultura,
 SUM(res) AS residencial,
 SUM(pub) AS alu_publico,
 SUM(com) AS comercio,
 SUM(ind) AS gran_ind,
 SUM(mind) AS med_ind,
 SUM(agr + res + pub + com + ind + mind) AS consumo_total
FROM public.consumo_elect_sector_mun_2022
GROUP BY region_con
ORDER BY region_con;
```

<p align="center">
  <img src="mapas/Regiones del Sistema Eléctrico Nacional con mayor consumo en 2022.jpg" alt="Mapa C1" width="70%">
</p>

#### 5. Porcentaje máximo por sector en cada región
``` sql
CREATE TABLE resultados.porcentajes_sector_region AS
SELECT
region_con,
ROUND(SUM(agr) * 100.0 / SUM(consumo_tot), 2) AS porc_agr,
ROUND(SUM(res) * 100.0 / SUM(consumo_tot), 2) AS porc_res,
ROUND(SUM(pub) * 100.0 / SUM(consumo_tot), 2) AS porc_pub,
ROUND(SUM(com) * 100.0 / SUM(consumo_tot), 2) AS porc_com,
ROUND(SUM(ind) * 100.0 / SUM(consumo_tot), 2) AS porc_ind,
ROUND(SUM(mind) * 100.0 / SUM(consumo_tot), 2) AS porc_mind
FROM public.consumo_elect_sector_mun_2022
GROUP BY region_con
ORDER BY region_con;
```
#### 6. ¿Cuál es la región CENACE que tiene los municipios con mayor consumo de energía eléctrica?
``` sql
SELECT region_con, COUNT(*) AS municipios_top10
FROM (
   SELECT nom_mun, region_con, consumo_tot
   FROM consumo_elect_sector_mun_2022
   ORDER BY consumo_tot desc
   limit 10
) AS top10
GROUP BY region_con
ORDER BY municipios_top10 DESC
LIMIT 10;
```
#### 7. ¿Qué tipo de tecnología es la principal a nivel nacional?
``` sql
SELECT tecno_simp, COUNT(*) AS total_plantas
   FROM public.plantas_generadoras_tecnologia
   GROUP BY tecno_simp
   ORDER BY COUNT(*) desc
```

#### 8. Tipo de tecnología principal
``` sql
create table resultados.plantas_tecnologia as
SELECT *
FROM public.plantas_generadoras_tecnologia
WHERE tecno_simp = (
   SELECT tecno_simp
   FROM public.plantas_generadoras_tecnologia
   GROUP BY tecno_simp
   ORDER BY COUNT(*) desc
   limit 1);
```
#### 9. ¿Qué tipo de tecnología es la principal en la region CENACE de mayor consumo nacional?
``` sql
SELECT tecno_simp, COUNT(*) AS total_plantas
FROM public.plantas_generadoras_tecnologia
where region_con = 'NOROESTE'
GROUP BY tecno_simp
ORDER BY total_plantas DESC
LIMIT 10;
```
#### 10. ¿Cuántas son las centrales del sector público y privado a nivel nacional?
``` sql
create table resultados.plantas_tecnologia_privado as
select central , empresa , tecno_simp , region_con, geom
FROM public.plantas_generadoras_tecnologia
where sector = 'Privado';

create table resultados.plantas_tecnologia_publico as
select central , empresa , tecno_simp , region_con, geom
FROM public.plantas_generadoras_tecnologia
where sector = 'Publico';
```
<p align="center">
  <img src="mapas/Tipo de propiedad de plantas generadoras de energía eléctrica.jpg" alt="Mapa C1" width="70%">
</p>


#### 11. ¿Qué estado tiene mas plantas del sector privado?
``` sql
SELECT cve_ent, entidad, matriz, COUNT(*) AS total_plantas_privadas
FROM public.plantas_generadoras_tecnologia
WHERE sector = 'Publico'
GROUP BY cve_ent, entidad, matriz
ORDER BY total_plantas_privadas DESC
LIMIT 10;
```

#### 12. ¿Obtener los 10 municipios de mayor consumo por cada region CENACE?
``` sql
create table resultados.mun_cenace as
WITH consumun_reg AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY region_con ORDER BY consumo_tot DESC) AS pos
  FROM public.consumo_elect_sector_mun_2022
)
SELECT *
FROM consumun_reg
WHERE pos <= 10;
```
#### 13. ¿Regiones CENACE con más subestaciones?
``` sql
SELECT rc.region_con,
  COALESCE(COUNT(DISTINCT se.id), 0) AS total_sub
FROM public.regiones_cenace rc
LEFT JOIN public.subestaciones se
ON ST_Intersects(se.geom, rc.geom)
GROUP BY rc.region_con
ORDER BY total_sub desc
limit 1;
```

#### 14. ¿Regiones CENACE con más centrales eléctricas?
``` sql
SELECT rc.region_con,
  COALESCE(COUNT(DISTINCT ce.id), 0) AS total_cent
FROM public.regiones_cenace rc
LEFT JOIN public.cent_elec ce
ON ST_Intersects(ce.geom, rc.geom)
GROUP BY rc.region_con
ORDER BY total_cent desc
limit 1;
```


