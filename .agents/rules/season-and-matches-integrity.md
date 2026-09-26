# REGLA ESTRICTA: AISLAMIENTO PERMANENTE DE TEMPORADAS Y SINCRONIZACIÓN FFCV

## Ámbito de Aplicación
Esta regla se aplica de manera ineludible a cualquier agente, desarrollador o script que interactúe con el código o la base de datos de Sporting Saladar.

## 1. Identificadores de Temporada
- **TEMPORADA ACTIVA 26/27**:
  - `season_id`: `663ed6ef-1dab-4350-9489-ed50f9e9ac15`
  - `ffcv_season_id`: `'22'`
- **TEMPORADA CERRADA Y ARCHIVADA 25/26**:
  - `season_id`: `584f508a-fc1a-4339-b5b2-4296ffde2f4c`
  - `ffcv_season_id`: `'21'`

## 2. Prohibición de Datos Históricos en Vistas Activas
- Ningún módulo activo (Matches, Dashboard, Calendario, Equipos, Estadísticas, Asistencia, Actas, Live) puede consultar o incorporar registros de la temporada 25/26.
- La temporada 25/26 solo debe consultarse en `/dashboard/archivo`.

## 3. Consultas y Sincronizaciones FFCV
- Todas las consultas a `ffcv_matches`, `ffcv_standings` y `ffcv_groups` deben incluir obligatoriamente:
  `.neq('ffcv_season_id', '21').eq('ffcv_season_id', '22')`
- La propagación de actas a `partidos` en `src/lib/ffcv/sync.ts` debe limitarse exclusivamente a la temporada activa 22.

## 4. Reconciliación y Cruce de Partidos (Matching)
- En `findMatchingFfcvMatch`, vincular prioritariamente por el identificador del acta (`CodPartido` o `ffcv_match_id`).
- Si se utiliza coincidencia por rival, exigir una proximidad temporal máxima de 7 días (`diffDays <= 7`).
- Nunca aplicar marcadores o el estado `Finalizado` a partidos con fechas futuras.

## 5. Integridad de Fechas de Calendario
- Las fechas en la tabla `partidos` deben coincidir exactamente con el calendario oficial federativo.
- No se permiten partidos duplicados en una misma jornada ni acumulaciones ficticias en mayo.
