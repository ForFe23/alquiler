# 🏥 Clinica2 - Base de Datos de Prueba

Esta base de datos **`clinica2`** fue creada para pruebas de gestión clínica y análisis de datos. Contiene información de usuarios, optometristas, pacientes y sus historias clínicas, generada de manera **masiva, coherente y randomizada**.

> ⚠️ Todos los datos son **simulados** y no representan personas reales.

---

## 📂 1. Estructura de la Base de Datos

| Tabla | Descripción |
|-------|-------------|
| `sv_rol` | Roles de usuario: `Administrador`, `Optometrista`, `Secretaria`. |
| `sv_usuarios` | Usuarios del sistema, vinculados a un rol. |
| `sv_optometrista` | Información de optometristas vinculados a un usuario. |
| `sv_paciente` | Pacientes con datos personales y motivo de consulta. |
| `sv_historia_clinica` | Historias clínicas vinculadas a pacientes y optometristas. |
| `stg_nombres_raw` | Tabla temporal para importar CSV inicial con nombres sucios. |
| `stg_nombres_limpios` | Tabla temporal con nombres y apellidos depurados. |

---

## 🧹 2. Limpieza y Preparación de Datos

### 2.1 Importación del CSV

Se importó un CSV con nombres, apellidos y cédulas a `stg_nombres_raw`.  
Luego se migró a `stg_nombres_limpios` para limpieza:

```sql
INSERT INTO stg_nombres_limpios(nombre, apellido)
SELECT nombre, apellido
FROM stg_nombres_raw;
2.2 Limpieza de apellidos
Se eliminaron números y caracteres extraños:

sql
Copiar código
UPDATE stg_nombres_limpios
SET apellido = REGEXP_REPLACE(apellido, '[^A-Za-zÁÉÍÓÚÑáéíóúñ]+', '', 'g')
WHERE apellido IS NOT NULL;
Si queda vacío, se asigna un apellido genérico:

sql
Copiar código
UPDATE stg_nombres_limpios
SET apellido = 'Apellido'
WHERE apellido IS NULL OR apellido = '';
🎲 3. Generación de Datos Coherentes y Randomizados
3.1 Roles iniciales
sql
Copiar código
INSERT INTO sv_rol(nombre_rol)
VALUES ('Administrador'), ('Optometrista'), ('Secretaria')
ON CONFLICT DO NOTHING;
3.2 Usuarios y Optometristas
Se crean usuarios tipo Optometrista y se vinculan a optometristas.

Correos, cédulas y nombres se generan de forma única y coherente.

3.3 Pacientes
Se generaron 2000 pacientes con:

Edad aleatoria 1–90 años

Teléfonos y direcciones simuladas

Motivos de consulta variados: Consulta General, Dolor ocular, Revisión de visión, Lentes nuevos

Sexo aleatorio M o F

Fechas de registro random entre 2010 y 2025

sql
Copiar código
INSERT INTO sv_paciente(tbl_nombre, tbl_apellido, tbl_cedula,
                        tbl_fecha_nac, tbl_direccion, tbl_telefono,
                        tbl_correo, tbl_motivo_consulta, tbl_fecha_registro, tbl_sexo)
SELECT
    nombre,
    COALESCE(apellido,'Apellido') || floor(random()*9999)::text,
    lpad((floor(random()*1e10)::bigint)::text,10,'0'),
    CURRENT_DATE - (floor(random()*32850 + 365)::int) * INTERVAL '1 day',
    'Calle ' || floor(random()*1000),
    '09' || floor(random()*89999999+10000000)::text,
    lower(nombre || '.' || COALESCE(apellido,'Apellido') || floor(random()*9999)::text || '@mail.com'),
    (ARRAY['Consulta General','Dolor ocular','Revisión de visión','Lentes nuevos'])[floor(random()*4)+1],
    DATE '2010-01-01' + (floor(random()*(DATE '2025-12-31' - DATE '2010-01-01'))::int) * INTERVAL '1 day',
    (ARRAY['M','F'])[floor(random()*2)+1]
FROM stg_nombres_limpios
ORDER BY random()
LIMIT 2000;
3.4 Historias Clínicas
Cada paciente tiene 1–5 historias clínicas

Cada historia está vinculada a un optometrista random

Fechas de consulta coherentes entre 2010 y 2025

Antecedentes, diagnósticos y notas aleatorias

sql
Copiar código
DO $$
DECLARE
    v_paciente RECORD;
    v_num_hist INT;
    i INT;
    v_optometrista_id INT;
BEGIN
    FOR v_paciente IN SELECT id_paciente FROM sv_paciente LOOP
        v_num_hist := floor(random()*5 + 1);
        FOR i IN 1..v_num_hist LOOP
            SELECT id_optometrista INTO v_optometrista_id
            FROM sv_optometrista
            ORDER BY random()
            LIMIT 1;

            INSERT INTO sv_historia_clinica(
                id_paciente, tbl_antecedente, tbl_diagnostico, tbl_notas_clinica, tbl_fecha, id_optometrista
            ) VALUES (
                v_paciente.id_paciente,
                (ARRAY['Ninguno','Hipertensión','Diabetes','Alergias Oculares'])[floor(random()*4)+1],
                (ARRAY['Miopía','Hipermetropía','Astigmatismo','Presbicia'])[floor(random()*4)+1],
                (ARRAY['Requiere lentes','Revisión cada 6 meses','Tratamiento ocular','Observación'])[floor(random()*4)+1],
                DATE '2010-01-01' + (floor(random()*(DATE '2025-12-31' - DATE '2010-01-01'))::int) * INTERVAL '1 day',
                v_optometrista_id
            );
        END LOOP;
    END LOOP;
END $$;
✅ 4. Verificaciones
sql
Copiar código
-- Conteo total de historias clínicas
SELECT COUNT(*) AS total_historia_clinica FROM sv_historia_clinica;

-- Pacientes con al menos una historia
SELECT COUNT(DISTINCT id_paciente) AS pacientes_con_historia 
FROM sv_historia_clinica;

-- Fechas mínimas y máximas de consultas
SELECT MIN(tbl_fecha) AS primera_consulta, MAX(tbl_fecha) AS ultima_consulta
FROM sv_historia_clinica;
🐳 5. Uso con Docker
bash
Copiar código
# Entrar al contenedor
docker exec -it postgres_grupo psql -U grupo -d postgres

# Usar esquema clinica2
SET search_path TO clinica2, public;

# Ejecutar scripts SQL
\i /ruta/al/script_generacion.sql
💡 6. Consideraciones
Los datos son simulados y no deben usarse en producción real.

Ideal para:

Pruebas de performance

Dashboards

Testing de aplicaciones clínicas

Escalable: se pueden generar más pacientes, optometristas e historias cambiando los límites en los bucles PL/pgSQL.
