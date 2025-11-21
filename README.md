# Custom Palletizing — Script Overview

Resumen corto
- Script principal: [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl) — contiene la lógica completa de pick/place y gestión del lifter.
- Utilidad para ajustar frames cuando sube el lifter: [scripts/up_frame.drl](scripts/up_frame.drl).
- Proyecto RoboDK (archivo de escena / datos binarios): [RoboDK/custom-palletizing.rdk](RoboDK/custom-palletizing.rdk).

Cómo funciona (alto nivel)
- El punto de entrada es la función [`main`](scripts/custom_palletizing.drl) — inicializa TCP/tool, verifica lifter y aplica compensación de frames.
- Flujo de paletizado por lado:
  - Usuario selecciona lado/cama/slot.
  - Se ejecuta la secuencia en [`run_from`](scripts/custom_palletizing.drl) que recorre capas y slots.
  - Para cada pieza: [`do_pick`](scripts/custom_palletizing.drl) toma la pieza desde el conveyor y [`do_place`](scripts/custom_palletizing.drl) la coloca en el pallet.
  - Lógica Half-A/B: orígenes definidos en [`half_plan_left`]/[`half_plan_right`]; `do_half` (en [`do_half`](scripts/custom_palletizing.drl)) gestiona colocar Half-A y luego Half-B en el destino.
- Lifter (Lift100):
  - Niveles lógicos por capa con función [`lifter_level_for_layer`](scripts/custom_palletizing.drl).
  - Mover y sincronizar: [`lifter_set_level`](scripts/custom_palletizing.drl) y helper interno [`_lifter_do_set_level`](scripts/custom_palletizing.drl).
  - Frames de trabajo (IDs 101/102/103) se recalculan desde frames originales (191/192/193) por [`apply_level_to_frames`](scripts/custom_palletizing.drl) usando el offset de nivel.

Archivos y símbolos clave
- [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl)
  - [`main`](scripts/custom_palletizing.drl)
  - [`run_from`](scripts/custom_palletizing.drl)
  - [`do_pick`](scripts/custom_palletizing.drl)
  - [`do_place`](scripts/custom_palletizing.drl)
  - [`do_half`](scripts/custom_palletizing.drl)
  - [`find_half`](scripts/custom_palletizing.drl)
  - [`is_half_destination`](scripts/custom_palletizing.drl)
  - [`lifter_set_level`](scripts/custom_palletizing.drl)
  - [`apply_level_to_frames`](scripts/custom_palletizing.drl)
  - [`lifter_restore_frames`](scripts/custom_palletizing.drl)
  - Datos: `left_layers`, `right_layers` (posición de PLACE por capa/slot) — en [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl)
  - Planes Half: `half_plan_left`, `half_plan_right` — en [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl)
  - Mid / Joint targets: `mid_left_a`, `mid_right_a`, `mid_left_b`, `mid_right_b` — en [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl)
- [scripts/up_frame.drl](scripts/up_frame.drl)
  - Utilidad para compensar frames tras mover físicamente el lifter (`compensar_frame_pallet` / `main` del script) — revisar antes de ejecutar en máquina real.
- [RoboDK/custom-palletizing.rdk](RoboDK/custom-palletizing.rdk)
  - Contiene la escena y datos exportados por RoboDK (binario). No editar a mano.

Parámetros importantes (configuración rápida)
- IDs de frames (editar si cambian en controlador): `conv_frame = 101`, `left_frame = 102`, `right_frame = 103` — ver [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl).
- Frames originales nivel 0: `conv_frame_orig = 191`, `left_frame_orig = 192`, `right_frame_orig = 193`.
- Offsets APPROACH/RETRACT: `dz_app`, `dz_ret`, `dx_app`, `dy_app` — se usan en `get_app`, `get_ret`.
- Velocidades y aceleraciones: `vj_fast`, `vj_slow`, `vl_fast`, `vl_slow`, etc. — ajustar según instalación.

Ejecutar (instrucciones)
1. Asegurar que la herramienta `custom_gripper` y `gripper_shape` están definidos en el controlador.
2. Abrir el proyecto en RoboDK / controlador Doosan y cargar [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl).
3. Ejecutar el script principal: llama a [`main`](scripts/custom_palletizing.drl).
4. Seguir las indicaciones del teach pendant para seleccionar lado, capa y slot.

Puntos de seguridad / limitaciones
- Las comprobaciones de sensores del conveyor y los detectores de tarima están comentadas (placeholders). Revisar funciones: `wait_conveyor_ready`, `wait_remove_finished_pallet`, `wait_new_pallet` en [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl).
- `lifter_safe_init` / `lifter_check_position` dependen del objeto `or_lift` (API del Lift100). Verificar conexión física antes de operar.
- El script modifica frames de trabajo temporalmente vía `overwrite_user_cart_coord(..., DR_TEMPORARY)`. Los frames originales nunca se modifican por código.
- Test en modo simulado o con movimientos lentos antes de correr en producción.

Resolución de problemas rápida
- Si las posiciones no coinciden tras subir el lifter: ejecutar [scripts/up_frame.drl](scripts/up_frame.drl) para compensar el frame manualmente.
- Si el lifter no responde: comprobar `or_lift.isconn()` y revisar `lifter_safe_init` en [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl).
- Para rastrear flujo: `tp_log` aparece en múltiples puntos (p. ej. en [`do_half`](scripts/custom_palletizing.drl) y [`run_from`](scripts/custom_palletizing.drl)).

Contacto rápido
- Ver los comentarios y docstrings dentro de [scripts/custom_palletizing.drl](scripts/custom_palletizing.drl) para detalles por función.