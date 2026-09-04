# Definición de los casos de uso - Modulo 1

Este documento explica brevemente los casos de uso encontrados en el diagrama, dando una información mas profunda de lo que se quiere conseguir con cada caso de uso.
### Actores
- **Administrador**: Este actor tiene a su cargo la fase operativa del galpón, es el único que puede registrar un galpón, editar un galpón, registrar un lote y actualizar el estado del galpón. 

- **Módulo 2**: Este modulo se encarga de remover pollos en caso de que muera, también genera una alerta al administrador en la que avisa que en el galpón se encuentran uno o varios pollos enfermos, además, el modulo necesita saber la edad del lote para poder llevar control de la dieta de los pollos.

- **Módulo 3**: Este modulo necesita conocer los pollos que se encuentran en el lote para calcular la venta bruta, además, necesita conocer cuanto costó el lote para calcular los costos de crianza.

- **Técnico de infraestructura**: Este actor tiene a su cargo generar una alerta al administrador para avisarle que se encontraron fallas en la infraestructura del galpón que necesitan ser atendidas.

---
## Casos de uso
### 1. Administrador
- **Registrar galpón**: En este caso de uso el administrador ocupa llenar un formulario donde ingresa los datos necesarios del galpón (nombre y capacidad).

- **Registrar Lote**: En este caso de uso el administrador ocupa llenar un formulario donde ingresa los datos necesarios del lote (nombre, fecha de ingreso, población inicial y el costo del lote).

- **Editar galpón**: En este caso de uso el administrador ocupa editar el galpón en caso de que sea ampliado o necesite un cambio de nombre.

- **Actualizar estado**: En este caso de uso el administrador ocupa actualizar el estado del galpón en caso de: 
	1. El modulo 2 generó la alerta para hacer un aislamiento al galpón.
	2. El actor "Técnico de infraestructura" generó la alerta para hacer un mantenimiento.
	3. Cuando finalice el vaciado sanitario el galpón queda disponible para un nuevo ingreso de un lote.
	4. Cuando el galpón este listo para cosechar.
	5. Cuando el galpón finalizó cosecha del lote y necesita un vaciado sanitario.

**Nota**: las transiciones tienen las siguientes restricciones:
- **Productivo**: Solo se cambia a este estado si el estado actual es "Disponible".
- **En cosecha**: Solo se cambia a este estado si el estado actual es "Productivo".
- **Disponible**: Solo se cambia a este estado si el estado actual es "Vaciado sanitario". 
- **Aislamiento**: Solo se cambia a este estado si el estado actual es "Productivo".
- **Vaciado sanitario**: Solo se cambia a este estado si el estado actual es "En cosecha".
- **Mantenimiento**: Solo se cambia a este estado si el estado actual es "Disponible". 

### 2. Módulo 2
- **Registrar retiro de ave**: En este caso de uso el modulo 2 avisa al administrador que se retiró un pollo del galpón porque murió. 

- **Actualizar población actual**: En este caso de uso el sistema resta uno o varios pollos que habían inicialmente en el galpón (Esta ligado obligatoriamente a el caso de uso "Registrar retiro de ave").

- **Registrar alerta sanitaria**: En este caso de uso el modulo 2 ocupa avisar al administrador de que en el galpón hay una enfermedad que contagió a uno o varios pollos y el galpón necesita estar en aislamiento.
### 3. Técnico de infraestructura
- **Generar alerta de mantenimiento**: En este caso de uso el técnico de infraestructura ocupa avisarle al administrador que hay fallas en el galpón y necesitan atención.

### 4. Compartidos (Administrador, Módulo 2 y Módulo 3)
- **Consultar galpón y/o lote**: En este caso de uso el: 
	1. **Administrador**: Ocupa visualizar los galpones y lotes activos en el sistema.
	2. **Módulo 2**: Ocupa visualizar los galpones y lotes activos para conocer la edad del lote para el control de la dieta de los pollos.
	3. **Módulo 3**: Ocupa visualizar los galpones y lotes activos para conocer el costo del lote y los pollos finales para poder calcular los costos de crianza y la venta bruta del lote. 
