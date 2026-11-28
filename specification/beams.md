# OSMF Elements and Beams

**Status:** Draft. All rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets.

Machine-readable schema: [`schemas/beams.schema.json`](../schemas/beams.schema.json).

This document defines the properties all elements share, the Beam and the Beam Property. Sections and Materials have their own documents.

## 1. Common element properties

Every element (Beam, Shell, Solid, Spring, Ground Spring, Constraint) has the common object properties of [`core-objects.md`](core-objects.md) §1, a `number` in the element sequence [O3] and a `group` label [O12]. In addition:

| Property | Type | Required | Description |
|---|---|---|---|
| `inactive` | boolean | no, default `false` | An inactive element stays in the Structural Model with its properties, but is excluded from the Analysis Model. Loads and mass on it are skipped with a `warning` [E1]. |

*Note:* `inactive` is not for construction stages. Those use Structural States [E1, T1].

## 2. Beam

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"Beam"`, `number`, `name`, `extensions`, `group`, `inactive` | | | See §1. |
| `node_i`, `node_j` | UUID → Node | yes | Start and end Node [B1]. |
| `beam_property` | UUID → Beam Property | no | Without it the Beam has no stiffness and no mass [B4]. |
| `arc_point` | array of 3 numbers | no | A point on the arc [B15], global `[x, y, z]` in m. Without it the Beam is straight [B2]. |
| `arc_segmentation` | Arc Segmentation | no | Mesh instruction for an arc [B3]; see §2.2. |

Rules:

- `node_i` and `node_j` MUST be different Nodes, and their positions MUST NOT coincide within 1e-6 m (`error`).
- Positions along the Beam are measured from Node I, along the arc length for an arc [B9].

### 2.1 Arc

A Beam is a circular arc from Node I to Node J through `arc_point` [B2]. `arc_point` may be any point strictly between I and J on the arc [B15]. If the three points are collinear within 1e-6 m, the Beam is invalid (`error`). Splines are outside the core [B2].

### 2.2 Arc segmentation

The Analysis Model receives straight segments [B3]. Arc Segmentation is an object with exactly one of [B16]:

| Key | Type | Meaning |
|---|---|---|
| `segments` | integer ≥ 1 | Fixed number of segments |
| `max_length` | number > 0 | Maximum segment length in m |
| `max_angle` | number > 0 | Maximum angle per segment in degrees |

Without `arc_segmentation`, an arc uses `{ "max_angle": 10 }`. `arc_segmentation` on a straight Beam is ignored with a `warning`.

### 2.3 Local axes

Local axes follow [`conventions.md`](conventions.md) §4 [B7, B8]. Local x runs from I to J, along the tangent for an arc, where the rule is applied at each point.

### 2.4 Beam without Beam Property

A Beam without `beam_property` has no stiffness and no mass [B4]:

- If it lies in or on a Shell or Solid, the mesh follows it.
- Loads on it are transferred to the underlying mesh. If it lies on nothing, a load on it is an `error`.
- It can serve as a Line in Line-to-Line and Line-to-Point Constraints.

## 3. Beam Property

A Beam Property is a shared definition referenced by Beams [O4, B5]. It has the common object properties and a `number` in its own sequence [O3].

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"BeamProperty"`, `number`, `name`, `extensions` | | | See [`core-objects.md`](core-objects.md) §1. |
| `element_type` | enum | yes | See §3.1 [B6]. |
| `material` | UUID → Material | yes | |
| `section_i` | UUID → Section | yes | Section at Node I, or over the full length. |
| `section_j` | UUID → Section | no | Section at Node J for a tapered Beam. Missing or equal to `section_i` means prismatic [B12, B18]. |
| `rotation` | number | no, default `0` | Section rotation about local x, in degrees [U2, B7]. |
| `offset_i`, `offset_j` | array of 2 numbers | no, default `[0, 0]` | Offset `[dy, dz]` in m [B10, B19]; see §3.2. |
| `releases_i`, `releases_j` | Release | no | Simple releases at I and J [B11, B20]; see §3.3. |

### 3.1 Element types

Enum values for B6 [B17]:

| Value | Meaning |
|---|---|
| `beam` | Beam with shear deformation |
| `beam_no_shear` | Beam without shear deformation |
| `bar` | Axial force only |
| `strut` | Axial force only, compression-only |
| `tie` | Axial force only, tension-only |

For `bar`, `strut` and `tie`, releases are ignored with a `warning`, because these types carry only axial force. This includes a release of `x`, so the member keeps its axial stiffness [B21].

### 3.2 Offsets

An offset is the vector from the Node to the actual member end (end = node + offset), in the local y and z axes of the node line I→J [B10]. There are no axial offsets. Offsets at I and J are independent. Node geometry is unchanged. Section alignments (such as "top flush") are converted to explicit offsets on input [B10].

### 3.3 Releases

A Release is an object with one key per local DOF at that end [B20], like the Simple Support of [`nodes.md`](nodes.md) §2:

| Key | DOF (local axes) |
|---|---|
| `x`, `y`, `z` | translation along local x, y, z |
| `xx`, `yy`, `zz` | rotation about local x, y, z |

Each value is `"connected"` or `"released"`. A missing key means connected.

*Example:* a pinned end about both bending axes is `"releases_j": { "yy": "released", "zz": "released" }`.

Rules [B11]:

- Releases along the length are not possible.
- Complex releases (partial stiffness, nonlinear) use a separate Release object, to be specified in a later document. A simple release and a complex release on the same DOF and end are invalid (`error`).
- Releasing `x` or `xx` at both ends creates a mechanism and is reported with a `warning` [B22]. It is not an error, because Springs, Constraints or other members may restrain it. Releasing `yy` and `zz` at both ends is a normal pin-ended member.

### 3.4 Taper

Tapering follows B13:

- Both Sections MUST use the same definition method and shape type, otherwise the Beam Property is invalid (`error`).
- Parametric: dimensions vary linearly, and properties follow from the interpolated geometry.
- User-defined: same topology, and coordinates vary linearly.
- Explicit properties: values vary linearly.
- Profile library Sections cannot taper. Convert them to parametric first.
- Non-linear haunches: split the member.

## 4. Example

The [`examples/beams.example.json`](../examples/beams.example.json) file shows a straight Beam, an arc Beam with segmentation, a Beam without Beam Property and a tapered Beam Property with an offset and releases.
