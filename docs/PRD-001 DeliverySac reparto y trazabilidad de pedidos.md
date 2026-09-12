# PRD-001: DeliverySac - reparto y trazabilidad de pedidos

## Contexto y Problema
En la empresa DeliverySac repartimos con 5 unidades (3 motos lineales y 2 furgonetas). Hoy el administrador asigna clientes y pedidos manualmente, 
sin visibilidad de dónde están las unidades, qué pedidos se entregaron ni a qué hora. 
Al final del día solo hay control cuando retornan las unidades: demoras, pérdida de trazabilidad y cero métricas. 
No tenemos presupuesto ni tiempo para un ERP de reparto; necesitamos algo simple que se haga cargo del control operativo diario.

Personas:
- Administrador de reparto: asigna pedidos cada mañana y quiere ver el estado sin llamar a nadie.
- Repartidor: necesita ver su ruta en el celular y marcar entregas rápido, sin fricción.
- Supervisor: consulta el reporte del día para tomar decisiones.

## Objetivos
Que al final del día el 100% de los pedidos asignados tengan un estado final (entregado, no entregado o reprogramado) registrado en el sistema, 
sin llamadas telefónicas. 
El Flujo es crea pedido → asigna → repartidor marca entregado → admin ve el estado actualizado.

## Requerimientos Funcionales
- RF-01: El sistema debe permitir iniciar sesión con usuario y contraseña.
- RF-02: El sistema debe permitir al administrador registrar un cliente con nombre y dirección.
- RF-03: El sistema debe permitir al administrador crear un pedido con cabecera (cliente, fecha de reparto, comentario) y una o más líneas de detalle (producto, cantidad, precio unitario).
- RF-04: El sistema debe calcular el monto total del pedido como la suma de cantidad × precio unitario de sus líneas.
- RF-05: El sistema debe permitir al administrador asignar un pedido pendiente a un repartidor y una unidad en una fecha.
- RF-06: El sistema debe mostrar al repartidor únicamente los pedidos asignados a él en la fecha actual.
- RF-07: El sistema debe mostrar al repartidor, para cada pedido, la cabecera y el detalle de líneas.
- RF-08: El sistema debe permitir al repartidor cambiar el estado de un pedido a EN_RUTA, ENTREGADO o NO_ENTREGADO.
- RF-09: El sistema debe mostrar al administrador el listado de pedidos del día con su estado actualizado.
- RF-10: El sistema debe paginar los listados con los parámetros page y size.

## Requerimientos No Funcionales
- RNF-01: El 95% de las peticiones debe responder en ≤ 2 segundos en red 4G.
- RNF-02: La interfaz del repartidor debe ser responsive y usable desde 320px de ancho.
- RNF-03: Las contraseñas deben almacenarse con hash bcrypt (coste ≥ 10).
- RNF-04: Toda comunicación debe usar HTTPS/TLS 1.2 o superior.
- RNF-05: El sistema debe funcionar sin depender de servicios externos.

## Criterios de Aceptación
- AC-01 (RF-01): Dado un usuario activo con credenciales válidas, cuando ingresa usuario y contraseña, entonces accede a su panel según rol.
- AC-02 (RF-03): Dado un administrador autenticado, cuando crea un pedido con dos líneas (2×10.00 y 1×5.00), entonces el pedido se guarda y el total es 25.00.
- AC-03 (RF-03): Dado un administrador que intenta guardar un pedido sin líneas, cuando confirma, entonces el sistema responde HTTP 400 y no lo crea.
- AC-04 (RF-05): Dado un pedido pendiente, cuando el administrador lo asigna a un repartidor y unidad, entonces el pedido cambia a asignado y aparece en la ruta del repartidor.
- AC-05 (RF-06): Dado un repartidor autenticado con 3 pedidos asignados hoy, cuando abre la vista web en su celular, entonces ve exactamente esos 3 pedidos.
- AC-06 (RF-08): Dado un pedido asignado, cuando el repartidor lo marca como ENTREGADO, entonces el estado cambia y el administrador lo ve reflejado en su listado.
- AC-07 (RF-10): Dados más de 20 pedidos, cuando se listan, entonces se devuelven paginados de a 20 (parámetros page/size).

## Fuera de Alcance
Reportes PDF/Excel · cierre de día · registro público de usuarios · GPS o ubicación en tiempo real · auditoría avanzada · multiempresa/multi-sucursal · 
app móvil nativa · funcionalidad offline · notificaciones, chat, integraciones con ERP, WhatsApp.

## Riesgos y Dependencias
- Riesgo: Que el administrador asigne un pedido a un repartidor y luego lo reasigne por error, dejando pedidos duplicados en dos rutas.
	→ Mitigación: Validación al reasignar: el pedido solo puede pertenecer a una asignación activa; si se reasigna, se elimina la anterior. 	
- Riesgo: que el repartidor no visualice bien la vista móvil → mitigación: diseño mobile-first y pruebas en pantalla de 5”.
- Riesgo: Que se cree un pedido con líneas de detalle vacías o con cantidades/precios inválidos (negativos, cero).
	→ Mitigación: Validaciones en backend y frontend: al menos una línea, cantidad > 0, precio ≥ 0, descripción obligatoria. Devolver HTTP 400 con mensaje claro.
- Riesgo: errores en la asignación manual de pedidos 
	→ mitigación: validaciones en el panel (no asignar pedidos ya entregados).
- Riesgo: Que un administrador edite un pedido que ya está en estado ASIGNADO, EN_RUTA o ENTREGADO, corrompiendo la trazabilidad.
	→ Mitigación: El backend rechaza la edición si el estado no es PENDIENTE. La interfaz oculta los botones de edición para esos estados.
- Dependencia: API de Claude.
- Dependencia: conexión 4G estable para actualizar estados.
