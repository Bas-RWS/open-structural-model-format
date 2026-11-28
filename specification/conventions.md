# OSMF Specification Conventions

**Status:** Draft. Items marked *(Proposed)* are new in this document and are not yet in the decision log. All other rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets.

## 1. Normative language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT** and **MAY** are used as described in RFC 2119.

Text marked as *Note* or *Example* is informative. Everything else is normative.

The specification text and the JSON Schemas in `schemas/` are normative together [G6]. If they conflict, the conflict is a defect and MUST be reported and fixed.

## 2. Decision status

Every design rule has one of three statuses:

- **Agreed:** explicitly approved and fixed unless reopened.
- **Proposed:** presented but not yet approved.
- **Open:** still requires a decision.

Specification documents state only Agreed rules as normative. Proposed rules are marked *(Proposed)*. Once approved, they are added to the decision log and the marker is removed.

## 3. Units

All values MUST be stored in the fixed SI units listed below [U1–U3]. Files contain no unit fields. Units follow from the quantity.

| Quantity | Unit |
|---|---|
| Length, coordinate, displacement | m |
| Force | N |
| Moment | N·m |
| Mass | kg |
| Time | s |
| Temperature, temperature difference | K (a ΔT is numerically equal in °C) |
| Stress, pressure, modulus | Pa (N/m²) |
| Density | kg/m³ |
| Line load | N/m |
| Area load | N/m² |
| Translational stiffness | N/m |
| Rotation (result, DOF) | rad |
| Rotational stiffness | N·m/rad |
| Input angle (Section rotation, Grid Plane distribution angle) | degree |

An optional preferred-units setting in the Project Core MAY record display preferences. It MUST NOT change stored values.

## 4. Coordinate systems

- The **global coordinate system** is a right-handed Cartesian system X, Y, Z.
- *(Proposed)* Global Z points up by convention. The direction and magnitude of gravity are stored explicitly as a vector in the Project Core, with the default (0, 0, −9.81) m/s². Self-weight is derived from that vector and the material density [M3].
- **Local axes** of Beams and Springs follow the agreed rule [B7, K2]:
  - local x runs from I to J (along the tangent for arcs);
  - Global Z is the reference for local y and z, with Global Y as fallback when Global Z is near-parallel to x;
  - the axes are right-handed;
  - Section rotation is an extra rotation about local x.
- The **reference-switch tolerance** is 1e-6 in the projection/parallelism test [B8].
- Nodes have no local orientation [N1].

## 5. Numeric values

- Numbers are JSON numbers, interpreted as IEEE 754 double precision [G5].
- Writers SHOULD write values with enough digits to round-trip a double exactly.
- `NaN` and infinity are not allowed.

## 6. Naming in files *(Proposed)*

- JSON property names use `snake_case`, for example `beam_property`.
- Object type names use PascalCase as in the specification text, for example `"type": "GroundSpring"`.
- Enumerated values use `snake_case` strings, for example `"compression_only"`.

## 7. Identity and references

- Every object has a UUID, and every reference to another object uses that UUID [O1]. The format is given in [`core-objects.md`](core-objects.md) §2.
- A reference MUST point to an object of the type the field allows. A reference to a missing object, or to an object of the wrong type, makes the file invalid.

## 8. Validation and reporting

OSMF processing never discards information silently (`docs/principles.md` §16).

*(Proposed)* Validation findings use three severities:

| Severity | Meaning |
|---|---|
| `error` | The file or model is invalid. Processing that depends on the faulty data MUST stop. |
| `warning` | Data was skipped, ignored or adapted. Processing may continue, but the finding MUST be reported. |
| `info` | Informative message only. |

Each finding SHOULD carry a severity, a stable message code, the UUIDs of the objects involved and a human-readable text.

*Example:* a Beam Line Load on an inactive Beam is skipped with a `warning` [E1]. Two Constraints that both drive the same target DOF are an `error` [C7].

## 9. Extensions

- Any object MAY carry an `extensions` object with one key per namespace [G7].
- Each namespace used MUST be declared in the manifest, with its version and a `required` flag.
- Readers MUST preserve unknown extensions unchanged.
- A reader that meets an unknown extension marked `required` MUST warn or refuse to open the file.

*(Proposed)* Namespace names are lowercase `[a-z][a-z0-9_]*`, for example `eurocode`, `sofistik` or `gsa`.

## 10. Versioning

- The format version follows SemVer (`MAJOR.MINOR.PATCH`) and is stored in the manifest [G8].
- Minor versions only add things.
- A reader MUST refuse a file with a higher major version than it supports.
