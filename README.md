🏥 Clinica2 — Base de Datos de Prueba para Gestión Clínica

Clinica2 es una base de datos relacional diseñada para pruebas, análisis y desarrollo de aplicaciones clínicas.
Contiene información totalmente simulada, generada de forma masiva, coherente y aleatoria, ideal para testing técnico y analítico.

⚠️ Aviso importante:
Todos los datos son ficticios. No representan personas reales ni información médica real.
No usar en producción.

📌 Características Principales

✅ Datos clínicos coherentes (usuarios, pacientes, historias clínicas)

🎲 Generación masiva y randomizada

🧹 Limpieza automática de nombres y apellidos

🧠 Relaciones realistas entre entidades

🐳 Compatible con PostgreSQL + Docker

📊 Ideal para BI, dashboards, pruebas de performance y QA

🧱 Estructura de la Base de Datos
Tabla	Descripción
sv_rol	Roles del sistema (Administrador, Optometrista, Secretaria)
sv_usuarios	Usuarios del sistema, asociados a un rol
sv_optometrista	Información de optometristas vinculados a usuarios
sv_paciente	Pacientes con datos personales y motivo de consulta
sv_historia_clinica	Historias clínicas por paciente y optometrista
stg_nombres_raw	Tabla staging para importar CSV con datos “sucios”
stg_nombres_limpios	Tabla staging con nombres y apellidos depurados
🧹 Limpieza y Preparación de Datos
1️⃣ Importación inicial desde CSV

Los datos crudos se cargan en stg_nombres_raw y luego se copian a una tabla limpia:

INSERT INTO stg_nombres_limpios(nombre, apellido)
SELECT nombre, apellido
FROM stg_nombres_raw;

2️⃣ Limpieza de apellidos

Se eliminan números y caracteres especiales

Se asigna un apellido genérico si queda vacío

UPDATE stg_nombres_limpios
SET apellido = REGEXP_REPLACE(apellido, '[^A-Za-zÁÉÍÓÚÑáéíóúñ]+', '', 'g')
WHERE apellido IS NOT NULL;

UPDATE stg_nombres_limpios
SET apellido = 'Apellido'
WHERE apellido IS NULL OR apellido = '';

🎲 Generación de Datos Simulados
3️⃣ Roles del sistema
INSERT INTO sv_rol(nombre_rol)
VALUES ('Administrador'), ('Optometrista'), ('Secretaria')
ON CONFLICT DO NOTHING;

4️⃣ Usuarios y Optometristas

Usuarios tipo Optometrista

Correos, cédulas y nombres únicos

Relación 1:1 usuario ↔ optometrista

5️⃣ Pacientes (2000 registros)

Cada paciente incluye:

Edad: 1–90 años

Sexo: M / F

Teléfono y dirección simulados

Motivo de consulta aleatorio

Fecha de registro entre 2010 y 2025

INSERT INTO sv_paciente(
    tbl_nombre, tbl_apellido, tbl_cedula,
    tbl_fecha_nac, tbl_direccion, tbl_telefono,
    tbl_correo, tbl_motivo_consulta,
    tbl_fecha_registro, tbl_sexo
)
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

6️⃣ Historias Clínicas

Cada paciente tiene 1 a 5 historias clínicas

Optometrista asignado aleatoriamente

Fechas coherentes entre 2010 y 2025

Diagnósticos y notas simuladas

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
            SELECT id_optometrista
            INTO v_optometrista_id
            FROM sv_optometrista
            ORDER BY random()
            LIMIT 1;

            INSERT INTO sv_historia_clinica(
                id_paciente,
                tbl_antecedente,
                tbl_diagnostico,
                tbl_notas_clinica,
                tbl_fecha,
                id_optometrista
            )
            VALUES (
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

✅ Verificaciones Rápidas
-- Total de historias clínicas
SELECT COUNT(*) FROM sv_historia_clinica;

-- Pacientes con al menos una historia
SELECT COUNT(DISTINCT id_paciente)
FROM sv_historia_clinica;

-- Rango de fechas de consulta
SELECT MIN(tbl_fecha), MAX(tbl_fecha)
FROM sv_historia_clinica;

🐳 Uso con Docker
# Entrar al contenedor PostgreSQL
docker exec -it postgres_grupo psql -U grupo -d postgres

# Usar el esquema
SET search_path TO clinica2, public;

# Ejecutar script
\i /ruta/al/script_generacion.sql

💡 Casos de Uso

Este proyecto es ideal para:

🧪 Pruebas de performance (SQL / índices)

📊 Dashboards BI (Power BI, Metabase, Superset)

🧠 Análisis de datos clínicos

🧩 Testing de ERPs o sistemas médicos

🧬 Pruebas de integridad y relaciones

📈 Escalabilidad

Puedes ajustar fácilmente:

Número de pacientes

Número de optometristas

Cantidad de historias clínicas por paciente

Rango de fechas

Catálogos de diagnósticos y antecedentes

Todo controlado desde los límites de los bucles y arrays en los scripts SQL.

📄 Licencia

Este proyecto se distribuye solo para fines educativos y de prueba.
El autor no se responsabiliza por usos indebidos de los datos generados.
