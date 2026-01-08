# 0001. Use Markdown for Architecture Decision Records

Date: 2026-01-08

## Status

Accepted

## Context

The Poise platform needs a way to document architecture decisions that is:
- Easy to read and write
- Version-controllable
- Accessible to all team members
- Searchable and maintainable over time

We need a format that allows for rich text formatting, code snippets, and links while remaining simple and portable.

## Decision

We will use Markdown format for all Architecture Decision Records (ADRs) stored in this repository.

Each ADR will:
- Be stored in the `docs/adr/` directory
- Follow a sequential numbering scheme (e.g., 0001, 0002, etc.)
- Use the naming format: `[number]-[short-title].md`
- Follow the structure defined in `adr-template.md`

## Consequences

### Positive

- Markdown is widely supported by development tools and platforms
- Easy to learn and write with minimal syntax
- Version control systems (like Git) handle Markdown files well
- Can be rendered beautifully on GitHub and other platforms
- Supports code snippets, tables, and links
- Plain text format ensures long-term accessibility

### Negative

- Less structured than specialized ADR tools
- No automated workflow for creating new ADRs (manual process)
- Requires discipline to maintain consistent formatting

### Neutral

- Team members need basic Markdown knowledge
- Need to establish a numbering convention for ADRs

## Alternatives Considered

### Use a specialized ADR tool (e.g., adr-tools, log4brains)

These tools provide automation and structure but add dependencies and complexity. For a platform-focused repository, simplicity and accessibility are prioritized over automation.

### Use a wiki or documentation platform

While wikis offer features like search and organization, they separate documentation from the codebase. Keeping ADRs in version control alongside code ensures they evolve together and provides better traceability.

### Use plain text files

Plain text is simple but lacks the formatting capabilities of Markdown, making documents harder to read and less expressive.

## References

- [Documenting Architecture Decisions by Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR GitHub Organization](https://adr.github.io/)
- [Markdown Guide](https://www.markdownguide.org/)
