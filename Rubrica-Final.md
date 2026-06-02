# Rúbrica de Evaluación — Examen Final

**Total: 100 puntos | Formato: Demo 15-20 min**

---

## Requisitos mínimos obligatorios

1. La aplicación debe estar desplegada y accesible mediante una URL pública funcional.
2. El proceso de build y despliegue debe estar automatizado mediante pipeline, workflow o mecanismo reproducible equivalente.
3. La demo debe realizarse sobre la versión desplegada, no solamente en local.

> **Nota:** La plataforma de hosting puede ser cualquiera. Lo evaluado no es el proveedor, sino que la aplicación esté realmente publicada, funcione de extremo a extremo y llegue a producción mediante un proceso automatizado. Si no existe URL pública funcional o no existe automatización del proceso de entrega, la evaluación no puede considerarse completa.

---

## 1. Aplicación Integral y Dominio del Proyecto *(40 pts — más importante)*

| Puntaje | Criterio |
|---------|----------|
| **36–40** | La aplicación pública funciona de extremo a extremo y demuestra valor real como producto. El estudiante muestra el flujo principal completo y puede responder preguntas sobre frontend, backend, base de datos, autenticación, estados, errores, despliegue y arquitectura sin quedarse en respuestas memorizadas. Justifica decisiones y trade-offs con criterio técnico. |
| **26–35** | La aplicación funciona en lo esencial y la integración principal existe, pero hay vacíos visibles en alguna capa, partes incompletas o respuestas débiles al profundizar en decisiones técnicas. |
| **16–25** | La aplicación corre parcialmente o solo algunos módulos están realmente integrados. El estudiante explica partes aisladas, pero no logra conectar el sistema completo ni defender cómo opera en conjunto. |
| **0–15** | La aplicación no demuestra un flujo real e integrado, depende de pantallas estáticas o el estudiante no puede explicar cómo funciona globalmente ni para quién fue construida. |

---

## 2. Despliegue Público y Operación de la Aplicación *(25 pts)*

| Puntaje | Criterio |
|---------|----------|
| **22–25** | Existe una URL pública funcional y estable. El estudiante demuestra que la versión desplegada es la que debe evaluarse, explica dónde corre cada componente, dónde viven los datos, cómo se consultan los logs y cómo verifica que la aplicación quedó saludable después del despliegue. |
| **16–21** | La aplicación está publicada y usable, pero hay debilidades en observabilidad, salud, persistencia, documentación operativa o claridad sobre cómo está montado el entorno. |
| **8–15** | La URL pública existe pero la demo es inestable, incompleta o depende de ajustes manuales. Hay vacíos importantes al explicar almacenamiento, logs, health checks o comportamiento de la app ya desplegada. |
| **0–7** | No hay URL pública funcional o no puede demostrarse una versión desplegada operativa del sistema. |

---

## 3. Automatización de Build y Entrega *(20 pts)*

| Puntaje | Criterio |
|---------|----------|
| **18–20** | El proceso de build y entrega está automatizado de forma clara y reproducible. Existe pipeline o workflow funcional que construye, valida y despliega la aplicación con mínima o nula intervención manual. El estudiante puede explicar qué lo dispara, qué etapas ejecuta y cómo protege la calidad antes de publicar cambios. |
| **14–17** | La automatización existe y resuelve la mayor parte del flujo, pero mantiene pasos manuales, validaciones limitadas o dependencias operativas poco claras. |
| **8–13** | Solo una parte del proceso está automatizada. El build o el deploy requieren varios pasos manuales, configuraciones ad hoc o intervención frecuente para funcionar. |
| **0–7** | No existe automatización real del proceso de entrega o el estudiante no puede demostrarla. |

---

## 4. Configuración y Criterio Profesional *(15 pts)*

| Puntaje | Criterio |
|---------|----------|
| **13–15** | El estudiante identifica con precisión los comandos de build y arranque, las variables de entorno requeridas, qué valores son secretos, cómo se inyectan sin exponerlos en el repositorio, dónde persisten los datos y cómo diagnosticar problemas con logs y checks de salud. Responde con criterio profesional, no solo recitando comandos. |
| **8–12** | Puede responder la mayoría de los puntos operativos y de seguridad, pero con uno o dos vacíos relevantes o explicaciones imprecisas. |
| **4–7** | Conoce comandos o configuraciones aisladas, pero no domina el sistema operativo de su aplicación ni el manejo seguro de configuración y secretos. |
| **0–3** | No puede identificar comandos, variables, secretos, almacenamiento, logs o validaciones básicas de salud del sistema. |

---

## Checklist obligatorio de demostración

Durante la demo, el estudiante debe poder mostrar o responder con precisión lo siguiente:

1. ¿Cuál es la URL pública?
2. ¿Qué comando construye la aplicación?
3. ¿Qué comando inicia la aplicación?
4. ¿Qué variables de entorno son requeridas?
5. ¿Qué valores son secretos y no deben vivir en el repositorio?
6. ¿Dónde se almacenan los datos?
7. ¿Cómo se consultarán los logs?
8. ¿Cómo verificamos que la app quedó saludable después del despliegue?

> **Nota:** No basta con nombrar estos elementos de memoria. El estudiante debe poder ubicar dónde están definidos, cómo se usan y por qué importan dentro del flujo real de entrega y operación.

---

## Banco de Preguntas de Validación por Área

### 1. Aplicación Integral y Arquitectura

1. Recorre tu aplicación desde la URL pública hasta la persistencia de datos. ¿Qué componentes intervienen y cómo se comunican?
2. Muéstrame el flujo principal del producto y explícame qué parte resuelve cada capa: frontend, backend, base de datos y autenticación si aplica.
3. Si cambio un requisito importante del negocio hoy, ¿qué parte de tu arquitectura sería la más sensible y por qué?
4. ¿Qué decisión técnica de tu proyecto tuvo el mayor impacto en el resultado final y qué alternativa descartaste?
5. Si yo te pregunto por cualquier archivo o módulo importante del proyecto, ¿cómo justificarías que existe y qué responsabilidad tiene?

### 2. Despliegue Público y Operación

1. Muéstrame la URL pública y explícame qué infraestructura o plataforma la sirve.
2. ¿Cuál es exactamente el comando de build y cuál es el comando de arranque? ¿Dónde están definidos?
3. ¿Qué variables de entorno necesita tu aplicación para funcionar y cuáles de ellas son secretas?
4. ¿Dónde se almacenan los datos y qué pasaría si recreas el contenedor, la instancia o el servicio donde corre la app?
5. ¿Cómo revisas logs y cómo compruebas que la aplicación está saludable después de un despliegue?

### 3. Automatización y Entrega Continua

1. ¿Qué evento dispara tu proceso automatizado y por qué ese trigger tiene sentido para tu flujo de trabajo?
2. Recorre tu pipeline como secuencia: build, validaciones, artefacto, deploy. ¿Qué riesgo reduce cada etapa?
3. ¿Qué parte de tu entrega sigue siendo manual y por qué todavía no la automatizaste?
4. ¿Cómo garantizas que los secretos no terminan en el repositorio, en la imagen o expuestos al cliente?
5. Si el deploy falla, ¿en qué parte del flujo buscarías primero y qué evidencia usarías para diagnosticarlo?

### 4. Criterio Profesional y Diagnóstico

1. Si clono tu repositorio en otra máquina, ¿qué necesito para construir y ejecutar la aplicación correctamente?
2. Si la URL pública responde pero el flujo principal falla, ¿cuál es tu orden de diagnóstico y por qué?
3. Si una variable de entorno falta o está mal configurada, ¿cómo lo notarías y cómo lo corregirías sin comprometer seguridad?
4. ¿Qué evidencia concreta usarías para demostrar que tu aplicación no solo está levantada, sino realmente saludable?
5. Si tuvieras una iteración más para mejorar este proyecto, ¿qué priorizarías primero para hacerlo más profesional y qué riesgo real reduciría?

---

> **Nota final:** En este examen no se evalúan piezas aisladas. Se evalúa la aplicación como sistema completo: publicada, operativa, explicable y entregada mediante un proceso automatizado.