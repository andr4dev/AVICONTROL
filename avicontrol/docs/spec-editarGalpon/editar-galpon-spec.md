# Feature Specification: Editar un galpón
**Created**: 2026-09-03

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Editar información básica del galpón (Priority: P3)

Como administrador de granja, quiero editar nombre y aforo máximo del galpón en caso de cambios en la infraestructura.

**Why this priority**: La edición del galpón no es recurrente, se da en casos especiales (ej. ampliaciones físicas).

**Independent Test**: Creación de un formulario donde aparezca el galpón que se va a editar con los campos nombre y aforo máximo. No debe mostrar el campo "Estado".

**Acceptance Scenarios**:

1. **Scenario**: Edición de nombre exitosa
   - **Given** un galpón ya registrado en cualquier estado
   - **When** el administrador ingresa un nuevo nombre válido
   - **Then** el sistema almacena el nuevo nombre.

2. **Scenario**: Edición de aforo bloqueada si no está en mantenimiento
   - **Given** un galpón en cualquier estado diferente a "Mantenimiento" (ej. "Productivo" o "Disponible")
   - **When** el administrador intenta modificar el campo "Aforo Máximo"
   - **Then** el sistema deshabilita o bloquea el campo de aforo e indica que requiere estar en Mantenimiento.

3. **Scenario**: Edición de aforo permitida en mantenimiento
   - **Given** un galpón cuyo estado actual es "Mantenimiento"
   - **When** el administrador modifica el aforo máximo a un nuevo valor válido
   - **Then** el sistema actualiza el aforo máximo del galpón.

## Edge Cases

- **Edición concurrente**: Dos usuarios editan el mismo galpón; se procesa el primero y se verifica reglas (ej. unicidad de nombre) en el segundo.
- **Estado del Galpón Inmutable Aquí**: Esta funcionalidad NO incluye la alteración del `estado` (se hace desde "Actualizar Estado").

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir editar el `nombre` de un galpón en cualquier momento (validando que no quede vacío ni duplicado).
- **FR-002**: El sistema DEBE permitir editar el `aforo_maximo` ÚNICAMENTE cuando el `estado` del galpón sea estrictamente "Mantenimiento".
- **FR-003**: El sistema NO DEBE permitir editar el estado del galpón en este formulario (solo muestra información o directamente oculta el campo de estado para edición).
- **FR-004**: El sistema DEBE generar automáticamente un histórico o rastro de auditoría de cada galpón editado.

### Key Entities

- **Galpón**: Representa la unidad de infraestructura.
  - `nombre`: Editable en cualquier momento.
  - `aforo_maximo`: Editable SOLO si `estado == Mantenimiento`.
  - `estado`: NO editable en esta funcionalidad.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 0% de ocurrencias donde el aforo máximo resulte menor a la población alojada tras una edición.
- **SC-002**: 100% de las ediciones de aforo máximo ocurren bajo la bandera de "Mantenimiento", garantizando un control estricto sobre cambios de infraestructura.
