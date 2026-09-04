## Feature Specification: Actualizar Estado del Galpón

**Created**: 2026-09-04

## User Scenarios & Testing

### User Story 1 - Actualización manual de estado siguiendo transiciones permitidas (Priority: P1)

Como administrador necesito cambiar el estado de un galpón para reflejar la fase operativa actual (mantenimiento, cosecha, aislamiento, vaciado sanitario, etc.). 

**Why this priority**: Es el mecanismo central para controlar el ciclo de vida del galpón y mantener la bioseguridad. Sin esta funcionalidad, no se puede reflejar mantenimiento, cosecha, aislamiento o limpieza, afectando la operación completa.

**Independent Test**: Puede probarse de forma aislada seleccionando un galpón en un estado conocido, verificando que solo se ofrecen las transiciones válidas, realizando una transición y comprobando que el estado cambia correctamente y el historial registra el cambio.

**Acceptance Scenarios**:

1. **Scenario**: Transición de Disponible a Mantenimiento
   - **Given** un galpón en estado "Disponible"
   - **When** el administrador selecciona "Actualizar estado", el sistema muestra solo "Mantenimiento" como opción, el administrador lo elige y confirma
   - **Then** el galpón pasa a "Mantenimiento", se registra en historial y se muestra mensaje de éxito en el listado

2. **Scenario**: Transición de Productivo a Aislamiento
   - **Given** un galpón en estado "Productivo" con un lote activo
   - **When** el administrador actualiza el estado a "Aislamiento" y confirma
   - **Then** el galpón queda en "Aislamiento" y el historial refleja el cambio

3. **Scenario**: Transición de Productivo a En cosecha
   - **Given** un galpón en estado "Productivo"
   - **When** el administrador actualiza el estado a "En cosecha" y confirma
   - **Then** el galpón pasa a "En cosecha"

4. **Scenario**: Transición de En cosecha a Vaciado sanitario
   - **Given** un galpón en estado "En cosecha"
   - **When** el administrador actualiza el estado a "Vaciado sanitario" y confirma
   - **Then** el galpón queda en "Vaciado sanitario"

5. **Scenario**: Transición de Aislamiento a Productivo (recuperación de enfermedad)
   - **Given** un galpón en estado "Aislamiento"
   - **When** el administrador selecciona "Productivo" (única opción junto con "Vaciado sanitario") y confirma
   - **Then** el galpón vuelve a "Productivo"

6. **Scenario**: Transición de Aislamiento a Vaciado sanitario (sacrificio sanitario)
   - **Given** un galpón en estado "Aislamiento"
   - **When** el administrador actualiza a "Vaciado sanitario" y confirma
   - **Then** el galpón pasa a "Vaciado sanitario"

7. **Scenario**: Transición de Vaciado sanitario a Disponible
   - **Given** un galpón en estado "Vaciado sanitario"
   - **When** el administrador actualiza a "Disponible" y confirma
   - **Then** el galpón queda "Disponible" y listo para un nuevo lote

8. **Scenario**: Transición de Mantenimiento a Disponible
   - **Given** un galpón en estado "Mantenimiento"
   - **When** el administrador actualiza a "Disponible" y confirma
   - **Then** el galpón pasa a "Disponible"

9. **Scenario**: Cancelación de la actualización
   - **Given** el administrador ha seleccionado un nuevo estado en el formulario
   - **When** en el mensaje de confirmación elige "Cancelar"
   - **Then** el galpón conserva su estado original y no se registra cambio

---

### Edge Cases

- ¿Qué sucede si el administrador intenta forzar una transición no permitida manipulando la petición? El sistema debe validar en el servidor y rechazar con error.
- ¿Qué sucede si el galpón no existe (URL manipulada)? Se muestra error "galpón no encontrado".
- ¿Qué sucede si se intenta cambiar a "Vaciado sanitario" cuando la población actual no es cero? El sistema debe impedirlo y mostrar un mensaje indicando que aún hay aves en el galpón. *(Asumido: no se debe permitir vaciado sanitario con aves presentes)*
- ¿Qué sucede si dos administradores intentan actualizar el mismo galpón simultáneamente? La última actualización sobrescribe; no se maneja bloqueo optimista.
- ¿Qué sucede si el estado actual es "Productivo" y el administrador quiere pasar a "Mantenimiento"? No se permite; solo desde Disponible.

## Requirements

### Functional Requirements

- **FR-001**: El sistema DEBE permitir al administrador actualizar el estado de un galpón existente.
- **FR-002**: El formulario de actualización DEBE mostrar únicamente las transiciones de estado permitidas desde el estado actual, según la siguiente matriz:
  - Desde **Disponible**: solo **Mantenimiento**
  - Desde **Productivo**: **Aislamiento**, **En cosecha**
  - Desde **En cosecha**: **Vaciado sanitario**
  - Desde **Vaciado sanitario**: **Disponible**
  - Desde **Mantenimiento**: **Disponible**
  - Desde **Aislamiento**: **Productivo**, **Vaciado sanitario**
- **FR-003**: El sistema DEBE validar que la transición solicitada esté permitida antes de persistir el cambio.
- **FR-004**: El sistema DEBE impedir la transición a **Productivo** de forma manual desde **Disponible** (solo automática al registrar lote).
- **FR-005**: El sistema DEBE impedir la transición de **Aislamiento** a **En cosecha** (no se pueden vender aves enfermas).
- **FR-006**: Antes de guardar el cambio, el sistema DEBE mostrar un mensaje de confirmación ("¿Estás seguro de actualizar el estado?").
- **FR-007**: Si el administrador confirma, el sistema DEBE actualizar el estado del galpón.
- **FR-008**: El sistema DEBE registrar en el historial de cambios: identificador del galpón, estado anterior, estado nuevo, timestamp. *(Usuario pendiente, se omite por autenticación no especificada)*
- **FR-009**: Tras la actualización exitosa, el sistema DEBE redirigir al listado de galpones y mostrar mensaje de éxito.
- **FR-010**: Si la transición es inválida o el galpón no existe, el sistema DEBE mostrar un mensaje de error y no realizar cambios.
- **FR-011**: El sistema DEBE validar que la población actual sea cero antes de permitir la transición a **Vaciado sanitario** desde **En cosecha** o **Aislamiento**. *(Asumido; si no es así, marcar como pendiente)*

### Key Entities

- **Galpón**: Entidad existente con atributo **Estado actual** que cambia según las reglas.
- **Historial de Cambios de Estado**: Registro de transiciones de estado.
  - Atributos: ID, Galpón ID (FK), Estado anterior, Estado nuevo, Timestamp.

## Success Criteria

### Measurable Outcomes

- **SC-001**: El administrador puede completar una actualización de estado válida en menos de 1 minuto (sin contar confirmación).
- **SC-002**: El 100% de las transiciones realizadas son válidas según la matriz definida (0% de transiciones inválidas exitosas).
- **SC-003**: El sistema previene el cambio manual de Disponible a Productivo y de Aislamiento a En cosecha en el 100% de los intentos.
- **SC-004**: El 100% de las actualizaciones confirmadas generan registro en el historial.
- **SC-005**: El 95% de las actualizaciones exitosas redirigen correctamente al listado con mensaje de éxito.