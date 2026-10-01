# REGLAS DEL PROYECTO: GESTIÓN INTEGRAL SPORTING SALADAR

## 1. REGLA DE ORO: AISLAMIENTO PERMANENTE DE TEMPORADAS

### Definición de Temporadas
- **TEMPORADA ACTIVA VIGENTE**: `TEMPORADA 26/27`
  - `season_id`: `663ed6ef-1dab-4350-9489-ed50f9e9ac15`
  - `ffcv_season_id`: `'22'`
- **TEMPORADA CERRADA Y ARCHIVADA**: `TEMPORADA 25/26`
  - `season_id`: `584f508a-fc1a-4339-b5b2-4296ffde2f4c`
  - `ffcv_season_id`: `'21'`

### Directrices Estrictas de Consulta y Visualización
1. **Exclusión Total de la Temporada 25/26**:
   - Queda **TERMINANTEMENTE PROHIBIDO** consultar, cruzar, fusionar o mostrar datos de la temporada 25/26 en ninguna vista activa del sistema (Dashboard general, `/dashboard/matches`, calendario, actas, asistencia, estadísticas, equipos, convocatorias, live, portal de familias, etc.).
   - La temporada 25/26 está cerrada y solo es accesible en modo lectura pasiva dentro del módulo de histórico: `/dashboard/archivo`.
2. **Consultas a Base de Datos (`partidos`, `teams`, `events`, `training_sessions`, etc.)**:
   - En cualquier consulta de datos deportivos activos, se debe filtrar explícitamente por la temporada activa (`.eq('season_id', activeSeasonId)`) o excluir de forma garantizada la temporada cerrada (`.neq('season_id', '584f508a-fc1a-4339-b5b2-4296ffde2f4c')`).
   - El selector de equipos de la barra lateral, menús móviles y vistas globales debe mostrar **únicamente** los equipos de la temporada activa 26/27.

---

## 2. REGLA ESTRICTA DE SINCRONIZACIÓN Y TABLAS FFCV

1. **Tablas Oficiales FFCV (`ffcv_matches`, `ffcv_standings`, `ffcv_groups`)**:
   - Toda consulta, importación o sincronización de datos federativos debe incluir el filtro obligatorio:
     ```ts
     .neq('ffcv_season_id', '21')
     .eq('ffcv_season_id', '22')
     ```
   - No se deben almacenar ni reutilizar datos de la temporada federativa `'21'` bajo ningún concepto.
2. **Propagación a la tabla interna `partidos` (`src/lib/ffcv/sync.ts`)**:
   - La función `propagateFfcvMatchesToClubPartidos` debe operar única y exclusivamente sobre partidos de la temporada activa (`ffcv_season_id = '22'`).
   - Jamás se actualizará el estado de un partido futuro a `Finalizado` basándose en partidos disputados de temporadas pasadas.

---

## 3. REGLA DE EMPAREJAMIENTO DE PARTIDOS (MATCHING INTEGRITY)

1. **Identificación por `CodPartido`**:
   - Para vincular un partido interno con el acta oficial federativa en `findMatchingFfcvMatch`, se debe extraer y contrastar en primer lugar el `CodPartido` de la URL oficial (`acta_oficial_url` o campo `ffcv_match_id`).
2. **Guardia Temporal en Búsqueda por Rival (Fallback Matching)**:
   - Si se recurre a coincidencias por nombre de equipo rival, es **OBLIGATORIO** verificar la proximidad temporal estricta:
     ```ts
     const diffDays = Math.abs((partidoDate.getTime() - fmDate.getTime()) / (1000 * 60 * 60 * 24));
     if (diffDays > 7) continue; // NUNCA emparejar partidos con más de 7 días de diferencia
     ```
   - Queda prohibido emparejar partidos de distintas vueltas (ida y vuelta) basándose solo en el nombre del rival.
   - Si un partido está programado para una fecha futura (`fecha_hora > now`), nunca debe recibir resultados o estado de un partido jugado en una fecha diferente.

---

## 4. REGLA DE INTEGRIDAD DEL CALENDARIO Y FECHAS

1. **Exactitud de Fechas Oficiales**:
   - Todo registro en `partidos` debe tener su `fecha_hora` exacta correspondiente a la jornada oficial asignada en la FFCV.
   - Queda prohibido asignar fechas por defecto de fin de temporada (como amontonar decenas de partidos en mayo) o colocar partidos de la segunda vuelta en jornadas tempranas.
2. **No Duplicidad de Jornadas**:
   - Un equipo nunca puede tener programados partidos de ida y vuelta en la misma fecha ni enfrentarse al mismo rival dos veces en el mismo fin de semana.

---

## 5. REGLA DE SINCRONIZACIÓN HORARIA Y PROPAGACIÓN UNIVERSAL FFCV

1. **Aplicación Universal a Todos los Equipos Federados**:
   - Todo equipo del club asignado a una competición oficial FFCV (`SENIOR`, `JUVENIL A`, `JUVENIL B`, `CADETE`, `INFANTIL`, etc.) debe mantener su calendario sincronizado con la FFCV.
   - Toda consulta federativa debe filtrar por los grupos de los equipos del club (`in('ffcv_group_id', clubGroupIds)`) y garantizar un límite de filas suficiente para evitar truncamientos de registros (>1.000 filas).
2. **Huso Horario Oficial Peninsular (Europe/Madrid)**:
   - Toda hora publicada por la FFCV debe transformarse con el desfase peninsular exacto (`formatSpainDateTimeToIso`): `+02:00` en horario de verano (CEST, marzo a octubre) y `+01:00` en horario de invierno (CET, noviembre a marzo).
   - Queda terminantemente prohibido almacenar marcas horarias UTC neutras (`+00:00` o `Z`) que generen un desfase en el navegador de familias y entrenadores.
3. **Propagación Conjunta a Partidos y Agenda (`team_events`)**:
   - Cuando la FFCV actualiza un horario o fecha oficial, la sincronización debe actualizar tanto la tabla `partidos` como los eventos asociados en `team_events` para que los calendarios de familias y cuerpo técnico coincidan al 100%.
