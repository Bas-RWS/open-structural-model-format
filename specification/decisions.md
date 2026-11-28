# OSMF Decision Log

This log lists the design decisions for OSMF, classified as described in
`docs/principles.md` §14:

- **Agreed** — explicitly approved. Fixed unless reopened.
- **Proposed** — presented, not yet approved.
- **Open** — requires a decision.

Each entry is one line. Detailed definitions belong in the relevant
specification file; this log is the index, not the specification.

Sources: the OSMF design baseline (transfer bundle, November 2026) and the
design review sessions of 28 November 2026. Where a later decision replaced
an earlier one, the entry states what it replaces.

SOFiSTiK and GSA were used as references during the review; they are not
part of the OSMF specification.

---

## 1. Architecture and domains

| # | Decision | Status |
|---|----------|--------|
| A1 | Standard data path is Geometry Model → Structural Model → Analysis Model. | Agreed |
| A2 | The path is not mandatory: an Analysis Model may be imported or created without a Structural Model (`docs/principles.md` §4 retained). | Agreed |
| A3 | The Structural Model holds mesh instructions (intent and settings); the Analysis Model holds the generated mesh. | Agreed |
| A4 | The Structural Model contains no solver-generated discretization or solver element settings. | Agreed |
| A5 | A project-level domain named **Calculations** holds Analysis Tasks, Cases, Combinations, Envelopes and Design Checks. The Analysis Model is the generated representation referenced by Tasks. | Agreed |
| A6 | Roadmap order: Structural Model, then Analysis Model, then Geometry Model. Phases reflect specification priority, not data flow. | Agreed |

## 2. Units

| # | Decision | Status |
|---|----------|--------|
| U1 | All values are stored in fixed SI units per quantity: m, N, kg, s, K (a ΔT is numerically equal in K and °C). | Agreed |
| U2 | Rotations and rotational stiffness in radians. Input angles (Section rotation, Grid Plane distribution angle) in degrees. | Agreed |
| U3 | Display units are an application concern. An optional preferred-units field in the Project Core never changes stored values. | Agreed |

## 3. Objects, identity, numbering, Groups and Lists

| # | Decision | Status |
|---|----------|--------|
| O1 | Objects have a UUID, an object number where appropriate, an optional name and explicit references. The UUID is the identity; references always use the UUID. | Agreed |
| O2 | Object numbers are positive integers with no encoded meaning; gaps are allowed. | Agreed |
| O3 | Beam, Shell, Solid, Spring, Ground Spring and Constraint share **one element number sequence** (e.g. Beam 3 and Shell 3 cannot coexist). Other object types (Node, Material, Section, Properties, Mass, Loads, …) each have their own sequence. | Agreed |
| O4 | Shared definitions (Materials, Sections, Properties) are separate objects referenced by instances. | Agreed |
| O5 | Copies receive new identities and numbers; selected internal relationships reconnect to the copies; unselected Nodes are recreated rather than linked; shared Materials/Sections/Properties are reused. | Agreed |
| O6 | Copying a Group creates new contained objects, reconnects internal relationships and drops external connections; the original is unchanged. | Agreed |
| O7 | Every object belongs to exactly one Group; sequential numbering, optional names; maximum three levels (e.g. G1, G1_1, G1_1_1). Parent Groups may contain objects and subgroups. | Agreed |
| O8 | A List is a set of typed terms, each *include* or *exclude*: Object (UUID, static), Group (dynamic, option to include subgroups), Property (dynamic), List (dynamic, no cycles). Ranges such as `1 to 100` are expanded to static Object terms at input. No text grammar in the core. *Replaces the earlier rule that membership is copied, not dynamic.* | Agreed |
| O9 | Any structural object type may be a List member; mixed-type Lists are allowed. Where a List is used, members of non-applicable types are skipped and reported, never silently. | Agreed |
| O10 | Lists do not affect Group membership or object properties. An object may be in multiple Lists. | Agreed |

## 4. Nodes and supports

| # | Decision | Status |
|---|----------|--------|
| N1 | Node: UUID, number, optional name, global X/Y/Z, Group, optional Simple Support. No local orientation. | Agreed |
| N2 | Simple Support: six DOFs (X, Y, Z, XX, YY, ZZ), each Free or Fixed, stored on the Node. | Agreed |
| N3 | Complex and angled supports use Ground Springs. Simple Support and Ground Spring must not conflict on the same DOF. | Agreed |
| N4 | Line and area supports are not dataset entities; postprocessing reconstructs them from nodal reactions and constraints. | Agreed |
| N5 | Documentation uses only Simple Support, Ground Spring, Spring and Constraint. "Rigid link" is not a separate concept: use a Rigid Point-to-Point Constraint. | Agreed |

## 5. Elements: common

| # | Decision | Status |
|---|----------|--------|
| E1 | Every element has an `inactive` flag (true/false). An inactive element stays in the Structural Model with its properties but is excluded from the Analysis Model; loads or mass on it are skipped and reported. Not used for construction stages (see Structural State). *Replaces the "Auxiliary Member" candidate.* | Agreed |

## 6. Beams, Beam Properties and Sections

| # | Decision | Status |
|---|----------|--------|
| B1 | Beam: UUID, number, optional name, Node I/J, Group, optional Beam Property reference. | Agreed |
| B2 | Beam shape is straight (default) or a circular arc from Node I to Node J through a third point (coordinates, not a Node). Splines are outside the core. *Reopens the earlier "Beam is straight" rule.* | Agreed |
| B3 | Arc segmentation is a Structural Model mesh instruction (number of segments, maximum length or maximum angle); the Analysis Model receives straight segments. | Agreed |
| B4 | A Beam without a Beam Property has no stiffness and no mass. If it lies in or on a Shell or Solid, the mesh follows it; loads on it transfer to the underlying mesh (error if it lies on nothing); it can serve as a Line in Line-to-Line and Line-to-Point Constraints. | Agreed |
| B5 | Beam Property: element type, Material, Section (or Sections at I and J), rotation about local x, offsets at I/J, releases. | Agreed |
| B6 | Element types: Beam (with shear deformation), Beam (no shear deformation), Bar, Strut (compression-only), Tie (tension-only). | Agreed |
| B7 | Local x from I to J (along the tangent for arcs). Global Z is the reference for local y/z, Global Y fallback when near-parallel; deterministic right-handed axes; Section rotation is additional rotation about local x. For arcs, the same rule is applied at each point. | Agreed |
| B8 | Reference-switch tolerance is 1e-6 in the projection/parallelism test (numeric, not an angle). | Agreed |
| B9 | Positions along a Beam (loads, taper) are measured from Node I, along the arc length for arcs. | Agreed |
| B10 | Offsets are (dy, dz) only, independently at I and J; no axial offsets. The offset is a vector from the Node to the actual member end (end = node + offset), in the local axes of the node line I→J. Node geometry is unchanged; section alignments are converted to explicit offsets. *Replaces the earlier (dx, dy, dz).* | Agreed |
| B11 | Simple releases on the Beam at I/J; complex releases may use a separate Release object; no conflicting definitions per DOF/end; no along-length releases. | Agreed |
| B12 | Tapered Beam: Sections at I and J; one Section applies over the full length; identical Sections mean prismatic. | Agreed |
| B13 | Taper requires the same definition method and shape type at both ends, otherwise the model is invalid (reported). Parametric: dimensions vary linearly, properties are derived from the interpolated geometry. User-defined: same topology, coordinates vary linearly. Explicit properties: values vary linearly. Profile library: not taperable (convert to parametric first). Non-linear haunches: split the member. | Agreed |
| B14 | Sections are reusable objects: explicit properties, parametric, profile library, or user-defined/composite geometry retaining design-check metadata. | Agreed |

## 7. Materials and libraries

| # | Decision | Status |
|---|----------|--------|
| M1 | Material sections: General, Properties, Strength, Additional Properties. | Agreed |
| M2 | Linear elastic is default; nonlinear behaviour optional. | Agreed |
| M3 | Density stored; self-weight derived as γ = ρ·g from project gravity; self-weight not stored separately. | Agreed |
| M4 | Damping belongs to analysis, not Material. Design codes are not part of the base Material. | Agreed |
| M5 | All library items (Materials, Sections, Properties, etc.) are copied into the project on import as standalone objects with provenance metadata (library, version, item ID); never live-linked. | Agreed |

## 8. Mass

| # | Decision | Status |
|---|----------|--------|
| S1 | Translational mass only; no rotational mass or mass moment of inertia. | Agreed |
| S2 | Mass is concentrated at a Node or uniform over a Beam; Beam Mass stores total mass, not mass per length. | Agreed |
| S3 | Mass object: UUID, number, optional name, value, target Node/Beam, Group. | Agreed |
| S4 | Mass may be generated during Analysis Model generation from self-weight or Loadcases with factors; generated mass is not duplicated as explicit mass. | Agreed |

## 9. Springs and Ground Springs

| # | Decision | Status |
|---|----------|--------|
| K1 | Ground Spring connects one Node to ground and references a Ground Spring Property; orientation belongs to the instance. | Agreed |
| K2 | Spring connects Nodes I/J with finite stiffness and references a Spring Property; local axes follow the Beam rule; orientation belongs to the instance. | Agreed |
| K3 | (Ground) Spring Properties define six DOFs; no coupled cross terms. | Agreed |
| K4 | Per-DOF behaviour type: Free; Fixed; Linear (k); Compression-only (k); Tension-only (k); Gap (k, gap + and −); Lockup (k, free travel + and −); Elastic-plastic (k, yield force + and −); Curve. | Agreed |
| K5 | Curve references a reusable **Spring Curve**: piecewise-linear table of (displacement, force) or (rotation, moment), strictly increasing x, passing through (0, 0), covering negative and positive range, with an explicit out-of-range rule (hold or extrapolate). Unloading follows the same curve. | Agreed |
| K6 | Sign convention: Spring deformation = u_J − u_I in local axes (positive = elongation); Ground Spring deformation = node displacement along the local axis. | Agreed |
| K7 | Friction and hysteresis are outside the base format. | Agreed |

## 10. Constraints

| # | Decision | Status |
|---|----------|--------|
| C1 | Constraints are kinematic couplings: Point-to-Point, Line-to-Line, Line-to-Point. Source/master order matters. | Agreed |
| C2 | Each Constraint stores its own six DOF switches and Global/Local coordinate system; no Constraint Property object. | Agreed |
| C3 | Coupling type per Constraint: **Rigid** (with lever arm) or **Equal** (selected DOFs equal, no lever arm). | Agreed |
| C4 | Local axes: Point-to-Point from vector A→B with the Beam rule (coincident nodes: Global only); Line constraints use the axes of the source Line. | Agreed |
| C5 | A Line is a Beam (with or without Beam Property) or a Shell edge (Shell + edge number). | Agreed |
| C6 | Line-to-Line mapping is proportional: a target node at relative position t along the target Line couples to the source point at the same t. Lines of different length are allowed. t = 0 is at the start of each Line. | Agreed |
| C7 | Releases act between element end and Node; Constraints act on Nodes; they do not conflict. Invalid (reported): the same target DOF is slave in two Constraints; a constrained target DOF also has a Simple Support on that DOF. | Agreed |

## 11. Loads and Loadcases

| # | Decision | Status |
|---|----------|--------|
| L1 | Loads reference a Loadcase directly; Load Groups are not used. | Agreed |
| L2 | Node Load: FX, FY, FZ, MX, MY, MZ; Global or Local; multiple per Node and Loadcase allowed. | Agreed |
| L3 | Beam loads: Line (start/end from Node I, linear interpolation), Point (six components), Thermal (ΔT, ΔT-y, ΔT-z). | Agreed |
| L4 | Shell loads (exactly three): Face, Edge, Thermal (ΔT, ΔT-x, ΔT-y, ΔT-z). | Agreed |
| L5 | Solid loads (exactly five): Body, Surface, Edge, Thermal, Strain. Buoyancy and Rotary excluded from core. | Agreed |
| L6 | Shell corner order is counter-clockwise seen from local +z; corner values follow that order. Triangle: linear; quadrilateral: bilinear (isoparametric). | Agreed |
| L7 | Edge k runs from corner k to corner k+1 (last edge back to corner 1); start value and positions are measured from corner k. | Agreed |
| L8 | Global face loads act on the true area (default) or the projected area. | Agreed |
| L9 | Solids: the specification fixes face and edge numbering per shape (tetrahedron, wedge, hexahedron), with the same rules. | Agreed |

## 12. Free Loads and Grid Plane

| # | Decision | Status |
|---|----------|--------|
| F1 | Free Loads are Global spatial loads projected onto elements during Analysis Model generation; each references a Grid Plane and a Loadcase. | Agreed |
| F2 | Free Point Load carries forces only (FX, FY, FZ); moments deferred to a possible extension. | Agreed |
| F3 | Free Line, Area and Temperature Loads as defined in the baseline. | Agreed |
| F4 | Grid Plane: three-point plane, projection direction and depth, Group selection, independent Beam/Shell/Solid Boolean filters, distribution method (Single, Two-way, Multi-way) and angle in degrees. Not duplicated in Free Loads. | Agreed |
| F5 | Grid Plane x-axis = p1→p2; normal = (p2−p1)×(p3−p1). The distribution angle is positive right-handed about the normal and applies only to Single (one-way). | Agreed |
| F6 | Conservation is normative: distributed loads must match the resultant force and moment of the Free Load. The algorithm is not fixed by the specification but is named in the Analysis Model generation settings. | Agreed |
| F7 | On Shells and Solids the load is applied directly to the face hit along the projection direction (no spanning). Spanning applies to Beams only. | Agreed |
| F8 | Load that lands on no eligible element is reported with the lost share, never silently dropped. | Agreed |
| F9 | Free Temperature Load: the linear field is evaluated at each analysis element centroid and applied as uniform ΔT; no gradients derived. | Agreed |
| F10 | Validation: the three Grid Plane points must not be collinear (tolerance 1e-6); polygons are closed, non-self-intersecting and projected onto the plane. | Agreed |

## 13. Structural State

| # | Decision | Status |
|---|----------|--------|
| T1 | Structural State represents cumulative deltas to the base Structural Model and lives in the Structural Model domain. The base model is implicit State 0. | Agreed |
| T2 | A State has an optional single parent, forming a tree (sequential stages and parallel variants). Its content is the ancestors' deltas plus its own; the youngest wins on conflict. | Agreed |
| T3 | Base delta types: Add (object exists from this State), Remove, Replace reference (Property, Material or Section), Change support (Simple Support DOFs). | Agreed |
| T4 | Moving Nodes is not a base delta (possible future extension). Stress history (lock-in, creep between stages) belongs to the analysis type, not the State. | Agreed |
| T5 | Invalid (reported): a delta on an object that does not exist in that State; conflicting deltas within one State. | Agreed |

## 14. Calculations: Tasks, Cases, Combinations

| # | Decision | Status |
|---|----------|--------|
| R1 | Analysis Task references exactly one Analysis Model, one Structural State, one Analysis Type and one settings set; may run multiple calculations with target Cases. | Agreed |
| R2 | Rerun overwrites its target Case; changed inputs (including changed Group or List content) mark results stale; deleting a Task does not delete Cases. | Agreed |
| R3 | Case stores the calculation snapshot and provenance. | Agreed |
| R4 | Combinations reference Analysis Cases, not Loadcases; support factors, addition, alternatives, ranges and nesting. | Agreed |
| R5 | Envelopes are extrema, not sums; strict compatibility; no silent exclusions; governing location and configuration stored. | Agreed |
| R6 | Sub-result indexing (e.g. A5_1 for modes); one-result Cases have no suffix. | Agreed |
| R7 | Base Analysis Types: Linear static; Nonlinear static (geometric y/n, material y/n, load steps, convergence criterion); Modal (number of modes, mass source); Linear buckling (number of modes, reference load); Staged (sequence of Structural States, linear or nonlinear per stage). | Agreed |
| R8 | Dynamics (time history, response spectrum, harmonic) and creep/shrinkage are planned for later versions. Solver-specific settings only in namespaced extensions. | Agreed |
| R9 | Design Checks are separate from Analysis Tasks. A Design Check object in Calculations references Cases, Combinations or Envelopes, target objects (via a List) and a namespaced code extension (e.g. `eurocode`). All code content lives in extensions. Full model in a later version. | Agreed |

## 15. Results

| # | Decision | Status |
|---|----------|--------|
| P1 | All computational result values are stored in external result datasets, regardless of size (`docs/principles.md` §12). | Agreed |
| P2 | Per result dataset OSMF stores: Case (with sub-index), quantity and components (fixed list), units (SI), coordinate system (Global, element local or node local), location type, references to Analysis Model UUIDs (never solver numbers), external file reference, checksum and provenance. | Agreed |
| P3 | Location types: node; element node; element centroid; relative positions along a Beam (0–1); user evaluation point (location ID). | Agreed |
| P4 | Unaveraged element-node values are stored; nodal averaging is postprocessing. | Agreed |
| P5 | Beam: N, Vy, Vz, Mt, My, Mz in local axes at relative positions; forces act on the cut face with outward normal +x and are positive in the positive local directions (N > 0 = tension). Node: ux, uy, uz, rx, ry, rz and reactions Rx, Ry, Rz, Mx, My, Mz, always global; reactions are the sum of Simple Support and Ground Springs at the node. Spring and Ground Spring: forces Fx…Mz and deformations in local axes, using the spring sign convention (K6). Warping bimoment via extension. | Agreed |
| P5a | Beam section results: σx, εx and plastic strain εpl per section point at each position along the Beam. Section point coordinates (y, z) in section axes are stored in the result metadata, so solvers may use their own point or fibre layout. Mandatory for Nonlinear static with material nonlinearity; optional otherwise. | Agreed |
| P6 | Shell: Nx, Ny, Nxy, Mx, My, Mxy, Qx, Qy (local axes); stresses σx, σy, τxy, σ1, σ2, Von Mises at layers through the thickness: at least top (+z), middle and bottom; any number of layers allowed, with layer positions stored in the result metadata. | Agreed |
| P7 | Solid: σx, σy, σz, τxy, τyz, τzx; σ1–σ3; Von Mises; τmax; strains εx, εy, εz, γxy, γyz, γzx. Plastic strain, energy and damage via extension. | Agreed |

## 16. Files, serialization and versioning

| # | Decision | Status |
|---|----------|--------|
| G1 | Project package: ZIP with extension **`.smf`**. Contents: `manifest.json` (format version, project UUID, domains, file index with checksums, external references), `project.json` (Project Core), `geometry/`, `structural/`, `analysis/`, `calculations/`, `relationships/`, `extensions/<namespace>/`. No `libraries/` folder. | Agreed |
| G2 | The Analysis Model mesh is always stored inside the package. | Agreed |
| G3 | Results: HDF5 files with extension **`.smr`**, stored next to the package and referenced by relative path and checksum in the manifest. Layout outline: one group per Case, one dataset per quantity, attributes referencing OSMF UUIDs. | Agreed |
| G4 | `.smr` layout: root attributes (format version, project UUID, file UUID, generator, creation date); one entity list per Analysis Model, stored once per file with UUIDs as 36-character text; `cases/<case-UUID>/` with attributes (label, Task, Analysis Model and State UUIDs, analysis type), sub-index groups `1/`, `2/`, … only when sub-results exist, then one group per quantity holding optional `positions` (Beam positions, section points or layer positions) and `values` (entities × [positions] × components, float64, chunked and compressed) with component, unit, coordinate system and location type attributes; `envelopes/<UUID>/<quantity>/` with `values` (max and min) and `governing` (Case and sub-index). | Agreed |
| G4a | By default one `.smr` file per Analysis Task; Combinations and Envelopes may use their own file. The manifest records which Case is in which file. | Agreed |
| G5 | Serialization inside the package is JSON (UTF-8), one or more files per domain listed in the manifest; numbers are double precision. Binary mesh tables may be added in a later minor version if needed. | Agreed |
| G6 | JSON Schema (2020-12) per domain in `schemas/`; schema and specification text are normative together. | Agreed |
| G7 | Any object may carry an `extensions` block keyed by namespace. Each namespace is declared in the manifest with a version and a `required` flag. Unknown extensions are preserved unchanged; an unknown required extension causes a warning or refusal. | Agreed |
| G8 | Format versioning uses SemVer (MAJOR.MINOR.PATCH) in the manifest. Minor versions are additive only; readers refuse a higher major version than they support. | Agreed |
