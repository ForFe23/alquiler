
# 🏥 Clinica2 — Base de Datos de Prueba para Gestión Clínica

**Clinica2** es una base de datos relacional diseñada para **pruebas, análisis y desarrollo** de aplicaciones clínicas.  
Contiene información **totalmente simulada**, generada de forma **masiva, coherente y aleatoria**, ideal para testing técnico y analítico.

> ⚠️ **Aviso importante:**  
> Todos los datos son **ficticios**. No representan personas reales ni información médica real.  
> **No usar en producción.**

---

## 📌 Características Principales

- ✅ Datos clínicos coherentes (usuarios, pacientes, historias clínicas)
- 🎲 Generación masiva y randomizada
- 🧹 Limpieza automática de nombres y apellidos
- 🧠 Relaciones realistas entre entidades
- 🐳 Compatible con **PostgreSQL + Docker**
- 📊 Ideal para **BI, dashboards, pruebas de performance y QA**

---

## 🧱 Estructura de la Base de Datos

| Tabla | Descripción |
|------|-------------|
| `sv_rol` | Roles del sistema (`Administrador`, `Optometrista`, `Secretaria`) |
| `sv_usuarios` | Usuarios del sistema, asociados a un rol |
| `sv_optometrista` | Información de optometristas vinculados a usuarios |
| `sv_paciente` | Pacientes con datos personales y motivo de consulta |
| `sv_historia_clinica` | Historias clínicas por paciente y optometrista |
| `stg_nombres_raw` | Tabla staging para importar CSV con datos “sucios” |
| `stg_nombres_limpios` | Tabla staging con nombres y apellidos depurados |

---

## 🧹 Limpieza y Preparación de Datos

### Importación inicial desde CSV

```sql
INSERT INTO stg_nombres_limpios(nombre, apellido)
SELECT nombre, apellido
FROM stg_nombres_raw;
```

### Limpieza de apellidos

```sql
UPDATE stg_nombres_limpios
SET apellido = REGEXP_REPLACE(apellido, '[^A-Za-zÁÉÍÓÚÑáéíóúñ]+', '', 'g')
WHERE apellido IS NOT NULL;

UPDATE stg_nombres_limpios
SET apellido = 'Apellido'
WHERE apellido IS NULL OR apellido = '';
```

---

## 🎲 Generación de Datos Simulados

### Roles del sistema

```sql
INSERT INTO sv_rol(nombre_rol)
VALUES ('Administrador'), ('Optometrista'), ('Secretaria')
ON CONFLICT DO NOTHING;
```

### Pacientes

- 2000 registros
- Edad 1–90 años
- Sexo M / F
- Fechas entre 2010 y 2025

### Historias Clínicas

- 1 a 5 historias por paciente
- Optometrista aleatorio
- Diagnósticos y notas simuladas

---

## ✅ Verificaciones

```sql
SELECT COUNT(*) FROM sv_historia_clinica;

SELECT COUNT(DISTINCT id_paciente)
FROM sv_historia_clinica;

SELECT MIN(tbl_fecha), MAX(tbl_fecha)
FROM sv_historia_clinica;
```

---

## 🐳 Uso con Docker

```bash
docker exec -it postgres_grupo psql -U grupo -d postgres
SET search_path TO clinica2, public;
\i /ruta/al/script_generacion.sql
```

---

## 💡 Casos de Uso

- Pruebas de performance
- Dashboards BI
- Testing de sistemas clínicos
- Análisis de datos

---

## 📈 Escalabilidad

El volumen de datos puede ajustarse modificando los límites de los bucles y arrays en los scripts SQL.

---

## 📄 Licencia

Proyecto con fines educativos y de prueba.  
No usar en producción.
