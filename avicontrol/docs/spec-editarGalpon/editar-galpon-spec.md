# Feature Specification: Editar un galpón
**Created**: 2026-09-03

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Editar un galpón (Priority: P3)

Como administrador de granja, quiero editar nombre y aforo máximo del galpón en caso de cambios en la infraestructura.

**Why this priority**: La edición del galpón no es recurrente, se da en casos especiales en las que el galpón este en estado de mantenimiento y se modifique area para que quepan mas pollos o el nombre por si cambió de ubicación. 

**Independent Test**: Creación de un formulario donde aparezca el galpón que se va a editar con los campos nombre y aforo máximo.

**Acceptance Scenarios**:

1. **Scenario**: Edición exitosa
   - **Given** un galpón ya registrado
   - **When** el administrador ingresa un nombre y aforo máximo
   - **Then** el sistema almacena los datos nuevos
   - **And** redirige al administrador a la lista de galpones

2. **Scenario**: Edición con nombre duplicado
   - **Given** un galpón registrado con el mismo nombre
   - **When** el administrador ingresa un nombre
   - **Then** el sistema verifica si el nombre ya existe
   - **And** muestra un mensaje específico indicando que el nombre ya existe

3. **Scenario**: Edición con aforo inválido
   - **Given**  el administrador ingresa un aforo vacío, cero, negativo o no entero
   - **When** intenta guardar el formulario
   - **Then** el sistema rechaza la edición
   - **And** señala el campo del aforo
   - **And** muestra un mensaje indicando que debe ingresar un entero positivo

4. **Scenario**: Edición con nombre vacío
   - **Given** El nombre está vacío o contiene únicamente espacios
   - **When** el administrador intenta guardar el formulario
   - **Then** el sistema rechaza la edición
   - **And** muestra un mensaje indicando que el nombre es obligatorio

## Edge Cases

- **Nombre con espacios al inicio o al final**: El sistema debe eliminar los espacios sobrantes antes de validar y guardar el nombre.
- **Nombre compuesto solo por espacios**: El sistema debe rechazar el registro y mostrar un mensaje indicando que el nombre es obligatorio.
- **Nombre duplicado**: El sistema debe rechazar la edición y mostrar un mensaje indicando que ya existe un galpón con ese nombre.
- **Nombre duplicado con diferencias de mayúsculas o espacios**: El sistema debe considerar los nombres equivalentes, rechazar la edición y mostrar el mensaje correspondiente.
- **Aforo igual a cero o negativo**: El sistema debe rechazar el valor, señalar el campo e indicar que debe ser un entero positivo.
- **Aforo con decimales, letras o caracteres inválidos**: El sistema debe rechazar el valor y solicitar un aforo entero positivo.
- **Aforo extremadamente alto**: El sistema debe permitirlo porque no se ha definido un límite superior, siempre que sea un entero positivo.
- **Error durante el guardado**: El sistema debe informar que no fue posible guardar el galpón y conservar los datos introducidos para permitir un nuevo intento.
- **Dos usuarios editan simultáneamente el mismo nombre**: El sistema debe validar la unicidad al guardar. Solo una edición debe crearse y el otro intento debe rechazarse.
- **Abandono del formulario**: El sistema no debe editar el galpón mientras el usuario no confirme el guardado.
- **Fallo en la generación del UUID**: El sistema debe cancelar la edición, informar el error y no guardar un galpón sin identificador único.
- **Nombre demasiado largo**: No existe una longitud máxima definida actualmente. El sistema debe aplicar el límite técnico disponible y mostrar un mensaje claro si se excede.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir al administrador de granja iniciar y completar la edición de un galpón.
- **FR-002**: El sistema DEBE solicitar un nombre obligatorio y diferente de un valor compuesto únicamente por espacios.
- **FR-003**: El sistema DEBE solicitar un aforo máximo entero y positivo.
- **FR-004**: El sistema DEBE eliminar los espacios sobrantes al inicio y al final del nombre antes de guardarlo.
- **FR-005**: El sistema DEBE impedir la edición con nombres duplicados.
- **FR-006**: El sistema DEBE mostrar mensajes de validación claros y específicos para cada campo.
- **FR-007**: El sistema DEBE generar automáticamente un histórico de cada galpón editado.
- **FR-008**: El sistema DEBE guardar la información del galpón cuando todos los datos sean válidos.
- **FR-009**: El sistema DEBE conservar los datos introducidos cuando ocurra un error durante el guardado.
- **FR-010**: El sistema DEBE redirigir al administrador a la lista de galpones después de una edición exitosa.
- **FR-011**: El sistema DEBE mostrar inmediatamente en la lista el galpón editado con su nombre, aforo máximo y estado.
- **FR-012**: El sistema DEBE validar nuevamente la unicidad del nombre al momento de guardar para evitar duplicados por ediciones en simultaneidad.

**Reglas pendientes de definición**:

### Key Entities *(include if feature involves data)*

- **Galpón**: Representa una unidad física de producción avícola.
  - UUID único
  - Nombre
  - Aforo máximo
  - Estado(Disponible, vaciado sanitario, productivo, en cosecha, mantenimiento, aislamiento)

- **Administrador de granja**: Usuario autorizado para registrar y administrar galpones.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de los galpones editados correctamente debe aparecer inmediatamente en la lista de galpones.
- **SC-002**: El 100 % de los registros válidos debe guardarse con nombre, aforo máximo y estado.
- **SC-003**: El 100 % de los intentos con nombre vacío, aforo inválido o nombre duplicado debe rechazarse antes de editar el galpón.
- **SC-004**: El 100 % de los errores de validación debe mostrar un mensaje claro y específico para que el administrador pueda corregir el campo correspondiente.
- **SC-005**: El 100 % de los errores durante el guardado debe conservar los datos ingresados para permitir un nuevo intento.
- **SC-006**: Ningún galpón debe guardarse sin un nombre válido, un aforo entero positivo.
