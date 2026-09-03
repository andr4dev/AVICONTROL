# Feature Specification: Recibir alerta sanitaria

**Created**: 2026-09-03

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Aislar un galpón por alerta sanitaria (Priority: P1)

Como administrador u operario de granja, quiero registrar una alerta sanitaria para que el sistema aísle el galpón afectado y evite que continúe en producción.

**Why this priority**: El aislamiento oportuno reduce el riesgo de propagación de enfermedades entre lotes y permite mantener la trazabilidad de las novedades sanitarias.

**Independent Test**: Se puede registrar una alerta sanitaria válida seleccionando un galpón productivo, verificar que la alerta quede guardada y comprobar que el estado del galpón cambie a “Aislamiento”.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso de una alerta de aislamiento
	- **Given** existe un galpón en estado “Productivo”
	- **When** un administrador u operario registra una alerta con acción “Aislamiento”, fecha del día actual, tipo de enfermedad o sospecha y descripción, seleccionando el galpón o un lote activo por su nombre
	- **Then** el sistema genera un UUID único para la alerta
	- **And** resuelve y guarda internamente el UUID del galpón afectado
	- **And** guarda el usuario autenticado que registró la alerta
	- **And** almacena la alerta sanitaria
	- **And** cambia atómicamente el estado del galpón a “Aislamiento”

2. **Scenario**: Alerta identificada mediante un lote activo
	- **Given** existe un lote activo asociado a un galpón productivo
	- **When** el usuario selecciona el lote por su nombre y registra una alerta de aislamiento válida
	- **Then** el sistema obtiene internamente el UUID del galpón asociado
	- **And** registra la alerta sobre ese galpón
	- **And** cambia el estado del galpón a “Aislamiento”

### User Story 2 - Autorizar la reanudación de la producción (Priority: P1)

Como administrador u operario de granja, quiero registrar una alerta sanitaria de reanudación para devolver a producción un galpón que estaba aislado.

**Why this priority**: La producción solo debe reanudarse cuando exista una autorización sanitaria registrada y trazable.

**Independent Test**: Se puede registrar una alerta válida con acción “Reanudación” para un galpón aislado y verificar que su estado cambie a “Productivo” sin modificar los datos del lote.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso de una autorización de reanudación
	- **Given** existe un galpón en estado “Aislamiento”
	- **When** un administrador u operario registra una alerta con acción “Reanudación”, fecha del día actual y selecciona el galpón o lote activo por su nombre
	- **Then** el sistema genera un UUID único para la alerta
	- **And** almacena la alerta y el usuario autenticado que la registró
	- **And** cambia atómicamente el estado del galpón a “Productivo”
	- **And** conserva sin cambios los atributos del lote asociado

### User Story 3 - Consultar la trazabilidad sanitaria (Priority: P2)

Como administrador u operario de granja, quiero consultar las alertas sanitarias registradas para conocer las acciones aplicadas a cada galpón.

**Why this priority**: El historial permite auditar las decisiones sanitarias y entender cuándo un galpón fue aislado o habilitado nuevamente.

**Independent Test**: Se puede consultar el historial de un galpón y verificar sus alertas ordenadas cronológicamente con los datos de la acción y del usuario responsable.

**Acceptance Scenarios**:

1. **Scenario**: Consulta del historial de alertas
	- **Given** existen alertas sanitarias registradas para un galpón
	- **When** el usuario consulta su historial
	- **Then** el sistema muestra el UUID de cada alerta, fecha y hora, acción, tipo y descripción cuando correspondan, gravedad si fue registrada, usuario responsable y estado aplicado
	- **And** muestra las alertas en orden cronológico descendente

2. **Scenario**: Galpón sin alertas registradas
	- **Given** no existen alertas sanitarias para el galpón seleccionado
	- **When** el usuario consulta el historial
	- **Then** el sistema informa que no existen alertas sanitarias registradas

## Edge Cases

- Si el nombre del galpón o lote no existe, el sistema debe rechazar la alerta y mostrar un mensaje específico.
- Si el usuario selecciona un lote histórico o no activo, el sistema debe rechazar la alerta; solo se permiten lotes activos.
- Si el galpón seleccionado no está en estado “Productivo” al solicitar aislamiento, el sistema debe rechazar la alerta.
- Si el galpón seleccionado no está en estado “Aislamiento” al solicitar reanudación, el sistema debe rechazar la alerta.
- Si el usuario no selecciona un galpón ni un lote activo, el sistema debe rechazar el registro.
- Si la fecha de la alerta no corresponde al día actual, el sistema debe rechazarla. La hora puede variar dentro del día actual.
- Si falta el tipo de enfermedad o sospecha o la descripción en una alerta de aislamiento, el sistema debe rechazarla y señalar el campo correspondiente.
- La gravedad es opcional; si no se registra, el sistema debe mostrar “No disponible” en las consultas.
- El usuario no ingresa UUID; el sistema genera el UUID de la alerta y resuelve internamente los UUID del galpón y del lote.
- Si llegan dos alertas simultáneas para el mismo galpón, ambas deben conservarse para trazabilidad. Cada cambio de estado debe procesarse dentro de una transacción atómica y respetar el estado vigente al momento de confirmar.
- Si la alerta de aislamiento y la de reanudación llegan simultáneamente, el estado final debe corresponder al orden de las transacciones confirmadas.
- Si falla la generación del UUID o la persistencia, el sistema no debe cambiar el estado del galpón ni guardar una alerta incompleta.
- Si ocurre un error durante el guardado, el sistema debe informar el error y conservar los datos diligenciados para permitir un nuevo intento.
- El sistema no debe modificar la población, fecha, costo ni demás atributos del lote al cambiar el estado del galpón.
- Un nombre ambiguo no debe poder seleccionarse; los nombres de galpones y lotes deben ser únicos.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir a administradores y operarios registrar alertas sanitarias manualmente.
- **FR-002**: El sistema DEBE permitir seleccionar el galpón directamente por su nombre o seleccionar un lote activo por su nombre.
- **FR-003**: El sistema DEBE resolver internamente el UUID del galpón y del lote cuando corresponda; el usuario NO DEBE ingresar UUID.
- **FR-004**: El sistema DEBE generar automáticamente un UUID único para cada alerta sanitaria.
- **FR-005**: El sistema DEBE registrar la fecha y hora de la alerta y aceptar únicamente fechas del día actual.
- **FR-006**: Una alerta con acción “Aislamiento” DEBE exigir tipo de enfermedad o sospecha y descripción.
- **FR-007**: La gravedad de la alerta DEBE ser opcional.
- **FR-008**: El sistema DEBE guardar automáticamente el usuario autenticado que registra la alerta.
- **FR-009**: El sistema DEBE validar que el galpón exista antes de guardar la alerta.
- **FR-010**: El sistema DEBE validar que el lote seleccionado exista, esté activo y esté asociado al galpón que será afectado.
- **FR-011**: El sistema DEBE permitir la transición de “Productivo” a “Aislamiento” únicamente mediante una alerta válida con acción “Aislamiento”.
- **FR-012**: El sistema DEBE permitir la transición de “Aislamiento” a “Productivo” únicamente mediante una alerta válida con acción “Reanudación”.
- **FR-013**: El sistema DEBE rechazar una alerta cuya acción no corresponda al estado actual del galpón.
- **FR-014**: El almacenamiento de la alerta y el cambio de estado del galpón DEBEN ejecutarse atómicamente dentro de una única transacción.
- **FR-015**: El sistema DEBE conservar el historial de todas las alertas sanitarias aceptadas, incluidas las registradas cuando el galpón ya tenga alertas anteriores.
- **FR-016**: El sistema DEBE mostrar el historial de alertas asociado al galpón seleccionado.
- **FR-017**: El sistema DEBE mostrar “No disponible” cuando la gravedad opcional no tenga valor.
- **FR-018**: El sistema NO DEBE guardar alertas sin UUID, galpón válido, usuario autenticado o fecha válida.
- **FR-019**: Si falla la generación del UUID, la validación o la persistencia, el sistema NO DEBE cambiar el estado del galpón.
- **FR-020**: El sistema DEBE mostrar mensajes claros y específicos para alertas inválidas, galpones inexistentes, lotes no activos y errores de persistencia.

### Key Entities *(include if feature involves data)*

- **Alerta sanitaria**: Representa una novedad sanitaria registrada sobre un galpón.
  - UUID único generado por el sistema
  - Fecha y hora
  - Acción: Aislamiento o Reanudación
  - Tipo de enfermedad o sospecha, obligatorio para aislamiento
  - Descripción, obligatoria para aislamiento
  - Gravedad opcional
  - UUID del galpón afectado
  - UUID del lote activo cuando la alerta se origine desde un lote
  - Usuario autenticado que registra la alerta
  - Estado o transición aplicada

- **Galpón**: Unidad física cuyo estado cambia como consecuencia de la alerta.
  - UUID único
  - Nombre único
  - Estado: Disponible, vaciado sanitario, productivo, en cosecha, mantenimiento o aislamiento

- **Lote**: Grupo de aves que puede utilizarse para identificar el galpón afectado.
  - UUID único
  - Nombre único
  - Estado o condición de lote activo
  - UUID del galpón referenciado

- **Administrador u operario de granja**: Usuario autenticado autorizado para registrar y consultar alertas sanitarias.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de las alertas válidas debe recibir un UUID único.
- **SC-002**: El 100 % de las alertas de aislamiento válidas debe dejar el galpón en estado “Aislamiento”.
- **SC-003**: El 100 % de las alertas de reanudación válidas debe dejar el galpón en estado “Productivo”.
- **SC-004**: El 100 % de los cambios de estado y registros de alerta debe ejecutarse atómicamente, sin galpones con transición aplicada sin su alerta correspondiente.
- **SC-005**: El 100 % de las alertas aceptadas debe conservar el usuario, fecha, acción y referencia interna del galpón.
- **SC-006**: El 100 % de las alertas con datos inválidos o estado incompatible debe rechazarse con un mensaje específico.
- **SC-007**: El 100 % de los historiales consultados debe mostrar las alertas registradas para el galpón correcto, sin mezclar información de otros galpones.
- **SC-008**: Ninguna alerta fallida debe modificar el estado del galpón ni crear registros incompletos.
- **SC-009**: El 100 % de los datos del lote debe conservarse sin cambios al aislar o reactivar su galpón.
