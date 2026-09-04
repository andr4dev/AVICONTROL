# Feature Specification: Actualizar Estado (Galpón)

**Created**: 2026-09-03  

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Actualización manual del ciclo productivo (Priority: P1)

Como administrador de la granja, quiero poder actualizar manualmente el estado de un galpón siguiendo las fases de su ciclo de vida, para reflejar la evolución del lote alojado o los procesos de limpieza de la infraestructura.

**Why this priority**: Garantiza que el sistema refleje la realidad operativa de la granja y que se cumplan las reglas lógicas del negocio (bioseguridad y cosecha).

**Independent Test**: Desde la interfaz, seleccionar un galpón en estado "Productivo", intentar cambiar su estado y verificar que solo se ofrezcan las opciones válidas ("En Cosecha" o "Aislamiento").

**Acceptance Scenarios**:

1. **Scenario**: Transición válida de Productivo a Cosecha
   - **Given** un galpón en estado "Productivo"
   - **When** el administrador actualiza su estado
   - **Then** el sistema solo permite seleccionar "En Cosecha" o "Aislamiento" y, al elegir "En Cosecha", guarda el cambio exitosamente.

2. **Scenario**: Transición válida post-cosecha
   - **Given** un galpón en estado "En Cosecha"
   - **When** el administrador actualiza su estado a "Vaciado Sanitario"
   - **Then** el sistema guarda el nuevo estado.

3. **Scenario**: Intento de transición manual inválida
   - **Given** un galpón en estado "Disponible"
   - **When** el administrador intenta cambiar el estado manualmente a "Productivo"
   - **Then** el sistema bloquea o no muestra esta opción, ya que este cambio solo ocurre automáticamente al registrar un Lote.

---

### User Story 2 - Entrada y Salida de Mantenimiento (Priority: P2)

Como administrador, necesito poder poner un galpón vacío en "Mantenimiento" para reflejar que está bajo reparaciones y bloquear temporalmente el ingreso de nuevos lotes.

**Why this priority**: Previene que se asigne un lote de aves a un galpón que no está en condiciones físicas para recibirlas.

**Acceptance Scenarios**:

1. **Scenario**: Paso a mantenimiento desde Disponible
   - **Given** un galpón en estado "Disponible" (o "Vaciado Sanitario")
   - **When** el administrador cambia el estado a "Mantenimiento"
   - **Then** el galpón adopta el nuevo estado y no aparecerá como opción al registrar un nuevo lote.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir al administrador actualizar manualmente el estado de un galpón, restringiendo las opciones según el estado actual:
  - De **Disponible** SOLO puede pasar a: `Mantenimiento` (manualmente). *Nota: a Productivo pasa de forma automática.*
  - De **Productivo** SOLO puede pasar a: `En Cosecha` o `Aislamiento`.
  - De **En Cosecha** SOLO puede pasar a: `Vaciado Sanitario`.
  - De **Vaciado Sanitario** SOLO puede pasar a: `Mantenimiento` o `Disponible`.
  - De **Aislamiento** SOLO puede pasar a: `Productivo` o `Vaciado Sanitario`.
  - De **Mantenimiento** SOLO puede pasar a: `Disponible`.
- **FR-002**: El sistema MUST automatizar las siguientes transiciones, bloqueando su ejecución manual:
  - Al crear un Galpón nuevo -> automáticamente a `Disponible`.
  - Al registrar un Lote en un Galpón -> automáticamente a `Productivo`.
- **FR-003**: El estado **Aislamiento** MUST actuar de manera puramente informativa (indicando posible contagio), sin bloquear funcionalidades o traslados en el sistema.

### Key Entities

- **Galpón**: Posee un único campo `Estado` que controla su fase operativa exclusiva. El estado de "Mantenimiento" forma parte de este mismo ciclo, asegurando que un galpón en reparación no pueda ser confundido con uno "Disponible".

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** (Integridad de Datos): 0% de ocurrencias de estados inválidos o saltos no permitidos en la base de datos gracias a las reglas estrictas de transición.
- **SC-002** (Trazabilidad): El 100% de los cambios de estado manuales y automáticos dejan un rastro de auditoría (audit trail) identificando la fecha, la hora, el usuario que hizo el cambio y el estado previo.
- **SC-003** (Bioseguridad): El sistema previene de manera proactiva el 100% de los intentos de alojar un lote (pasar a Productivo) si el galpón no se encuentra previamente en "Disponible", reduciendo riesgos sanitarios.
- **SC-004** (Usabilidad): El administrador puede realizar un cambio de estado válido en la interfaz en menos de 3 clics y recibir confirmación visual inmediata del éxito de la operación.
