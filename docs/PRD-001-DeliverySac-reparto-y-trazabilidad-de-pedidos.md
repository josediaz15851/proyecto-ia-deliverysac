# PRD-001: DeliverySac - Reparto y Trazabilidad de Pedidos

## Contexto y Problema
En la empresa DeliverySac repartimos con 5 unidades (3 motos lineales y 2 furgonetas). Hoy el administrador asigna clientes y pedidos manualmente, sin visibilidad de dónde están las unidades, qué pedidos se entregaron ni a qué hora. Al final del día solo hay control cuando retornan las unidades: demoras, pérdida de trazabilidad y cero métricas. No tenemos presupuesto ni tiempo para un ERP de reparto; necesitamos algo simple que se haga cargo del control operativo diario.

**Personas:**
* **Administrador de reparto:** Asigna pedidos cada mañana y quiere ver el estado y la hora exacta de actualización sin llamar a nadie.
* **Repartidor:** Necesita ver su ruta en el celular, marcar entregas rápido y sincronizar estados incluso ante intermitencias de señal en ruta.
* **Supervisor:** Consulta el reporte del día y los motivos de no entrega para tomar decisiones operativas.

---

## Objetivos
Que al final del día el 100% de los pedidos asignados tengan un estado final (entregado, no entregado o reprogramado) registrado en el sistema con su respectiva hora y usuario, sin llamadas telefónicas. 

*Flujo operativo:* Creación de pedido -> Asignación -> Repartidor marca estado (con soporte offline/reintento) -> Administrador visualiza el estado y la hora actualizados.

---

## Requerimientos Funcionales (RF)
* **RF-01:** El sistema debe permitir iniciar sesión con usuario y contraseña según rol (Administrador, Repartidor, Supervisor).
* **RF-02:** El sistema debe permitir al administrador registrar un cliente con nombre y dirección obligatorios.
* **RF-03:** El sistema debe permitir al administrador crear un pedido con cabecera (cliente, fecha de reparto, comentario) y una o más líneas de detalle. *Regla de negocio:* Cada línea requiere producto, cantidad > 0, precio unitario >= 0 y descripción obligatoria. Si no cumple, el sistema rechaza la solicitud respondiendo HTTP 400.
* **RF-04:** El sistema debe calcular automáticamente el monto total del pedido como la suma de cantidad por precio unitario de sus líneas.
* **RF-05:** El sistema debe permitir al administrador asignar un pedido pendiente a un repartidor y una unidad en una fecha. *Reglas de negocio:* 
  * Un pedido solo puede pertenecer a una asignación activa (si se reasigna, se invalida la anterior automáticamente).
  * No se pueden asignar ni editar pedidos cuyos estados sean diferentes de `PENDIENTE`; el backend debe rechazar cualquier intento de modificación si el estado ya avanzó.
* **RF-06:** El sistema debe mostrar al repartidor únicamente los pedidos asignados a él en la fecha actual, bloqueando estrictamente el acceso a pedidos de otros usuarios (retornando HTTP 403 en intentos no autorizados).
* **RF-07:** El sistema debe mostrar al repartidor, para cada pedido, la cabecera completa y el detalle de sus líneas de productos.
* **RF-08:** El sistema debe permitir al repartidor cambiar el estado de un pedido a `EN_RUTA`, `ENTREGADO`, `NO_ENTREGADO` o `REPROGRAMADO`. *Regla de negocio:* Si el estado seleccionado es `NO_ENTREGADO` o `REPROGRAMADO`, el ingreso del motivo del suceso es de carácter obligatorio.
* **RF-09:** El sistema debe registrar de forma automática e inalterable la hora exacta y el ID del usuario responsable cada vez que ocurra un cambio de estado en el pedido.
* **RF-10:** El sistema debe mostrar al administrador el listado en tiempo real de los pedidos del día con su estado actualizado, hora de última modificación y motivo en caso de incidencias.
* **RF-11:** El celular del repartidor debe almacenar localmente los cambios de estado si la red 4G se interrumpe momentáneamente, reintentando la sincronización en segundo plano de forma automática al recuperar la conectividad.
* **RF-12:** El sistema debe paginar todos los listados principales utilizando los parámetros de consulta `page` y `size`.

---

## Requerimientos No Funcionales (RNF)
* **RNF-01:** El 95% de las peticiones HTTP debe responder en <= 2 segundos bajo condiciones normales de red.
* **RNF-02:** La interfaz del repartidor debe ser responsive, adaptada y totalmente usable desde pantallas de 320px de ancho (diseño mobile-first).
* **RNF-03:** Las contraseñas de usuario deben almacenarse de forma segura utilizando hash bcrypt con coste >= 10.
* **RNF-04:** Toda comunicación cliente-servidor debe efectuarse mediante HTTPS utilizando TLS 1.2 o superior.
* **RNF-05:** El sistema operativo funcional del backend y frontend no debe depender de servicios externos o pasarelas pagas de terceros.

---

## Criterios de Aceptación (AC)
* **AC-01 (RF-01):** Dado un usuario con credenciales válidas, cuando ingresa usuario y contraseña, entonces accede correctamente a su panel según su rol asignado.
* **AC-02 (RF-02):** Dado un administrador autenticado, cuando registra un cliente con nombre y dirección válidos, entonces el cliente se guarda exitosamente en la base de datos.
* **AC-03 (RF-03):** Dado un administrador que intenta guardar un pedido sin líneas o con cantidades en 0, cuando confirma, entonces el sistema responde HTTP 400 y rechaza la creación.
* **AC-04 (RF-04):** Dado un conjunto de líneas de detalle en un pedido, cuando el sistema procesa la creación o consulta, entonces calcula automáticamente el monto total como la suma aritmética de cantidad por precio unitario sin intervención manual.
* **AC-05 (RF-05):** Dado un pedido en estado pendiente, cuando el administrador lo asigna a un repartidor y unidad, entonces el pedido cambia a asignado y aparece reflejado exclusivamente en la ruta de dicho repartidor.
* **AC-06 (RF-06 & Aislamiento):** Dado un repartidor autenticado que intenta consultar o modificar mediante URL/API el pedido asignado a otro compañero, entonces el sistema deniega el acceso respondiendo con HTTP 403.
* **AC-07 (RF-07):** Dado un repartidor autenticado con pedidos asignados, cuando abre el detalle de un pedido en su celular (>= 320px), entonces visualiza claramente la cabecera y todas sus líneas de productos.
* **AC-08 (RF-08 & RF-09):** Dado un pedido asignado, cuando el repartidor lo marca como `ENTREGADO`, entonces el estado cambia, el sistema registra automáticamente la hora exacta y el ID del usuario, y el administrador lo visualiza en su listado.
* **AC-09 (RF-08):** Dado un pedido asignado, cuando el repartidor intenta marcarlo como `NO_ENTREGADO` o `REPROGRAMADO` sin especificar un motivo, entonces el sistema bloquea la acción y exige el texto del motivo.
* **AC-10 (RF-10):** Dado el panel del administrador, cuando consulta el listado del día, entonces visualiza los pedidos actualizados con su estado, hora de último cambio y los motivos de incidencias en tiempo real.
* **AC-11 (RF-11):** Dado un repartidor que actualiza el estado de un pedido sin conexión 4G, cuando el celular recupera la señal, entonces el sistema sincroniza automáticamente el cambio pendiente en segundo plano.
* **AC-12 (RF-12):** Dados más de 20 pedidos registrados en el sistema, cuando se realiza la petición de listado enviando los parámetros `page` y `size`, entonces el backend devuelve estrictamente un subconjunto paginado respetando los límites solicitados.

---

## Fuera de Alcance
Reportes PDF/Excel · cierre de día automatizado · registro público de usuarios · GPS o ubicación en tiempo real por mapa · auditoría avanzada de bases de datos · arquitectura multiempresa/multi-sucursal · aplicación móvil nativa (se emplea PWA/Web móvil) · notificaciones push masivas, chat interno, o integraciones con ERP y WhatsApp.

---

## Riesgos y Dependencias
* **Riesgo:** Intermitencia o pérdida de señal 4G mientras el repartidor se desplaza en moto.
  * *Mitigación:* Mecanismo de cola local en el almacenamiento del navegador móvil que reintenta automáticamente el envío del cambio de estado al recuperar la conexión (implementado en RF-11).
* **Riesgo:** Que un administrador intente editar un pedido ya en curso (`ASIGNADO`, `EN_RUTA`, etc.), alterando la trazabilidad operativa.
  * *Mitigación:* Validación estricta en el backend que rechaza cualquier edición si el estado actual es diferente de `PENDIENTE` (regla incorporada en RF-05).
* **Dependencia:** Estabilidad básica de la red móvil celular (4G/3G) para las sincronizaciones periódicas en ruta.