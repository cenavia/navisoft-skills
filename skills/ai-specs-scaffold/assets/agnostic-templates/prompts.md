# Reusable prompt templates

Use these as reference when invoking .commands or when asking the AI to follow a workflow.

## Enrich User Story

Use the command or prompt that invokes `ai-specs/.commands/enrich-us.md` with the ticket id or reference. Ensures the user story has enough detail for autonomous implementation.

## Plan backend ticket

Use the command that invokes `ai-specs/.commands/plan-backend-ticket.md` with the ticket id. Produces a step-by-step plan in `ai-specs/changes/[ticket_id]_backend.md`.

## Plan frontend ticket

Use the command that invokes `ai-specs/.commands/plan-frontend-ticket.md` with the ticket id. Produces a plan in `ai-specs/changes/[ticket_id]_frontend.md`.

## Update documentation

Use `ai-specs/.commands/update-docs.md` or refer to `ai-specs/specs/documentation-standards.mdc` to update docs after code changes.
