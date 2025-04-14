# Antes de empezar

Los datos utilizados en este proyecto se pueden desgaragar de  siguiente enlace [link](https://drive.google.com/drive/folders/1hzKPipsvvtnzqzkHqaQQ9VLM45J4P6X7?usp=sharing)

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



#### Asignar sistema de coordenadas o proyección
Algunas de las capas utilizadas en este trabajo no contaban con un sistema de referencia espacial. Por ello, antes de iniciar las consultas, se les asignó uno, para posteriormente ser reproyectadas.

``` sql
create database energy;
create extension postgis;
create schema ej01;
```


