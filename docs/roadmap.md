# OSMF Roadmap

This roadmap is a working proposal. Phases may overlap or change as the specification develops.

Phases reflect specification priority, not data flow. The data flow is Geometry Model → Structural Model → Analysis Model, but the Structural Model is specified first because most agreed design work concerns it.

## Phase 1 — Project foundations

- Establish vision, principles and architecture
- Define decision-making and specification conventions (decision log: `specification/decisions.md`)
- Establish project versioning approach (SemVer)
- Define terminology

## Phase 2 — Core specification

- Project metadata
- Stable identifiers and object numbering
- Units (fixed SI)
- Coordinate systems
- Versioning
- Package structure (`.smf`), manifest and JSON serialization
- JSON Schema conventions
- Extension mechanism
- Relationship framework
- External dataset references
- Validation principles

## Phase 3 — Structural Model

- Nodes and Simple Supports
- Beams (straight and arc), Beam Properties, local axes, offsets, releases and taper
- Shells and Solids
- Springs, Ground Springs, Spring Properties and Spring Curves
- Constraints
- Materials and Sections
- Mass
- Groups and Lists
- Loadcases and Loads
- Free Loads and Grid Planes
- Structural States
- Mesh instructions and geometry references

## Phase 4 — Analysis Model and Calculations

- Generated mesh: nodes, elements, connectivity and element types
- Mapping of supports, constraints, loads and mass onto the mesh
- Free Load projection and distribution rules
- Generation settings and provenance
- Analysis Tasks, Analysis Types and settings
- Cases, Combinations and Envelopes
- Design Check hook
- Model validation

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
- Formal specification of the agreed `.smr` HDF5 layout
- Case sub-indexing, steps, increments and result locations
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

## Later versions

- Dynamic analysis types (time history, response spectrum, harmonic)
- Creep and shrinkage
- Full Design Check model
- Free Point Load moments
- Moving Nodes as a Structural State delta

## Development principle

The specification should be developed and reviewed before major implementation work. Reference tools should help validate and clarify the specification, not dictate the format prematurely.
