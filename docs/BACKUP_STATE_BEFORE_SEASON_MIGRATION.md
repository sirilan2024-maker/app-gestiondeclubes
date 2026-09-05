# Punto de Restauración / Backup previo a la Migración de Temporadas

**Fecha de registro:** 5 de septiembre de 2026  
**Propósito:** Proteger y documentar el estado exacto de la aplicación, el repositorio de código y la base de datos antes de iniciar cualquier cambio estructural o migración de temporadas.

---

## 1. Identificación del Código y Git

* **Rama activa:** `main`
* **Último Commit:** `f0cb0371aa42cbeaa2ca946bf86427a151b72e9d` (`f0cb037`)
  * Mensaje: `feat(ffcv): integrate FFCV data, calendar J1-J30, standings responsive view and hourly cron sync`
* **Tag de Restauración:** `backup-before-season-migration-2026-09-05`
  * Sincronizado en remoto (`origin`): `https://github.com/sirilan2024-maker/Aplicacion-de-Gestion-de-clubes.git`
* **Estado del árbol de trabajo (Working Tree):** Limpio (`clean`), sin modificaciones pendientes ni ficheros sin seguimiento.

---

## 2. Identificación del Despliegue en Producción

* **Entorno:** Vercel Production
* **Repositorio vinculado:** `sirilan2024-maker/Aplicacion-de-Gestion-de-clubes` (`origin/main`)
* **Cron programado:** `/api/ffcv/cron` (`0 * * * *`)
* **Estado de compilación TypeScript:** `0 errores` (`npx tsc --noEmit` completado exitosamente).

---

## 3. Snapshot / Conteo Real de la Base de Datos (Supabase)

Conteo exacto verificado contra la base de datos de producción:

| Tabla / Entidad | Conteo Real de Registros | Descripción / Estado |
| :--- | :---: | :--- |
| `teams` | **8** | Equipos internos registrados en la plataforma |
| `partidos` | **386** | Partidos internos históricos y programados |
| `convocatorias` | **3.648** | Convocatorias internas de jugadores para partidos |
| `match_events` | **1.472** | Eventos, goles, tarjetas y sustituciones en partidos |
| `ffcv_groups` | **1** | Grupo 905431618 (Segona FFCV Grup 8 - Sporting Saladar) |
| `ffcv_matches` | **240** | 30 jornadas completas (J1 a J30) del Grupo 8 |
| `ffcv_standings` | **16** | 16 clasificados oficiales del Grupo 8 |
| `players` | **171** | Jugadores dados de alta |
| `clubs` | **5** | Clubes registrados en la plataforma |
| `profiles` | **30** | Usuarios y miembros de cuerpo técnico |

---

## 4. Estado de los Datos FFCV Actuales

* **Grupo sincronizado:** `cod_grupo: 905431618` (Segona FFCV - Grup 8, Temporada 22 / 2026-2027)
* **Equipo oficial enlazado:** Sporting Saladar (`codequipo: 18233`)
* **Jornadas disponibles:** 30 (240 partidos en total)
* **Clasificación:** 16 equipos procesados correctamente con estadísticas oficiales
* **Cron horaria:** Operativa mediante endpoint `/api/ffcv/cron`

---

## 5. Procedimiento de Rollback (En caso de ser necesario)

Si en cualquier momento se requiriera volver al estado exacto previo a la migración:

1. **Restaurar código a este punto:**
   ```bash
   git checkout backup-before-season-migration-2026-09-05
   ```
2. **Re-desplegar versión exacta en producción:**
   ```bash
   git push origin backup-before-season-migration-2026-09-05:main --force-with-lease
   ```
3. Los datos de base de datos (`partidos`, `convocatorias`, `match_events`, etc.) se encuentran íntegros y preservados con las cifras detalladas en la Sección 3.
