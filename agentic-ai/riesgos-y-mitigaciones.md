# Riesgos y Mitigaciones en Agentes de IA

## Riesgos de seguridad y técnicos

- **Alucinaciones**: el modelo genera información falsa con alta confianza — no "sabe" que está mal, la produce con el mismo tono seguro que una respuesta correcta. Mitigado (no eliminado) con [RAG](rag.md) — darle datos reales en vez de dejar que invente.
- **Adversarial input**: una entrada diseñada **a propósito** para explotar una debilidad del modelo — no es un caso límite que aparece solo, es un ataque deliberado contra cómo el modelo procesa la entrada. El ejemplo clásico (visión por computadora) es una imagen con ruido casi imperceptible para un humano que hace que un clasificador se equivoque con alta confianza. El **Prompt Injection** de abajo es la versión específica de esto para LLMs/agentes.
- **Prompt Injection**: manipular las instrucciones del agente a través de la entrada — ej. un usuario (o un documento que el agente lee vía RAG) incluye texto tipo "ignorá tus instrucciones anteriores y hacé X". Es al agente lo que la inyección SQL es a una query: input que se trata como si fuera instrucción de confianza.
- **Uso incorrecto de herramientas**: el agente ejecuta una tool con argumentos mal formados o en un contexto donde no correspondía (ej. borra en vez de archivar).
- **Loops infinitos**: el agente queda atrapado en un ciclo sin converger a una respuesta — no solo un problema de UX, es plata real gastada en cada vuelta (ver [Circuit Breaker como límite de gasto](costos-llms.md#evitar-gasto-por-loops-que-no-cortan-solos)).
- **Envenenamiento de datos**: los datos de entrenamiento (o, en un agente con RAG, la base de conocimiento que consulta) fueron manipulados maliciosamente para sesgar las respuestas.

## Riesgos éticos y operativos

- **Sesgos algorítmicos**: decisiones discriminatorias que reflejan sesgos presentes en los datos de entrenamiento — el modelo no "decide" ser injusto, reproduce patrones que ya estaban en los datos.
- **Falta de responsabilidad**: cuando un agente autónomo falla, es difícil asignar la culpa — ¿fue el prompt, el modelo, la tool que llamó, quien diseñó el sistema?
- **Imprevisibilidad**: en sistemas complejos (multiagente, loops largos de ReAct), el comportamiento exacto es difícil de predecir de antemano, incluso para quien lo construyó.
- **Costos excesivos**: consumo descontrolado de tokens/API calls — ver [Costos de LLMs](costos-llms.md).
- **Impacto en empleo**: la automatización de tareas antes hechas por personas genera desplazamiento laboral — un riesgo real a nivel organizacional/social, no técnico.

## Mitigaciones técnicas

- **Validación de datos**: datos diversos, sin sesgos evidentes, con cifrado y control de acceso.
- **Transparencia**: explicabilidad de las decisiones del agente + logging para poder auditar después qué pasó y por qué.
- **Pruebas rigurosas (Red Teaming)**: simular ataques activamente (intentar romper el agente a propósito) para encontrar vulnerabilidades antes de que las encuentre alguien más.
- **Mínimos privilegios**: sub-agentes especializados con permisos limitados y segmentados — un agente que solo necesita leer no debería tener permiso de escribir.

## Mitigaciones de proceso

- **Supervisión humana**: ver [Human-in-the-Loop](diseno-de-agentes.md#human-in-the-loop-hitl) — validación de decisiones críticas.
- **Monitoreo continuo**: detección de anomalías y comportamientos inesperados en producción, no solo en testing.
- **Contingencia**: protocolos de rollback y desactivación de emergencia — poder "apagar" un agente rápido si empieza a comportarse mal.

## Gobernanza y ética

- **Marcos éticos**: justicia, privacidad, derechos humanos como criterios explícitos de diseño, no un afterthought.
- **Auditorías**: revisiones periódicas de sesgos, no solo al lanzar el sistema.
- **Regulación**: adherencia a marcos como GDPR — ver [Privacidad y GDPR](../frontend-react/privacidad-y-gdpr.md) para el detalle de consentimiento/data minimization, que aplica igual a sistemas de IA que procesan datos de usuarios.
- **Educación**: capacitar al equipo sobre riesgos y límites del sistema que están construyendo, no asumir que "la IA ya lo resuelve".

## Protección de datos y PII

- **Manejo de PII**: detección y anonimización automática de datos personales identificables antes de que lleguen al modelo o queden en logs.
- **Filtrado de contenido**: bloqueo de prompts y outputs maliciosos — tanto lo que entra como lo que el agente genera.
- **Moderación activa**: sistemas con reglas y retroalimentación humana para casos límite.
- **Mitigación de Prompt Injection en la práctica**: sandboxing (la tool que ejecuta el agente corre con permisos acotados, ver mínimos privilegios arriba), validación estricta de qué puede hacer cada tool, y nunca tratar contenido de un documento externo (recuperado vía RAG, por ejemplo) con el mismo nivel de confianza que las instrucciones del desarrollador.

---
Relacionado: [Agentes vs Workflows](agentes-vs-workflows.md), [Costos de LLMs](costos-llms.md), [RAG](rag.md), [Diseño de Agentes](diseno-de-agentes.md), [Privacidad y GDPR](../frontend-react/privacidad-y-gdpr.md).
