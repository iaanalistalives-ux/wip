# CLAUDE.md — WIP de Muestras e Inventario de telas (wip)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Seguimiento de muestras (protos) de Planta Barranco, por área.
- `index.html` — **el sistema WIP** completo:
  - Menú: ingresar requerimiento Comercial (se pegan las filas copiadas del Excel de gestión, desde la columna Proto), ingresar requerimiento Desarrollo, Programa WIP ("Por entregar", con filtros e impresión) y Búsqueda de estilos (historial y Excel).
  - Ficha por estilo: hitos (TELA_LLEGO, MOLDE, CORTE, COSTURA, ENTREGA), novedades (SERVICIO, VOLVIO, REPROCESO, ANULAR, STANDBY, CANCELAR), edición, comentarios y esperas.
  - Modo por área con `?area=`: corte, costura, inspeccion, servicio, comercial (seguimiento de protos, solo mira), moldes (comenta moldes) e ingenieria (formulario de Desarrollo, fase "Mock Up Ingenieria").
  - Seguimiento de protos (Comercial), desde el 23/11/2026: filtro «Ver» (En proceso por defecto / Entregados / Todos). Los entregados van agrupados por fecha de entrega, el más reciente arriba. La ruta («Cómo va») de un entregado termina en «ENTREGADO — fecha»: la del hito Entregado (`despacho_inicial`) o, en los que vienen del Excel antiguo sin hito, `xf_real`. La fecha DDP es la «Fecha comercial» (`fecha_comercial`, aclaración de José, 23/11/2026): sale como punto propio «FECHA COMERCIAL (DDP)» antes del último punto, con la fecha vigente; cada cambio queda como «FECHA COMERCIAL CAMBIADA — de … a …» con su motivo. La columna «Entrega DDP» del Excel de Comercial no se guarda al pegar.
  - Excel "Descargar Excel" (SheetJS): hojas BASE e INFO, archivo `WIP_<filtro>_<fecha>.xlsx`.
- `comercial.html`, `corte.html`, `costura.html`, `ingenieria.html`, `inspeccion.html`, `moldes.html`, `servicio.html` — solo **redirigen** a `index.html?area=<área>` (una entrada por área en el portal).
- `telas.html` — **Inventario de telas**: registrar bultos (partida, cliente, artículo, color, peso, ubicación E1–E7), buscar y editar, y devolución (Desarrollo marca, Corte confirma la salida, Desarrollo descarga el Excel LIVES `DEVOLUCION_TELAS_<fecha>.xlsx`). Se abre también como ventana desde el área Corte del WIP.
- `index.html` todavía usa la forma antigua `google.script.run`, pero es una imitación: por dentro todo va a la base. No queda Apps Script ni Sheet.

## Base de datos (Supabase lives-datos)
- WIP, leer: `wm_inicio` (permisos de la persona), `wm_datos` (carga todo: `wm_ordenes`, `wm_base`, `wm_movimientos`, `wm_clientes`, `wm_personas`, `wm_config`, `wm_feriados`, `wm_fases_dias`, `wm_servicio_cliente`), `wm_ficha_datos`, `wm_registros_etapas` (Excel), `wm_ultimo_comercial`, `wm_catalogo_telas`.
- WIP, registrar: `wm_registrar`, `wm_novedad`, `wm_editar`, `wm_confirmar_fecha`, `wm_avios`, `wm_confirmar_servicio`, `wm_comentario`, `wm_quitar_comentarios`, `wm_guardar_ingreso`, `wm_guardar_desarrollo`, `wm_espera`, `wm_fin_espera` (tablas `wm_ordenes`, `wm_base`, `wm_movimientos`; `wm_tokens` evita guardar dos veces).
- Telas: `tl_inicio` (leer) y `tl_registrar`, `tl_editar`, `tl_marcar`, `tl_desmarcar`, `tl_devolver` (tabla `tl_telas`, historial en `tl_movimientos`).
- Selección desde la base (desde el 11/11/2026): lo que existe en `ops` o `partidas` se elige, no se escribe; si no está, se toma de lo ya registrado; un proto sin OP (su número de estilo no está en `ops`) se puede escribir. Lectura `wm_catalogo_base`; `wm_guardar_desarrollo` y `wm_editar` validan código, color y partida con `wm_valida_base_` (solo lo que se envía o se cambia). En Requerimiento de Desarrollo el estilo va en dos casillas (número y versión). No aplica al ingreso de Comercial (pegar del Excel) ni al Inventario de telas.
- Clientes desde el catálogo general (desde el 17/11/2026, Wm17): la lista de clientes que devuelve `wm_datos` es `wm_clientes` más los activos de `cat_clientes` que no estén ya (por sigla = prefijo o por nombre). `wm_sigla_` y `wm_cliente_oficial_` buscan también en `cat_clientes`. Un cliente nuevo creado en Catálogos aparece solo en el WIP, con su prefijo como sigla (grupo `PREFIJO-000`). Los clientes de muestras que solo existen en `wm_clientes` se quedan ahí. Los prefijos escritos en `wm_config.CLIENTES_NO_MUESTRAS` (separados por coma; hoy `QMA`, QUIMA es solo de auditores) no salen en la lista del WIP (Wm18). `index.html` no cambió.
- Puertas únicas (desde el 23/11/2026, Wm20): `wm_api(p_accion, p)` y `tl_api(p_accion, p)`, SECURITY INVOKER (cada acción conserva los permisos de su función: leer sin usuario, guardar con sesión). La acción es el nombre de la función sin prefijo (`datos`, `registrar`, `ficha_datos` con `p.orden`…; en Telas `marcar`/`desmarcar`/`devolver` reciben `p.ids`, `p.obs`, `p.fecha`). Las pantallas ya llaman por la puerta (publicado el 23/11/2026). Las funciones de siempre siguen existiendo.
- Telas tiene `cliente_cod` (Wm20): sale de la sigla del grupo (columna `cliente`) con `tl_cliente_cod_` y el disparador `tg_tl_cliente_cod`; THYM = THY; nunca crea códigos nuevos (RES y VAR quedan vacíos hasta que José responda).
- Limpieza de datos con las respuestas de José Palacios (26/11/2026, registro `An28`, respaldo en `respaldo_an28`):
  - Ejecutiva comercial: el Excel de Comercial trae el usuario (columna Usuario). `wm_guardar_ingreso` lo traduce al nombre de Sistemas con `wm_ejecutiva_` y la tabla `wm_ejecutivas` (usuario, nombre; cerrada a la web). Un usuario que no está en la tabla se guarda tal cual, en mayúsculas: si entra una ejecutiva nueva, se agrega su fila ahí.
  - Fases sugeridas al escribir (`fasesUso` de `wm_datos`): salen de `wm_ordenes.fase` (ya no de `wm_base`, que guarda la fila original tal como se pegó y no se toca) y se omiten las de `wm_config.FASES_NO_VAN` (separadas por coma; función `wm_fase_no_va_`). La casilla sigue siendo de texto libre: no bloquea escribirlas. Las órdenes antiguas con esas fases conservan su fase.
  - Nombres de fase acordados: Size Set, Wear Test (15 días, palabra `WEAR TEST` en `wm_fases_dias`), Pre Top Sample, PP Reference, Prenda OTM, Mock Up (el número o nombre va en la observación). «Mock Up Ingenieria» sigue sin tilde porque está escrita así en `index.html`.
  - Grupos: THYM va SIN guion (THYM1084); VERONICA BEARD es VB-000 (sigla VB en `wm_clientes`). Personas (analista, ejecutiva, historial) con el nombre de Sistemas. Los cortadores siguen por código (9714, A330, A823).
- Fase por lista cerrada y cortador con nombre (26/11/2026, registro `Wm22`; respuestas de la parte 4 de José Palacios):
  - Catalogo `wm_fases` (codigo interno `FAS-01`..., nombre, orden, activo): 22 fases. `wm_datos` lo entrega en `fasesCat`. Para agregar o retirar una fase se cambia esa tabla (activo), no el HTML.
  - `wm_guardar_desarrollo` y `wm_editar` rechazan una fase fuera de la lista (`wm_fase_oficial_`), solo cuando se escribe o se cambia; una orden antigua con otra fase se puede seguir editando. Ingenieria conserva su fase fija. El pegado de Comercial no se rechaza: la pantalla avisa en la revision y se graba igual.
  - En `index.html` la fase del Requerimiento de Desarrollo es una lista (antes texto libre) y Editar usa la misma lista.
  - Cortadores: `wm_personas.nombre_completo` (9714 JORGE BERNABE, A330 ROBERT GUTIERREZ, A823 JHONNY MACHACA); `wm_datos` lo entrega en `personasNombre`. Se sigue guardando el codigo; el nombre solo se muestra al lado (campo del corte y ruta de la ficha).
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
