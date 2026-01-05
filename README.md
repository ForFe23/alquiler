
Clinica2 - Base de Datos de Prueba

Este proyecto contiene la base de datos clinica2 diseñada para pruebas de gestión clínica, generación de pacientes, optometristas y sus historias clínicas de manera masiva, coherente y randomizada.

El propósito es tener un dataset robusto para pruebas de aplicaciones, dashboards o análisis de datos clínicos.

1. Estructura de la Base de Datos

La base de datos clinica2 contiene las siguientes tablas:

Tabla	Descripción
sv_rol	Roles de usuario (Administrador, Optometrista, Secretaria).
sv_usuarios	Usuarios del sistema, vinculados a un rol.
sv_optometrista	Información de optometristas, vinculados a un usuario.
sv_paciente	Pacientes con datos personales, contacto y motivo de consulta.
sv_historia_clinica	Historias clínicas, vinculadas a pacientes y optometristas.
stg_nombres_raw	Tabla temporal donde se importa CSV inicial de nombres sucios.
stg_nombres_limpios	Tabla temporal con nombres y apellidos depurados.
2. Proceso de Importación y Limpieza
2.1. Importación del CSV

Se importó un CSV con nombres, apellidos y cédulas a la tabla temporal stg_nombres_raw.

La estructura del CSV era irregular, con números y caracteres extraños en apellidos.

2.2. Limpieza de Datos

Se depuraron los datos mediante SQL:

Se eliminaron números y caracteres no alfabéticos de los apellidos:

UPDATE stg_nombres_limpios
SET apellido = REGEXP_REPLACE(apellido, '[^A-Za-zÁÉÍÓÚÑáéíóúñ]+', '', 'g')
WHERE apellido IS NOT NULL;


Se asignó un apellido genérico cuando quedó vacío:

UPDATE stg_nombres_limpios
SET apellido = 'Apellido'
WHERE apellido IS NULL OR apellido = '';


Esta limpieza asegura que los nombres y apellidos sean coherentes para generar pacientes y usuarios.

3. Generación de Datos Coherentes y Randomizados
3.1. Roles Iniciales

Se insertaron roles básicos si no existían:

INSERT INTO sv_rol(nombre_rol)
VALUES ('Administrador'), ('Optometrista'), ('Secretaria')
ON CONFLICT DO NOTHING;

3.2. Creación de Optometristas y Usuarios

Se generaron usuarios y optometristas randomizados a partir de los nombres limpios.

Cada usuario de tipo Optometrista se vinculó a un optometrista.

Los correos y cédulas se generaron de manera única y coherente.

3.3. Generación de Pacientes

Se crearon 2000 pacientes con datos randomizados:

Edad: entre 1 y 90 años.

Direcciones y teléfonos simulados.

Correo electrónico basado en nombre y apellido.

Motivo de consulta aleatorio (Consulta General, Dolor ocular, Revisión de visión, Lentes nuevos).

Sexo aleatorio M o F.

INSERT INTO sv_paciente(...)
SELECT ...
FROM stg_nombres_limpios
ORDER BY random()
LIMIT 2000;

3.4. Generación de Historias Clínicas

Se generaron historias clínicas para todos los pacientes existentes.

Cada paciente tiene 1 a 5 historias clínicas.

Cada historia se vincula a un optometrista random.

Fechas de consulta randomizadas entre 2010 y 2025.

Antecedentes, diagnósticos y notas coherentes y randomizadas.

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

4. Validaciones y Verificaciones

Para asegurarse de que todo quedó coherente:

-- Conteo total de historias clínicas
SELECT COUNT(*) AS total_historia_clinica FROM sv_historia_clinica;

-- Pacientes con al menos una historia
SELECT COUNT(DISTINCT id_paciente) AS pacientes_con_historia 
FROM sv_historia_clinica;

-- Verificar fechas máximas y mínimas
SELECT MIN(tbl_fecha) AS primera_consulta, MAX(tbl_fecha) AS ultima_consulta
FROM sv_historia_clinica;


Cada paciente tiene al menos una historia clínica.

Las fechas de consulta están dentro del rango 2010–2025.

Todas las relaciones (paciente -> historia, optometrista -> historia) son válidas.

5. Consideraciones

Se respetó la coherencia entre tablas.

Los datos son simulados y no representan información real.

Se pueden generar más pacientes y más historias simplemente ajustando los límites en los bucles PL/pgSQL.

Ideal para pruebas de performance, dashboards y análisis de datos clínicos.

6. Uso en Docker

Si se usa PostgreSQL en Docker:

# Entrar al contenedor
docker exec -it postgres_grupo psql -U grupo -d postgres

# Usar el esquema clinica2
SET search_path TO clinica2, public;

# Ejecutar scripts de generación y verificación
\i /path/to/tu_script.sql
