# Feature Specification: Registrar Lote 
**Created** : 2026-09-01

#### User Scenarios & Testing  *(mandatory)*

### User Story 1 - Registrar un Lote (Priority: P1)

Como Administrador de la Granja, quiero registrar un lote de pollos indicando la cantidad de pollos, el nombre, la fecha de ingreso y el costo del lote.

**Why this priority**: Al registrarse un lote este permite obtener la información crucial para la operaciones del sistema como lo es el control de la dieta de los pollos (Modulo 2) y el calculo de la matriz de venta final (Modulo 3).

**Independent Test** : Creación de un formulario donde el sistema solicita la fecha de ingreso, la población inicial, el costo del lote.

**Acceptance Scenarios** :
1.  **Scenario** : Registro exitoso del lote
   - **Given** el administrador intente registrar un lote
   * **When** ingrese la población inicial, la fecha de ingreso, nombre y cuanto costó el lote
   * **Then** el sistema genera un UUID único
   * **And** el sistema le da a elegir un galpón disponible
   * **And** el sistema debe almacenar el lote

2. **Scenario** : La población inicial de pollos supera el aforo máximo del galpón
   - **Given** el administrador intente ingresar una población de pollos
   * **When** la población de pollos es mayor al aforo máximo
   * **Then** el sistema lanza una alerta 
   * **And** el sistema permite ingresar un nuevo valor

3. **Scenario** : La fecha de ingreso posterior a la fecha actual
   - **Given** el administrador intente ingresar una fecha de ingreso 
   * **When** la fecha es posterior a la fecha actual del sistema
   * **Then** el sistema lanza una alerta 
   * **And** el sistema permite ingresar una nueva fecha

4. **Scenario** : Población inicial igual a 0 o menor.
   - **Given** el administrador intente ingresar la población inicial
   * **When** el valor es menor o igual a 0
   * **Then** el sistema lanza una alerta
   * **And** el sistema permite ingresar una nueva población inicial
   
5. **Scenario** : Población inicial con valores inválidos
   - **Given** el administrador intente ingresar la población inicial
   * **When** el valor ingresado es decimal, letras o caracteres inválidos
   * **Then** el sistema lanza una alerta
   * **And** el sistema permite ingresar una nueva población inicial

6. **Scenario**: Registro con nombre vacío
   - **Given** El nombre está vacío o contiene únicamente espacios
   - **When** el administrador intenta guardar el formulario
   - **Then** el sistema rechaza el registro
   - **And** muestra un mensaje indicando que el nombre es obligatorio
### User Story 2 - Asignar lote a galpón (Priority: P2)

Como Administrador de la Granja, quiero poder ingresar un lote de pollos dentro de en un galpón.

**Why this priority**: El lote debe estar alojado en un galpón, necesario para que el galpón pueda realizar su ciclo de vida (En cosecha o productivo).

**Independent Test** : Creación de una lista desplegable dentro del formulario donde se aparezcan los galpones disponibles.

**Acceptance Scenarios** :
1.  **Scenario** : Asignación de galpón exitoso.
   - **Given** el administrador intente elegir un lote
   * **When** elige un galpón
   * **Then** el sistema debe guardar el UUID del galpón en los atributos del lote
   * **And** el sistema debe almacenar el lote
   
### Edge Cases

¿Qué pasa si dos usuarios intentan registrar un lote al mismo tiempo sobre el mismo galpón? → El sistema debe procesar la primera transacción y bloquear de inmediato la segunda, arrojando un error de concurrencia ya que el estado pasa a productivo en la primera confirmación.

¿Qué pasa si se da un error durante el guardado? → El sistema debe informar que no fue posible guardar el lote y conservar los datos introducidos para permitir un nuevo intento.

¿Qué pasa si el usuario abandona el formulario? →  El sistema no debe crear ningún lote mientras el usuario no confirme el guardado.

¿Qué pasa si hay fallo en la generación del UUID? →  El sistema debe cancelar la creación, informar el error y no guardar un galpón sin identificador único.

### Requirements  *(mandatory)*

 **Functional Requirements**
*   **FR-001** : El sistema DEBE generar automáticamente un identificador único universal (UUID) en cada registro exitoso.
*   **FR-002** : El sistema DEBE verificar que el galpón exista en la base de datos y que su estado actual sea estrictamente disponible antes de procesar el registro del lote.
*   **FR-003** : El sistema DEBE bloquear cualquier registro donde la población inicial sea superior al aforo máximo del galpón.
*   **FR-004** : El sistema DEBE cambiar el estado actual del galpón a productivo inmediatamente después de registrar el lote de forma exitosa en la base de datos de manera atómica.
*   **FR-007** : El sistema debe permitir que un usuario con rol de Administrador registre lotes de forma retroactiva (por ejemplo, con fecha de ingreso del día anterior si hubo retrasos en el reporte)

##### Key Entities  *(include if feature involves data)*
*   **Lote**: Representa el conjunto biológico de pollos alojado en un galpón específico durante un ciclo productivo cerrado.
    *  Identificador único.
    *  Identificador único del galpón.
    *  Nombre 
    *  Fecha de ingreso del lote
    *  Edad del lote en días transcurridos.
    *  Cantidad original de aves ingresadas.
    *  Población actual de pollos en el lote
    
*   **Galpón**: Se incluye porque actúa como el registro maestro o de referencia que debe existir previamente para poder persistir el lote.
    * UUID único
	- Nombre
	- Aforo máximo
	- Estado (Disponible, vaciado sanitario, productivo, en cosecha, mantenimiento, aislamiento)

- **Administrador de granja**: Usuario autorizado para registrar y administrar galpones.
---

#### Success Criteria  *(mandatory)*

##### Measurable Outcomes
*  **SC-001** : El 100% de los lotes registrados deben quedar asociados a galpones físicos en estado disponible.
*  **SC-002** : El cambio de estado del galpón de disponible a productivo tras el registro debe ejecutarse de forma atómica bajo una transacción única de base de datos en menos de 150ms.
- **SC-003**: El 100 % de los lotes creados debe recibir un UUID único.
- **SC-004**: El 100 % de los errores de validación debe mostrar un mensaje claro y específico para que el administrador pueda corregir el campo correspondiente.
- **SC-005**: El 100 % de los errores durante el guardado debe conservar los datos ingresados para permitir un nuevo intento.
- **SC-006**: Ningún lote debe guardarse sin una fecha valida y una población valida.
