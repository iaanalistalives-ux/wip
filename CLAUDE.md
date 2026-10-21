# CLAUDE.md — WIP de Muestras e Inventario de telas (wip)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Seguimiento de muestras (protos) de Planta Barranco, por área.
- `index.html` — **el sistema WIP** completo:
  - Menú: ingresar requerimiento Comercial (se pegan las filas copiadas del Excel de gestión, desde la columna Proto), ingresar requerimiento Desarrollo, Programa WIP ("Por entregar", con filtros e impresión) y Búsqueda de estilos (historial y Excel).
  - Ficha por estilo: hitos (TELA_LLEGO, MOLDE, CORTE, COSTURA, ENTREGA), novedades (SERVICIO, VOLVIO, REPROCESO, ANULAR, STANDBY, CANCELAR), edición, comentarios y esperas.
  - Modo por área con `?area=`: corte, costura, inspeccion, servicio, comercial (seguimiento de protos, solo mira), moldes (comenta moldes) e ingenieria (formulario de Desarrollo, fase "Mock Up Ingenieria").
  - Excel "Descargar Excel" (SheetJS): hojas BASE e INFO, archivo `WIP_<filtro>_<fecha>.xlsx`.
- `comercial.html`, `corte.html`, `costura.html`, `ingenieria.html`, `inspeccion.html`, `moldes.html`, `servicio.html` — solo **redirigen** a `index.html?area=<área>` (una entrada por área en el portal).
- `telas.html` — **Inventario de telas**: registrar bultos (partida, cliente, artículo, color, peso, ubicación E1–E7), buscar y editar, y devolución (Desarrollo marca, Corte confirma la salida, Desarrollo descarga el Excel LIVES `DEVOLUCION_TELAS_<fecha>.xlsx`). Se abre también como ventana desde el área Corte del WIP.
- `index.html` todavía usa la forma antigua `google.script.run`, pero es una imitación: por dentro todo va a la base. No queda Apps Script ni Sheet.

## Base de datos (Supabase lives-datos)
- WIP, leer: `wm_inicio` (permisos de la persona), `wm_datos` (carga todo: `wm_ordenes`, `wm_base`, `wm_movimientos`, `wm_clientes`, `wm_personas`, `wm_config`, `wm_feriados`, `wm_fases_dias`, `wm_servicio_cliente`), `wm_ficha_datos`, `wm_registros_etapas` (Excel), `wm_ultimo_comercial`, `wm_catalogo_telas`.
- WIP, registrar: `wm_registrar`, `wm_novedad`, `wm_editar`, `wm_confirmar_fecha`, `wm_avios`, `wm_confirmar_servicio`, `wm_comentario`, `wm_quitar_comentarios`, `wm_guardar_ingreso`, `wm_guardar_desarrollo`, `wm_espera`, `wm_fin_espera` (tablas `wm_ordenes`, `wm_base`, `wm_movimientos`; `wm_tokens` evita guardar dos veces).
- Telas: `tl_inicio` (leer) y `tl_registrar`, `tl_editar`, `tl_marcar`, `tl_desmarcar`, `tl_devolver` (tabla `tl_telas`, historial en `tl_movimientos`).
- No usa Storage ni lee tablas directo.

## Quién la usa
Según las entradas por el portal de los últimos 30 días al 21/10/2026: 9 personas, sobre todo de Desarrollo y Comercial (gerente y analistas de Desarrollo, analistas comerciales, digitadora, responsable de registro).

## Quién puede registrar
- WIP: lo decide `wm_inicio` (área o áreas que registra cada persona y si es de Desarrollo). Sin área, la pantalla abre solo para consultar; comercial y moldes se pueden mirar siempre.
- Telas: lo decide `tl_inicio` con el rol DESARROLLO, CORTE o VER (solo consulta y Excel).
- A Sistemas: `wm_inicio` (con `wm_ve_`) y `tl_rol_` son las piezas que reemplazan con su seguridad.

## Pendientes vistos al escribir esto (21/10/2026)
- El Excel del WIP sale con SheetJS (sin formato LIVES); el de Telas ya tiene formato LIVES.
- Comentarios viejos que hablan de Apps Script y Sheet (incluida la nota de `URL_TELAS` en `index.html`).
