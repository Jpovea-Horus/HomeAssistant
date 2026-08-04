# Blueprints de Ocupación — Aula & Hotel

Automatizaciones (blueprints) para [Home Assistant](https://www.home-assistant.io/) que gestionan ocupación, actuadores (`switch`) y termostatos (`climate`) según sensores y el modo del plugin.

| Plugin | Versión | Archivo | Sensores |
|:---|:---|:---|:---|
| **Aula** | `v1.3.2.4` | [`Plugin Aula v1-3-2-4 - motion_sensor.yaml`](../Code%20BluePrint%20-%20Aula/Version%20Actual/Plugin%20Aula%20v1-3-2-4%20-%20motion_sensor.yaml) | Movimiento |
| **Hotel** | `v1.3.4` | [`_Bp-Plugin Hotel v1.3.4 - SensorPuerta.yaml`](Version%20Actual/_Bp-Plugin%20Hotel%20v1.3.4%20-%20SensorPuerta.yaml) | Puerta + movimiento |

> **Idea rápida:** Aula es el flujo simple (solo movimiento). Hotel añade puerta, `librePuertaAb` y delays diferenciados.

---

## Tabla de contenidos

- [Descripción](#descripción)
- [Requisitos](#requisitos)
- [Modos del plugin](#modos-del-plugin)
- [Funcionamiento](#funcionamiento)
- [Escenarios principales](#escenarios-principales)
- [Configuración (Hotel)](#configuración-hotel)
- [Historial de cambios](#historial-de-cambios)
- [Instalación](#instalación)
- [Notas](#notas)
- [Licencia](#licencia)

---

## Descripción

### Aula (`v1.3.2.4`)

Gestiona ocupación en un aula con **sensor de movimiento**. Controla switches y climates según:

- Estados: `libre` · `ocupado` · `scan`
- Modos: `auto` · `manual` · `apagado`

Ideal para espacios con ocupación variable **sin** control de puerta.

### Hotel (`v1.3.4`)

Gestiona ocupación en habitación de hotel con **puerta + movimiento**. Añade:

- Estados extra: `puertAbierta` · `librePuertaAb`
- Apagado de AA por puerta abierta prolongada (`mode_puertAb`)
- Delays según origen de la ocupación (`delay_off_sp` / `delay_off_actuadores`)
- Transición segura `librePuertaAb` (evita pasar a `libre` con la puerta abierta)

---

## Requisitos

Configura estas entidades **antes** de crear la automatización. Las opciones deben coincidir **exactamente**.

### Input Select

| Input | Aula | Hotel |
|:---|:---|:---|
| `state_plugin` | `libre`, `ocupado` | `libre`, `ocupado` |
| `action_plugin` | `libre`, `ocupado`, `scan` | `libre`, `ocupado`, `scan`, `puertAbierta`, `librePuertaAb` |
| `mode_plugin` | `auto`, `manual`, `apagado` | `auto`, `manual`, `apagado` |

### Input Boolean

| Flag | Aula | Hotel | Función (solo modo `auto`, salvo apagados) |
|:---|:---:|:---:|:---|
| Actuadores On | `mode_activation` | `mode_actuadoresOn` | Encender switches al iniciar ocupación |
| Termostato | `mode_thermostat` | `mode_thermostat` | Aplicar SetPointOn al iniciar |
| SetPointOff | `mode_Setpointoff` | `mode_Setpointoff` | Temperatura al pasar a libre |
| Puerta abierta | — | `mode_puertAb` | Apagar AA si puerta abierta ≥ `delay_puertAb` |

> Si un boolean no está anexado, Hotel lo trata como `off` por defecto.

---

## Modos del plugin

| Modo | Qué hace |
|:---|:---|
| **`auto`** | Encendido al iniciar + apagado/ajuste al finalizar. Soporta SetPointOff. En Hotel: AA por puerta abierta si `mode_puertAb` está `on`. |
| **`manual`** | Solo apagado al finalizar (**sin** SetPointOff). No enciende al iniciar. |
| **`apagado`** | Solo actualiza `state_plugin` / `action_plugin`. **No** acciona equipos. |

Ambos blueprints corren en modo automatización **`queued`**.

---

## Funcionamiento

### Aula

```text
SM ON  → ocupado  (enciende en auto si venía de no-ocupado)
SM OFF → scan     (inmediato)
scan + delay_off sin movimiento → libre  (apaga / SetPointOff)
```

Detalles:

- No reenciende el AA si se apagó manualmente estando `ocupado` (hasta pasar por `libre`).
- `libre` por UI → sincroniza `action_plugin` y aplica apagado según modo.
- Respaldo 1 min en `libre`: reenvía apagados; **no** apaga climates si SetPointOff está `on`.

### Hotel

```text
libre + puerta abierta     → puertAbierta + ocupado
libre + movimiento/actuador → scan → ocupado
puertAbierta + inactividad + puerta sigue abierta → librePuertaAb + libre
librePuertaAb + SM / actuador On → puertAbierta + ocupado  (solo estados)
```

Detalles:

- Delays: `delay_off_sp` (entrada por puerta), `delay_off_actuadores` (SM/actuadores), `delay_scan`, `delay_puertAb`.
- En `librePuertaAb`, movimiento o actuador On **solo cambian estados** (no encienden equipos; la puerta sigue abierta).
- Evita comandos redundantes a climates; reintentos de switches consultando estado.

---

## Escenarios principales

### Aula

1. Movimiento detectado → `ocupado` · enciende (si `auto` + flags).
2. Sensor off → `scan`.
3. Movimiento en `scan` → `ocupado`.
4. Inactividad (`delay_off`) → `libre` · apaga / SetPointOff.
5. Cambio a `libre` por UI → sincroniza y apaga.
6. Respaldo 1 min en `libre` → reasegura apagados.

### Hotel

Incluye la lógica de ocupación y, además:

1. Puerta abierta en `libre` → `puertAbierta` + `ocupado` · puede encender.
2. Cierre de puerta (`puertAbierta` / `librePuertaAb`) → `scan` · evalúa movimiento.
3. Puerta abierta ≥ `delay_puertAb` + `mode_puertAb` → apaga AA (`auto`).
4. `puertAbierta` + inactividad + sin movimiento → `librePuertaAb` + `libre` · apaga.
5. `librePuertaAb` + actuador On → `puertAbierta` + `ocupado` (**solo estados**).
6. `librePuertaAb` + movimiento → `puertAbierta` + `ocupado` (**solo estados**).

---

## Configuración (Hotel)

Resumen de inputs del blueprint Hotel (el más completo). Aula comparte sensores de movimiento, actuadores, climates, SetPoints y `retry_time`.

### Sensores y equipos

| Input | Dominio | Uso |
|:---|:---|:---|
| `door_sensor` | `binary_sensor` | Apertura/cierre (Hotel) |
| `motion_sensor` | `binary_sensor` | Ocupación |
| `target_actuadoresOn` | `switch` | Encender al iniciar (`auto`) / apagar al liberar |
| `target_actuadoresOff` | `switch` | Apagar al liberar |
| `target_climates` | `climate` | AA / termostatos |

### Termostato

| Input | Rango / defecto |
|:---|:---|
| `temperature_SetPointOn` | 16–30 °C · default `24` |
| `fanMode_SetPointOn` | Auto low / Low / Medium / High |
| `temperature_SetPointOff` | 16–30 °C · default `27` |
| `fanMode_SetPointOff` | Auto low / Low / Medium / High |

### Retardos (Hotel)

| Input | Descripción | Defecto |
|:---|:---|:---|
| `delay_off_sp` | Scan → libre (entrada por puerta) | `900` s |
| `delay_off_actuadores` | Scan → libre (entrada por SM / actuadores) | `900` s |
| `delay_scan` | Evaluación de movimiento en `scan` | `260` s |
| `delay_puertAb` | Puerta abierta → apagar AA | `300` s |
| `retry_time` | Reintento switches Z-Wave/Zigbee | `1` s |

En **Aula**, el tiempo de liberación es un único `delay_off` (default `600` s). El scan al SM OFF es inmediato.

---

## Historial de cambios

### Hotel

| Versión | Cambio |
|:---|:---|
| **1.3.2.9** | Fix `librePuertaAb`: actuador On o SM → `puertAbierta` + `ocupado` (solo estados). Escenas: `Scena_SW_librePuertaAb_a_puertAb_ocup`, `Scena_SMov_librePuertaAb_a_puertAb_ocup`. |
| **1.3.3.0** | `delay_off_sp` (puerta) y `delay_off_actuadores` (SM / actuadores). |
| **1.3.3.1** | Evita comandos redundantes en AA/termostatos. |
| **1.3.3.1.2** | Reintentos de switches consultando estado. |
| **1.3.3.1.3** | Evalúa flags de modo; default `off` si no anexados. |
| **1.3.4** | `mode_plugin`: `auto` / `manual` / `apagado`. |

### Aula

| Versión | Cambio |
|:---|:---|
| **1.3.2.4** | Modos `auto` / `manual` / `apagado`. SM OFF → `scan` inmediato; liberación con `delay_off`. Fix: no reencender AA tras apagado manual en ocupado; respaldo respeta SetPointOff; listas aplanadas. |

---

## Instalación

1. Copia el YAML a `config/blueprints/automation/<tu_usuario>/`.
2. En **Ajustes → Automatizaciones y escenas → Blueprints**, crea una automatización.
3. Selecciona entidades (`input_select`, `input_boolean`, sensores, switches, climates).
4. Guarda. Si hace falta, recarga automatizaciones o reinicia Home Assistant.

---

## Notas

- Usa entidades **válidas** y disponibles en Home Assistant.
- Revisa **Logs** si algo no dispara o no apaga/enciende.
- **Aula** → entornos sin puerta controlada.  
  **Hotel** → habitaciones con puerta, AA y lógica de ocupación más estricta.

---

## Licencia

Ambos bajo [MIT](LICENSE).
https://www.home-assistant.io/developers/license/

## Contribuciones

1. Fork del repositorio  
2. Cambios en una rama  
3. Pull request con descripción clara  

## Contacto

Ing. Jefry Povea
jefry.p@horus-sc.com

```
