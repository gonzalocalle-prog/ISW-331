# Rúbrica de Evaluación — Semanas 9–14

**Total: 100 puntos | Formato: Demo individual 15-20 min**

---


## 1. Base de Datos y Autenticación *(20 pts)*

| Puntaje | Criterio |
|---------|----------|
| **18–20** | El proyecto tiene persistencia funcional y autenticación implementada correctamente. La base de datos está conectada al backend, los datos se guardan y recuperan de forma consistente, y el flujo de login/registro o protección de rutas funciona. El estudiante puede explicar el modelo de datos y las decisiones básicas de seguridad. |
| **14–17** | La base de datos y la autenticación funcionan en lo esencial, pero hay detalles incompletos en validación, protección de rutas, manejo de sesiones/tokens o consistencia de los datos. |
| **8–13** | Solo una parte está implementada correctamente: la base de datos o la autenticación. Hay fallas visibles en la integración con el backend o en el flujo de acceso. |
| **0–7** | No tiene base de datos funcional conectada al proyecto o no puede demostrar autenticación operativa. |

---

## 2. Dockerización y Ejecución de la Aplicación *(30 pts)*

| Puntaje | Criterio |
|---------|----------|
| **27–30** | La aplicación se levanta correctamente con Docker o docker-compose. Los servicios principales están bien definidos, la configuración es clara y el estudiante puede explicar cómo correr el proyecto y qué resuelve cada contenedor. |
| **20–26** | La aplicación corre en contenedores, pero con ajustes manuales, documentación incompleta o problemas menores de configuración. El estudiante entiende la mayor parte de la configuración. |
| **10–19** | Solo una parte del sistema corre en Docker o la ejecución es inestable. Hay errores de configuración, puertos, variables de entorno o dependencias no resueltas. |
| **0–9** | La aplicación no corre en contenedores o la configuración no permite demostrar el funcionamiento del proyecto. |

---

## 3. Pipeline de CI/CD *(30 pts)*

| Puntaje | Criterio |
|---------|----------|
| **27–30** | Tiene un pipeline funcional que se ejecuta automáticamente y valida pasos relevantes del proyecto, como lint, tests, build o imagen Docker. El estudiante puede explicar cuándo se activa, qué verifica y por qué está organizado de esa manera. |
| **20–26** | El pipeline existe y corre, pero valida solo una parte del flujo o tiene pasos poco claros. El estudiante entiende su propósito general, aunque no justifica bien algunas decisiones. |
| **10–19** | El pipeline está incompleto, falla con frecuencia o depende de intervención manual para funcionar. La automatización es limitada. |
| **0–9** | No tiene pipeline funcional o no puede demostrar su ejecución. |

---


## 4. Testing y Calidad Técnica *(10 pts)*

| Puntaje | Criterio |
|---------|----------|
| **9–10** | Tiene tests relevantes pasando y conectados al flujo de validación. El estudiante puede explicar qué cubren, por qué esos tests importan y cómo ayudan a evitar regresiones. |
| **7–8** | Tiene tests básicos pasando, pero la cobertura es limitada o la explicación sobre su valor es superficial. |
| **4–6** | Tiene pocos tests, no están integrados claramente al pipeline o su utilidad para el proyecto es dudosa. |
| **0–3** | No tiene tests funcionales o no puede demostrar que están pasando. |

---

## 5. Explicación Técnica y Criterio Profesional *(10 pts)*

| Puntaje | Criterio |
|---------|----------|
| **9–10** | Explica con claridad las decisiones tomadas en base de datos, autenticación, Docker, CI/CD y testing. Puede responder preguntas, justificar trade-offs y conectar la solución técnica con un flujo de trabajo profesional. |
| **7–8** | Explica la mayoría de decisiones, aunque con vacíos en algunos conceptos o justificaciones poco precisas. |
| **4–6** | Describe lo implementado de forma mecánica, pero no demuestra comprensión suficiente de por qué funciona o por qué fue elegido ese enfoque. |
| **0–3** | No puede explicar las decisiones técnicas principales ni defender el flujo implementado. |

---

> **Nota:** Si el estudiante no puede explicar cómo se relacionan base de datos, autenticación, Docker, pipeline y tests dentro de un flujo real de desarrollo y entrega, el criterio no se considera completo aunque la demo funcione parcialmente.