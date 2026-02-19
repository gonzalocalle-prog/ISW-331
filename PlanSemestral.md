# Aplicaciones Web II

## Información General

| Campo | Valor |
|-------|-------|
| **Código** | ISW-331 |
| **Duración** | 20 semanas |
| **Horas/semana** | 6 horas |
| **Estudiantes** | Máximo 10 |

---

## Sobre el Curso

Este curso prepara estudiantes para el mundo laboral real. Al finalizar, cada estudiante tendrá:

- Un proyecto desplegado en producción
- Un portafolio actualizado
- Experiencia con herramientas y flujos de trabajo profesionales
- Confianza para aplicar a posiciones junior/trainee

---

## Estrategia Pedagógica

### Blended Learning

Cada semana se divide en:

| Modalidad | Duración | Actividad |
|-----------|----------|-----------|
| **Asíncrono** | ~2 hrs | Video/lectura + quiz de comprensión |
| **Sincrónico 1** | 2 hrs | Discusión + ejercicio guiado |
| **Sincrónico 2** | 2 hrs | Trabajo en proyecto personal + mentoría |

### Sistema de Starters

<!-- Cada tema tiene un repositorio plantilla funcional. Esto permite:

- Aprender conceptos nuevos sin depender del avance del proyecto personal
- Practicar en código que ya funciona
- Avanzar todos juntos en los conceptos, independientemente del proyecto -->

Cada estudiante elige un stack y se compromete con él durante el semestre.

### Sistema de Capas (Proyecto Personal)

| Capa | Requisitos | Nota máxima |
|------|------------|-------------|
| **Base** | CRUD funcional, auth, deploy básico | Hasta 70 |
| **Intermedia** | CI/CD, tests, Docker | Hasta 85 |
| **Avanzada** | Optimización, seguridad avanzada, features extra | Hasta 100 |

### Proyecto de Rescate

Si a la **semana 10** un estudiante va muy retrasado:

- Cambiar proyecto predefinido más simple
- Recibe acompañamiento adicional

---

## Sistema de Evaluación

| Evaluación | Semana | Peso | Contenido |
|------------|--------|------|-----------|
| **Parcial 1** | 7 | 25% | Demo: Frontend + Backend conectados |
| **Parcial 2** | 14 | 25% | Demo: CI/CD + Docker + Deploy funcional |
| **Final** | 20 | 50% | Producto completo + Pitch + Portafolio |

---

## Plan Semanal

### BLOQUE 1: Fundamentos y Setup (Semanas 1-3)

#### Semana 1 - Kickoff y Visión del Curso

**Asíncrono:**
- Qué hace un desarrollador Full Stack en 2026
- Tour de una aplicación en producción real

**Sincrónico 1:**
- Presentación del curso (estilo onboarding laboral)
- Dinámica: "Recruiter" - cada estudiante presenta su CV/portafolio actual

**Sincrónico 2:**
- Lluvia de ideas para proyectos personales
- Definir problema, usuario objetivo, features principales

**Entregable:**
- One-pager del proyecto (problema, usuario, 5 features principales)

---

#### Semana 2 - Arquitectura y CSS Moderno

**Asíncrono:**
- MVC vs Clean Architecture vs Microservicios
- Caso de uso: Utility-first CSS (Tailwind)

**Sincrónico 1:**
- Discusión: ¿Cuándo usar cada patrón arquitectónico?
- Ejercicio: Clonar UI de app conocida

**Sincrónico 2:**
- Definir arquitectura de su proyecto personal
- Crear wireframes básicos**

**Entregable:**
- Diagrama de arquitectura de su proyecto (C4)
- Wireframes de 3 pantallas principales *

---

#### Semana 3 - Setup Profesional

**Asíncrono:**
- Git Flow, Github Flow, Trunk based y trabajo con branches
- Configuración de linters y formatters

**Sincrónico 1:**
- Setup guiado: ESLint, Prettier, Husky
- Práctica: crear PR, hacer code review entre compañeros

**Sincrónico 2:**
- Configurar repositorio de proyecto personal
- Escribir README profesional

**Entregable:**
- Repositorio con estructura profesional, linters configurados, README completo

---

### BLOQUE 2: Frontend Avanzado (Semanas 4-6)

#### Semana 4 - React/Vue** Avanzado

**Asíncrono:**
- Estado global - Context vs Redux vs Zustand
- Lectura: Custom hooks patterns**

**Sincrónico 1:**
- Ejercicio: Implementar auth context
- Refactorizar componentes a custom hooks**

**Sincrónico 2:**
- Implementar estructura de estado en proyecto personal

**Entregable:**
- Componente funcional con estado global implementado

---

#### Semana 5 - Comunicación Asíncrona con APIs

**Asíncrono:**
- Fetch vs Axios vs React Query**
- Lectura: Manejo de estados de carga y error

**Sincrónico 1:**
- Ejercicio: Consumir API pública
- Implementar loading skeletons, error boundaries

**Sincrónico 2:**
- Conectar proyecto personal a API mock (JSON Server o similar)

**Entregable:**
- Frontend conectado a API con manejo correcto de estados

---

#### Semana 6 - Preparación Parcial 1

**Asíncrono:**
- Revisar documentación de su proyecto
- Preparar demo

**Sincrónico 1:**
- Sesión de debugging grupal
- Resolver dudas técnicas

**Sincrónico 2:**
- Mentoría 1:1 para revisar avances
- Ensayo de demo

**Entregable:**
- Frontend listo para presentar

---

### BLOQUE 3: Backend y Base de Datos (Semanas 7-10)

#### Semana 7 - PARCIAL 1

**Evaluación:**
- Demo individual (15 min por estudiante)
- Mostrar frontend funcional conectado a API (mock está ok)**
- Explicar decisiones de arquitectura

**Criterios:**
- Código limpio y organizado
- Manejo de estado correcto
- UI responsive
- Manejo de errores

---

#### Semana 8 - Backend Fundamentals

**Asíncrono:**
- RESTful API design principles
- Lectura: Estructura de proyecto backend

**Sincrónico 1:**
- Ejercicio (Node o Flask**): Crear CRUD endpoints
- Validación de datos, manejo de errores HTTP

**Sincrónico 2:**
- Diseñar endpoints de proyecto personal
- Comenzar implementación

**Entregable:**
- API con al menos 3 endpoints funcionales (documentados)

---

#### Semana 9 - Base de Datos

**Asíncrono:**
- SQL vs NoSQL - cuándo usar cada uno**
- Lectura: Modelado de datos y relaciones

**Sincrónico 1:**
- Ejercicio: Conectar backend a MongoDB/PostgreSQL
- Diseñar schemas, crear índices

**Sincrónico 2:**
- Implementar base de datos en proyecto personal

**Entregable:**
- Schema de base de datos implementado y conectado al backend

---

#### Semana 10 - Autenticación y Autorización

**Asíncrono:**
- JWT, sessions, OAuth**
- Lectura: RBAC (Role-Based Access Control)**

**Sincrónico 1:**
- Ejercicio: Implementar auth completo
- Login, registro, refresh tokens, roles

**Sincrónico 2:**
- Implementar auth en proyecto personal

**Checkpoint de rescate:**
- Evaluar quién necesita cambiar a proyecto de rescate

**Entregable:**
- Sistema de auth funcional con al menos 2 roles

---

### BLOQUE 4: DevOps y CI/CD (Semanas 11-14)

#### Semana 11 - Docker

**Asíncrono:**
- Containers 101 - qué problema resuelven
- Lectura: Dockerfile y docker-compose

**Sincrónico 1:**
- Ejercicio: Dockerizar aplicación full stack
- Multi-stage builds, optimización de imágenes

**Sincrónico 2:**
- Dockerizar proyecto personal

**Entregable:**
- docker-compose.yml funcional que levanta toda la aplicación

---

#### Semana 12 - CI/CD con GitHub Actions

**Asíncrono:**
- Qué es CI/CD y por qué importa
- Lectura: GitHub Actions syntax

**Sincrónico 1:**
- Ejercicio: Crear pipeline completo
- Lint → Test → Build → Deploy

**Sincrónico 2:**
- Implementar pipeline en proyecto personal

**Entregable:**
- Pipeline de CI/CD funcional

---

#### Semana 13 - Testing

**Asíncrono:**
- Pirámide de testing
- Lectura: Jest/Vitest para unit tests, testing-library para integración**

**Sincrónico 1:**
- Ejercicio: Escribir tests unitarios y de integración
- TDD: escribir test primero, luego el código**

**Sincrónico 2:**
- Agregar tests a proyecto personal

**Entregable:**
- Al menos 5 tests unitarios y 2 tests de integración

---

#### Semana 14 - PARCIAL 2

**Evaluación:**
- Demo individual (15 min por estudiante)
- Mostrar: aplicación dockerizada, pipeline CI/CD, deploy automático

**Criterios:**
- Docker funcional
- Pipeline ejecutándose correctamente
- Deploy automático a ambiente de prueba
- Tests pasando

---

### BLOQUE 5: Cloud y Producción (Semanas 15-17)

#### Semana 15 - Azure Fundamentals

**Asíncrono:**
- Servicios de Azure para desarrolladores
- Lectura: Azure App Service, Blob Storage

**Sincrónico 1:**
- Ejercicio: Deploy a Azure App Service
- Configurar variables de entorno, secretos

**Sincrónico 2:**
- Migrar proyecto personal a Azure

**Entregable:**
- Aplicación corriendo en Azure con dominio accesible

---

#### Semana 16 - Base de Datos en la Nube

**Asíncrono:**
- Managed databases - pros y contras
- Lectura: Optimización de queries, índices

**Sincrónico 1:**
- Ejercicio: Migrar a MongoDB Atlas o Azure Cosmos DB
- Configurar backups, connection pooling

**Sincrónico 2:**
- Migrar base de datos de proyecto personal

**Entregable:**
- Base de datos en la nube conectada a la aplicación

---

#### Semana 17 - Monitoreo y Logging

**Asíncrono:**
- Por qué el monitoreo salva vidas (y trabajos)**
- Lectura: Application Insights, Sentry

**Sincrónico 1:**
- Ejercicio: Configurar error tracking y métricas
- Crear alertas básicas

**Sincrónico 2:**
- Implementar monitoreo en proyecto personal

**Entregable:**
- Dashboard de monitoreo con al menos 3 métricas

---

### BLOQUE 6: Seguridad y Cierre (Semanas 18-20)

#### Semana 18 - Seguridad Web

**Asíncrono:**
- OWASP Top 10
- Lectura: XSS, CSRF, SQL Injection - cómo prevenirlos**

**Sincrónico 1:**
- Mini-hackathon: encontrar vulnerabilidades en apps de compañeros**
- Documentar hallazgos y fixes

**Sincrónico 2:**
- Aplicar fixes de seguridad en proyecto personal

**Entregable:**
- Reporte de seguridad con al menos 3 vulnerabilidades encontradas/arregladas

---

#### Semana 19 - Pulido Final y Portafolio

**Asíncrono:**
- Lectura: Cómo presentar proyectos técnicos
- Portfolios que destacan

**Sincrónico 1:**
- Workshop: mejorar README, agregar screenshots, GIFs
- Documentación técnica profesional

**Sincrónico 2:**
- Preparar pitch de 10 minutos
- Actualizar portafolio/CV

**Entregable:**
- Portafolio actualizado con el proyecto destacado

---

#### Semana 20 - FINAL

**Evaluación:**
- Pitch individual (20 min presentación + 5 min preguntas)
- Formato: simulación de entrevista técnica

**Contenido del pitch:**
1. El problema que resuelve
2. Demo en vivo de la aplicación
3. Arquitectura técnica (diagrama C4)
4. Decisiones técnicas y trade-offs
5. Qué harías diferente / próximos pasos

**Criterios:**
- Producto funcional en producción
- Código limpio y documentado
- Pipeline CI/CD funcionando
- Capacidad de explicar decisiones técnicas
- Portafolio profesional

**Cierre:**
- Retrospectiva grupal
- Feedback individual
- Networking: compartir LinkedIns, GitHubs

---

## Herramientas Recomendadas (Free Tier)**

| Propósito | Opciones |
|-----------|----------|
| **Frontend hosting** | Vercel, Netlify |
| **Backend hosting** | Railway, Render, Azure App Service |
| **Base de datos** | MongoDB Atlas, PlanetScale, Azure Cosmos DB |
| **CI/CD** | GitHub Actions |
| **Monitoreo** | Sentry, Azure Application Insights |
| **Containers** | Docker Desktop, Podman |
| **Diseño** | Figma, Excalidraw |

---

## Recursos Adicionales

### Recursos online
- [roadmap.sh](https://roadmap.sh) - Roadmaps de desarrollo
- [patterns.dev](https://patterns.dev) - Patrones de diseño en JS
- [web.dev](https://web.dev) - Best practices de Google
- 
- 
-

---

## Contacto y Soporte

- **Horario de consultas:** [Definir]
- **Canal de comunicación:** [Whatsapp]
- **Repositorio del curso:** [URL]
