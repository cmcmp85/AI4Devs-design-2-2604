## PASO 1 — Generar las User Stories
### US-01: Postulación de Candidato y Carga de CV

**Como** Candidate,
**quiero** aplicar a una posición abierta y subir mi currículum en formato PDF o DOCX,
**para** postularme oficialmente al puesto y que el equipo de reclutamiento evalúe mi perfil.

**Criterios INVEST:**

* [x] Independent: Es el punto de entrada de la entidad Candidate sin depender de flujos avanzados.
* [x] Negotiable: El peso del archivo y los campos requeridos se pueden ajustar con diseño.
* [x] Valuable: Permite alimentar el sistema con nuevos perfiles de manera automatizada.
* [x] Estimable: Frontend sencillo con un endpoint de carga de archivos estándar.
* [x] Small: Se centra únicamente en la creación del perfil y la postulación inicial.
* [x] Testable: Se valida mediante la creación del registro en la base de datos y la persistencia del archivo.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el candidato está en el formulario público de una posición con estado "visible"
* **When** completa los campos requeridos (nombre, email, teléfono) y adjunta un archivo PDF válido de 2MB
* **Then** el sistema crea el registro del `Candidate`, la relación `Application` en el paso inicial, almacena el CV y muestra un mensaje de éxito.

*Scenario 2 — Error / caso alternativo:*

* **Given** que el candidato está completando el formulario de postulación
* **When** intenta enviar el formulario adjuntando un archivo con formato no permitido (ej: `.png` o `.exe`)
* **Then** el sistema bloquea el envío, resalta el campo del CV en rojo y muestra el error: "Formato no válido. Solo se admiten archivos .pdf y .docx".

*Scenario 3 — Edge case:*

* **Given** que un candidato con un email ya existente en la base de datos intenta aplicar a una posición diferente
* **When** envía el formulario de postulación
* **Then** el sistema no duplica la entidad `Candidate`, pero crea una nueva `Application` asociada a ese ID existente y vincula el nuevo CV si este fue actualizado.

**Notas técnicas:**

* Dependencia de un servicio de almacenamiento de archivos (S3 o similar) y del modelo `Candidate` / `Application` en Prisma.
* Límite de subida inicial recomendado: 5MB.

---

### US-02: Gestión Visual de Candidatos en Tablero Kanban

**Como** Recruiter,
**quiero** mover las tarjetas de los candidatos entre las diferentes columnas del tablero Kanban,
**para** actualizar la fase del flujo de selección de un candidato de forma visual y ágil.

**Criterios INVEST:**

* [x] Independent: Modifica el estado de `Application` de forma aislada a las notas de la entrevista.
* [x] Negotiable: El drag-and-drop puede tener animaciones complejas o transiciones simples según el sprint.
* [x] Valuable: Optimiza radicalmente el tiempo del reclutador en la gestión diaria.
* [x] Estimable: Uso de librerías estándar de React para drag-and-drop y una mutación simple en backend.
* [x] Small: Solo cubre el movimiento visual y la actualización de la fase en base de datos.
* [x] Testable: Comprobando que el campo `InterviewStep` cambia en la base de datos tras soltar la tarjeta.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el reclutador está visualizando el tablero Kanban de una posición específica
* **When** arrastra la tarjeta del candidato "Juan Pérez" desde la columna "Screening" a la columna "Técnica"
* **Then** la tarjeta se posiciona en la nueva columna y el sistema actualiza el estado de la `Application` de Juan a la fase "Técnica".

*Scenario 2 — Error / caso alternativo:*

* **Given** que el reclutador está moviendo un candidato en el tablero y ocurre un error de conectividad de red
* **When** suelta la tarjeta en la columna de destino
* **Then** el sistema revierte visualmente la tarjeta a su columna original ("Screening") y muestra una notificación: "Error de conexión. No se pudo actualizar el estado".

*Scenario 3 — Edge case:*

* **Given** que el reclutador intenta mover un candidato a una etapa de entrevista técnica, pero dicha etapa requiere la asignación obligatoria previa de un examinador
* **When** arrastra la tarjeta a esa columna
* **Then** el sistema detiene el movimiento, abre un modal flotante para seleccionar el examinador (`Employee`) antes de confirmar el cambio de etapa.

**Notas técnicas:**

* Se requiere implementar componentes de React específicos para drag-and-drop (ej: `@hello-pangea/dnd`).
* El backend en Node/Express debe validar que el `InterviewStep` de destino pertenezca genuinamente al `InterviewFlow` de esa posición.

---

### US-03: Configuración de Flujos de Entrevistas Personalizados

**Como** Admin,
**quiero** crear y estructurar flujos de entrevistas reutilizables con pasos específicos,
**para** estandarizar los procesos de selección según la naturaleza de cada rol (ej: Técnico, Ejecutivo).

**Criterios INVEST:**

* [x] Independent: Define plantillas de flujos sin necesidad de que haya candidatos activos asociados.
* [x] Negotiable: Se puede limitar inicialmente el número máximo de pasos por flujo.
* [x] Valuable: Evita tener que configurar el proceso desde cero para cada nueva vacante.
* [x] Estimable: CRUD estándar enfocado en las entidades `InterviewFlow` e `InterviewStep`.
* [x] Small: Se enfoca solo en la creación y ordenamiento del flujo, no en su asignación a la vacante.
* [x] Testable: Validando las inserciones relacionales correctas en PostgreSQL mediante Prisma.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el administrador está en la sección de configuración de flujos
* **When** crea un flujo titulado "Proceso Senior Backend" y añade secuencialmente los pasos: "Screening", "Live Coding" y "Cultural Fit"
* **Then** el sistema guarda el `InterviewFlow` con sus 3 `InterviewSteps` ordenados correctamente y confirma la creación con un mensaje en pantalla.

*Scenario 2 — Error / caso alternativo:*

* **Given** que el administrador está configurando un nuevo flujo
* **When** intenta guardar el flujo con el título vacío o sin añadir ningún paso (`InterviewStep`)
* **Then** el sistema deshabilita el botón de guardar y muestra alertas específicas indicando los campos obligatorios faltantes.

*Scenario 3 — Edge case:*

* **Given** que el administrador intenta eliminar un paso de un flujo existente
* **When** ese flujo específico ya está siendo utilizado activamente por una posición con candidatos en proceso
* **Then** el sistema bloquea la eliminación, alertando que el flujo está en uso y sugiriendo la duplicación del flujo como alternativa para no romper la integridad de los datos.

**Notas técnicas:**

* Arquitectura DDD: El agregado `InterviewFlow` debe manejar la consistencia e integridad del orden secuencial de sus `InterviewSteps` mediante un campo `order` o `sequence`.

---

### US-04: Registro de Evaluaciones y Feedback de Entrevistas

**Como** Hiring Manager,
**quiero** registrar la puntuación numérica y las notas de feedback de la entrevista de un candidato,
**para** dejar constancia de su desempeño y avanzar o descartar su postulación con base en criterios claros.

**Criterios INVEST:**

* [x] Independent: La puntuación se guarda en el registro de la entrevista independientemente de si el candidato se mueve de columna inmediatamente o no.
* [x] Negotiable: La escala de puntuación puede ser del 1 al 5 o del 1 al 10.
* [x] Valuable: Centraliza el feedback del equipo técnico y evita la pérdida de información en canales externos.
* [x] Estimable: Formulario sencillo con campos de texto enriquecido y selectores numéricos.
* [x] Small: Centrado únicamente en la edición/guardado de la entidad `Interview`.
* [x] Testable: Verificación del guardado correcto del feedback numérico y de texto en la base de datos.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el Hiring Manager ha sido asignado a una entrevista específica de un candidato
* **When** accede a la ficha de la entrevista, selecciona una puntuación de "4/5" y escribe en las notas: "Excelente dominio de Node.js y patrones arquitectónicos"
* **Then** el sistema guarda los cambios en la entidad `Interview` y actualiza el estado de la entrevista a "Completada".

*Scenario 2 — Error / caso alternativo:*

* **Given** que el Hiring Manager está completando la evaluación
* **When** intenta guardar el registro dejando el campo de puntuación numérica vacío
* **Then** el sistema impide el guardado, resalta el selector de puntuación y muestra el mensaje: "La calificación numérica es obligatoria para cerrar la evaluación".

*Scenario 3 — Edge case:*

* **Given** que un Hiring Manager intenta modificar las notas de una entrevista que ya fue cerrada y firmada hace más de 48 horas
* **When** hace clic en el botón de guardar cambios
* **Then** el sistema le deniega el acceso de edición (campo de solo lectura) indicando que el periodo de modificación ha expirado para asegurar la transparencia del proceso.

**Notas técnicas:**

* Requiere control de accesos a nivel de aplicación (RBAC) para garantizar que solo el `Employee` asignado (o un Admin) pueda escribir notas en esa `Interview`.

---

### US-05: Búsqueda y Filtrado Avanzado de Candidatos

**Como** Recruiter,
**quiero** buscar candidatos por nombre o filtrar la lista global por habilidades y nivel de experiencia,
**para** localizar rápidamente perfiles específicos sin tener que revisar manualmente todo el pool de talento.

**Criterios INVEST:**

* [x] Independent: Lee datos de la base de datos sin alterar los estados de los procesos de selección directos.
* [x] Negotiable: Se puede priorizar la búsqueda por texto exacto antes de implementar una búsqueda difusa (fuzzy search).
* [x] Valuable: Ahorra horas de scroll y revisión manual cuando la base de datos crece.
* [x] Estimable: Implementación de queries con filtros condicionales mediante Prisma ORM.
* [x] Small: No maneja acciones sobre los candidatos, solo la visualización filtrada del listado.
* [x] Testable: Ejecución de búsquedas con strings de prueba para contrastar los resultados devueltos con los registros reales.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el reclutador está en el listado global de candidatos
* **When** escribe "TypeScript" en la barra de búsqueda y selecciona el filtro de educación "Universitaria completa"
* **Then** el sistema renderiza exclusivamente las tarjetas de los candidatos que cumplen con ambos criterios en tiempo real.

*Scenario 2 — Error / caso alternativo:*

* **Given** que el reclutador aplica un término de búsqueda muy específico (ej: "Zxyw123") o una combinación imposible de filtros
* **When** el sistema procesa la consulta
* **Then** la interfaz oculta la lista y muestra una ilustración limpia con el mensaje: "No se encontraron candidatos que coincidan con tu búsqueda. Intenta con otros términos".

*Scenario 3 — Edge case:*

* **Given** que el reclutador introduce caracteres especiales de inyección de código (ej: `' OR 1=1 --` o `<script>`) en la barra de búsqueda rápida
* **When** presiona enter o ejecuta la consulta
* **Then** el sistema sanitiza la entrada en el backend, no rompe la query de Prisma y devuelve 0 resultados de forma segura sin comprometer la base de datos.

**Notas técnicas:**

* Utilizar indexación en campos críticos de la base de datos PostgreSQL (`name`, `email`) para optimizar el rendimiento de las consultas `LIKE` o `ILIKE` en Prisma.

---

### US-06: Creación y Publicación de Vacantes

**Como** Recruiter,
**quiero** registrar una nueva posición laboral especificando título, descripción y asignándole un flujo de entrevistas preconfigurado,
**para** habilitar las postulaciones y coordinar al equipo que evaluará el puesto.

**Criterios INVEST:**

* [x] Independent: Crea la entidad `Position` de forma autónoma, requiriendo solo asociar un ID de flujo ya existente.
* [x] Negotiable: La descripción de la vacante puede ser texto plano o texto enriquecido (Rich Text) según el alcance del sprint.
* [x] Valuable: Es el punto de partida operativo para cualquier proceso de reclutamiento dentro de la plataforma.
* [x] Estimable: Formulario estándar conectado a las entidades `Position`, `Company` e `InterviewFlow`.
* [x] Small: Se limita al proceso de alta y cambio de visibilidad de la posición.
* [x] Testable: Creando la vacante y verificando que aparezca listada correctamente en el portal público/privado.

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*

* **Given** que el reclutador está en el módulo de "Nueva Posición"
* **When** introduce "Senior Backend Developer", añade una descripción, selecciona la empresa, vincula el flujo "Proceso Senior Backend" y marca el estado como "Visible"
* **Then** el sistema guarda la entidad `Position`, genera la URL pública para postulaciones y la muestra en la lista de vacantes activas.

*Scenario 2 — Error / caso alternativo:*

* **Given** que el reclutador intenta guardar una vacante
* **When** olvida asociar un flujo de entrevistas (`InterviewFlow`) del menú desplegable
* **Then** el sistema bloquea el guardado, resalta el campo de selección del flujo y muestra el mensaje: "Debes asignar un flujo de entrevistas para poder abrir la posición".

*Scenario 3 — Edge case:*

* **Given** que el reclutador decide cambiar el estado de una posición activa con postulantes a "Oculta"
* **When** confirma la acción en la interfaz
* **Then** la vacante deja de ser accesible al público externo inmediatamente (evitando nuevas postulaciones), pero mantiene intacto el acceso del equipo interno en el dashboard Kanban con todos sus candidatos activos.

## PASO 2 — Armar el Backlog priorizado



### 1. Tabla de Priorización (Matriz Valor vs. Esfuerzo)

Para calcular el **Ratio V/E**, dividimos el Valor entre el Esfuerzo. A mayor ratio, mayor prioridad estratégica tendrá la historia en etapas tempranas.

| ID | Título | Valor (1-5) | Esfuerzo (1-5) | Ratio V/E | Prioridad |
| --- | --- | --- | --- | --- | --- |
| **US-06** | Creación y Publicación de Vacantes

 | 5 | 2 | **2.50** | **Alta** |
| **US-01** | Postulación de Candidato y Carga de CV

 | 5 | 3 | **1.67** | **Alta** |
| **US-04** | Registro de Evaluaciones y Feedback

 | 4 | 3 | **1.33** | **Media-Alta** |
| **US-02** | Gestión Visual en Tablero Kanban

 | 5 | 4 | **1.25** | **Media** |
| **US-03** | Configuración de Flujos Personalizados

 | 3 | 3 | **1.00** | **Media-Baja** |
| **US-05** | Búsqueda y Filtrado Avanzado

 | 3 | 3 | **1.00** | **Media-Baja** |



### 2. Orden Final del Backlog (Secuencia de Ejecución)

El orden propuesto no solo sigue estrictamente el **Ratio V/E**, sino también las dependencias lógicas del negocio (no se puede postular un candidato a una vacante que no existe):

1. **US-06: Creación y Publicación de Vacantes** (Ratio: 2.50) — *Bloqueante inicial.*


2. **US-01: Postulación de Candidato y Carga de CV** (Ratio: 1.67) — *Esencial para captar datos.*


3. **US-04: Registro de Evaluaciones y Feedback** (Ratio: 1.33) — *Aporta valor inmediato tras recibir perfiles.*


4. **US-02: Gestión Visual en Tablero Kanban** (Ratio: 1.25) — *La joya de la corona operativa, pero requiere base previa.*


5. **US-03: Configuración de Flujos Personalizados** (Ratio: 1.00) — *Optimización de procesos.*


6. **US-05: Búsqueda y Filtrado Avanzado** (Ratio: 1.00) — *Solo es útil cuando el volumen de datos crece.*




### 3. Quick Wins (Alto Valor, Bajo Esfuerzo)

* **US-06: Creación y Publicación de Vacantes:** Es el disparador de todo el ecosistema de LTI. Técnicamente es un formulario estructurado relativamente sencillo (CRUD básico conectado a tablas maestras), pero a nivel de negocio define el punto de partida operativo indispensable. Al asignarle un flujo por defecto, resolvemos el flujo inicial sin fricciones.




### 4. Ítems a Evitar Ahora (Bajo Valor Relativo, Mayor Esfuerzo)

En esta fase del MVP, no tenemos historias de "bajo valor y alto esfuerzo" crítico (antipatrones), pero sí características que **debemos posponer**:

* **US-05: Búsqueda y Filtrado Avanzado:** Si el sistema está naciendo, el pool de candidatos en la base de datos será pequeño. Invertir tiempo de desarrollo en indexación compleja de PostgreSQL, sanitización de búsquedas y queries condicionales mediante Prisma no mueve la aguja del negocio en el día uno. Puede esperar a que el volumen de candidatos lo justifique.


* **US-03: Configuración de Flujos Personalizados:** Aunque el diseño DDD contempla la flexibilidad, podemos iniciar el producto con 2 o 3 flujos fijos en base de datos (*hardcoded* mediante un *seed* de Prisma). Darle una interfaz al Administrador para crear pasos dinámicos, ordenarlos y validar que no rompan procesos activos añade una capa de complejidad que se puede mitigar al principio.




### Conclusión y Estrategia de Sprints

Para garantizar un flujo de entrega constante de valor (Early ROI), organizaría el desarrollo en **dos Sprints de 2 semanas**:

* **Sprint 1 (El Core Operativo / MVP):** Enfocado en **US-06**, **US-01** y **US-04**. Con esto cubrimos el ciclo completo básico: el reclutador publica la vacante, el candidato aplica cargando su CV, y el Hiring Manager puede entrar a calificar y dejar feedback de forma plana. El sistema ya es funcional y aporta valor real.


* **Sprint 2 (La Experiencia de Usuario y Escalabilidad):** Enfocado en **US-02** (Tablero Kanban con Drag & Drop), permitiendo que la gestión deje de ser a través de formularios y pase a ser totalmente visual. Al final de este sprint, e introduciendo de forma simplificada la **US-03** o **US-05**, el producto adquiere la madurez necesaria para competir en el mercado de sistemas ATS.

## PASO 3 — Generar Tickets de trabajo


#### TICKET-01: Modelado de Base de Datos y Migración con Prisma

**Tipo:** Base de datos
**Descripción técnica:** Diseñar la persistencia en el archivo `schema.prisma` para las entidades del agregado `InterviewFlow`. Se deben mapear las relaciones uno a muchos entre `InterviewFlow` e `InterviewStep`. El modelo de `InterviewStep` debe contener un campo numérico (ej. `sortOrder` o `sequence`) para mantener la integridad secuencial requerida por las reglas de negocio.
**Archivos o componentes afectados:**

* `infrastructure/database/prisma/schema.prisma`
* Carpeta de migraciones generada automáticamente (`prisma/migrations/`)
**Criterios de aceptación técnicos:**
* [ ] Ejecutar `prisma migrate dev` sin errores, generando la estructura correspondiente en PostgreSQL.
* [ ] El modelo `InterviewFlow` debe incluir restricciones de unicidad en el título (opcional pero recomendado para evitar duplicados accidentales).
* [ ] El modelo `InterviewStep` debe contar con una clave foránea apuntando a `InterviewFlow` con borrado/actualización en cascada restringida si está en uso.
**Dependencias:** Ninguna.
**Notas de implementación:** Prisma maneja las relaciones de forma nativa. Asegurar que el campo que gestiona el orden de los pasos sea un entero indexado para agilizar las consultas de ordenamiento.

---

#### TICKET-02: Definición del Dominio y Lógica del Agregado (Domain Layer)

**Tipo:** Backend (Domain Layer)
**Descripción técnica:** Implementar las entidades de dominio y el Agregado Raíz (`InterviewFlow`) dentro de la capa Core. Se deben codificar las reglas y validaciones de negocio descritas en la User Story: un flujo no puede crearse sin título, no puede guardarse vacío (sin pasos) y se debe encapsular el comportamiento para añadir, remover u ordenar los `InterviewSteps` asegurando la consistencia de su secuencia.
**Archivos o componentes afectados:**

* `domain/interview-flow/entities/InterviewFlow.ts`
* `domain/interview-flow/entities/InterviewStep.ts`
* `domain/interview-flow/value-objects/` (ej. FlowTitle)
* `domain/interview-flow/exceptions/` (excepciones de dominio personalizadas)
**Criterios de aceptación técnicos:**
* [ ] La entidad `InterviewFlow` debe validar que su estado sea íntegro antes de ser persistida (mínimo 1 paso obligatoriamente).


* [ ] Los métodos del agregado para manipular pasos deben autogestionar el índice secuencial de los `InterviewSteps` para que no queden huecos en la numeración.
**Dependencias:** TICKET-01
**Notas de implementación:** No inyectar infraestructura ni librerías externas aquí. Es TypeScript puro (POJO). Toda mutación del flujo debe pasar centralizada por el Agregado Raíz `InterviewFlow`.

---

#### TICKET-03: Caso de Uso de Creación de Flujos e Infraestructura (Application & Infrastructure Layer)

**Tipo:** Backend (Application & Infrastructure)
**Descripción técnica:** Implementar el caso de uso (Use Case) de creación en la capa de aplicación, coordinando la validación del dominio, y desarrollar la implementación concreta del repositorio utilizando Prisma en la capa de infraestructura.
**Archivos o componentes afectados:**

* `application/interview-flow/use-cases/CreateInterviewFlowUseCase.ts`
* `domain/interview-flow/repositories/IInterviewFlowRepository.ts` (Interface)
* `infrastructure/interview-flow/repositories/PrismaInterviewFlowRepository.ts` (Implementación)
**Criterios de aceptación técnicos:**
* [ ] El Repositorio Prisma debe realizar la inserción del flujo y de todos sus pasos de manera transaccional (`prisma.$transaction`).
* [ ] El caso de uso debe capturar las excepciones del dominio y mapearlas correctamente para las capas superiores.
**Dependencias:** TICKET-02
**Notas de implementación:** Asegurarse de mapear las entidades de persistencia de Prisma a las entidades de dominio puro al retornar o guardar datos en el repositorio (patrón Data Mapper).

---

#### TICKET-04: Controlador REST y Enrutamiento (Presentation Layer)

**Tipo:** Backend (Presentation Layer)
**Descripción técnica:** Exponer el endpoint HTTP POST `/api/v1/interview-flows` en la capa de presentación (Express) para permitir la recepción de la carga útil del frontend. Deberá encargarse de la sanitización básica de los datos de entrada y de invocar al caso de uso correspondiente.
**Archivos o componentes afectados:**

* `presentation/express/controllers/InterviewFlowController.ts`
* `presentation/express/routes/interviewFlowRoutes.ts`
* `presentation/express/dtos/CreateInterviewFlow.dto.ts`
**Criterios de aceptación técnicos:**
* [ ] El endpoint debe validar que el payload cumpla con el formato JSON requerido (ej. usando `class-validator` o `zod`).
* [ ] Responder con código `201 Created` e incluir el objeto creado en el cuerpo de la respuesta en el Happy Path.


* [ ] Retornar un código de estado `400 Bad Request` ante errores de validación de campos obligatorios o de reglas de negocio del dominio.
**Dependencias:** TICKET-03
**Notas de implementación:** Mantener el controlador "delgado" (*skinny controller*). Su única responsabilidad es el HTTP parsing, la validación sintáctica del DTO y delegar la ejecución a la capa de aplicación.



---

#### TICKET-05: Componentes de Interfaz de Usuario y Formulario Dinámico (Frontend)

**Tipo:** Frontend
**Descripción técnica:** Desarrollar las vistas y componentes en React 18 para la creación de los flujos de entrevistas utilizando React Bootstrap. El formulario debe permitir ingresar el título general y añadir interactivamente una lista dinámica de pasos (inputs dinámicos que se agregan/eliminan en tiempo real).
**Archivos o componentes afectados:**

* `src/components/interview-flow/InterviewFlowForm.tsx`
* `src/pages/admin/CreateInterviewFlowPage.tsx`
* `src/services/api/interviewFlowService.ts` (Cliente Axios/Fetch para consumir la API)
**Criterios de aceptación técnicos:**
* [ ] El formulario debe bloquear la acción de envío (deshabilitar botón) si el título está vacío o si la lista no cuenta con pasos añadidos.


* [ ] Conectarse adecuadamente con el endpoint desarrollado en el TICKET-04 enviando la estructura JSON esperada.
* [ ] Renderizar alertas visuales de error de React Bootstrap ante fallas devueltas por la API.
**Dependencias:** TICKET-04
**Notas de implementación:** Se puede usar `react-hook-form` o el estado nativo de React para controlar el array dinámico de pasos. El diseño debe respetar los lineamientos estéticos del dashboard de administración de LTI.



---

#### TICKET-06: Automatización de Pruebas Unitarias e Integración

**Tipo:** Testing
**Descripción técnica:** Crear la suite de pruebas automatizadas para asegurar la cobertura del flujo desarrollado, enfocándose en la lógica de negocio del agregado de dominio y en la integración del endpoint de la API.
**Archivos o componentes afectados:**

* `tests/unit/domain/interview-flow/InterviewFlow.test.ts`
* `tests/integration/presentation/interviewFlow.api.test.ts`
**Criterios de aceptación técnicos:**
* [ ] El test unitario del dominio debe verificar que se lance una excepción si se intenta guardar un flujo con título vacío o sin pasos asociados.


* [ ] El test de integración de la API debe simular una llamada HTTP POST válida, verificar la persistencia simulada o real en base de datos de pruebas y comprobar el código de respuesta `201`.
**Dependencias:** TICKET-04 (puede ejecutarse en paralelo con el TICKET-05)
**Notas de implementación:** Usar Jest o Vitest como ejecutor de pruebas. Para las pruebas de integración a la API se sugiere emplear `supertest`. Asegurar un entorno limpio o mocks de base de datos adecuados para evitar la contaminación entre ejecuciones de pruebas.