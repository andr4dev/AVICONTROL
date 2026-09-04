# Feature Specification: Registrar galpón

**Created**: 2026-08-24

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar un galpón (Priority: P1)

Como administrador de granja, quiero registrar un galpón indicando su nombre y aforo máximo para incorporarlo al control operativo de la granja.

**Why this priority**: El registro de galpones es la base para gestionar posteriormente la población, alimentación, sanidad, inventario y rentabilidad.

**Independent Test**: Se puede probar ingresando un nombre y un aforo válidos, guardando el formulario y verificando que el galpón aparezca inmediatamente en la lista con sus valores iniciales.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso
   - **Given** el administrador intenta ingresar un galpón
   - **When** ingresa un nombre válido y un aforo máximo entero positivo
   - **Then** el sistema crea el galpón con un UUID automático
   - **And** establece el estado inicial en “Disponible” (con Mantenimiento en false)
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
   - **Then** el nuevo galpón aparece inmediatamente con su nombre, aforo y estado "Disponible".

## Edge Cases

- **Nombre con espacios al inicio o al final**: El sistema debe eliminar los espacios sobrantes antes de validar y guardar el nombre.
- **Nombre compuesto solo por espacios**: El sistema debe rechazar el registro y mostrar un mensaje indicando que el nombre es obligatorio.
- **Nombre duplicado**: El sistema debe rechazar el registro y mostrar un mensaje indicando que ya existe un galpón con ese nombre (ignorando diferencias de mayúsculas/espacios).
- **Error durante el guardado**: El sistema debe informar que no fue posible guardar el galpón y conservar los datos introducidos para permitir un nuevo intento.
- **Dos usuarios registran simultáneamente el mismo nombre**: El sistema debe validar la unicidad al guardar. Solo un registro debe crearse y el otro intento debe rechazarse.
- **Fallo en la generación del UUID**: El sistema debe cancelar la creación, informar el error y no guardar un galpón sin identificador único.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir al administrador de granja iniciar y completar el registro de un galpón.
- **FR-002**: El sistema DEBE solicitar un nombre obligatorio y diferente de un valor compuesto únicamente por espacios.
- **FR-003**: El sistema DEBE solicitar un aforo máximo entero y positivo.
- **FR-004**: El sistema DEBE eliminar los espacios sobrantes al inicio y al final del nombre antes de guardarlo.
- **FR-005**: El sistema DEBE impedir el registro de nombres duplicados.
- **FR-006**: El sistema DEBE generar automáticamente un UUID único para cada galpón registrado.
- **FR-007**: El sistema DEBE establecer el estado de fase inicial del galpón en “Disponible”.
- **FR-008**: El sistema DEBE establecer el indicador de "Mantenimiento" en falso (false) al registrar.
- **FR-009**: El sistema NO DEBE permitir asociar un lote durante el registro inicial (es un flujo separado).

### Key Entities 

- **Galpón**: Representa una unidad física de producción avícola.
  - `id`: UUID único
  - `nombre`: String
  - `aforo_maximo`: Entero positivo
  - `estado_fase`: Enum (Disponible, Productivo, En Cosecha, Vaciado Sanitario, Aislamiento). Valor por defecto al registrar: "Disponible".
  - `en_mantenimiento`: Booleano. Valor por defecto al registrar: false.

- **Administrador de granja**: Usuario autorizado para registrar y administrar galpones.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100% de los galpones nuevos nacen estrictamente con estado "Disponible" y sin la bandera de mantenimiento.
- **SC-002**: El 100% de los intentos de registros con nombres duplicados o valores nulos son rechazados antes de llegar a la base de datos.
