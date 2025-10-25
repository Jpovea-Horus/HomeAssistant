# Bp-Plugin Hotel versiones 1...

## Descripción
Esta automatización para Home Assistant está diseñada para gestionar la ocupación en una habitación, como en un entorno de hotel. Controla switches y termostatos en función de sensores de puerta y movimiento, permitiendo un manejo eficiente de los dispositivos según el estado de ocupación (libre, ocupado, etc.) y los modos configurados (automático o manual).

## Requisitos
Para que la automatización funcione correctamente, asegúrate de configurar los siguientes elementos en Home Assistant:

### Input Select
Los siguientes `input_select` deben tener las opciones exactas indicadas:
- **state_plugin**: Opciones: `libre`, `ocupado`
- **state_internal**: Opciones: `libre`, `ocupado`, `scan`, `puertAbierta`
- **mode_plugin**: Opciones: `auto`, `manual`

### Input Boolean
Los siguientes `input_boolean` deben estar configurados (valores `on`/`off`):
- **ModoPlg**: Habilita/deshabilita el modo del plugin.
- **ModoAct**: Activa/desactiva los actuadores al iniciar la ocupación.
- **ModoThermo**: Habilita/deshabilita el control de termostatos al iniciar la ocupación.
- **ModoPuertAb**: Habilita/deshabilita el apagado de aire acondicionado si la puerta está abierta por un tiempo prolongado.
- **ModoSetPointOff**: Habilita/deshabilita el control de temperatura al apagar la ocupación.

**Nota**: Si las opciones no están configuradas correctamente, la automatización puede no funcionar como se espera.

## Instalación
1. Copia el contenido del archivo YAML proporcionado en tu carpeta de automatizaciones de Home Assistant (normalmente en `automations.yaml` o en una subcarpeta).
2. Configura las entidades requeridas en Home Assistant (sensores, switches, termostatos, etc.).
3. Asegúrate de que los `input_select` e `input_boolean` estén creados con las opciones indicadas.
4. Reinicia Home Assistant o recarga las automatizaciones desde la interfaz.

## Configuración
La automatización utiliza las siguientes entradas configurables:

### Sensores
- **Sensor de Puerta** (`door_sensor`): Sensores binarios que detectan la apertura/cierre de la puerta.
- **Sensor de Movimiento** (`motion_sensor`): Sensores binarios que detectan movimiento para determinar la ocupación.

### Actuadores
- **Switches a Encender** (`target_switchesOn`): Lista de switches que se encenderán al iniciar la ocupación y se apagarán al pasar a estado libre.
- **Switches a Apagar** (`target_switchesOff`): Lista de switches que se apagarán cuando el estado sea libre.
- **Termostatos a Encender** (`target_climates`): Lista de termostatos que se configurarán según la ocupación.

### Configuraciones de Termostato
- **Temperatura SetPointOn** (`temperature_SetPointOn`): Temperatura a establecer cuando los termostatos se encienden (16–30 °C, por defecto 24 °C).
- **Modo de Ventilador SetPointOn** (`fanMode_SetPointOn`): Modo de ventilador al iniciar ocupación (opciones: `Auto low`, `Low`, `Medium`, `High`).
- **Temperatura SetPointOff** (`temperature_SetPointOff`): Temperatura a establecer al apagar ocupación si `mode_Setpointoff` está activo (16–30 °C, por defecto 27 °C).
- **Modo de Ventilador SetPointOff** (`fanMode_SetPointOff`): Modo de ventilador al apagar ocupación (opciones: `Auto low`, `Low`, `Medium`, `High`).

### Retardos
- **Retardo Apagar** (`delay_off`): Tiempo de espera antes de apagar equipos tras inactividad (1–60 minutos, por defecto 15 minutos).
- **Retardo de Scan al SM** (`delay_scan`): Tiempo de espera para escanear si el sensor de movimiento está activo (1–60 minutos, por defecto 5 minutos).
- **Retardo de Switches** (`delay_switch`): Tiempo de espera antes de apagar switches en estado libre (1–60 minutos, por defecto 10 minutos).
- **Retardo para Apagar AA** (`delay_puertAb`): Tiempo de espera por puerta abierta antes de apagar el aire acondicionado (1–60 minutos, por defecto 5 minutos).

## Funcionamiento
La automatización utiliza varios desencadenantes (`triggers`) basados en los estados de los sensores de puerta, movimiento, switches y variables internas. Según el estado del plugin (`state_plugin`), el estado interno (`state_internal`) y el modo del plugin (`mode_plugin`), se ejecutan diferentes acciones, como:

- **Encendido de dispositivos**: Cuando la habitación pasa de `libre` a `ocupado` (por apertura de puerta o detección de movimiento), se encienden los switches y termostatos si el modo automático está activo.
- **Apagado de dispositivos**: Cuando la habitación pasa a `libre` (tras inactividad o puerta abierta por un tiempo prolongado), se apagan los switches y, opcionalmente, los termostatos, o se ajustan a una temperatura de apagado (`SetPointOff`).
- **Gestión de puerta abierta**: Si la puerta permanece abierta por más del tiempo configurado (`delay_puertAb`) y `mode_puertAb` está activo, los termostatos se apagan.
- **Modo manual/automático**: En modo manual, las acciones automáticas de encendido/apagado se desactivan, permitiendo control manual.

## Escenarios Principales
1. **Puerta abierta (`libre` → `puertAbierta`)**: Cambia el estado a `ocupado`, enciende switches y termostatos (si modo automático).
2. **Detección de movimiento (`libre` → `scan`)**: Cambia el estado a `ocupado`, enciende switches y termostatos (si modo automático).
3. **Switches encendidos manualmente**: Cambia el estado a `scan` y luego a `ocupado`.
4. **Puerta cerrada**: Cambia de `puertAbierta` o `libre` a `scan` y verifica movimiento.
5. **Inactividad prolongada**: Cambia a `libre`, apaga switches y ajusta/apaga termostatos según configuración.
6. **Puerta abierta prolongada**: Apaga termostatos si `mode_puertAb` está activo.

## Notas
- La automatización opera en modo `queued`, procesando los desencadenantes en orden.
- Asegúrate de que las entidades seleccionadas (sensores, switches, termostatos) sean válidas y estén correctamente configuradas en Home Assistant.
- Si necesitas soporte o encuentras errores, revisa los logs de Home Assistant o crea un issue en este repositorio.

## Licencia
Este proyecto está licenciado bajo la [Licencia MIT](LICENSE). Siéntete libre de usarlo y modificarlo según tus necesidades.

## Contribuciones
¡Las contribuciones son bienvenidas! Si deseas mejorar este blueprint, por favor:
1. Crea un fork del repositorio.
2. Realiza tus cambios.
3. Envía un pull request con una descripción clara de las mejoras.

## Contacto
Para preguntas o soporte, crea un issue en el repositorio o contacta a través de [tu canal preferido, si aplica].
