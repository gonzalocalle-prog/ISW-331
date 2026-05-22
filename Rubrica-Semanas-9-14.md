# Rúbrica de Evaluación — Semanas 9–14

**Total: 100 puntos | Formato: Demo individual 15-20 min**

---


## 1. Base de Datos y Autenticación *(10 pts)*

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


## 4. Testing y Calidad Técnica *(20 pts)*

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

---

## Banco de Preguntas de Validación por Área

### 1. Base de Datos y Autenticación

*Semanas base: Week 09 y Week 10*

1. Muéstrame con tu diagrama ER cómo identificaste entidades, atributos y relaciones. 
¿Qué parte de tu modelo quedó como 1:N y cuál como N:M?

2. ¿Qué redundancia o anomalía de inserción, actualización o borrado evitaste con tu diseño de base de datos? Dame un ejemplo concreto de lo que habría pasado con una tabla mal modelada.

3. ¿Qué reglas protege tu base de datos además del backend, por ejemplo con `NOT NULL`, `UNIQUE`, `FOREIGN KEY` o restricciones equivalentes, y por qué esas reglas no deberían vivir solo en el código?

4. Explícame el flujo completo de autenticación en tu proyecto: registro o login, verificación de credenciales, emisión de token o sesión, y protección de una ruta o recurso.

5. ¿Por qué elegiste tu estrategia de autenticación actual y no otra? Justifica cómo manejas contraseñas, dónde guardas el token o la sesión, y qué riesgo principal tuviste que mitigar.

### 2. Dockerización y Ejecución de la Aplicación

*Semanas base: Week 11*

1. ¿Qué problema concreto de portabilidad o de "en mi máquina sí funciona" resuelve Docker en tu proyecto?

2. Muéstrame tu `Dockerfile` y justifica el orden de las instrucciones. Si cambias solo código fuente pero no dependencias, 
¿qué parte debería quedar cacheada y por qué?

3. ¿Cómo se comunican tus servicios en `docker-compose` o en tu configuración equivalente, y por qué dentro de Docker el backend no debería conectarse a la base de datos usando `localhost`?

4. ¿Qué información necesita persistir fuera del ciclo de vida del contenedor y cómo la resolviste con volúmenes? ¿Qué pasaría si eliminas el contenedor sin esa configuración?

5. ¿Qué diferencia hay entre tu forma de correr el proyecto en desarrollo y una versión más cercana a producción? Señala al menos una decisión de optimización o seguridad que aplicaste o que te faltaría aplicar.

### 3. Pipeline de CI/CD

*Semanas base: Week 12, Week 12.1 y Week 12.2*

1. ¿Qué evento dispara tu pipeline y por qué ese trigger tiene sentido para tu flujo de trabajo: `push`, `pull_request`, ramas específicas u otro caso?

2. Recorre tu pipeline como una secuencia de quality gates: ¿qué riesgo detecta cada etapa y por qué están en ese orden?

3. En tu workflow, ¿qué parte modelaste como `job` y qué parte como `step`? Explica qué estado se comparte dentro de un job y qué deja de existir al pasar a otro.

4. Si tu pipeline genera una imagen Docker o un artefacto de build, ¿cómo aseguras que lo validado sea lo mismo que luego desplegarías o promoverías a otro entorno?

5. Muéstrame una práctica de madurez o seguridad aplicada en GitHub Actions, por ejemplo cache, `concurrency`, permisos mínimos, acciones pineadas, `environments` o workflows reutilizables, y justifica su valor en tu caso.

### 4. Testing y Calidad Técnica

*Semanas base: Week 13*

1. Muéstrame un test unitario que consideres valioso y explícame qué comportamiento de negocio protege. ¿Por qué ese test merece existir?
2. Muéstrame un test de integración y explícame qué interacción real entre componentes, API, autenticación o base de datos valida que un unit test no alcanzaría a cubrir.

3. ¿Cómo se refleja la pirámide de testing en tu proyecto? Indica dónde concentraste la mayor cantidad de pruebas y por qué elegiste esa distribución.

4. ¿Cómo ejecutas tus tests dentro del flujo de validación del proyecto y qué riesgo aparece si los tests quedan fuera del pipeline?

5. Dame un ejemplo de un test que decidiste no escribir, o de una dependencia que decidiste no mockear por completo, y justifica ese trade-off.

### 5. Explicación Técnica y Criterio Profesional

*Semanas base: integracion de Weeks 09 a 13*

1. Si tuvieras que explicarle este proyecto a una persona nueva como flujo real de entrega, ¿cómo conectas base de datos, autenticación, contenedores, pipeline y tests desde el commit hasta la ejecución final?
2. ¿Qué trade-off técnico tomaste de forma consciente en tu proyecto y qué alternativa descartaste? La respuesta debe incluir el costo de la decisión, no solo su beneficio.
3. ¿Qué secretos o variables de entorno intervienen en tu proyecto y cómo evitaste exponerlos en el repositorio, en la imagen Docker o en el cliente?
4. Si hoy clono tu repositorio en otra máquina y algo falla, ¿cuál sería tu orden de diagnóstico y por qué? Quiero escuchar un proceso, no una lista de comandos memorizados.
5. Si tuvieras dos semanas extra para llevar este proyecto a un estándar más profesional, ¿qué mejora priorizarías primero y qué riesgo real reduciría?