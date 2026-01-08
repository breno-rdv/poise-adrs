# poise-adrs

Architectural Decision Records for Poise platform

## Overview

This repository contains Architecture Decision Records (ADRs) for the Poise platform. ADRs are documents that capture important architectural decisions made along with their context and consequences.

## Structure

```
docs/
  adr/
    adr-template.md           # Template for creating new ADRs
    0001-[title].md           # Individual ADR files
    0002-[title].md
    ...
```

## Creating a New ADR

1. Copy the template file: `docs/adr/adr-template.md`
2. Rename it with the next sequential number and a descriptive title:
   - Format: `[number]-[short-kebab-case-title].md`
   - Example: `0002-use-postgresql-database.md`
3. Fill in each section of the template:
   - **Title**: Short, descriptive title
   - **Date**: Date when the decision was made (YYYY-MM-DD)
   - **Status**: Proposed, Accepted, Deprecated, or Superseded
   - **Context**: What is the issue being addressed?
   - **Decision**: What decision was made?
   - **Consequences**: What are the positive, negative, and neutral outcomes?
   - **Alternatives Considered**: What other options were evaluated?
   - **References**: Links to relevant resources
4. Commit and push your ADR to the repository

## ADR Lifecycle

- **Proposed**: The ADR is under discussion
- **Accepted**: The decision has been approved and should be implemented
- **Deprecated**: The decision is no longer relevant but kept for historical context
- **Superseded**: The decision has been replaced by a newer ADR (reference the new ADR)

## Examples

See [0001-use-markdown-for-adrs.md](docs/adr/0001-use-markdown-for-adrs.md) for an example of a completed ADR.

## Why ADRs?

Architecture Decision Records help teams:
- Document the reasoning behind important decisions
- Understand the context that led to current architecture
- Avoid revisiting settled decisions
- Onboard new team members more effectively
- Track the evolution of the system over time

## Resources

- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) by Michael Nygard
- [ADR GitHub Organization](https://adr.github.io/)
