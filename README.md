## Descripción General
- **Aula vers**: Gestiona ocupación en un aula utilizando un sensor de movimiento para controlar switches y termostatos según los estados `libre`, `ocupado` y `scan`, con modos `auto` o `manual`. Ideal para optimizar el uso de energía en espacios con ocupación variable.
- **Hotel vers**: Gestiona ocupación en una habitación de hotel usando sensores de movimiento y puerta. Controla switches y termostatos según los estados `libre`, `ocupado`, `scan` y `puertAbierta`, con modos `auto` o `manual`. Incluye funciones avanzadas como manejo de puerta abierta y ajuste de temperatura al apagar.

## Requisitos
### Input Select
- **Aula**: 
  - `state_plugin`: `libre`, `ocupado`
  - `state_internal`: `libre`, `ocupado`, `scan`
  - `mode_plugin`: `auto`, `manual`
- **Hotel**: 
  - Igual que Aula, pero `state_internal` incluye `puertAbierta` entre otras adicionalmente.

## Funcionamiento
- **Aula**: 
  - Detecta ocupación vía sensor de movimiento, cambiando entre `libre`, `ocupado` y `scan`.
  - En modo `auto`, enciende switches y termostatos al detectar ocupación; los apaga tras inactividad (`off_delay`) o si los switches están encendidos en `libre` (`switch_delay`).
  - Modo `single`: Procesa un desencadenante a la vez.

- **Hotel**: 
  - Usa sensores de movimiento y puerta, con estados adicionales (`puertAbierta`).
  - En modo `auto`, enciende dispositivos al detectar ocupación (movimiento o puerta abierta); apaga switches y ajusta/apaga termostatos tras inactividad (`delay_off`) o puerta abierta prolongada (`delay_puertAb`).
  - Modo `queued`: Procesa desencadenantes en orden.
  - **Diferencia**: Hotel incluye lógica para apagar termostatos por puerta abierta y ajustar temperatura al apagar ocupación.

## Escenarios Principales
- **Aula**: 
  1. Movimiento detectado: `libre` → `ocupado`, enciende dispositivos.
  2. Sensor apagado: `ocupado` → `scan`.
  3. Movimiento en scan: `scan` → `ocupado`.
  4. Inactividad: `scan` → `libre`, apaga dispositivos.
  5. Switches encendidos en `libre`: Apaga tras `switch_delay`.
  6. Cambio manual a `libre`: Apaga dispositivos en modo `auto`.

- **Hotel**: 
  - Incluye los de Aula más:
    1. Puerta abierta: `libre` → `puertAbierta`, enciende dispositivos.
    2. Puerta cerrada: `puertAbierta`/`libre` → `scan`, verifica movimiento.
    3. Puerta abierta prolongada: Apaga termostatos si `mode_puertAb` está activo.

## Notas
- **Aula**: Más simple, centrada en sensores de movimiento. Ideal para entornos sin puertas controladas.

- **Hotel**: Más compleja, con gestión de puertas y ajustes de temperatura al apagar. Adecuada para entornos hoteleros.

- **Común**: Requiere entidades válidas y configuración correcta. Revisa logs de Home Assistant para depurar errores.

## Licencia
Ambos están licenciados bajo la [Licencia MIT](LICENSE).

## Contribuciones
Ambos proyectos aceptan contribuciones mediante forks y pull requests con descripciones claras de las mejoras.

## Contacto
Para soporte, crea un issue en el repositorio o usa el canal de contacto preferido.

---

**Resumen Final**: 
- **Aula version** es más simple, enfocado en aulas con control basado solo en movimiento. 
- **Hotel version** es más avanzado, ideal para hoteles, con soporte para sensores de puerta, gestión de puerta abierta y control de temperatura al apagar. Ambos optimizan el uso de energía, pero Hotel ofrece mayor flexibilidad para entornos complejos.