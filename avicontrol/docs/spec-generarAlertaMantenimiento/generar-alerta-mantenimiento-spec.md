# Feature Specification: Generar alerta de mantenimiento

**Created**: 2026-09-04

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Reportar un daño o necesidad de infraestructura (Priority: P1)

Como Técnico de infraestructura, quiero generar una alerta de mantenimiento para un galpón específico indicando el problema, para que el administrador esté enterado y pueda programar las reparaciones necesarias.

**Why this priority**: Es vital capturar los daños físicos a tiempo para evitar que un galpón se deteriore o que se asigne un nuevo lote a una infraestructura defectuosa.

**Independent Test**: El técnico inicia sesión, selecciona un galpón de la lista, redacta la descripción del daño y envía la alerta. El sistema la guarda con estado "Pendiente" y notifica al administrador.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso de la alerta por el técnico
   - **Given** un técnico de infraestructura autenticado en el sistema
   - **When** selecciona un galpón por su nombre y registra una descripción detallada del problema y la urgencia (Baja, Media, Alta)
   - **Then** el sistema genera un UUID único para la alerta
   - **And** almacena la alerta vinculada al galpón con el estado inicial "Pendiente"
   - **And** el galpón CONSERVA su estado operativo actual (no cambia automáticamente).

### User Story 2 - Revisión y aprobación de la alerta por el administrador (Priority: P1)

Como administrador de granja, quiero revisar las alertas de mantenimiento pendientes y, si el galpón está en condiciones (vacío), cambiar su estado a "Mantenimiento" para iniciar la reparación.

**Why this priority**: El administrador es el único responsable del flujo productivo y debe decidir el momento adecuado para inhabilitar el galpón respetando el ciclo biológico.

**Acceptance Scenarios**:

1. **Scenario**: Aprobación de alerta y pase a mantenimiento (Galpón Vacío)
   - **Given** una alerta en estado "Pendiente" sobre un galpón que actualmente está en "Disponible" o "Vaciado Sanitario"
   - **When** el administrador revisa la alerta y decide aprobar el inicio del mantenimiento
   - **Then** el sistema cambia el estado de la alerta a "Aprobada / En Reparación"
   - **And** cambia automáticamente el estado del galpón a "Mantenimiento".

2. **Scenario**: Aprobación postergada (Galpón Ocupado)
   - **Given** una alerta sobre un galpón que está en "Productivo" (con aves adentro)
   - **When** el administrador revisa la alerta
   - **Then** el sistema NO permite cambiar el estado del galpón a "Mantenimiento" (por reglas de bioseguridad)
   - **And** permite al administrador marcar la alerta como "Programada para fin de ciclo", posponiendo el mantenimiento físico real hasta que el galpón se vacíe.

## Edge Cases

- **Múltiples alertas**: Un técnico puede registrar varias alertas para el mismo galpón si hay distintos daños. El administrador puede consolidarlas.
- **Fallo de UUID**: El sistema debe cancelar la creación e informar el error, no se debe guardar una alerta sin identificador único.
- **El técnico selecciona un galpón inexistente**: El sistema debe rechazar el registro y mostrar un error.
- **Intento técnico de cambiar estado**: El técnico de infraestructura NO tiene permisos para alterar el estado operativo del galpón directamente; su función llega hasta la generación de la alerta.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir a los usuarios con rol de "Técnico de infraestructura" generar alertas de mantenimiento.
- **FR-002**: El sistema DEBE solicitar obligatoriamente la selección de un galpón, una descripción del daño y el nivel de urgencia.
- **FR-003**: El sistema DEBE generar la alerta con estado inicial "Pendiente" sin alterar el estado operativo (`estado`) del galpón.
- **FR-004**: El sistema DEBE permitir al rol "Administrador" visualizar un listado de las alertas de mantenimiento pendientes.
- **FR-005**: El sistema DEBE permitir al administrador cambiar el estado del galpón a "Mantenimiento" desde la alerta, ÚNICAMENTE si el galpón se encuentra en estado "Disponible" o "Vaciado Sanitario".
- **FR-006**: El sistema DEBE bloquear el paso a "Mantenimiento" si el galpón está en fases ocupadas (Productivo, En Cosecha), permitiendo al administrador solo "programar" o agendar la alerta para más adelante.
- **FR-007**: El sistema DEBE guardar un registro de auditoría indicando qué técnico creó la alerta y qué administrador la aprobó.

### Key Entities

- **Alerta de Mantenimiento**: 
  - `id`: UUID único.
  - `galpon_id`: UUID del galpón afectado.
  - `tecnico_id`: UUID del técnico que reporta.
  - `admin_id`: UUID del administrador que gestiona la alerta (nulo al inicio).
  - `descripcion`: String detallando el daño.
  - `urgencia`: Enum (Baja, Media, Alta).
  - `estado_alerta`: Enum (Pendiente, Programada, En Reparación, Finalizada).
  - `fecha_creacion`: Timestamp.

- **Galpón**:
  - `estado`: Debe ser evaluado por el administrador. Solo si es "Disponible" o "Vaciado Sanitario" se le permitirá pasar a "Mantenimiento" al gestionar la alerta.

- **Técnico de infraestructura**: Nuevo actor autorizado para registrar daños pero sin poder de decisión sobre el ciclo del galpón.

- **Administrador de granja**: Actor que toma decisiones sobre las alertas.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100% de las alertas generadas por los técnicos se guardan sin alterar el ciclo productivo (estado) del galpón hasta que un administrador intervenga.
- **SC-002**: El 100% de los intentos de un administrador por pasar a "Mantenimiento" un galpón ocupado (Productivo/Cosecha) a raíz de una alerta, son bloqueados por el sistema, garantizando la integridad de las aves.
- **SC-003**: Trazabilidad completa: El 100% de las alertas resueltas identifican al técnico creador y al administrador aprobador.
