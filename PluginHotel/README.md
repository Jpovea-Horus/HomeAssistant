# Bp-Plugin Hotel v1.3.4.2.1

## Descripción

Automatización (blueprint) para Home Assistant orientada a gestionar la ocupación de una habitación de hotel. Controla actuadores (`switch`) y termostatos (`climate`) según sensores de puerta y movimiento, según el estado de ocupación y el modo del plugin (`auto`, `manual`, `apagado`).

Incluye el estado interno `librePuertaAb` para evitar transiciones erróneas a `libre` mientras la puerta permanece abierta.

## Requisitos

Configura en Home Assistant los siguientes elementos **antes** de usar el blueprint.

### Input Select (obligatorios)

Las opciones deben coincidir **exactamente**:

| Entidad (input del blueprint) | Opciones requeridas |
| --- | --- |
| **state_plugin** | `libre`, `ocupado` |
| **action_plugin** (Estado Interno) | `libre`, `ocupado`, `scan`, `puertAbierta`, `librePuertaAb` |
| **mode_plugin** | `auto`, `manual`, `apagado` |

### Input Boolean (opcionales)

Valores `on` / `off`. **No son obligatorios**: si no creas la entidad o no la anexas al blueprint, el plugin la trata como `off`.

| Entidad (input del blueprint) | Ayudante habitual | Función |
| --- | --- | --- |
| **mode_actuadoresOn** | ModoAct | Habilita el encendido de actuadores al iniciar ocupación (solo modo `auto`) |
| **mode_thermostat** | ModoThermo | Habilita el control de termostatos al iniciar ocupación (solo modo `auto`) |
| **mode_puertAb** | ModoPuertAb | Habilita el apagado de AA si la puerta permanece abierta ≥ `delay_puertAb` (solo modo `auto`) |
| **mode_Setpointoff** | ModoSetPointOff | Habilita SetPointOff al pasar a libre (solo modo `auto`). Si está `off`, apaga el AA al liberar |

> Si las opciones de `input_select` no están bien configuradas, la automatización puede no comportarse como se espera.

## Instalación

1. Copia el archivo YAML del blueprint (p. ej. `_Bp-Plugin Hotel v1.3.4.2.1 - SensorMov.yaml`) a la carpeta de blueprints de Home Assistant, normalmente:
   - `config/blueprints/automation/<tu_usuario>/`
2. En **Ajustes → Automatizaciones y escenas → Blueprints**, crea una automatización a partir del blueprint.
3. Crea y selecciona las entidades requeridas (`input_select` obligatorios; `input_boolean`, sensores, switches y climates según tu instalación).
4. Guarda la automatización. Si hace falta, recarga automatizaciones o reinicia Home Assistant.

## Configuración

### Sensores

- **Sensor de Puerta** (`door_sensor`): `binary_sensor` (uno o varios) que detectan apertura/cierre.
- **Sensor de Movimiento** (`motion_sensor`): `binary_sensor` (uno o varios) para ocupación.

### Actuadores y climates

- **Actuadores a Encender** (`target_actuadoresOn`): switches que se encienden al iniciar ocupación (solo modo `auto`) y se apagan al pasar a libre (`auto` / `manual`).
- **Actuadores a Apagar** (`target_actuadoresOff`): switches que se apagan al pasar a libre (`auto` / `manual`).
- **Termostatos** (`target_climates`): climates configurados según ocupación y modos.

### Termostato

- **Temperatura SetPointOn** (`temperature_SetPointOn`): 16–30 °C (por defecto `24`). Se aplica al iniciar ocupación si `mode_thermostat` está `on` y `mode_plugin` es `auto`.
- **Modo de Ventilador SetPointOn** (`fanMode_SetPointOn`): `Auto low`, `Low`, `Medium`, `High` (por defecto `Auto low`). Solo se envía si el climate expone `fan_modes`.
- **Temperatura SetPointOff** (`temperature_SetPointOff`): 16–30 °C (por defecto `27`). Al pasar a libre si `mode_Setpointoff` está `on` y `mode_plugin` es `auto` — incluso si el AA fue apagado antes por puerta abierta prolongada.
- **Modo de Ventilador SetPointOff** (`fanMode_SetPointOff`): `Auto low`, `Low`, `Medium`, `High` (por defecto `Auto low`). Solo se envía si el climate expone `fan_modes`.

### Retardos y reintentos

| Input | Descripción | Valores / defecto |
| --- | --- | --- |
| **delay_off_sp** | Tiempo en `scan` antes de pasar a libre cuando la entrada fue por sensor de puerta (`libre` → `puertAbierta` → cierra → `scan`) | `120`, `300`, `600`, `900` (defecto), `1200`, `1800` (segundos) |
| **delay_off_actuadores** | Tiempo en `scan` antes de pasar a libre cuando la entrada fue por movimiento o por encender actuadores (`libre` → `scan`) | `120`, `300`, `600`, `900` (defecto), `1200`, `1800`, `3600` (segundos) |
| **delay_scan** | Tiempo en `scan` + evaluación del sensor de movimiento | `140`, `260` (defecto), `500`, `920` (segundos) |
| **delay_puertAb** | Tiempo con puerta abierta (`puertAbierta`) antes de apagar AA | 1–3600 s (por defecto `300`) |
| **delay_door_debounce** | La puerta debe permanecer abierta o cerrada este tiempo para disparar el trigger | 0–10 s (por defecto `0` = inmediato). Subir si hay rebotes que saturan la cola |
| **retry_time** | Intervalo entre reintentos para switches (Z-Wave/Zigbee), consultando estado antes de reenviar orden | 1–10 s (por defecto `1`) |

## Modos del plugin (`mode_plugin`)

| Modo | Comportamiento |
| --- | --- |
| **auto** | Control total: encendido al iniciar y apagado/ajuste al finalizar. Soporta SetPointOff. Apagado de AA por puerta abierta prolongada si `mode_puertAb` está activo. |
| **manual** | Solo apagado al finalizar (sin SetPointOff). No enciende nada al iniciar. |
| **apagado** | Solo actualiza variables de estado (`state_plugin` / `action_plugin`). No acciona equipos. El respaldo de 1 min en `libre` tampoco actúa en este modo. |

## Funcionamiento

La automatización opera en modo `queued` (cola máxima **20** ejecuciones) y reacciona a cambios de puerta, movimiento, actuadores y estados internos (`action_plugin`).

### Escenarios principales

1. **Puerta abierta (`libre` → `puertAbierta`)**: pasa a ocupado; en modo `auto` puede encender actuadores y termostatos según flags.
2. **Movimiento en libre (`libre` → `scan`)**: marca ocupación; en modo `auto` puede encender equipos según flags.
3. **Actuadores encendidos manualmente**: `libre` → `scan` → `ocupado`.
4. **Puerta cerrada**: desde `puertAbierta` o `librePuertaAb` pasa a `scan` y evalúa movimiento. Desde `librePuertaAb`, si el AA estaba `off` y `mode_thermostat` está activo, puede reencenderlo en modo `auto`.
5. **Inactividad prolongada**: tras `delay_off_sp` o `delay_off_actuadores`, pasa a `libre`. Los actuadores se apagan siempre (independiente de SetPointOff); el AA se apaga o ajusta según `mode_Setpointoff`.
6. **Puerta abierta prolongada**: si `mode_puertAb` está `on`, `mode_plugin` es `auto` y no hay movimiento, apaga AA tras `delay_puertAb`.
7. **`librePuertaAb`**: si la puerta sigue abierta al “liberar”, evita un `libre` incorrecto; movimiento o actuador On solo actualizan estados a `puertAbierta` + `ocupado` (sin encender equipos mientras la puerta sigue abierta).

### Robustez y optimizaciones

- **Comandos a climates**: solo apaga si hay alguno encendido; solo aplica SetPoint/Fan si procede; `set_fan_mode` solo en climates con `fan_modes`.
- **Switches**: reintentos (hasta 3) consultando estado antes de reenviar; `continue_on_error` para no bloquear la cola ante fallos puntuales.
- **Ayudantes opcionales**: ModoAct, ModoThermo, ModoPuertAb y ModoSetPointOff → `off` si no se anexan.
- **Debounce de puerta** (`delay_door_debounce`): filtra rebotes del sensor antes de disparar.
- **SetPointOff (v1.3.4.1+)**: temperatura de cortesía al quedar libre aunque el AA se hubiera apagado por puerta abierta.
- **Apagado de actuadores (v1.3.4.2.1)**: en `Scena_Apagar_a_libre` es independiente de la lógica SetPointOff del AA.

## Historial breve

| Versión | Cambio |
| --- | --- |
| **1.3.2.9** | Fix `librePuertaAb`: actuador On o movimiento en `librePuertaAb`+`libre` → `puertAbierta`+`ocupado` (solo estados). |
| **1.3.3.0** | `delay_off_sp` (entrada por puerta) y `delay_off_actuadores` (entrada por SM o actuadores). |
| **1.3.3.1** | Evita comandos redundantes en AA/termostatos. |
| **1.3.3.1.2** | Reintentos para switches (Z-Wave/Zigbee). |
| **1.3.3.1.3** | Evalúa variables de modo antes de accionar (`off` por defecto si no anexadas). |
| **1.3.4** | `mode_plugin` con `auto` / `manual` / `apagado` y reglas de actuación por modo. |
| **1.3.4.1** | SetPointOff: AA pasa a temperatura de cortesía al liberar incluso si fue apagado por puerta abierta. |
| **1.3.4.2** | Debounce puerta (default 0), `set_fan_mode` condicional, `continue_on_error` en switches, reencendido AA al cerrar desde `librePuertaAb`, cola `max: 20`. |
| **1.3.4.2.1** | Ayudantes boolean opcionales explícitos (OFF si no anexados). Apagado de actuadores al liberar independiente de SetPointOff. |

## Notas

- Usa entidades válidas y disponibles en Home Assistant.
- Revisa los logs de Home Assistant si algo no dispara o no apaga/enciende.
- Archivo de referencia actual: `Version Actual/_Bp-Plugin Hotel v1.3.4.2.1 - SensorMov.yaml`.

## Licencia

MIT. Puedes usar y modificar el blueprint según tus necesidades.

## Contribuciones

1. Haz fork del repositorio.
2. Aplica tus cambios.
3. Abre un pull request con una descripción clara.

## Contacto

Para soporte o errores, abre un issue en el repositorio.
