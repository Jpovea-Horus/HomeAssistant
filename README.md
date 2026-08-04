Tu borrador está bien orientado, pero **desactualizado** respecto a Aula **v1.3.2.4** y Hotel **v1.3.4**. Correcciones clave:

| Borrador | Real |
|---|---|
| `state_internal` | `action_plugin` |
| modos `auto` / `manual` | + **`apagado`** (ambos) |
| Aula `mode: single` | Aula también es **`queued`** |
| `off_delay` / `switch_delay` | Aula: **`delay_off`** (SM OFF → scan inmediato) |
| Hotel `delay_off` único | `delay_off_sp` + `delay_off_actuadores` + `delay_scan` + `delay_puertAb` |
| Historial solo v1.3.2.8 | Fix `librePuertaAb` en **v1.3.2.9**; actual **v1.3.4** |

Solo existe `Code BluePrint - Hotel/README.md`. Estoy en **Ask mode**: no puedo editarlo. Cambia a Agent si quieres que lo escriba. Mientras, este es el texto alineado a los YAML:

---

```markdown
# Blueprints de Ocupación — Aula & Hotel

Comparativa de los plugins de ocupación para Home Assistant.

| | **Aula** | **Hotel** |
|---|---|---|
| Versión actual | **v1.3.2.4** (`Plugin Aula v1-3-2-4 - motion_sensor.yaml`) | **v1.3.4** (`_Bp-Plugin Hotel v1.3.4 - SensorPuerta.yaml`) |
| Sensores | Movimiento | Puerta + movimiento |
| Estados internos (`action_plugin`) | `libre`, `ocupado`, `scan` | + `puertAbierta`, `librePuertaAb` |
| Modo automatización | `queued` | `queued` |

## Descripción general

- **Aula**: Gestiona ocupación con sensor de movimiento. Controla switches y climates según `libre` / `ocupado` / `scan`, con modos `auto`, `manual` o `apagado`. Ideal para aulas u oficinas sin control de puerta.
- **Hotel**: Añade sensor de puerta y estados `puertAbierta` / `librePuertaAb`. Incluye apagado de AA por puerta abierta prolongada, delays diferenciados según origen de ocupación y el mismo trío de modos del plugin.

## Requisitos

### Input Select (opciones exactas)

| Entidad | Aula | Hotel |
|---|---|---|
| **state_plugin** | `libre`, `ocupado` | `libre`, `ocupado` |
| **action_plugin** | `libre`, `ocupado`, `scan` | `libre`, `ocupado`, `scan`, `puertAbierta`, `librePuertaAb` |
| **mode_plugin** | `auto`, `manual`, `apagado` | `auto`, `manual`, `apagado` |

### Input Boolean (on/off)

| Flag | Aula | Hotel | Función |
|---|---|---|---|
| Modo Actuadores On | `mode_activation` | `mode_actuadoresOn` | Encendido de actuadores al iniciar (solo `auto`) |
| Modo Thermostat | `mode_thermostat` | `mode_thermostat` | SetPointOn al iniciar (solo `auto`) |
| Modo SetPointOff | `mode_Setpointoff` | `mode_Setpointoff` | Temperatura al pasar a libre (solo `auto`) |
| Modo Puerta Abierta | — | `mode_puertAb` | Apaga AA si puerta abierta ≥ `delay_puertAb` (solo `auto`) |

## Modos del plugin (`mode_plugin`)

| Modo | Comportamiento |
|---|---|
| **auto** | Encendido al iniciar + apagado/ajuste al finalizar. SetPointOff soportado. En Hotel, AA por puerta abierta si `mode_puertAb` está on. |
| **manual** | Solo apagado al finalizar (sin SetPointOff). No enciende al iniciar. |
| **apagado** | Solo actualiza `state_plugin` / `action_plugin`. No acciona equipos. |

## Funcionamiento

### Aula (v1.3.2.4)

- SM ON → `ocupado` (enciende en `auto` solo si venía de no-ocupado; no reenciende AA si se apagó manualmente estando ocupado).
- SM OFF → `scan` **inmediato**.
- Tras `delay_off` en `scan` sin movimiento → `libre` (apaga / SetPointOff según modo).
- Cambio a `libre` por UI: sincroniza `action_plugin` y aplica apagado.
- Respaldo 1 min en `libre`: reenvía apagados; **no** apaga climates si SetPointOff está activo.

### Hotel (v1.3.4)

- Puerta abierta / movimiento / actuador On desde `libre` inician ocupación (con o sin encendido según flags y modo).
- Delays: `delay_off_sp` (entrada por puerta), `delay_off_actuadores` (entrada por SM o actuadores), `delay_scan` (evaluación de movimiento), `delay_puertAb` (apagado AA con puerta abierta).
- **`librePuertaAb`**: si la puerta sigue abierta al “liberar”, evita un `libre` incorrecto. Movimiento o actuador On → `puertAbierta` + `ocupado` **solo estados** (sin encender equipos).
- Optimizaciones: evita comandos redundantes a climates; reintentos de switches consultando estado; flags no anexados = `off` por defecto.

## Escenarios principales

### Aula

1. Movimiento: → `ocupado`, enciende (modo `auto`, flags on).
2. SM OFF: → `scan`.
3. Movimiento en `scan`: → `ocupado`.
4. Inactividad (`delay_off`): `scan` → `libre`, apaga / SetPointOff.
5. `libre` por UI: sincroniza y apaga según modo.
6. Respaldo 1 min en `libre`: reasegura apagados.

### Hotel (incluye lógica de puerta)

1. Puerta abierta en `libre`: → `puertAbierta` + `ocupado`, puede encender.
2. Movimiento / actuador On en `libre`: → `scan` → `ocupado`.
3. Cierre de puerta desde `puertAbierta` / `librePuertaAb`: → `scan`, evalúa movimiento.
4. Inactividad: → `libre` (o `librePuertaAb` si la puerta sigue abierta), apaga / ajusta.
5. Puerta abierta ≥ `delay_puertAb` + `mode_puertAb`: apaga AA (solo `auto`).
6. `librePuertaAb` + actuador On o SM: → `puertAbierta` + `ocupado` (solo estados).

## Historial de cambios

### Hotel

| Versión | Cambio |
|---|---|
| **1.3.2.8 / 1.3.2.9** | Fix `librePuertaAb`: `Scena_SW_librePuertaAb_a_puertAb_ocup` y `Scena_SMov_librePuertaAb_a_puertAb_ocup` (solo estados). |
| **1.3.3.0** | `delay_off_sp` vs `delay_off_actuadores`. |
| **1.3.3.1** | Evita comandos redundantes en AA/termostatos. |
| **1.3.3.1.2** | Reintentos switches (Z-Wave/Zigbee). |
| **1.3.3.1.3** | Evalúa flags de modo; default `off` si no anexados. |
| **1.3.4** | `mode_plugin`: `auto` / `manual` / `apagado`. |

### Aula

| Versión | Cambio |
|---|---|
| **1.3.2.4** | Modos `auto` / `manual` / `apagado`. SM OFF → scan inmediato; `delay_off` para fin de scan. Fix: no reencender AA si se apagó manual en ocupado; respaldo no apaga clima con SetPointOff on; listas aplanadas / helpers. |

## Notas

- **Aula**: más simple; solo movimiento. Ruta: `Code BluePrint - Aula/Version Actual/`.
- **Hotel**: puerta, `librePuertaAb`, delays duales y apagado AA por puerta abierta. Ruta: `Code BluePrint - Hotel/Version Actual/`.
- Ambos requieren entidades válidas; revisar logs de HA ante fallos.

## Licencia

https://www.home-assistant.io/developers/license/

## Contribuciones

Fork + pull request con descripción clara de la mejora.

## Contacto

Ing. Jefry Povea
jefry.p@horus-sc.com

```

---

**Resumen:** el README de Hotel ya documenta bien v1.3.4; tu texto comparativo conviene **fusionarlo** con esa precisión (modos `apagado`, `action_plugin`, delays reales, Aula `queued`). ¿Lo aplico sobre `Code BluePrint - Hotel/README.md`, o prefieres un README padre compartido para ambos? En Agent mode lo escribo donde indiques.
