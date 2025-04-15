## Objetivo 2
 
**Caracterizar los municipios con rezago energético en el consumo de energía del sector residencial a partir del porcentaje de viviendas sin luz eléctrica.** 

Previo a calcular los índices de ingreso per-cápita, se realizaron cambios a la columna de la población total de tipo string a numérico

``` sql
ALTER TABLE public.poptot_2020
ALTER COLUMN valor TYPE NUMERIC
USING valor::NUMERIC;
```
**a. Crear campo consumo percapita, cuartil, categoria percapita**
   ``` sql
alter table public.consumo_elect_sector_mun_2022 add column consumo_per_cap numeric;
ALTER TABLE public.consumo_elect_sector_mun_2022 ADD COLUMN cuartil_percap INTEGER;
ALTER TABLE consumo_elect_sector_mun_2022 ADD COLUMN cat_percap TEXT;
```

**b. Actualizar campo consumo percapita**
   ``` sql
UPDATE public.consumo_elect_sector_mun_2022 AS ce
SET consumo_per_cap = ce.res / pop.valor
FROM poptot_2020 AS pop
WHERE ce.cvegeomun = pop.cvegeo;

```
**c. Actualizar campo cuartil percapita**
``` sql
WITH cuartiles AS (
 SELECT cvegeomun, NTILE(4) OVER (ORDER BY consumo_per_cap) AS cuartil
 FROM public.consumo_elect_sector_mun_2022
)
UPDATE public.consumo_elect_sector_mun_2022 AS c
SET cuartil_percap = q.cuartil
FROM cuartiles q
WHERE c.cvegeomun = q.cvegeomun;
```

**d. Actualizar campo categoria percapita**
``` sql
UPDATE public.consumo_elect_sector_mun_2022
SET cat_percap = CASE
   WHEN cuartil_percap = 1 THEN 'bajo'
   WHEN cuartil_percap IN (2, 3) THEN 'medio'
   WHEN cuartil_percap = 4 THEN 'alto'
   ELSE NULL
END;
```
## 1. Municipios con alto consumo per capita y alto porcentaje de viviendas sin luz eléctrica (2%) 
**combinar registros donde el % de viviendas sin acceso a electrcidad sea mayor al 2% y el consumo per capita sea de categoria 'alto'**   
``` sql
create table resultados.mun_desigualdad as
SELECT c.*, a.v_sep_2020
FROM consumo_elect_sector_mun_2022 AS c
JOIN acceso_elect_mun_2020 AS a
ON c.cvegeomun = a.cvegeo
WHERE c.cat_percap = 'alto' AND a.v_sep_2020 > 2
ORDER BY a.v_sep_2020 DESC;
```
## 2. Porcentaje de viviendas sin acceso a la electricidad por localidad que pertenecen a municipios con alto consumo per çapita
``` sql
CREATE TABLE resultados.localidades_percapita AS
SELECT
 l.*,
 m.cat_percap
FROM public."loc_5_viv_sin_elect_2020." AS l
JOIN public.consumo_elect_sector_mun_2022 AS m
 ON l.cve_mun = m.cvegeomun
WHERE m.cat_percap = 'alto'
 AND l.vph_s_elec IS NOT NULL
 AND l.vph_s_elec > 0
ORDER BY l.vph_s_elec DESC;
```





<p align="center">
  <img src="img/c1.png" alt="Mapa C1" width="45%">
</p>



<p align="center">
  <img src="img/c1a.png" alt="c1a" width="30%" style="margin-right: 100%;">
  <img src="img/c1a1.png" alt="c1a1" width="55%">
</p>

