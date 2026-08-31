# CLAUDE.md

Instrucciones de proyecto para Claude Code. Estas indicaciones tienen prioridad sobre el comportamiento por defecto.

## Sobre este proyecto

Proyecto de **aprendizaje y construcción de agentes de IA** con el objetivo de llegar a **soluciones a nivel empresarial y monetizables** en AWS.

- Plan de estudio completo: [`plan-estudio-agentes-ia.md`](./plan-estudio-agentes-ia.md) — es la fuente de verdad de la ruta, recursos y cronograma. Consultalo antes de proponer temas o pasos.
- Estructura del plan: 8 fases (Prerrequisitos → 0 a 7) + capstone, con mini-entregable por fase.
- Ritmo: **~14 h/semana → ~14 semanas (~3,5 meses)**. Hitos: primer agente desplegado en AgentCore ≈ semana 9; solución SaaS completa ≈ semana 14. Ver el cronograma en el plan.
- Estado actual: fase inicial (aún sin código base). El repo va a crecer siguiendo las fases del plan.
- Objetivo final: solución de agentes **empresarial, multi-tenant y monetizable** (listable en AWS Marketplace).

## Stack técnico (decidido)

- **Lenguaje:** Python (con `async/await`)
- **Modelos / inferencia:** Amazon Bedrock (Converse API vía Boto3)
- **Framework de agentes:** Strands Agents (SDK open-source de AWS)
- **Integración de herramientas:** MCP (Model Context Protocol)
- **Conocimiento/RAG:** Amazon Bedrock Knowledge Bases
- **Producción:** Amazon Bedrock AgentCore (Runtime, Memory, Gateway, Identity, Observability)
- **Validación/datos:** Pydantic
- **Observabilidad:** AgentCore Observability (CloudWatch + OpenTelemetry); Langfuse como complemento

> Al recomendar librerías o servicios, mantenerse dentro de este stack salvo que haya una razón fuerte para desviarse (explicarla).

## Cómo debe asistir Claude

1. **Enseñar, no solo resolver.** El objetivo es que Guido *aprenda a construir agentes*. Explicá el porqué de las decisiones; no entregues solo código sin contexto.
2. **Teoría antes que práctica**, pero con foco en entregables. Cada tema debería terminar en algo ejecutable (ver mini-entregables del plan).
3. **Empezar simple.** Preferir el loop agéntico explícito antes que abstracciones de framework. Un framework es azúcar sobre el loop.
4. **Distinguir workflow vs agente autónomo.** Por defecto proponer workflows (pasos definidos); usar agentes autónomos solo cuando el caso lo justifique.
5. **Costos y economía unitaria.** Iterar con modelos chicos y baratos (Claude Haiku, Amazon Nova Lite); subir a modelos frontera solo cuando se mida que hace falta. Como el objetivo es monetizar, razonar en **costo por request vs precio** (tokens, prompt caching, margen) y señalar riesgos de gasto material.
6. **Seguridad y control humano desde el inicio.** Least-privilege en tools e IAM, cuidado con prompt injection, y considerar el EU AI Act cuando aplique. Para acciones con impacto (modificar datos, operaciones sensibles), diseñar **human-in-the-loop** (aprobación/confirmación), no dejar al agente actuar sin control.

## Verificación de información

- Para docs de librerías/servicios (AWS, Strands, MCP, Anthropic), **consultar la documentación actual** (context7 / AWS docs / búsqueda) en vez de responder de memoria — el ecosistema cambia rápido.
- Al citar precios, benchmarks o features de modelos, verificar contra la fuente primaria del proveedor, no solo agregadores.

## Convenciones (a medida que aparezca código)

- Python: type hints + Pydantic para I/O de tools y structured outputs.
- Tools/MCP: descripciones claras y explícitas (impactan directo en el comportamiento del agente).
- Nunca hardcodear credenciales; usar perfiles/roles de AWS y variables de entorno.
- Tests y evals junto al código que validan.

## Idioma

Responder en **español** salvo que Guido escriba en inglés.

## Memoria (Engram)

Este proyecto usa el protocolo Engram del CLAUDE.md global. Guardar proactivamente decisiones de arquitectura, patrones y descubrimientos con `mem_save`.
