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
   - **Given** un galpón que NO tiene la bandera "En Mantenimiento"
   - **When** el administrador intenta modificar el campo "Aforo Máximo"
   - **Then** el sistema deshabilita o bloquea el campo de aforo e indica que requiere estar en Mantenimiento.

3. **Scenario**: Edición de aforo permitida en mantenimiento (vacío)
   - **Given** un galpón con "En Mantenimiento" activado y sin ningún lote alojado (ej. estado "Disponible" o "Vaciado Sanitario")
   - **When** el administrador modifica el aforo máximo a un nuevo valor válido
   - **Then** el sistema actualiza el aforo máximo del galpón.

4. **Scenario**: Edición de aforo bloqueada si contiene lote
   - **Given** un galpón en "Mantenimiento" pero que actualmente contiene un lote activo (ej. estado "Productivo")
   - **When** el administrador intenta modificar el campo "Aforo Máximo"
   - **Then** el sistema rechaza la edición y deshabilita el campo, indicando que el aforo es intocable mientras el galpón esté ocupado.

## Edge Cases

- **Edición concurrente**: Dos usuarios editan el mismo galpón; se procesa el primero y se verifica reglas (ej. unicidad de nombre) en el segundo.
- **Aforo Intocable Ocupado**: Cualquier intento de manipulación del aforo vía interfaz o API será rechazado estrictamente si el galpón tiene un lote activo.
- **Estado del Galpón Inmutable Aquí**: Esta funcionalidad NO incluye la alteración de la `estado_fase` ni de la bandera de `en_mantenimiento` (se hace desde "Actualizar Estado").

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir editar el `nombre` de un galpón en cualquier momento (validando que no quede vacío ni duplicado).
- **FR-002**: El sistema DEBE permitir editar el `aforo_maximo` ÚNICAMENTE cuando el galpón cumpla dos condiciones: tener la bandera `en_mantenimiento` activa Y NO contener ningún lote activo.
- **FR-003**: El sistema DEBE bloquear y hacer completamente inmodificable el campo `aforo_maximo` si el galpón está ocupado por un lote (ej. en fase "Productivo" o "En Cosecha").
- **FR-004**: El sistema NO DEBE permitir editar el estado del galpón en este formulario (solo muestra información o directamente oculta el campo de estado para edición).
- **FR-005**: El sistema DEBE generar automáticamente un histórico o rastro de auditoría de cada galpón editado.

### Key Entities

- **Galpón**: Representa la unidad de infraestructura.
  - `nombre`: Editable en cualquier momento.
  - `aforo_maximo`: Editable SOLO si `en_mantenimiento == true`.
  - `estado_fase`: NO editable en esta funcionalidad.
  - `en_mantenimiento`: Booleano (Bandera necesaria para permitir la edición de aforo).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 0% de ocurrencias donde el aforo máximo resulte menor a la población alojada tras una edición.
- **SC-002**: 100% de las ediciones de aforo máximo ocurren bajo la bandera de "Mantenimiento", garantizando un control estricto sobre cambios de infraestructura.
