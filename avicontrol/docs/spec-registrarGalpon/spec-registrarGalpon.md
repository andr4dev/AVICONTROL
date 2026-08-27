# Feature Specification: Registrar Galpón (Módulo 1)

**Created**: 2026-08-24

  

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Registro Básico de Infraestructura (Priority: P1)

Como Administrador, quiero registrar un nuevo galpón en el sistema ingresando su identificación única y capacidad técnica, para digitalizar la capacidad física de la granja y establecer la base de datos central de activos.

  

**Why this priority**: Es la funcionalidad más crítica del Módulo 1. Sin la existencia de la infraestructura física en el sistema, es imposible distribuir la población inicial, gestionar el ciclo biológico o calcular la rentabilidad.

  

**Independent Test**: Se puede probar independientemente mediante la interfaz de usuario ingresando un nombre/ID y un valor de aforo máximo. El valor entregado es la creación exitosa del registro en la base de datos que podrá ser consultado posteriormente.

  

**Acceptance Scenarios**:

  

1. **Scenario**: Registro exitoso de un nuevo galpón
    
      
    - **Given** que el Administrador ha iniciado sesión en el sistema y se encuentra en el Módulo 1.
        
          
        
    - **When** ingresa un ID/Nombre único (ej. "Galpón Norte") y una Capacidad Técnica válida (ej. 10000).
        
          
        
    - **Then** el sistema guarda el registro y el galpón aparece en el listado de activos.
        
          
        
2. **Scenario**: Fallo por identificador duplicado
    
      
    - **Given** que ya existe un galpón registrado con el ID "Galpón 01".
        
          
        
    - **When** el Administrador intenta registrar un nuevo galpón utilizando el mismo ID "Galpón 01".
        
          
        
    - **Then** el sistema rechaza el registro y muestra un mensaje de error indicando que la identificación debe ser un ID único (UUID), número o nombre.
        
          
        

### User Story 2 - Inicialización Automática de Estados y Población (Priority: P2)

Como Administrador, quiero que al registrar un nuevo galpón, el sistema asigne automáticamente los valores iniciales de población, edad y estado sanitario, para evitar errores humanos y garantizar el cumplimiento de las normativas de sanidad.

  

**Why this priority**: Asegura la integridad de los datos desde el momento de la creación. Un galpón recién creado no puede tener pollos vivos ni estar en etapa de engorde sin antes pasar por una validación.

  

**Independent Test**: Puede ser probado creando un galpón solo con los metadatos de capacidad y verificando en la base de datos que los estados del ciclo de vida se asignen por defecto.

  

**Acceptance Scenarios**:

  

1. **Scenario**: Asignación del estado inicial por defecto
    
      
    - **Given** que el Administrador está registrando un nuevo galpón físico.
        
          
        
    - **When** finaliza y guarda el registro exitosamente.
        
          
        
    - **Then** el sistema asigna automáticamente el estado "Vaciado Sanitario", asegurando el periodo de limpieza y desinfección obligatorio antes de ingresar cualquier lote.
        
          
        
2. **Scenario**: Inicialización de métricas biológicas
    
      
    - **Given** que un nuevo galpón acaba de ser creado.
        
          
        
    - **When** se consulta el galpón recién registrado.
        
          
        
    - **Then** el sistema muestra que la "Población Actual" es 0 y la "Edad del Lote" es 0 días.
        
          
        

### Edge Cases

- What happens when el Administrador ingresa una Capacidad Técnica negativa o igual a cero? (El sistema debe bloquear la creación, ya que el aforo máximo basado en metros cuadrados no puede ser nulo o negativo).
    
      
    
- How does system handle la concurrencia si dos administradores intentan registrar galpones diferentes simultáneamente pero el sistema de autogeneración de IDs (UUID) colisiona?
    
      
    
- What happens when la conexión a la base de datos central de activos se interrumpe justo después de hacer clic en "Guardar"? (El sistema debe manejar el error y prevenir la creación de registros huérfanos o corruptos).
    
      
    

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST permitir al Administrador ingresar metadatos para definir la capacidad operativa del galpón.
    
      
    
- **FR-002**: System MUST requerir y validar que la Identificación sea un ID Único (UUID), número o nombre del galpón que no exista previamente en el sistema.
    
      
    
- **FR-003**: System MUST requerir el ingreso de la Capacidad Técnica (aforo máximo de pollos), calculada opcionalmente en base a metros cuadrados y ventilación.
    
      
    
- **FR-004**: System MUST inicializar la Población Actual (cantidad de pollos vivos) en cero al momento del registro.
    
      
    
- **FR-005**: System MUST inicializar la Edad del Lote en cero (0) días transcurridos al momento de crear el galpón.
    
      
    
- **FR-006**: System MUST asignar automáticamente el estado inicial del ciclo de vida a "Vaciado Sanitario" tras la creación del galpón, para forzar el periodo de limpieza.
    
      
    

### Key Entities _(include if feature involves data)_

- **Galpón**: Representa la unidad física de infraestructura.
    
      
    - _Atributos clave_: ID Único, Capacidad Técnica, Población Actual, Edad del Lote.
        
          
        
    - _Estados del Ciclo de Vida_: Vaciado Sanitario, Productivo (Ocupado), En Cosecha (Venta), Mantenimiento.
        
          
        
    - _Relaciones_: Es consultado por el Módulo 3 para liquidación, y es actualizado y consultado por el Módulo 2 para nutrición y sanidad.
        
          
        

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: El Administrador puede completar el registro de un nuevo galpón en la interfaz en menos de 1 minuto.
    
      
    
- **SC-002**: El 100% de los galpones registrados en el sistema poseen un ID único sin colisiones en la base de datos.
    
      
    
- **SC-003**: Reducción del 100% en errores de ingreso de lotes en galpones sucios, gracias a que el sistema fuerza el estado de "Vaciado Sanitario" al momento del registro.
    
      
    
- **SC-004**: El sistema previene el ingreso de aforos inválidos (valores menores o iguales a cero) con un 100% de efectividad durante la validación del formulario.