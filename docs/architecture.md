# OSMF Architecture

## 1. Overview

OSMF is organized into independent domains connected through shared project metadata, identifiers, relationships and external dataset references.

Domains are optional. Multiple instances of the same model type may exist in one project. No mandatory workflow or dependency is imposed.

## 2. Project Core

The Project Core provides shared foundations, including:

- Project identity and metadata
- Format version
- Stable identifiers
- Units and unit conventions
- Coordinate reference systems
- Georeferencing
- External dataset descriptors
- Shared metadata and provenance

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

Potential entities include:

- Beams, columns, trusses and cables
- Plates, shells and membranes
- Solid structural entities
- Springs, supports and rigid links
- Connections and assemblies
- Materials and sections
- Thickness, stiffness and mass properties
- Eccentricities, offsets and releases
- Local coordinate systems

The Structural Model may reference geometry, but geometry is not mandatory.

Mesh generation intent and settings belong to the Structural Model. Settings may include global defaults, entity-specific overrides, local refinements, target regions, mesh types, element order, size controls and mesher extensions.

Mesh setting inheritance and conflict resolution require a formal definition.

## 5. Analysis Model

Describes a solver-oriented computational model.

Potential entities include:

- Nodes
- Elements and connectivity
- Element types
- Materials and sections
- Supports and constraints
- Loads and load cases
- Load combinations
- Analysis steps and settings
- Mesh and groups

An Analysis Model may be independently created, imported or generated from another model. It does not require a Geometry Model or Structural Model.

Meshes may be manually created, imported or generated. Mesh generation provenance may be recorded.

## 6. Results

Computational results are represented by metadata and references in OSMF. Actual result values are stored externally.

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

Result metadata may identify the analysis run, solver and version, Analysis Model, load cases, combinations, steps, increments, units, available result types, provenance and external datasets.

A dedicated result-file extension with HDF5 storage is under consideration.

## 7. Inspection

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

## 8. Reality Capture

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

## 9. Measurements and Monitoring

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

## 10. Libraries

Provides reusable definitions, including:

- Materials
- Sections and profiles
- Connections
- Springs and bearings
- Loads
- Sensors
- Inspection classifications
- Vehicle classes
- Standards and reference data

Library definitions may be embedded or referenced externally. Versioning is important for traceability.

## 11. Relationships

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

## 12. Global-local models and submodels

A local or detailed model is represented as an independent Analysis Model linked to another Analysis Model.

The models need not share identical nodes or meshes.

A relationship may define an interface using nodes, elements, surfaces, geometric boundaries, named regions or user-defined mappings.

Initial scope may support one-way displacement transfer from a global model to a local model, including beam-to-solid kinematic mapping. Future extensions may include load transfer, iterative coupling and two-way coupling.

## 13. External datasets

Large or specialized datasets are referenced rather than embedded.

Potential external data include:

- Results
- Measurements and time series
- Point clouds and scans
- Photographs and videos
- Large meshes
- Raster and imaging datasets

External dataset metadata may include identifier, locator, type, format, size, checksum, version, timestamps, source, description, units and coordinate system.

Relative paths should be supported for portable project arrangements. Other locators may include absolute paths, URIs, network storage and cloud or repository locations.

Missing external datasets should be detectable and reported without making the remaining project metadata inaccessible.

## 14. Extensions

Extensions add specialized capabilities without redefining the core.

Potential namespaces include bridge, building, tunnel, Eurocode, prestressing, fatigue, construction stages, reinforcement, monitoring, WIM, solver-specific and mesher-specific information.

Extensions should be namespaced, documented and preservable by applications that do not support them.

## 15. Package and file structure

A ZIP-based OSMF package is a candidate packaging approach.

The package may contain a manifest, model definitions, relationships, domain metadata, libraries and extensions.

Large external datasets remain outside the package.

The exact package layout, serialization, schema and file extensions remain open design topics.
