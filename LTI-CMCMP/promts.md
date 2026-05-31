

## PASO 1 — Generar las User Stories

### Prompt 1.1 — Establecer el rol y contexto (enviar primero, en solitario)


Actúa como un Product Manager y Business Analyst senior con experiencia en sistemas ATS (Applicant Tracking System).

Voy a darte el contexto completo de nuestro producto LTI para que lo tengas en cuenta en todos los pasos que seguirán. No generes nada todavía, solo confirma que has entendido el contexto.

**Producto:** LTI — Sistema de Seguimiento de Talento (ATS)
**Objetivo:** Plataforma web para gestionar procesos de selección de personal: candidatos, posiciones abiertas, flujos de entrevistas y evaluaciones.

**Entidades principales del dominio:**
- Candidate: nombre, email, teléfono, dirección, educación, experiencia laboral, CV (PDF/DOCX)
- Position: título, descripción, estado (visible/oculta), empresa, flujo de entrevistas asignado
- Application: relación candidato ↔ posición, con fase actual en el flujo
- Interview: entrevista realizada en un paso concreto, puntuación, notas, entrevistador (Employee)
- InterviewFlow: flujo de entrevistas reutilizable (ej: "Proceso Senior Backend") compuesto de InterviewSteps
- InterviewStep: paso dentro de un flujo (ej: "Screening", "Técnica", "Cultural Fit")
- Company / Employee: empresa y sus empleados que actúan como entrevistadores

**Funcionalidades principales ya identificadas:**
1. Reserva y gestión de candidatos con subida de CV
2. Visualización de posiciones abiertas (dashboard tipo Kanban por etapas)
3. Gestión de posiciones y flujos de entrevistas personalizables
4. Movimiento de candidatos entre etapas del flujo
5. Registro de entrevistas con puntuaciones y notas

**Stack técnico:**
- Frontend: React 18, TypeScript, React Bootstrap, React Router
- Backend: Node.js, Express, TypeScript, arquitectura DDD
- Base de datos: PostgreSQL, Prisma ORM
- Comunicación: API REST

Confirma que has entendido el contexto y espera mis instrucciones.




### Prompt 1.2 — Generar las User Stories con plantilla


Perfecto. Ahora genera un mínimo de 6 User Stories para el sistema LTI cubriendo los distintos tipos de usuarios y funcionalidades clave.

**Usa OBLIGATORIAMENTE esta plantilla para cada User Story:**

---
### US-[número]: [Título corto]

**Como** [tipo de usuario],
**quiero** [acción o funcionalidad],
**para** [beneficio o valor que aporta].

**Criterios INVEST:**
- [ ] Independent
- [ ] Negotiable
- [ ] Valuable
- [ ] Estimable
- [ ] Small
- [ ] Testable

**Acceptance Criteria (formato Given/When/Then):**

*Scenario 1 — Happy path:*
- **Given** [precondición]
- **When** [acción del usuario]
- **Then** [resultado esperado]

*Scenario 2 — Error / caso alternativo:*
- **Given** [precondición]
- **When** [acción del usuario]
- **Then** [resultado esperado]

*Scenario 3 — Edge case:*
- **Given** [precondición]
- **When** [acción del usuario]
- **Then** [resultado esperado]

**Notas técnicas:** [dependencias o consideraciones relevantes para el equipo]

---

**Tipos de usuario a cubrir:**
- Recruiter (reclutador): gestiona posiciones y candidatos
- Hiring Manager: revisa candidatos y da feedback
- Candidate (candidato): aplica y sube su CV
- Admin: configura flujos de entrevistas

**Cobertura funcional mínima requerida:**
- Al menos 1 story sobre gestión de candidatos
- Al menos 1 story sobre el flujo Kanban de posiciones
- Al menos 1 story sobre configuración de flujos de entrevistas
- Al menos 1 story sobre registro de entrevistas/evaluaciones


## PASO 2 — Armar el Backlog priorizado

Dado el siguiente conjunto de User Stories del sistema LTI:

@USerStories-CMCMP

Actúa como Product Manager y prioriza el backlog usando una **matriz Valor de negocio vs Esfuerzo de implementación**.

Para cada User Story, asigna:
- **Valor** (1-5): impacto en usuarios y negocio
- **Esfuerzo** (1-5): complejidad técnica e incertidumbre
- **Prioridad calculada**: Valor / Esfuerzo (ratio)

Entrega:
1. Una **tabla markdown** con columnas: | ID | Título | Valor (1-5) | Esfuerzo (1-5) | Ratio V/E | Prioridad |
2. El **orden final del backlog** de mayor a menor prioridad
3. Las **Quick Wins** (alto valor, bajo esfuerzo) destacadas
4. Los **ítems a evitar ahora** (bajo valor, alto esfuerzo)

Incluye una conclusión breve sobre qué iteraciones o sprints formarías con estos grupos.


## PASO 3 — Generar Tickets de trabajo 

### Prompt 3.1 — Desglose técnico en tickets


Actúa como Tech Lead y desglosa la User Story 'US-03: Configuración de Flujos de Entrevistas Personalizados' en **tickets de trabajo técnicos**, tal como se haría en una sesión de Sprint Planning.

Para cada ticket usa esta estructura:

---
#### TICKET-[número]: [Título técnico]

**Tipo:** [Frontend / Backend / Base de datos / Testing / DevOps]
**Descripción técnica:** Qué hay que implementar exactamente, con referencias a capas DDD si aplica (domain / application / presentation / infrastructure).
**Archivos o componentes afectados:** Lista de ficheros o módulos que se crearán o modificarán.
**Criterios de aceptación técnicos:**
- [ ] criterio 1
- [ ] criterio 2
**Dependencias:** [otros tickets que deben completarse antes]
**Notas de implementación:** Consideraciones, gotchas, o decisiones técnicas relevantes.

---

**Stack de referencia:**
- Frontend: React 18 + TypeScript + React Bootstrap
- Backend: Node.js + Express + TypeScript, arquitectura DDD (domain / application / presentation / infrastructure)
- ORM: Prisma con PostgreSQL
- API: REST

Incluye tickets para: modelo de datos / migración Prisma, endpoint(s) de la API, lógica de servicio en application layer, componente(s) React, y tests unitarios mínimos.


