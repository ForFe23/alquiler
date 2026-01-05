
# 👓 Proceso de Minería de Datos en el Sector Óptico  
## Caso de Estudio: **Veoptics**

---

## 📌 Descripción del Proyecto

El proyecto **“Proceso de minería de datos en el sector óptico: Caso Veoptics”** consiste en el diseño, depuración y preparación de una **base de datos clínica óptica** orientada al **análisis de la pérdida de visión en jóvenes** durante el período **2010–2025**.

La base de datos ha sido creada con fines académicos y analíticos, permitiendo aplicar técnicas de **minería de datos, análisis estadístico y modelado predictivo**, con énfasis en:

- Evolución de ametropías  
- Agudeza visual  
- Exposición a pantallas  
- Frecuencia de consultas oftálmicas  

> ⚠️ **Aviso**  
> Todos los datos son **simulados** y **no corresponden a personas reales**.  
> Este proyecto es exclusivamente **educativo y de investigación**.

---

## 🎯 Objetivos del Proyecto

- 📊 Determinar el **porcentaje de jóvenes con pérdida de visión** entre 2010 y 2025.  
- 📈 Analizar la **progresión y severidad** de la pérdida visual en población joven.  
- 🔮 **Predecir tendencias** de pérdida de visión para los próximos **25 años** mediante técnicas de minería de datos.

---

## 🗂 Estructura de la Base de Datos

El esquema principal es **`clinica2`**, compuesto por las siguientes tablas:

| Tabla | Descripción |
|------|-------------|
| `sv_paciente` | Datos personales del paciente, motivo de consulta y fecha de registro |
| `sv_rol` | Roles del sistema (Administrador, Optometrista, Secretaria) |
| `sv_usuarios` | Usuarios del sistema vinculados a roles |
| `sv_optometrista` | Información profesional de optometristas |
| `sv_historia_clinica` | Historias clínicas con diagnóstico y métricas visuales |
| `stg_nombres_raw` | Datos crudos importados desde CSV |
| `stg_nombres_limpios` | Datos depurados listos para generación masiva |

---

## 🧹 Proceso de Limpieza de Datos

### 1️⃣ Importación de datos crudos

```sql
COPY stg_nombres_raw(nombre, apellido, cedula)
FROM '/ruta/al/csv/nombres.csv'
DELIMITER ',' CSV HEADER;
```

---

### 2️⃣ Limpieza de nombres y apellidos

```sql
UPDATE clinica2.stg_nombres_limpios
SET apellido = REGEXP_REPLACE(apellido, '[^A-Za-zÁÉÍÓÚÑáéíóúñ]+', '', 'g')
WHERE apellido IS NOT NULL;

UPDATE clinica2.stg_nombres_limpios
SET apellido = 'Apellido'
WHERE apellido IS NULL OR apellido = '';
```

---

### 3️⃣ Eliminación de duplicados

```sql
DELETE FROM stg_nombres_limpios a
USING stg_nombres_limpios b
WHERE a.ctid < b.ctid
  AND a.nombre = b.nombre
  AND a.apellido = b.apellido;
```

---

### 4️⃣ Normalización de correos

```sql
UPDATE sv_usuarios
SET tbl_correo = LOWER(tbl_nombre || '.' || tbl_apellido || id_usuario || '@veoptics.com');
```

---

## ⚡ Generación de Datos Masivos

### Roles del sistema

```sql
INSERT INTO sv_rol(nombre_rol)
VALUES ('Administrador'), ('Optometrista'), ('Secretaria')
ON CONFLICT DO NOTHING;
```

---

### Usuarios y Optometristas

- Generación automática de usuarios tipo **Optometrista**
- Asociación 1:1 usuario ↔ optometrista
- Datos únicos y consistentes

---

### Pacientes

- 📌 **2000 pacientes simulados**
- 👶👴 Edad entre **1 y 90 años**
- ⚥ Sexo: M / F
- 📅 Fechas de registro entre **2010–2025**
- 🏠 Direcciones y teléfonos simulados

---

### Historias Clínicas

- 🧾 Cada paciente tiene **al menos una historia clínica**
- 👓 Tipo de ametropía
- 🔢 Dioptrías reales (0–5)
- 👁 Agudeza visual
- 📱 Exposición a pantallas
- 📅 Fechas coherentes de consulta

---

## 🔄 Coherencia y Fiabilidad

- Relaciones referenciales consistentes:
  - `sv_historia_clinica.id_paciente → sv_paciente.id_paciente`
  - `sv_historia_clinica.id_optometrista → sv_optometrista.id_optometrista`

- Coherencia clínica:
  - Edad vs fecha de nacimiento
  - Dioptrías dentro de rangos realistas
  - Fechas entre 2010 y 2025

---

## 🧮 Variables para Análisis de Minería de Datos

| Variable | Tabla | Tipo |
|--------|------|------|
| Año del control | sv_historia_clinica.tbl_fecha | Fecha |
| Dioptrías | sv_historia_clinica.dioptrias | Numérico |
| Tipo de ametropía | sv_historia_clinica.tipo_ametropia | Categórico |
| Agudeza visual | sv_historia_clinica.agudeza_visual | Categórico |
| Edad | sv_paciente.edad | Numérico |
| Sexo | sv_paciente.tbl_sexo | Categórico |
| Exposición a pantallas | sv_historia_clinica.exposicion_pantallas | Categórico |
| Frecuencia de consultas | Derivada | Categórico |
| % pérdida visual | Derivada | Numérico |

---

## 📊 Consulta de Validación

```sql
SELECT p.id_paciente, p.tbl_nombre, p.tbl_apellido, p.edad, p.tbl_sexo,
       h.tbl_fecha, h.tipo_ametropia, h.dioptrias,
       h.agudeza_visual, h.exposicion_pantallas
FROM clinica2.sv_paciente p
LEFT JOIN clinica2.sv_historia_clinica h
ON p.id_paciente = h.id_paciente
LIMIT 10;
```

---

## 📈 Diagrama Conceptual (ER)

```
sv_rol ──< sv_usuarios >── sv_optometrista
                 |
                 v
            sv_historia_clinica >── sv_paciente
```

- Cada paciente tiene al menos una historia clínica  
- Cada historia clínica pertenece a un optometrista  

---

## 📝 Flujo de Trabajo

1. Creación del esquema y tablas  
2. Carga de datos crudos desde CSV  
3. Limpieza y normalización  
4. Generación masiva de datos clínicos  
5. Validación de coherencia  
6. Preparación para minería de datos y predicción  

---

## ✅ Conclusión

La base de datos **Veoptics** es completamente apta para:

- 📊 Análisis de pérdida visual por edad y sexo  
- 📈 Estudio de progresión de ametropías  
- 🔮 Modelos predictivos de salud visual  
- 🧠 Aplicación de técnicas de minería de datos  

Los datos generados son **masivos, coherentes, realistas y confiables** para investigación en **salud visual y análisis óptico**.

---

## 📄 Licencia

Proyecto de uso **académico y educativo**.  
Prohibido su uso en producción real.
