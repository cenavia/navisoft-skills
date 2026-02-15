# Fragmentos para archivos en ai-specs/specs/

Usar al generar los `.mdc` y demás archivos. Extraer del documento de arquitectura los principios y convenciones; aquí solo se muestran estructuras y frontmatter típicos.

## base-standards.mdc

```yaml
---
description: Development rules and guidelines for this project, applicable to all AI agents.
alwaysApply: true
---

## 1. Core Principles
<!-- Del documento: TDD, tipado, naming, pasos incrementales, etc. -->

## 2. Language Standards
<!-- Del documento: idioma (p. ej. English only) -->

## 3. Specific standards
- [Documentation Standards](./documentation-standards.mdc)
- [Java Standards](./java-standards.mdc)   <!-- Uno por cada tecnología detectada: java, node, angular, react, vue -->
- [Node Standards](./node-standards.mdc)
- [Angular Standards](./angular-standards.mdc)
```

**Importante:** Catalogar por **tecnología**. Generar un enlace por cada tecnología (java, node, angular, react, vue). No usar nombres de proyecto ni "Backend" / "Frontend" genéricos.

## <tecnologia>-standards.mdc (uno por tecnología)

Generar **un** archivo por cada **tecnología** detectada (ej. java-standards.mdc, node-standards.mdc, angular-standards.mdc). Slugs: `java`, `node`, `angular`, `react`, `vue`.

```yaml
---
description: Standards and conventions for <tecnologia> projects (e.g. Java/Spring Boot, Node/Express, Angular).
globs: ["demo-api/**", "other-java-app/**"]   <!-- Todas las carpetas que usen esta tecnología -->
alwaysApply: true
---

# <Tecnologia> Standards
## Overview
## Technology Stack
## Architecture / Structure
## Coding Standards
## API / UI conventions (según stack)
## Testing Standards
## Development Workflow (branching, scripts)
```

Ajustar secciones según la tecnología (DDD para Java, capas para Node, componentes para Angular). Los `globs` deben abarcar **todas** las carpetas que usen esa tecnología.

## documentation-standards.mdc

```yaml
---
description: Standards for technical documentation and AI specs updates.
alwaysApply: true
---

# Documentation and AI specs rules
## General rules
- Language (e.g. English) for all docs and comments.
## Technical documentation
- When to update: data model → data-model.md; API → api-spec.yml; setup/tooling → *-standards.mdc.
- Process: review changes → identify files → update in English → verify → report.
## AI specs
- Process for incorporating user feedback into rules (optional, if in architecture doc).
```

## api-spec.yml (mínimo OpenAPI 3.0)

```yaml
openapi: 3.0.3
info:
  title: <Project Name> API
  version: 1.0.0
  description: API specification for <Project Name>. See ai-specs/specs for development rules.
servers:
  - url: http://localhost:3000
    description: Local
paths: {}
# Añadir paths según documento de arquitectura o primer endpoint (e.g. /health).
```

## data-model.md (esqueleto)

```markdown
# Data Model

This document describes the domain and persistence model for the project.
See ai-specs/specs/backend-standards.mdc for implementation conventions.

## Entities
<!-- List or table of entities and main attributes -->

## Schemas / Database
<!-- Tables, relations, naming if applicable -->

## Conventions
<!-- Naming, IDs, timestamps, etc. from architecture doc -->
```

## development_guide.md

```markdown
# Development Guide

## Prerequisites
## Setup
## Scripts (install, test, lint, run)
## Branching / Workflow
## Running tests
## Documentation updates
```
Contenido según documento de arquitectura y stack detectado.

## prompts.md

```markdown
# Reusable prompt templates

## Enrich User Story
## Plan backend ticket
## Plan frontend ticket
## Update documentation
```
Incluir solo las plantillas que el documento de arquitectura o los .commands referencien.
