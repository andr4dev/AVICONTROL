# Feature Specification: Registrar galpon

**Created**: 2026-08-24

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar un galpón (Priority: P1)

Como administrador de granja, quiero registrar un galpón indicando su nombre y aforo máximo para incorporarlo al control operativo de la granja.

**Why this priority**: El registro de galpones es la base para gestionar posteriormente la población, alimentación, sanidad, inventario y rentabilidad.

**Independent Test**: Se puede probar ingresando un nombre y un aforo válidos, guardando el formulario y verificando que el galpón aparezca inmediatamente en la lista con sus valores iniciales.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso
   - **Given** el administrador intente ingresar un galpón
   - **When** ingresa un nombre válido y un aforo máximo entero positivo
   - **Then** el sistema crea el galpón con un UUID automático
   - **And** establece el estado inicial en “Disponible”
   - **And** redirige al administrador a la lista de galpones

2. **Scenario**: Registro con nombre duplicado
   - **Given** existe un galpón registrado con el mismo nombre
   - **When** el administrador intenta registrar otro galpón con ese nombre
   - **Then** el sistema rechaza el registro
   - **And** muestra un mensaje específico indicando que el nombre ya existe

3. **Scenario**: Registro con aforo inválido
   - **Given**  el administrador ingresa un aforo vacío, cero, negativo o no entero
   - **When** intenta guardar el formulario
   - **Then** el sistema rechaza el registro
   - **And** señala el campo del aforo
   - **And** muestra un mensaje indicando que debe ingresar un entero positivo

4. **Scenario**: registro con nombre vacío
   - **Given** El nombre está vacío o contiene únicamente espacios
   - **When** el administrador intenta guardar el formulario
   - **Then** el sistema rechaza el registro
   - **And** muestra un mensaje indicando que el nombre es obligatorio

### User Story 2 - Consultar el galpón registrado (Priority: P2)

Como administrador de granja, quiero visualizar el galpón recién registrado en la lista para confirmar que la información fue guardada correctamente.

**Why this priority**: La confirmación inmediata permite verificar el registro y utilizar el galpón en los siguientes procesos operativos.

**Independent Test**: Se puede probar registrando un galpón válido y comprobando que aparezca en la lista con sus datos iniciales.

**Acceptance Scenarios**:

1. **Scenario**: Galpón visible en la lista
   - **Given** el galpón fue registrado correctamente
   - **When** el sistema redirige a la lista de galpones
   - **Then** el nuevo galpón aparece inmediatamente con su nombre.
## Edge Cases

- **Nombre con espacios al inicio o al final**: El sistema debe eliminar los espacios sobrantes antes de validar y guardar el nombre.
- **Nombre compuesto solo por espacios**: El sistema debe rechazar el registro y mostrar un mensaje indicando que el nombre es obligatorio.
- **Nombre duplicado**: El sistema debe rechazar el registro y mostrar un mensaje indicando que ya existe un galpón con ese nombre.
- **Nombre duplicado con diferencias de mayúsculas o espacios**: El sistema debe considerar los nombres equivalentes, rechazar el registro y mostrar el mensaje correspondiente.
- **Aforo igual a cero o negativo**: El sistema debe rechazar el valor, señalar el campo e indicar que debe ser un entero positivo.
- **Aforo con decimales, letras o caracteres inválidos**: El sistema debe rechazar el valor y solicitar un aforo entero positivo.
- **Aforo extremadamente alto**: El sistema debe permitirlo porque no se ha definido un límite superior, siempre que sea un entero positivo.
- **Error durante el guardado**: El sistema debe informar que no fue posible guardar el galpón y conservar los datos introducidos para permitir un nuevo intento.
- **Dos usuarios registran simultáneamente el mismo nombre**: El sistema debe validar la unicidad al guardar. Solo un registro debe crearse y el otro intento debe rechazarse.
- **Abandono del formulario**: El sistema no debe crear ningún galpón mientras el usuario no confirme el guardado.
- **Fallo en la generación del UUID**: El sistema debe cancelar la creación, informar el error y no guardar un galpón sin identificador único.
- **Nombre demasiado largo**: No existe una longitud máxima definida actualmente. El sistema debe aplicar el límite técnico disponible y mostrar un mensaje claro si se excede.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir al administrador de granja iniciar y completar el registro de un galpón.
- **FR-002**: El sistema DEBE solicitar un nombre obligatorio y diferente de un valor compuesto únicamente por espacios.
- **FR-003**: El sistema DEBE solicitar un aforo máximo entero y positivo.
- **FR-004**: El sistema DEBE eliminar los espacios sobrantes al inicio y al final del nombre antes de guardarlo.
- **FR-005**: El sistema DEBE impedir el registro de nombres duplicados.
- **FR-006**: El sistema DEBE mostrar mensajes de validación claros y específicos para cada campo.
- **FR-007**: El sistema DEBE generar automáticamente un UUID único para cada galpón registrado.
- **FR-010**: El sistema DEBE establecer el estado inicial del galpón en “Disponible”.
- **FR-011**: El sistema DEBE guardar la información del galpón cuando todos los datos sean válidos.
- **FR-012**: El sistema DEBE conservar los datos introducidos cuando ocurra un error durante el guardado.
- **FR-013**: El sistema DEBE redirigir al administrador a la lista de galpones después de un registro exitoso.
- **FR-014**: El sistema DEBE mostrar inmediatamente en la lista el nuevo galpón con su nombre, aforo máximo y estado.
- **FR-015**: El sistema NO DEBE crear un registro si falla la generación del UUID o la persistencia de los datos.
- **FR-016**: El sistema DEBE validar nuevamente la unicidad del nombre al momento de guardar para evitar duplicados por registros simultáneos.

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

- **SC-001**: El 100 % de los galpones registrados correctamente debe aparecer inmediatamente en la lista de galpones.
- **SC-002**: El 100 % de los galpones creados debe recibir un UUID único.
- **SC-003**: El 100 % de los registros válidos debe guardarse con nombre, aforo máximo y estado.
- **SC-004**: El 100 % de los nuevos galpones debe iniciar con población actual igual a 0, edad del lote igual a 0 días y estado “Disponible”.
- **SC-005**: El 100 % de los intentos con nombre vacío, aforo inválido o nombre duplicado debe rechazarse antes de crear el registro.
- **SC-006**: El 100 % de los errores de validación debe mostrar un mensaje claro y específico para que el administrador pueda corregir el campo correspondiente.
- **SC-007**: El 100 % de los errores durante el guardado debe conservar los datos ingresados para permitir un nuevo intento.
- **SC-008**: Ningún galpón debe guardarse sin un nombre válido, un aforo entero positivo o un UUID único.
