# Plan de Estudio — Desarrollo de Agentes de IA (v2)

> **Objetivo:** ser capaz de **construir soluciones de agentes a nivel empresarial y monetizables**
> **Perfil:** Python + AWS (Bedrock / AgentCore) · estilo: teoría antes que práctica
> **Duración estimada:** ~12-14 semanas a ritmo part-time
> **Última actualización:** 2026-08-30

---

## Contexto

El desarrollo de agentes es, en esencia, **un LLM dentro de un loop** que razona, llama herramientas (*tools*), observa el resultado y decide el siguiente paso. Todo framework o plataforma es azúcar alrededor de ese loop. Este plan va de los fundamentos conceptuales hacia **producción empresarial y comercialización**, aprovechando que AWS ofrece un camino limpio en Python: el framework open-source **Strands Agents** + la plataforma de producción **Amazon Bedrock AgentCore** + los canales de venta de **AWS Marketplace**.

**Regla de oro:** distinguir *workflows* (vos definís los pasos, el LLM rellena) de *agents* (el LLM decide el flujo). El 80% de lo que vale en producción son workflows. Empezar por lo autónomo es el error más común.

**Advertencia honesta:** leer y ver cursos NO te vuelve capaz. La capacidad viene de **construir y depurar**. Por eso cada fase cierra con un **mini-entregable**. Apuntá a 30% teoría / 70% práctica.

---

## Curso práctico de referencia (AWS Samples) ⭐

Repo oficial de AWS con la contraparte *hands-on* de este plan, en **el mismo stack** (Strands + Bedrock + MCP + AgentCore). Python 3.10+, licencia MIT-0, ~5-6 h de labs. Es el recurso práctico más alineado; se referencia fase por fase abajo.

- **Repo:** https://github.com/aws-samples/sample-getting-started-with-strands-agents-course
- Ruta de 4 cursos progresivos: Fundamentos → MCP avanzado → Multi-agente → Producción.

| Curso del repo | Encaje con el plan |
|---|---|
| **Course 1 — Getting Started with Strands** (agents, model providers, MCP intro, A2A, observabilidad con LangFuse/RAGAS) | Fases 2, 3, 6 |
| **Course 2 — Advanced Strands with MCP** (SDK avanzado, hooks/async/retry, `@tool` + MCP, session managers, memoria FAISS/OpenSearch/Mem0 + Knowledge Bases) | Fases 2, 3, 4 |
| **Course 3 — Building Multi-Agent Systems** (swarm, agent graph, agents-as-tools) | Fase 2 (orquestación) + opcionales |
| **Course 4 — Production Deployment with AgentCore** (Runtime serverless, deployment, session mgmt) | Fase 5 |

> **Nota de stack:** el Course 2 usa API key de Anthropic como proveedor por defecto (Bedrock es opcional ahí). Para mantenernos Bedrock-first, corré esos labs apuntando al **provider de Bedrock** de Strands. El Course 4 sí exige cuenta AWS + Bedrock (Claude 3.5 Haiku habilitado, política `BedrockAgentCoreFullAccess`).
>
> **Qué NO cubre el repo (sigue siendo tuyo):** el loop agéntico a mano (Fase 1), gran parte de Fase 6 (HITL, guardrails, prompt injection, audit, EU AI Act) y toda la Fase 7 (multi-tenancy, FinOps, Well-Architected, Marketplace). El deployment del Course 4 es manual (coincide con el pendiente de IaC + CI/CD).

---

## Prerrequisitos (verificar antes de empezar)

Si alguno flojea, atajalo primero o tropezás en las Fases 2-4.

- [ ] **Python intermedio**, especialmente **`async/await`** (Strands y Bedrock lo usan intensamente)
- [ ] APIs REST, JSON, manejo de errores/excepciones
- [ ] **Fundamentos de AWS/IAM**: cuentas, roles, políticas, least-privilege
- [ ] Git y entornos virtuales (venv/poetry/uv)
- [ ] **Pydantic** (validación de datos; base para *structured outputs*)

**Recursos:**
- Skill Builder — *AWS Cloud Practitioner Essentials* (si el AWS base flojea): https://aws.amazon.com/training/learn-about/ai
- Docs de Pydantic: https://docs.pydantic.dev/

---

## Fase 0 — Modelo mental + Prompt engineering (1-2 semanas)

**Objetivo:** entender *qué* es un agente y dominar el skill que más impacta en que funcione: **escribir buenos prompts y descripciones de herramientas**.

**Conceptos (solo teoría):**
- Tool use / function calling · el loop agéntico
- Context window, system prompt, structured output
- Memoria: corto vs largo plazo · RAG
- Workflow vs agente autónomo
- **Prompt engineering**: instrucciones claras, few-shot, chain-of-thought, descripciones de tools que el modelo entienda

**Lectura:**
- Anthropic — *Building effective agents* (texto fundacional): https://www.anthropic.com/engineering/building-effective-agents
- Anthropic — Guía de prompt engineering: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Cursos Skill Builder:**
- *Foundations of Prompt Engineering* (~1 h): https://explore.skillbuilder.aws/learn/course/17763/foundations-of-prompt-engineering
- *Fundamentals of Prompt Engineering with Claude* (casos AWS reales: infra, security reviews, documentación): https://skillbuilder.aws/learn/TU33983VQ7/fundamentals-of-prompt-engineering-with-claude/SB2G63EBUR

**Mini-entregable:** un system prompt + 2 tool specs bien escritas, probadas en el playground de Bedrock.

---

## Fase 1 — Fundamentos de LLM en Python (1-2 semanas)

**Objetivo:** hablar directo con un modelo vía API, sin frameworks.

**Herramienta:** Boto3 → Amazon Bedrock, específicamente la **Converse API** (interfaz unificada que maneja tool use de forma estándar entre modelos).

**Lectura:**
- Getting started con Converse API en Python: https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started-api-ex-python.html
- Ejemplos de código Bedrock Runtime (incluye **tool use con Converse**): https://docs.aws.amazon.com/code-library/latest/ug/python_3_bedrock-runtime_code_examples.html

**Curso Skill Builder:**
- *Getting Started with Amazon Bedrock*: https://explore.skillbuilder.aws/learn/course/external/view/elearning/17508/getting-started-with-amazon-bedrock

**Tip de costos:** iterá con modelos chicos y baratos (Claude Haiku, Amazon Nova Lite). Subí a modelos frontera solo cuando midas que hace falta.

**Mini-entregable:** el **loop agéntico a mano** (~80 líneas): prompt + 2 tools → el modelo pide una → la ejecutás → devolvés resultado → repetís. Esto demuestra que un framework no es magia.

---

## Fase 2 — Un framework: Strands Agents (2-3 semanas)

**Objetivo:** dominar UN framework, no cinco. Strands es el SDK open-source de AWS en Python, agnóstico de modelo, y el que mejor se integra con AgentCore.

**Qué aprender:**
- Definir agentes y herramientas de forma declarativa
- Manejo de memoria y sesiones
- Orquestación multi-agente (un agente que delega en subagentes)
- **Structured outputs con Pydantic** (confiabilidad)

**Lectura:**
- Documentación de Strands Agents: https://strandsagents.com/
- Provider de Bedrock en Strands (config, token counting): https://strandsagents.com/docs/user-guide/concepts/model-providers/amazon-bedrock/
- Desplegar Strands en AgentCore: https://strandsagents.com/docs/user-guide/deploy/deploy_to_bedrock_agentcore/

*(Alternativa a conocer: LangGraph, el más usado en la industria para grafos con estado. Empezar por Strands para este stack.)*

**Práctica (repo AWS Samples):**
- **Course 1** — Fundamentos: inicialización de agentes, system prompts, model providers (Claude/Bedrock), integración S3/DynamoDB con `use_aws`.
- **Course 2, Labs 1-3 y 5** — SDK avanzado: ciclo de vida y métricas del agente, structured output, thinking mode, hooks event-driven, async/retry logic, y session managers (Null / SlidingWindow / Summarizing, persistencia en archivos y S3). *El Summarizing manager conecta con FinOps de Fase 7 (truncar historial = menos tokens).*
- **Course 3** — Orquestación multi-agente (swarm, agent graph, agents-as-tools). *Opcional según lo pida el capstone.*

**Mini-entregable:** un agente en Strands con 3+ tools, memoria de sesión y salida validada con Pydantic.

---

## Fase 3 — MCP: Model Context Protocol (2 semanas)

**Objetivo:** conectar agentes a sistemas reales (APIs, DBs, servicios internos). El "USB-C de la IA". Ventaja de perfil para quien viene de arquitectura cloud/enterprise.

**Qué aprender:**
- Concepto de servidor MCP: tools, resources, prompts
- Escribir un servidor MCP en Python
- Cómo **AgentCore Gateway** convierte APIs y funciones Lambda en herramientas MCP sin reescribir código
- Seguridad de MCP: auth (OAuth 2.1), identidad, prompt injection

**Lectura:**
- Documentación oficial de MCP: https://modelcontextprotocol.io
- Roadmap de MCP (identidad + seguridad enterprise): https://blog.modelcontextprotocol.io/posts/mcp-roadmap

**Práctica (repo AWS Samples):**
- **Course 1, Lab 4** — MCP & Tools: notebook interactivo, servidores MCP (calculadora/clima).
- **Course 2, Lab 4** — Integración `@tool` + MCP con servidores reales (AWS Documentation, AWS Pricing), *self-extending agents* y meta tooling.

**Mini-entregable:** un servidor MCP propio que exponga una API interna como herramienta, consumido por tu agente de la Fase 2.

---

## Fase 4 — RAG y grounding con Bedrock Knowledge Bases (2 semanas)

**Objetivo:** dar al agente conocimiento propietario y actualizado. Casi toda solución empresarial lo necesita.

**Qué aprender:**
- Embeddings, chunking, vector stores, retrieval
- **Bedrock Knowledge Bases**: Managed vs Customer-managed
- *Agentic retrieval* (multi-hop), re-ranking, filtrado por permisos (ACL)
- Integración nativa Knowledge Base ↔ AgentCore Gateway (se descubre como tool MCP)

**Lectura:**
- Bedrock Knowledge Bases (cómo funciona y tipos): https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html
- Grounding y RAG (guía prescriptiva, enfoque enterprise): https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-serverless/grounding-and-rag.html

**Cursos Skill Builder:**
- Lab: *Build and Evaluate RAG Applications using Knowledge Bases for Amazon Bedrock*: https://explore.skillbuilder.aws/learn/course/external/view/elearning/21658/lab-build-and-evaluate-retrieval-augmented-generation-rag-applications-using-knowledge-bases-for-amazon-bedrock
- *Generative AI: Amazon Bedrock (Advanced)* — Jam Journey (prompt eng + RAG + agentes): https://explore.skillbuilder.aws/learn/courses/22256/generative-ai-amazon-bedrock-advanced

**Práctica (repo AWS Samples):**
- **Course 2, Lab 6** — Memoria persistente y RAG: backends FAISS / OpenSearch / Mem0, integración con **Amazon Bedrock Knowledge Bases**, políticas de retención, búsqueda web. Buen on-ramp antes de armar la KB gestionada.

**Mini-entregable:** agente que responde citando una base de conocimiento propia (RAG), con evaluación básica de precisión.

---

## Fase 5 — De demo a producción con AgentCore (2-3 semanas)

**Objetivo:** operar agentes con escala, seguridad y observabilidad. Terreno natural para perfil cloud.

**Servicios de Bedrock AgentCore (modulares):**

| Servicio | Para qué |
|---|---|
| **Runtime** | Hosting serverless con microVM aislada por sesión |
| **Memory** | Memoria corto y largo plazo gestionada |
| **Gateway** | Convierte APIs/Lambda en herramientas MCP |
| **Identity** | Identidad verificable por agente + Cognito/Entra/Okta |
| **Observability** | Tracing y monitoreo vía CloudWatch + OpenTelemetry |
| **Built-in tools** | Code Interpreter y Browser en sandbox |

Es agnóstico de modelo (Claude, Nova, Llama, incluso modelos fuera de Bedrock).

**Lectura:**
- ¿Qué es Bedrock AgentCore? (dev guide): https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- AgentCore en la guía prescriptiva de AWS: https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-frameworks/amazon-bedrock-agent-core.html

**Cursos Skill Builder:**
- *Building Generative AI Applications Using Amazon Bedrock* (RAG, agents, guardrails): https://skillbuilder.aws/learn/TM4ZAXTGEZ/building-generative-ai-applications-using-amazon-bedrock/WM6Z6ZHU7K
- **Agentic AI Technical Learning Plan** (+15 h: AgentCore, multi-agente, MCP, A2A, gobernanza) — *el plato fuerte*, otorga badge: https://aws.amazon.com/blogs/apn/introducing-the-agentic-ai-technical-learning-plan-in-aws-skill-builder
- *Building Production-Ready AI Agents with Amazon Bedrock AgentCore* (hands-on): https://skillbuilder.aws/learn/4G7V8NQB5B/building-productionready-ai-agents-with-amazon-bedrock-agentcore/7DY16CFWTC

**Práctica (repo AWS Samples):**
- **Course 4** — Production Deployment con AgentCore: dev vs producción, componentes de AgentCore, y despliegue real de un *calculator agent* en **AgentCore Runtime** con session management e invocación en producción. *(Requiere AWS + Claude 3.5 Haiku en Bedrock + política `BedrockAgentCoreFullAccess`. El deployment es manual — la automatización IaC/CI-CD queda como pendiente.)*

**Mini-entregable:** agente desplegado en AgentCore Runtime con Identity + Observability activos.

---

## Fase 6 — Confiabilidad, evals y seguridad (2 semanas)

**Objetivo:** lo que separa un demo de una solución empresarial vendible.

**Confiabilidad (reliability engineering):**
- Reintentos, manejo de errores, timeouts
- **Structured outputs con Pydantic** + validación
- Idempotencia y fallbacks entre modelos

**Human-in-the-loop (HITL) — imprescindible para enterprise:**
- Puntos de control y **flujos de aprobación** antes de que el agente ejecute acciones con impacto (modificar datos, operaciones sensibles)
- Niveles de autonomía: sugerir vs actuar-con-confirmación vs actuar-y-notificar
- Ningún cliente enterprise deja un agente actuar sin control; diseñar esto desde el inicio, no como parche
- Patrones: colas de aprobación, *dry-run*, límites de acción por rol, checkpoints en workflows largos

**Evals:**
- Cómo medir si el agente funciona: datasets de prueba, LLM-as-judge, métricas
- Sin evals, volás a ciegas

**Observabilidad:**
- Tracing con **AgentCore Observability** (CloudWatch + OpenTelemetry)
- Alternativa/complemento open-source: **Langfuse** — https://langfuse.com/docs

**Seguridad y gobernanza:**
- Least-privilege en tools (IAM + AgentCore Identity)
- Prompt injection, kill switches, audit logging
- Conexión con el **EU AI Act** (transparencia y alto riesgo vigentes desde ago-2026)

**Práctica (repo AWS Samples):**
- **Course 1, Lab 6** — Observabilidad y evals: tracing con **LangFuse** + evaluación con **RAGAS** sobre un agente de recomendación. Cubre la parte de evals/observabilidad; HITL, guardrails, prompt injection y gobernanza no están en el repo y quedan de tu lado.

**Mini-entregable:** suite de evals + guardrails + dashboard de tracing sobre tu agente.

---

## Fase 7 — Nivel empresarial y monetización (2-3 semanas)

**Objetivo:** diseñar soluciones vendibles y llevarlas al mercado. Esta fase es lo que convierte "saber construir agentes" en "construir un negocio de agentes".

**Arquitectura empresarial:**
- **AWS Well-Architected — Generative AI Lens**: seguridad, confiabilidad, performance, costos, sostenibilidad para cargas GenAI: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html
- Multi-tenancy, aislamiento de tenants, data partitioning (fundamentos SaaS)

**Economía unitaria / FinOps de LLMs — clave para monetizar:**
- El tema de fondo del negocio: **costo por request vs precio que cobrás**. Un agente puede funcionar y aun así perder plata en cada llamada.
- Medir tokens (input + output) por interacción y traducirlo a $ por transacción/tenant
- Palancas de reducción: elegir el modelo más barato que resuelva la tarea, **prompt caching**, limitar context/tools, cachear resultados de RAG, truncar historial
- Definir el margen: `precio − (costo de tokens + infra + soporte)`; esto determina si el modelo de pricing es viable
- Monitorear costo por tenant en producción (tie-in con Observability de Fase 6)

**Monetización:**
- **Listar agentes de IA como SaaS en AWS Marketplace** (guía oficial, product type específico para agentes): https://docs.aws.amazon.com/marketplace/latest/userguide/listing-saas-ai-agents.html
- Vender en AWS Marketplace (ISVs, modelos de pricing: pay-as-you-go, contratos, private offers): https://aws.amazon.com/partners/marketplace
- **AI for Software Companies** (cómo los ISV monetizan IA: pricing outcome/usage-based, co-sell): https://aws.amazon.com/isv/ai-for-software-companies
- **AWS ISV Accelerate** (co-sell con AWS, cierra deals ~50% más rápido): https://aws.amazon.com/partners/programs/isv-accelerate

**Modelos de pricing a considerar:** por uso (usage-based), por resultado (outcome-based), suscripción, contratos anuales. AWS Marketplace maneja metering, billing y cobros por vos.

**Mini-entregable:** un one-pager de tu solución con arquitectura Well-Architected + modelo de pricing + plan de listado en Marketplace.

---

## Proyecto integrador (capstone)

Un **agente de soporte/documentación empresarial** que:
1. Consulte una base de conocimiento propia (RAG con Knowledge Bases)
2. Ejecute acciones vía una API interna expuesta como **servidor MCP**
3. Se despliegue en **AgentCore Runtime** con Identity y Observability
4. Tenga evals + guardrails + tracing
5. Esté diseñado como **SaaS multi-tenant listable en Marketplace**

Toca todo el stack, es realista para trabajo de documentación/cloud, y es la base de un producto monetizable.

---

## Pendientes futuros (no olvidar)

Temas fuera del alcance de esta versión, a incorporar más adelante:

- **IaC + CI/CD (automatización de despliegue)** — para nivel empresarial real: infraestructura como código (**AWS CDK / CloudFormation / Terraform**) y pipelines de despliegue automatizado del agente. Hoy el plan llega a desplegar en AgentCore de forma manual. Terreno cómodo para perfil cloud; es "hacerlo explícito" más que un tema difícil. Encaja como extensión de la Fase 5.

**Opcionales (según el producto):** patrones multi-agente / A2A en profundidad · fine-tuning / customización de modelos (RAG suele alcanzar) · UX/frontend del agente.

---

## Cómo mantenerse actualizado (el campo cambia cada mes)

- **AWS Machine Learning Blog** — https://aws.amazon.com/blogs/machine-learning/
- **Anthropic Engineering** — https://www.anthropic.com/engineering
- **MCP Blog** — https://blog.modelcontextprotocol.io
- Agregadores/benchmarks: llm-stats.com, aireleasetracker.com (útiles, pero verificá cifras contra la fuente primaria)

---

## Ruta rápida (orden recomendado de cursos Skill Builder)

1. *Foundations of Prompt Engineering* → Fase 0
2. *Getting Started with Amazon Bedrock* → Fase 1
3. *Building Generative AI Applications Using Amazon Bedrock* → Fases 4-5
4. Lab *Build and Evaluate RAG Applications* → Fase 4
5. **Agentic AI Technical Learning Plan** (el plato fuerte) → Fases 2-6
6. *Building Production-Ready AI Agents with AgentCore* → Fase 5

> **Nota:** algunos links de `skillbuilder.aws` piden login (con la cuenta de empresa abren directo). Los catálogos rotan; si algún curso da 404, buscá el título exacto en el buscador de skillbuilder.aws.

**Práctica hands-on en paralelo (repo AWS Samples):** después del loop a mano (Fase 1), Course 1 → Course 2 (Fases 2-4) → Course 3 (multi-agente, opcional) → Course 4 (Fase 5, AgentCore). Ver la sección *Curso práctico de referencia* arriba.

---

## Cronograma (14 h/semana → ~14 semanas / ~3,5 meses)

Marcá cada semana al completarla. 🎯 = hitos grandes.

| Semana | Foco | Hito al cerrar |
|---|---|---|
| [ ] 1 | Prerrequisitos + Fase 0 (prompts, lecturas) | Prompts y tool specs bien escritas |
| [ ] 2 | Fase 0 fin + Fase 1 (loop a mano) | 🎯 Loop agéntico funcionando con Converse API |
| [ ] 3 | Fase 2 — Strands (parte 1) · repo: Course 1 | — |
| [ ] 4 | Fase 2 fin + Fase 3 — MCP (inicio) · repo: Course 2 Labs 1-5 | Agente Strands con tools + memoria |
| [ ] 5 | Fase 3 — MCP (fin) · repo: Course 1 Lab 4 + Course 2 Lab 4 | 🎯 Servidor MCP propio en Python |
| [ ] 6 | Fase 4 — RAG / Knowledge Bases (parte 1) · repo: Course 2 Lab 6 | — |
| [ ] 7 | Fase 4 fin + Fase 5 — AgentCore (inicio) | Agente con RAG citando base propia |
| [ ] 8 | Fase 5 — AgentCore + Agentic Learning Plan · repo: Course 4 | — |
| [ ] 9 | Fase 5 (fin) | 🎯🚀 **Primer agente desplegado en AgentCore** |
| [ ] 10 | Fase 6 — evals, HITL, seguridad (parte 1) | — |
| [ ] 11 | Fase 6 fin + Fase 7 (inicio) | Evals + guardrails + tracing + HITL |
| [ ] 12 | Fase 7 — empresarial + FinOps + monetización | Diseño Well-Architected + plan de pricing |
| [ ] 13 | Capstone (parte 1) | — |
| [ ] 14 | Capstone (fin) | 🎯🚀 **Solución SaaS multi-tenant completa** |

> Ritmo: ~14 h/semana. Si una semana rendís menos, se corre todo hacia adelante sin drama. Las fases de construcción (2, 4, 5, capstone) piden bloques largos (2-3 h), no sesiones de 30 min.

---

## Checklist de progreso

- [ ] Prerrequisitos — Python async, IAM, Pydantic OK
- [ ] Fase 0 — *Building effective agents* leído + prompts/tool specs bien escritas
- [ ] Fase 1 — Loop agéntico a mano con Converse API
- [ ] Fase 2 — Agente en Strands con tools, memoria y structured outputs
- [ ] Fase 3 — Servidor MCP propio en Python
- [ ] Fase 4 — Agente con RAG sobre Knowledge Bases
- [ ] Fase 5 — Agente desplegado en AgentCore con Identity + Observability
- [ ] Fase 6 — Evals + guardrails + tracing + human-in-the-loop
- [ ] Fase 7 — Diseño Well-Architected + economía unitaria (FinOps) + plan de monetización en Marketplace
- [ ] Capstone — Solución SaaS multi-tenant completa
