# Feature Specification: Registrar Lote 
**Created** : 2026-09-01

#### User Scenarios & Testing  *(mandatory)*

### User Story 1 - Registrar un Lote (Priority: P1)

Como Administrador de la Granja, quiero registrar un lote de pollos indicando la cantidad de pollos y la fecha de ingreso.

**Why this priority**: Al registrarse un lote este permite obtener la información crucial para la operaciones del sistema como lo es el control de la dieta de los pollos (Modulo 2) y el calculo de la matriz de venta final (Modulo 3).

**Independent Test** : Creación de un formulario donde el sistema solicita la fecha de ingreso, la población inicial, el costo del lote.

**Acceptance Scenarios** :
1.  **Scenario** : Registro exitoso del lote
   - **Given** el administrador intente registrar un lote
   * **When** ingrese la población inicial, la fecha de ingreso y cuanto costó el lote
   * **Then** el sistema debe almacenar el lote
   * **And** generar un UUID único
   * **And** elige galpón

2. **Scenario** : La población de pollos supera el aforo máximo del galpón
   - **Given** el administrador ingresa una población de pollos
   * **When** la población de pollos es mayor al aforo máximo
   * **Then** lanza una alerta 
   * **And** se permite ingresar un nuevo valor
##### Edge Cases

¿Qué ocurre si un usuario ingresa una fecha ingreso posterior a la fecha actual del sistema? → 

¿Qué pasa si dos usuarios intentan registrar un lote al mismo tiempo sobre el mismo galpón? → El sistema debe procesar la primera transacción y bloquear de inmediato la segunda, arrojando un error de concurrencia ya que el estado pasa a productivo en la primera confirmación.

¿Qué pasa si la población igual a cero o negativo? → El sistema debe rechazar el valor, señalar el campo e indicar que debe ser un entero positivo.

¿Qué pasa si la población se ingresa con decimales, letras o caracteres inválidos? → El sistema debe rechazar el valor y solicitar una Población entero positivo.

- **Error durante el guardado**: El sistema debe informar que no fue posible guardar el lote y conservar los datos introducidos para permitir un nuevo intento.
- **Abandono del formulario**: El sistema no debe crear ningún galpón mientras el usuario no confirme el guardado.
- **Fallo en la generación del UUID**: El sistema debe cancelar la creación, informar el error y no guardar un galpón sin identificador único.


---

#### Requirements  *(mandatory)*

##### Functional Requirements
*   **FR-001** : El sistema DEBE generar automáticamente un identificador único universal (UUID) como id_lote en cada registro exitoso.
*   **FR-002** : El sistema DEBE verificar que el galpón exista en la base de datos y que su estado actual sea estrictamente DISPONIBLE antes de procesar el registro del lote.
*   **FR-003** : El sistema DEBE bloquear cualquier registro donde la población inicial sea superior al aforo máximo del galpón.
*   **FR-004** : El sistema DEBE cambiar el estado actual del galpón a PRODUCTIVO inmediatamente después de registrar el lote de forma exitosa en la base de datos de manera atómica.
*   **FR-005** : El sistema DEBE calcular el atributo edad del lote en dias cada vez que sea consultado mediante la fórmula.
*   **FR-006** : El sistema DEBE calcular dinámicamente en tiempo de consulta la población actual de aves vivas.
*   **FR-007** : El sistema debe permitir que un usuario con rol de Administrador registre lotes de forma retroactiva (por ejemplo, con fecha de ingreso del día anterior si hubo retrasos en el reporte)

##### Key Entities  *(include if feature involves data)*
*   **Lote**: Representa el conjunto biológico de aves alojado en un galpón específico durante un ciclo productivo cerrado.
    *   (UUID): Identificador único.
    *   (UUID, FK): Relación con el galpón receptor.
    *  fecha del día del alojamiento.
    *   Edad del lote en días transcurridos.
    *  Cantidad original de aves ingresadas.
    *   Población actual
    
*   **Galpón**: Se incluye porque actúa como el registro maestro o de referencia que debe existir previamente para poder persistir el lote.
    * UUID único
	- Nombre
	- Aforo máximo
	- Estado (Disponible, vaciado sanitario, productivo, en cosecha, mantenimiento, aislamiento)

- **Administrador de granja**: Usuario autorizado para registrar y administrar galpones.
---

#### Success Criteria  *(mandatory)*

##### Measurable Outcomes
*  **SC-001** : El 100% de los lotes registrados deben quedar asociados a galpones físicos en estado DISPONIBLE.
*  **SC-002** : El cambio de estado del galpón de DISPONIBLE a PRODUCTIVO tras el registro debe ejecutarse de forma atómica bajo una transacción única de base de datos en menos de 150ms.
- **SC-003**: El 100 % de los lotes creados debe recibir un UUID único.
- **SC-004**: El 100 % de los errores de validación debe mostrar un mensaje claro y específico para que el administrador pueda corregir el campo correspondiente.
- **SC-005**: El 100 % de los errores durante el guardado debe conservar los datos ingresados para permitir un nuevo intento.
- **SC-006**: Ningún lote debe guardarse sin una fecha valida y una población valida.
