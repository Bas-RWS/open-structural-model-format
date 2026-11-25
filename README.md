# Open Structural Model Format (OSMF)

## Purpose

OSMF is an open, vendor-neutral data format for representing, exchanging,
preserving and connecting structural engineering data throughout the lifecycle
of structures and infrastructure.

The format aims to support structural modeling, analysis, inspection,
monitoring, measurement data, reality capture and engineering results.

OSMF is intended to be independent of specific software vendors, solvers,
modeling tools or engineering workflows.

## Vision

Structural engineering data should be accessible, exchangeable, interpretable
and reusable throughout the entire lifecycle of a structure.

OSMF aims to provide a common semantic framework that connects different
types of engineering data while allowing each domain to exist independently.

## Core principles

- Open and vendor-neutral
- Modular and domain-independent
- Semantically explicit
- Extensible
- Traceable
- Suitable for long-term data preservation
- AI-compatible
- No mandatory workflow or modeling sequence
- Large datasets stored externally and referenced
- All computational results stored in external result datasets

## Domains

OSMF may contain or reference the following domains:

- Project Core
- Geometry Model
- Structural Model
- Analysis Model
- Results
- Inspection
- Reality Capture
- Measurements and Monitoring
- Libraries
- Relationships
- Extensions

Domains are independent and optional. A project may contain any combination
of these domains.

## Project status

OSMF is currently in the conceptual design and specification phase.

The architecture, data structures, schemas, file packaging and technical
implementation are under development. Proposals are not considered final
until explicitly agreed upon.

## Repository structure

- `docs/` — vision, principles, architecture and roadmap
- `specification/` — formal format specification
- `schemas/` — machine-readable schemas
- `examples/` — example OSMF projects and datasets
- `tools/` — reference implementations and utilities

## Contributing

Design decisions and specification changes should be discussed, documented
and reviewed before implementation.

The project aims to maintain a clear distinction between agreed decisions,
proposals and open questions.

## License

License not yet determined.
