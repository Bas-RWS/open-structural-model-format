# OSMF Architecture

## 1. Overview

OSMF is organized into independent domains connected through shared project metadata, identifiers, relationships and external dataset references.

Domains are optional. Multiple instances of the same model type may exist in one project. No mandatory workflow or dependency is imposed.

The standard path is Geometry Model → Structural Model → Analysis Model. The Structural Model holds mesh instructions; the Analysis Model holds the generated mesh. The project-level Calculations domain holds Analysis Tasks, Cases, Combinations, Envelopes and Design Checks, which use Analysis Models and Structural States. An Analysis Model may also be imported or created without a Structural Model.

Agreed design decisions are indexed in [`specification/decisions.md`](../specification/decisions.md).

## 2. Project Core

The Project Core provides shared foundations, including:

- Project identity and metadata
- Format version
- Stable identifiers
- Units and unit conventions
- Project gravity
- Coordinate reference systems
- Georeferencing
- External dataset descriptors
- Shared metadata and provenance

All values are stored in fixed SI units per quantity. An optional preferred-units setting may record display preferences; it never changes stored values.

## 3. Geometry Model

Describes geometry, which may be physical, conceptual, design-based or parametric.

Potential entities include:

- Points, lines, curves and arcs
- Polylines
- Surfaces
- Solids
- Profiles and cross-sections
- Transformations
- Extrusions, sweeps, lofts and revolves
- Boolean operations
- Visualization representations

Geometry may exist without a Structural Model or Analysis Model.

Visualization tessellation should be distinguishable from original geometry.

## 4. Structural Model

Describes the structural idealization and engineering interpretation of a structure.

Entities include:

- Nodes, with optional Simple Support
- Beams (straight or circular arc; a Beam without Beam Property has no stiffness and serves as a mesh, load and coupling line)
- Shells and Solids
- Springs and Ground Springs
- Constraints (Point-to-Point, Line-to-Line, Line-to-Point)
- Materials and Sections
- Beam Properties, Spring Properties, Ground Spring Properties and Spring Curves
- Offsets and releases
- Mass
- Groups and Lists
- Loadcases and Loads, including Free Loads and Grid Planes
- Structural States
- Mesh instructions

Every element has an `inactive` flag; inactive elements are kept in the Structural Model but excluded from the Analysis Model.

The Structural Model may reference geometry, but geometry is not mandatory.

Mesh generation intent and settings belong to the Structural Model. Settings may include global defaults, entity-specific overrides, local refinements, target regions, mesh types, element order, size controls, arc segmentation and mesher extensions. The Structural Model contains no solver-generated discretization.

Mesh setting inheritance and conflict resolution require a formal definition.

Structural States represent cumulative changes to the base Structural Model (added or removed objects, replaced Properties, Materials or Sections, changed supports). States form a tree through an optional parent reference.

## 5. Analysis Model

Describes the discretized computational representation of a structure.

Entities include:

- Nodes
- Elements and connectivity
- Element types
- References to Materials, Sections and Properties
- Supports and constraints as applied to the mesh
- Loads as mapped onto the mesh, including projected Free Loads
- Generated mass
- Generation settings and provenance

An Analysis Model is normally generated from a Structural Model. It may also be independently created or imported; it does not require a Geometry Model or Structural Model.

Meshes may be manually created, imported or generated. Mesh generation provenance, including the Free Load distribution algorithm used, is recorded.

Loadcases, Combinations and analysis settings are not part of the Analysis Model: Loadcases belong to the Structural Model and Combinations to Calculations.

## 6. Calculations

The project-level Calculations domain holds:

- Analysis Tasks, each referencing exactly one Analysis Model, one Structural State, one Analysis Type and one settings set
- Cases, which store the calculation snapshot, provenance and result references
- Combinations, which reference Cases
- Envelopes, which find extrema across compatible Cases
- Design Checks, which reference Cases, Combinations or Envelopes and a namespaced code extension

Base Analysis Types are Linear static, Nonlinear static, Modal, Linear buckling and Staged. Further types may be added in later versions. Solver-specific settings belong in extensions.

Deleting a Task does not delete its Cases. Changed inputs mark results stale.

## 7. Results

Computational results are represented by metadata and references in OSMF. Actual result values are always stored externally.

Potential result types include:

- Displacements and rotations
- Reactions
- Element forces
- Stresses and strains
- Modal properties and mode shapes
- Buckling results
- Nonlinear increments
- Time histories
- Energy and convergence data

Result metadata identifies the Case (with sub-index), quantity and components, units, coordinate system, location type, references to Analysis Model entities by UUID, the external dataset and its checksum, and provenance. Unaveraged element-node values are stored; averaging is postprocessing.

Result values are stored in HDF5 files with the extension `.smr`. The detailed layout remains open.

## 8. Inspection

Describes condition assessments and inspection observations.

Potential information includes:

- Inspection records, date and method
- Inspector and organization
- Scope and environmental conditions
- Observations and condition classifications
- Severity or rating
- Crack dimensions, corrosion, spalling, deformation and other measured properties
- Locations and references to model entities
- Photographs, videos, sketches and reports
- Links to previous observations

Inspections are historical records. New inspections should not overwrite earlier observations.

## 9. Reality Capture

Describes reality-capture datasets and their metadata.

Potential sources include:

- Laser scanning and LiDAR
- Point clouds
- Photogrammetry
- Drone imagery
- Depth maps
- 3D reconstructions
- Raster imagery

Metadata may include acquisition information, device, source format, coordinate system, accuracy, coverage, registration status and external file references.

Large capture datasets are stored externally.

## 10. Measurements and Monitoring

Describes quantitative measurements and structural health monitoring.

Potential information includes:

- Sensors and their specifications
- Measurement points
- Measurement definitions
- Measurement campaigns
- Time series and event data
- Acquisition and sampling metadata
- Calibration and sensor history

Examples include strain, vibration, displacement, rotation, temperature, wind, moisture, pressure and Weight-In-Motion (WIM).

Measurement points should be independent of sensors so sensor replacement does not invalidate the point's identity.

Large measurement datasets are stored externally.

## 11. Libraries

Libraries are sources of reusable definitions, including:

- Materials
- Sections and profiles
- Connections
- Springs and bearings
- Loads
- Sensors
- Inspection classifications
- Vehicle classes
- Standards and reference data

Library items are copied into the project on import as standalone objects. Each copy keeps provenance metadata (library, version and item identifier). Library items are never live-linked, so a library update never changes an existing project silently.

## 12. Relationships

Relationships are represented in a separate relationship layer instead of being rigidly embedded in individual domains.

Relationships may connect models, entities, datasets and other objects.

Examples include:

- references
- derived_from
- idealized_from
- discretized_from
- located_on
- measured_on
- observed_on
- represents
- registered_to
- compared_with
- submodel_of
- boundary_transfer
- coupled_with
- supersedes
- corresponds_to

Relationships may include mapping, transformation, provenance and extension information.

## 13. Global-local models and submodels

A local or detailed model is represented as an independent Analysis Model linked to another Analysis Model.

The models need not share identical nodes or meshes.

A relationship may define an interface using nodes, elements, surfaces, geometric boundaries, named regions or user-defined mappings.

Initial scope may support one-way displacement transfer from a global model to a local model, including beam-to-solid kinematic mapping. Future extensions may include load transfer, iterative coupling and two-way coupling.

## 14. External datasets

Large or specialized datasets are referenced rather than embedded.

Potential external data include:

- Results
- Measurements and time series
- Point clouds and scans
- Photographs and videos
- Raster and imaging datasets

Analysis Model meshes are not external datasets; they are always stored inside the package.

External dataset metadata may include identifier, locator, type, format, size, checksum, version, timestamps, source, description, units and coordinate system.

Relative paths should be supported for portable project arrangements. Other locators may include absolute paths, URIs, network storage and cloud or repository locations.

Missing external datasets should be detectable and reported without making the remaining project metadata inaccessible.

## 15. Extensions

Extensions add specialized capabilities without redefining the core.

Potential namespaces include bridge, building, tunnel, Eurocode, prestressing, fatigue, construction stages, reinforcement, monitoring, WIM, solver-specific and mesher-specific information.

Any object may carry an `extensions` block keyed by namespace. Each namespace used is declared in the manifest with a version and a `required` flag. Applications that do not support an extension preserve it unchanged; an unknown required extension causes a warning or refusal.

## 16. Package and file structure

An OSMF project is a ZIP package with the extension `.smf`:

```
project.smf
├─ manifest.json        format version, project UUID, domains, file index and checksums, external references
├─ project.json         Project Core
├─ geometry/            Geometry Model (optional)
├─ structural/          Structural Model
├─ analysis/            Analysis Model(s), including the mesh
├─ calculations/        Tasks, Cases, Combinations, Envelopes, Design Checks
├─ relationships/       relationship layer
└─ extensions/<namespace>/
```

Result files (`.smr`, HDF5) and other large external datasets are stored outside the package and referenced from the manifest.

Files inside the package are JSON (UTF-8), validated by JSON Schema (2020-12) per domain. The format is versioned with SemVer: minor versions are additive only, and readers refuse a higher major version than they support.
