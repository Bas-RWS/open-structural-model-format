# OSMF Roadmap

This roadmap is a working proposal. Phases may overlap or change as the specification develops.

## Phase 1 — Project foundations

- Establish vision, principles and architecture
- Define decision-making and specification conventions
- Establish project versioning approach
- Define terminology

## Phase 2 — Core specification

- Project metadata
- Stable identifiers
- Units
- Coordinate systems
- Versioning
- Relationship framework
- External dataset references
- Validation principles

## Phase 3 — Analysis Model

- Nodes and elements
- Connectivity and element types
- Materials and sections
- Supports, constraints and DOFs
- Loads, cases and combinations
- Analysis steps and settings
- Mesh representation
- Model validation

## Phase 4 — Structural Model

- Structural entities and assemblies
- Materials, sections and properties
- Geometry references
- Local axes, releases, offsets and connections
- Mesh settings and refinement intent
- Mesh generation provenance

## Phase 5 — Geometry Model

- Geometric primitives
- Curves, surfaces and solids
- Parametric operations
- Transformations
- Coordinate systems
- Visualization representations

## Phase 6 — Results

- Result metadata and indexing
- External result datasets
- HDF5-based result layout investigation
- Dedicated result file extension investigation
- Load cases, steps, increments and result locations
- Selective result access
- Integrity and provenance

## Phase 7 — Inspection and Reality Capture

- Inspection records and observations
- Condition classifications and numeric properties
- Attachments and media references
- Point cloud and capture metadata
- Registration and comparison relationships

## Phase 8 — Measurements and Monitoring

- Sensor definitions
- Measurement points
- Measurement definitions
- Campaigns
- Time series and event datasets
- Strain, vibration and WIM use cases

## Phase 9 — Advanced relationships and submodels

- Global-local model relationships
- Interface definitions
- Boundary transfer
- Mapping methods
- Beam-to-solid transfer
- Validation of transfer completeness
- Future coupling approaches

## Phase 10 — Reference tooling and ecosystem

- Python reader and writer
- Validator
- Example projects
- Conversion utilities
- Documentation and implementation guidance
- Interoperability testing

## Development principle

The specification should be developed and reviewed before major implementation work. Reference tools should help validate and clarify the specification, not dictate the format prematurely.
