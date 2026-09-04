# Feature Specification: Actualizar Estado (Galpón)

**Created**: 2026-09-03  

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Actualización manual del ciclo productivo (Priority: P1)

Como administrador de la granja, quiero poder actualizar manualmente el estado de un galpón siguiendo estrictamente las fases de su ciclo de vida, para reflejar la evolución del lote alojado o los procesos de limpieza de la infraestructura.

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

### User Story 2 - Gestión de Mantenimiento Simultáneo (Priority: P2)

Como administrador, necesito poder indicar que un galpón está en "Mantenimiento" sin perder su estado operativo actual (ej. que siga siendo "Productivo"), para reflejar reparaciones en curso mientras está ocupado.

**Why this priority**: Evita cuellos de botella informáticos donde el sistema obligaría a sacar las aves (cambiar estado) solo para poder registrar una reparación.

**Acceptance Scenarios**:

1. **Scenario**: Asignar mantenimiento a un galpón ocupado
   - **Given** un galpón en estado "Productivo"
   - **When** el administrador lo marca como "En Mantenimiento"
   - **Then** el galpón refleja ambos estados al mismo tiempo ("Productivo" y "Mantenimiento").

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir al administrador actualizar manualmente el estado de un galpón, pero restringiendo las opciones según el estado actual:
  - De **Productivo** SOLO puede pasar a: `En Cosecha` o `Aislamiento`.
  - De **En Cosecha** SOLO puede pasar a: `Vaciado Sanitario`.
  - De **Vaciado Sanitario** SOLO puede pasar a: `Mantenimiento` o `Disponible`.
  - De **Aislamiento** SOLO puede pasar a: `Productivo` o `Vaciado Sanitario`.
- **FR-002**: El sistema MUST automatizar las siguientes transiciones, bloqueando su ejecución manual:
  - Al crear un Galpón nuevo -> automáticamente a `Disponible`.
  - Al registrar un Lote en un Galpón -> automáticamente a `Productivo`.
- **FR-003**: El sistema MUST permitir que un galpón posea el estado de **Mantenimiento** simultáneamente con cualquier otro estado de su ciclo biológico (ej: Productivo + Mantenimiento).
- **FR-004**: El estado **Aislamiento** MUST actuar de manera puramente informativa (indicando posible contagio), sin bloquear funcionalidades o traslados en el sistema.

### Key Entities

- **Galpón**: Posee el control de su fase operativa. Dado que "Mantenimiento" puede coexistir con otros estados, el modelo de datos debe soportar multiplicidad de estados (por ejemplo, tener un campo para la fase del ciclo `Disponible/Productivo/Cosecha/Vaciado/Aislamiento` y un campo o flag booleano adicional como `enMantenimiento`).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** (Integridad de Datos): 0% de ocurrencias de estados inválidos o saltos no permitidos en la base de datos gracias a las reglas estrictas de transición.
- **SC-002** (Trazabilidad): El 100% de los cambios de estado manuales y automáticos dejan un rastro de auditoría (audit trail) identificando la fecha, la hora, el usuario que hizo el cambio y el estado previo.
- **SC-003** (Bioseguridad): El sistema previene de manera proactiva el 100% de los intentos de alojar un lote (pasar a Productivo) si el galpón no se encuentra previamente en "Disponible", reduciendo riesgos sanitarios.
- **SC-004** (Usabilidad): El administrador puede realizar un cambio de estado válido en la interfaz en menos de 3 clics y recibir confirmación visual inmediata del éxito de la operación.
