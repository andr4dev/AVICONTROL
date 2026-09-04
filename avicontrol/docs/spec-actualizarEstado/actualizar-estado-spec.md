# Feature Specification: Actualizar estado de galpón
**Created** : 2026-09-03

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Actualizar estado operativo del galpón (Priority: P1)

Como administrador de la granja, quiero modificar de forma manual el estado operativo de un galpón.

**Why this priority** : Permite al administrador coordinar el flujo bioseguro de la granja, indicando cuándo se está cosechando el lote, cuándo inicia el vaciado sanitario obligatorio, cuando se genera la alerta de aislamiento o cuándo la infraestructura requiere mantenimiento.

**Independent Test** : Se puede probar seleccionando un galpón en un estado determinado, eligiendo un nuevo estado válido de una lista de transiciones permitidas, confirmando la acción y verificando que el estado cambie de inmediato en la base de datos y en la lista de galpones.

**Acceptance Scenarios** :
1. **Scenario** : Inicio de mantenimiento desde estado Disponible
   * **Given** existe un galpón en estado "Disponible"
   * **When** el administrador actualiza manualmente el estado a "Mantenimiento"
   * **Then** el sistema cambia el estado del galpón a "Mantenimiento"
   * **And** registra el cambio en el historial con fecha, hora y usuario responsable.

2. **Scenario** : Inicio de cosecha desde estado Productivo
   * **Given** existe un galpón en estado "Productivo (Ocupado)" 
   * **When** el administrador actualiza manualmente el estado a "En Cosecha (Venta)"
   * **Then** el sistema cambia el estado del galpón a "En Cosecha (Venta)" 
   * **And** mantiene sin alteraciones los datos del lote alojado.

3. **Scenario** : Fin de cosecha e inicio de vaciado sanitario obligatorio
   * **Given** existe un galpón en estado "En Cosecha (Venta)" 
   * **When** el administrador actualiza el estado a "Vaciado Sanitario"
   * **Then** el sistema cambia el estado del galpón a "Vaciado Sanitario"
   * **And** archiva el lote activo en el historial.

4. **Scenario** : Fin de mantenimiento y habilitación del galpón
   * **Given** existe un galpón en estado "Mantenimiento" [4]
   * **When** el administrador actualiza el estado a "Disponible"
   * **Then** el sistema establece el estado del galpón en "Disponible"
   * **And** el galpón queda apto para recibir un nuevo lote.

### User Story 3 - Auditoría de cambios de estado (Priority: P2)

Como administrador de la granja, quiero que cada actualización de estado quede registrada en un historial detallado para mantener la trazabilidad de las operaciones físicas de la granja.

**Why this priority** : Es una operación secundaria para el funcionamiento real de la granja, sirviendo para las auditorías operativas, permitiendo conocer con precisión los tiempos muertos de mantenimiento, la duración real del vaciado sanitario.

**Independent Test** : Se puede probar realizando una actualización de estado exitosa, ingresando luego a la consulta detallada del galpón e inspeccionando que aparezca la nueva entrada en el historial de estados.

**Acceptance Scenarios** :
1. **Scenario** : Registro automático de trazabilidad de estados
   * **Given** el administrador realiza un cambio de estado válido
   * **When** el sistema confirma la transacción
   * **Then** genera una entrada en el historial de estados con: UUID único, UUID del galpón, estado anterior, estado nuevo, fecha y hora del servidor.


## Edge Cases

* **Actualización concurrente del estado de un galpón** : Si dos usuarios intentan cambiar el estado del mismo galpón simultáneamente, el sistema debe aplicar bloqueo de concurrencia optimista, procesar la primera solicitud y rechazar la segunda con un error indicando que el estado del galpón cambió en el origen.

* **Fallo en la base de datos durante la transición** : Si ocurre una falla de red al registrar la transición, la operación debe revertirse en su totalidad. No debe cambiarse el estado del galpón si no se pudo registrar el registro en el historial de estados.

* **Modificación de estado en galpones con lotes activos** : Al pasar de "Productivo" a "En Cosecha", la población actual del lote no debe alterarse.


---

## Requirements *(mandatory)*

##### Functional Requirements
* **FR-001** : El sistema DEBE permitir al administrador de granja actualizar el estado de un galpón de manera manual.
* **FR-002** : El sistema DEBE validar y aplicar estrictamente la siguiente **Máquina de Estados** para el galpón:
	  - Desde **Disponible** se permite transicionar únicamente a: **Mantenimiento**. (La transición a *Productivo* ocurre de forma atómica y automática al registrar un lote).
	  - Desde **Productivo** se permite transicionar únicamente a: **En Cosecha** [4] (La transición a *Aislamiento* ocurre únicamente vía Alerta Sanitaria de Aislamiento).
	  - Desde **En Cosecha** se permite transicionar únicamente a: **Vaciado Sanitario** [4].
	  - Desde **Vaciado Sanitario** se permite transicionar únicamente a: **Disponible** o **Mantenimiento** [4].
	  - Desde **Mantenimiento** se permite transicionar únicamente a: **Disponible** [4].
	  - Desde **Aislamiento** puede pasar a "en cosecha" con la aprobación del modulo 2.
* **FR-003** : El sistema DEBE rechazar cualquier cambio de estado manual que intente omitir o saltar estados del ciclo bioseguro (ej. saltar de "En Cosecha" a "Disponible" directamente sin vaciado sanitario).
* **FR-004** : El sistema DEBE generar automáticamente un UUID único para cada registro en el historial de estados de galpón.
* **FR-005** : El sistema DEBE registrar de forma obligatoria los siguientes datos en cada cambio de estado:
  * UUID del galpón afectado.
  * Estado anterior.
  * Estado nuevo.
  * Identificación del usuario autenticado que realiza el cambio.
  * Fecha y hora del servidor (marca de tiempo).
* **FR-006** : El sistema DEBE procesar la actualización del estado del galpón y la creación del registro del historial de estados de forma **atómica** bajo una única transacción de base de datos.
* **FR-007** : El sistema DEBE mostrar mensajes de error claros y específicos en la interfaz cuando el usuario intente una transición prohibida o concurrente.
* **FR-009** : El sistema DEBE actualizar inmediatamente la lista y el detalle del galpón tras una transición de estado exitosa.

##### Key Entities *(include if feature involves data)*

* **Galpón** : Representa la unidad física de producción cuyo estado se actualiza .
  * UUID único 
  * Nombre 
  * Aforo máximo 
  * Estado actual (Disponible, vaciado sanitario, productivo, en cosecha, mantenimiento, aislamiento) 

* **Lote** : Representa el conjunto de aves alojado en el galpón. No debe verse afectado en sus datos internos durante las transiciones operativas generales.
  * UUID único
  * Llave foránea del galpón 
  * Población inicial 
  * Población actual
  
* **Administrador de granja** : Usuario autenticado autorizado para ejecutar este caso de uso de actualización manual de estado.

---

## Success Criteria *(mandatory)*

##### Measurable Outcomes
* **SC-001** : El 100 % de los cambios de estado exitosos debe verse reflejado inmediatamente en la base de datos y actualizarse en la pantalla de listado y detalle del galpón.
* **SC-002** : El 100 % de las transiciones inválidas especificadas en la máquina de estados debe ser rechazado antes de procesarse, mostrando un error amigable al usuario.
* **SC-003** : El 100 % de los cambios de estado confirmados debe generar un registro atómico correspondiente en el Historial de Estados, sin excepciones .
* **SC-004** : Cero (0) galpones deben quedar en estados inconsistentes (por ejemplo, en estado "Aislamiento" sin su respectiva alerta o en "Productivo" sin un lote asociado en base de datos).
* **SC-005** : El 100 % de los errores por concurrencia o fallos de base de datos debe revertir completamente el estado del galpón al valor previo de forma automática.
