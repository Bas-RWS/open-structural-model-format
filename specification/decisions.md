# OSMF Decision Log

This log lists the design decisions for OSMF, classified as described in
`docs/principles.md` §14:

- **Agreed** — explicitly approved. Fixed unless reopened.
- **Proposed** — presented, not yet approved.
- **Open** — requires a decision.

Each entry is one line. Detailed definitions belong in the relevant
specification file; this log is the index, not the specification.

Sources: the OSMF design baseline (transfer bundle, November 2026) and the
repository review of 28 November 2026.

---

## 1. Architecture and domains

| # | Decision | Status |
|---|----------|--------|
| A1 | Standard data path is Geometry Model → Structural Model → Analysis Model. | Agreed |
| A2 | The path is not mandatory: an Analysis Model may be imported or created without a Structural Model (`docs/principles.md` §4 retained). | Agreed |
| A3 | The Structural Model holds mesh instructions (intent and settings); the Analysis Model holds the generated mesh. | Agreed |
| A4 | The Structural Model contains no solver-generated discretization or solver element settings. | Agreed |
| A5 | A project-level Analysis domain holds Analysis Tasks, Cases, Combinations and Envelopes. The Analysis Model is the generated representation referenced by Tasks. | Agreed |
| A6 | Final name of the project-level Analysis domain. | Open |
| A7 | Placement of Structural State (likely alongside the Structural Model). | Open |
| A8 | Roadmap order: Structural Model, then Analysis Model, then Geometry Model. Phases reflect specification priority, not data flow. | Agreed |

## 2. Objects, identity, Groups and Lists

| # | Decision | Status |
|---|----------|--------|
| O1 | Objects have UUID, object number where appropriate, optional name and explicit references. | Agreed |
| O2 | Shared definitions (Materials, Sections, Properties) are separate objects referenced by instances. | Agreed |
| O3 | Copies receive new identities; selected internal relationships reconnect to the copies; unselected Nodes are recreated rather than linked; shared Materials/Sections/Properties are reused. | Agreed |
| O4 | Copying a Group creates new contained objects, reconnects internal relationships and drops external connections; the original is unchanged. | Agreed |
| O5 | Every object belongs to exactly one Group; sequential numbering, optional names; maximum three levels (e.g. G1, G1_1, G1_1_1). Parent Groups may contain objects and subgroups. | Agreed |
| O6 | Lists are manual, non-hierarchical selections of objects, ranges, Groups, Properties or Lists. Membership is copied at insertion, deduplicated, and does not affect Groups or object properties. An object may be in multiple Lists. | Agreed |
| O7 | List reference grammar. | Open |
| O8 | Object-number rules. | Open |

## 3. Nodes and supports

| # | Decision | Status |
|---|----------|--------|
| N1 | Node: UUID, number, optional name, global X/Y/Z, Group, optional Simple Support. No local orientation. | Agreed |
| N2 | Simple Support: six DOFs (X, Y, Z, XX, YY, ZZ), each Free or Fixed, stored on the Node. | Agreed |
| N3 | Complex and angled supports use Ground Springs. Simple Support and Ground Spring must not conflict on the same DOF. | Agreed |
| N4 | Line and area supports are not dataset entities; postprocessing reconstructs them from nodal reactions/constraints. | Agreed |
| N5 | Documentation uses only Simple Support, Ground Spring, Spring and Constraint; "rigid link" is not a separate concept (use a full-DOF Point-to-Point Constraint). | Agreed |

## 4. Beams, Beam Properties and Sections

| # | Decision | Status |
|---|----------|--------|
| B1 | Beam is straight from Node I to Node J; curved geometry uses multiple straight Beams. | Agreed |
| B2 | Beam: UUID, number, optional name, Node I/J, Group, Beam Property reference. | Agreed |
| B3 | Beam Property: element type, Material, Section (or Sections at I and J), rotation about local x, offsets at I/J, releases. | Agreed |
| B4 | Element types: Beam (with shear deformation), Beam (no shear deformation), Bar, Strut (compression-only), Tie (tension-only). | Agreed |
| B5 | Local x from I to J; Global Z is the reference for local y/z, Global Y fallback when near-parallel; deterministic right-handed axes; Section rotation is additional rotation about local x. | Agreed |
| B6 | Reference-switch tolerance is 1e-6 in the projection/parallelism test (numeric, not an angle). | Agreed |
| B7 | Offsets at I/J independent, stored explicitly as (dx, dy, dz) in Beam-local coordinates; Node geometry unchanged; only final offsets stored. | Agreed |
| B8 | Simple releases on Beam at I/J; complex releases may use a separate Release object; no conflicting definitions per DOF/end; no along-length releases. | Agreed |
| B9 | Tapered Beam: Sections at I and J; one Section applies over the full length; identical Sections mean prismatic. | Agreed |
| B10 | Sections are reusable objects: explicit properties, parametric, profile library, or user-defined/composite geometry retaining design-check metadata. | Agreed |
| B11 | Offset sign/reference interpretation. | Open |
| B12 | Taper compatibility and interpolation rules. | Open |
| B13 | Auxiliary Member (geometry-only member) name and definition. | Open |
| B14 | Explicit curved-member concept. | Open |

## 5. Materials and libraries

| # | Decision | Status |
|---|----------|--------|
| M1 | Material sections: General, Properties, Strength, Additional Properties. | Agreed |
| M2 | Linear elastic is default; nonlinear behaviour optional. | Agreed |
| M3 | Density stored; self-weight derived as γ = ρ·g from project gravity; self-weight not stored separately. | Agreed |
| M4 | Damping belongs to Analysis, not Material. Design codes are not part of the base Material. | Agreed |
| M5 | All library items (Materials, Sections, Properties, etc.) are copied into the project on import as standalone objects with provenance metadata (library, version, item ID); never live-linked. | Agreed |

## 6. Mass

| # | Decision | Status |
|---|----------|--------|
| S1 | Translational mass only; no rotational mass or mass moment of inertia. | Agreed |
| S2 | Mass is concentrated at a Node or uniform over a Beam; Beam Mass stores total mass, not mass per length. | Agreed |
| S3 | Mass object: UUID, number, optional name, value, target Node/Beam, Group. | Agreed |
| S4 | Mass may be generated during Analysis Model generation from self-weight or Loadcases with factors; generated mass is not duplicated as explicit mass. | Agreed |

## 7. Springs, Ground Springs and Constraints

| # | Decision | Status |
|---|----------|--------|
| K1 | Ground Spring connects one Node to ground and references a Ground Spring Property; orientation belongs to the instance. | Agreed |
| K2 | Spring connects Nodes I/J with finite stiffness and references a Spring Property; local axes follow the Beam rule; orientation belongs to the instance. | Agreed |
| K3 | (Ground) Spring Properties define six DOFs, each Free/Fixed/Linear/Nonlinear, with stiffness and optional curves; no coupled cross terms. | Agreed |
| K4 | Constraints are kinematic couplings: Point-to-Point, Line-to-Line, Line-to-Point. Source/master order matters. | Agreed |
| K5 | Each Constraint stores its own six DOF switches and Global/Local coordinate system; no Constraint Property object. | Agreed |
| K6 | Nonlinear Spring curve schema. | Open |
| K7 | Constraint orientation, coupling variants and interaction with releases. | Open |

## 8. Loads and Loadcases

| # | Decision | Status |
|---|----------|--------|
| L1 | Loads reference a Loadcase directly; Load Groups are not used. | Agreed |
| L2 | Node Load: FX, FY, FZ, MX, MY, MZ; Global or Local; multiple per Node and Loadcase allowed. | Agreed |
| L3 | Beam loads: Line (start/end from Node I, linear interpolation), Point (six components), Thermal (ΔT, ΔT-y, ΔT-z). | Agreed |
| L4 | Shell loads (exactly three): Face, Edge, Thermal (ΔT, ΔT-x, ΔT-y, ΔT-z). | Agreed |
| L5 | Solid loads (exactly five): Body, Surface, Edge, Thermal, Strain. Buoyancy and Rotary excluded from core. | Agreed |
| L6 | Shell/Solid varying-load interpolation and edge conventions. | Open |

## 9. Free Loads and Grid Plane

| # | Decision | Status |
|---|----------|--------|
| F1 | Free Loads are Global spatial loads projected onto elements during Analysis Model generation; each references a Grid Plane and a Loadcase. | Agreed |
| F2 | Free Point Load carries forces only (FX, FY, FZ); moments deferred to a possible extension. | Agreed |
| F3 | Free Line, Area and Temperature Loads as defined in the baseline. | Agreed |
| F4 | Grid Plane: three-point plane, projection direction and depth, Group selection, independent Beam/Shell/Solid Boolean filters, distribution method (Single, Two-way, Multi-way) and angle in degrees. Not duplicated in Free Loads. | Agreed |
| F5 | Projection/distribution algorithm, load conservation, angle reference and sign, temperature mapping, geometry validation. | Open |

## 10. Analysis, Cases, Combinations and Results

| # | Decision | Status |
|---|----------|--------|
| R1 | Structural State represents cumulative deltas to the base Structural Model. | Agreed |
| R2 | Analysis Task references exactly one Analysis Model, one Structural State, one Analysis Type and one settings set; may run multiple calculations with target Cases. | Agreed |
| R3 | Rerun overwrites its target Case; changed inputs mark results stale; deleting a Task does not delete Cases. | Agreed |
| R4 | Case stores the calculation snapshot and provenance. | Agreed |
| R5 | Design Checks are separate from Analysis Tasks. | Agreed |
| R6 | Combinations reference Analysis Cases, not Loadcases; support factors, addition, alternatives, ranges and nesting. | Agreed |
| R7 | Envelopes are extrema, not sums; strict compatibility; no silent exclusions; governing location/configuration stored. | Agreed |
| R8 | Sub-result indexing (e.g. A5_1 for modes); one-result Cases have no suffix. | Agreed |
| R9 | All computational result values are stored in external result datasets, regardless of size (`docs/principles.md` §12). | Agreed |
| R10 | HDF5 result file and ZIP `.osmf` package. | Proposed |
| R11 | Analysis Type catalogue, result storage contract, Shell/Solid result components, Design Check model. | Open |

## 11. Format-wide

| # | Decision | Status |
|---|----------|--------|
| G1 | Serialization, schema, extension and version policy. | Open |
| G2 | Units conventions. | Open |
| G3 | Structural State delta types, inheritance and validation. | Open |
