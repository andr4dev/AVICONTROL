# Feature Specification: Registrar Lote 
**Created** : 2026-09-01

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar un Lote (Priority: P1)

Como Administrador de la Granja, quiero registrar un lote de pollos indicando la cantidad de pollos, el nombre, la fecha de ingreso y el costo del lote.

**Why this priority**: Al registrarse un lote este permite obtener la información crucial para la operaciones del sistema como lo es el control de la dieta de los pollos (Modulo 2) y el calculo de la matriz de venta final (Modulo 3).

**Independent Test** : Creación de un formulario donde el sistema solicita la fecha de ingreso, la población inicial, el costo del lote y un galpón destino.

**Acceptance Scenarios** :
1.  **Scenario** : Registro exitoso del lote
   - **Given** el administrador intente registrar un lote
   - **When** ingrese la población inicial, la fecha de ingreso, nombre, costo y asigne un galpón en estado "Disponible"
   - **Then** el sistema genera un UUID único para el lote
   - **And** guarda el lote asociado al galpón
   - **And** cambia automáticamente el estado del galpón a "Productivo".

2. **Scenario** : La población supera el aforo del galpón
   - **Given** el administrador seleccionó un galpón
   - **When** ingresa una población inicial mayor al aforo máximo de ese galpón
   - **Then** el sistema lanza una alerta y no permite guardar.

3. **Scenario** : La fecha de ingreso posterior a la fecha actual
   - **Given** el administrador intente ingresar una fecha de ingreso 
   - **When** la fecha es posterior a la fecha actual del sistema
   - **Then** el sistema lanza una alerta y solicita una fecha válida.

### User Story 2 - Asignar lote a galpón (Priority: P2)

Como Administrador de la Granja, quiero poder ingresar un lote de pollos dentro de en un galpón.

**Why this priority**: El lote debe estar alojado en un galpón obligatoriamente.

**Acceptance Scenarios** :
1.  **Scenario** : Lista de galpones permitidos
   - **Given** el administrador despliega la lista de galpones para el lote
   - **Then** el sistema SOLO le muestra los galpones cuya `estado_fase` es "Disponible".

## Edge Cases

- **Concurrencia**: ¿Qué pasa si dos usuarios intentan registrar un lote al mismo tiempo sobre el mismo galpón? → El sistema debe procesar la primera transacción y cambiar el galpón a "Productivo". La segunda transacción fallará porque el galpón ya no estará "Disponible".
- **Error durante guardado**: Conservar datos del formulario para un nuevo intento.
- **Fallo de UUID**: El sistema debe cancelar la creación e informar el error, no se debe guardar un lote sin identificador único.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE generar automáticamente un UUID único en cada registro exitoso.
- **FR-002**: El sistema DEBE verificar que el galpón seleccionado exista y su estado_fase sea ESTRICTAMENTE "Disponible" antes de procesar el registro del lote.
- **FR-003**: El sistema DEBE bloquear cualquier registro donde la población inicial sea superior al aforo máximo del galpón seleccionado.
- **FR-004**: El sistema DEBE cambiar el estado_fase del galpón a "Productivo" inmediatamente después de registrar el lote de forma exitosa (transacción atómica).
- **FR-005**: El sistema DEBE permitir el registro retroactivo (fechas anteriores a la actual), pero bloquear fechas futuras.

### Key Entities

- **Lote**: Representa el conjunto biológico de pollos alojado en un galpón.
    - `id`: UUID único.
    - `galpon_id`: UUID del galpón (Foránea).
    - `nombre`: String.
    - `fecha_ingreso`: Date.
    - `poblacion_inicial`: Entero positivo.
    - `poblacion_actual`: Entero positivo (igual a inicial al registrar).
    - `costo`: Decimal.
    
- **Galpón**: Registro maestro de referencia.
    - `id`: UUID único
    - `aforo_maximo`: Límite superior para la población del lote.
    - `estado_fase`: Debe ser "Disponible" para aceptar el lote, pasa a "Productivo".

## Success Criteria *(mandatory)*

### Measurable Outcomes
- **SC-001**: El 100% de los lotes nuevos nacen vinculados a un galpón.
- **SC-002**: El cambio de estado del galpón (Disponible -> Productivo) ocurre en el 100% de los registros de manera atómica, garantizando cero inconsistencias en la BD.
