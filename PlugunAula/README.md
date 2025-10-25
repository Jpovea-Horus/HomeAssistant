# Bp-Plugin Aula versiones 1...

## Descripción
Este blueprint para Home Assistant permite gestionar la ocupación y el control unificado de switches y termostatos en un entorno como un aula. La automatización utiliza un sensor de movimiento para detectar ocupación y controla los dispositivos según el estado (`libre`, `ocupado`, `scan`) y el modo del plugin (`auto` o `manual`). Es ideal para optimizar el uso de energía en espacios con ocupación variable.

## Requisitos
Para que la automatización funcione correctamente, configura los siguientes elementos en Home Assistant:

### Input Select
Los siguientes `input_select` deben tener las opciones exactas indicadas:
- **state_plugin**: Opciones: `libre`, `ocupado`
- **state_internal**: Opciones: `libre`, `ocupado`, `scan`
- **mode_plugin**: Opciones: `auto`, `manual`

### Input Boolean
Los siguientes `input_boolean` deben estar configurados (valores `on`/`off`):
- **mode_activation**: Habilita/deshabilita la activación de switches al iniciar la ocupación.
- **mode_thermostat**: Habilita/deshabilita el control de termostatos al iniciar la ocupación.

**Nota**: Si las opciones no están configuradas correctamente, la automatización puede no funcionar como se espera.

## Instalación
1. Copia el contenido del archivo YAML proporcionado en tu carpeta de automatizaciones de Home Assistant (normalmente en `automations.yaml` o una subcarpeta).
2. Configura las entidades requeridas en Home Assistant (sensor de movimiento, switches, termostatos, etc.).
3. Asegúrate de que los `input_select` e `input_boolean` estén creados con las opciones indicadas.
4. Reinicia Home Assistant o recarga las automatizaciones desde la interfaz.

## Configuración
La automatización utiliza las siguientes entradas configurables:

### Sensores
- **Sensor de Movimiento** (`motion_sensor`): Sensor binario (tipo `switch`) que detecta ocupación al cambiar de estado.

### Actuadores
- **Switches a Encender** (`target_switchesOn`): Lista de switches que se encenderán al detectar ocupación y se apagarán al pasar a estado libre.
- **Switches a Apagar** (`target_switchesOff`): Lista de switches que se apagarán cuando el estado sea libre.
- **Termostatos a Encender** (`target_climates`): Lista de termostatos que se configurarán según la ocupación.

### Configuraciones de Termostato
- **Temperatura Objetivo** (`target_temperature`): Temperatura a establecer cuando los termostatos se encienden (16–30 °C, por defecto 24 °C).
- **Modo de Ventilador** (`fan_mode`): Modo de ventilador para los termostatos (opciones: `Auto low`, `Low`, `Medium`, `High`, por defecto `Auto low`).

### Retardos
- **Retardo para Apagar** (`off_delay`): Tiempo de espera antes de apagar los equipos tras inactividad (1–60 minutos, por defecto 15 minutos).
- **Retardo de Switches** (`switch_delay`): Tiempo de espera antes de apagar switches si están encendidos en estado libre (1–60 minutos, por defecto 10 minutos).

## Funcionamiento
La automatización utiliza varios desencadenantes (`triggers`) basados en el estado del sensor de movimiento, los switches, y las variables internas. Según el estado del plugin (`state_plugin`), el estado interno (`state_internal`) y el modo del plugin (`mode_plugin`), se ejecutan las siguientes acciones:

- **Encendido de dispositivos**: Cuando el sensor de movimiento detecta ocupación (`libre` → `ocupado`), se encienden los switches y termostatos (si `mode_plugin` es `auto`, `mode_activation` y `mode_thermostat` están en `on`).
- **Modo scan**: Si el sensor de movimiento se apaga, el estado interno pasa a `scan` para verificar si la ocupación persiste.
- **Apagado de dispositivos**: Tras un período de inactividad (`off_delay`) o si los switches están encendidos en estado libre por el tiempo especificado (`switch_delay`), el estado pasa a `libre`, apagando switches y termostatos.
- **Control manual**: En modo `manual`, las acciones automáticas de encendido/apagado se desactivan, permitiendo control manual.
- **Control por estado del plugin**: Si el plugin cambia a `libre` (por ejemplo, desde la interfaz web), los dispositivos se apagan si `mode_plugin` es `auto`.

## Escenarios Principales
1. **Detección de movimiento (`libre` → `ocupado`)**: Enciende switches y termostatos (si modo automático).
2. **Sensor de movimiento apagado (`ocupado` → `scan`)**: Cambia al estado `scan` para verificar ocupación.
3. **Movimiento detectado en scan (`scan` → `ocupado`)**: Vuelve al estado `ocupado`.
4. **Inactividad prolongada (`scan` → `libre`)**: Apaga switches y termostatos tras el retardo configurado.
5. **Switches encendidos en estado libre**: Apaga switches y termostatos tras el retardo `switch_delay`.
6. **Cambio manual a libre**: Apaga todos los dispositivos si el modo es automático.

## Notas
- La automatización opera en modo `single`, procesando un desencadenante a la vez.
- Asegúrate de que las entidades seleccionadas (sensor de movimiento, switches, termostatos) sean válidas y estén correctamente configuradas en Home Assistant.
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