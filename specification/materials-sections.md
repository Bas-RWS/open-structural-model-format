# OSMF Materials and Sections

**Status:** Draft. All rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets. Items marked *(Proposed)* are not yet approved.

Machine-readable schema: [`schemas/materials-sections.schema.json`](../schemas/materials-sections.schema.json).

Materials and Sections are shared definitions. Beam Properties and other Properties reference them [O4]. They have the common object properties of [`core-objects.md`](core-objects.md) §1 and a `number` in their own sequence [O3]. They do not belong to a Group [O12].

## 1. Provenance

All library items are copied into the project on import as standalone objects. They are never live-linked [M5].

*(Proposed)* An object copied from a library carries a `provenance` object:

| Key | Type | Required | Description |
|---|---|---|---|
| `library` | string | yes | Library name, for example `"EN 10365"` or `"EN 1992-1-1"` |
| `version` | string | yes | Library version |
| `item_id` | string | yes | Identifier of the item in the library, for example `"HEA300"` or `"C30/37"` |

The values in the project are leading. A library update never changes them [M5].

## 2. Material

A Material has four parts [M1]. *(Proposed)* Each part is a nested object.

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"Material"`, `number`, `name`, `extensions` | | | See [`core-objects.md`](core-objects.md) §1. |
| `general` | object | yes | §2.1 |
| `properties` | object | yes | §2.2 |
| `strength` | object | no | §2.3 |
| `additional` | object | no | §2.4 |
| `provenance` | Provenance | no | §1 |

Linear elastic behaviour is the default. Nonlinear behaviour is optional and is specified in a later document [M2]. Damping belongs to the analysis, not to the Material [M4].

### 2.1 General

| Key | Type | Required | Description |
|---|---|---|---|
| `category` | enum | yes | *(Proposed)* `concrete`, `steel`, `reinforcement`, `prestressing_steel`, `timber`, `aluminium`, `masonry` or `other` |
| `grade` | string | no | Grade name, for example `"S355"` or `"C30/37"` |

### 2.2 Properties

*(Proposed)* The core Material is isotropic.

| Key | Unit | Required | Description |
|---|---|---|---|
| `elastic_modulus` | Pa | yes | E |
| `poisson_ratio` | – | yes | ν, with −1 < ν < 0.5 |
| `shear_modulus` | Pa | no | G. *(Proposed)* If missing, G = E / (2(1 + ν)). |
| `density` | kg/m³ | yes | ρ. Self-weight is derived as ρ·g from the project gravity and is not stored [M3, U4]. |
| `thermal_expansion` | 1/K | no | α |

Anisotropic and orthotropic materials are outside the core and use an extension.

### 2.3 Strength

Design codes are not part of the base Material [M4]. *(Proposed)* `strength` holds characteristic values in Pa under fixed key names, without partial factors:

| Key | Description | Typical categories |
|---|---|---|
| `f_y` | yield strength | steel, reinforcement, aluminium |
| `f_u` | tensile strength | steel, reinforcement, prestressing_steel |
| `f_p01` | 0.1 % proof stress | prestressing_steel |
| `f_ck` | compressive strength (cylinder) | concrete |
| `f_ctm` | mean tensile strength | concrete |
| `f_m` | bending strength | timber |

Other strength values MAY be added as extra keys in Pa. Code-specific values (partial factors, `f_cd` and so on) belong in a code extension, for example `extensions.eurocode`.

### 2.4 Additional

`additional` is a free object for values that are not part of the analysis, for example colour or a supplier code.

## 3. Section

A Section is a reusable object. It is defined in one of four ways [B14]:

| `definition` | Content |
|---|---|
| `explicit` | Explicit section properties (§3.2) |
| `parametric` | A shape type and its dimensions (§3.3) |
| `library` | A profile from a library: its designation and its explicit properties, copied in (§3.4) |
| `geometry` | User-defined or composite geometry (§3.5) |

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"Section"`, `number`, `name`, `extensions` | | | See [`core-objects.md`](core-objects.md) §1. |
| `definition` | enum | yes | One of the four above |
| `properties` | Section Properties | see below | Required for `explicit` and `library`. MAY be stored for `parametric` and `geometry` as computed values. |
| `shape` | object | for `parametric` | §3.3 |
| `geometry` | object | for `geometry` | §3.5 |
| `provenance` | Provenance | for `library` | §1 |
| `design_metadata` | object | no | Information for design checks, such as fabrication type or section class [B14] |

### 3.1 Section axes

*(Proposed)* Section coordinates use the local y and z axes of the Beam (`conventions.md` §4). The Beam axis passes through the centroid of the Section. A different reference point, such as the top of a flange, is converted to an explicit Beam offset [B10].

### 3.2 Section Properties

*(Proposed)* Properties are given about the centroidal y and z axes, not the principal axes. Solvers derive the principal axes from `i_yz`.

| Key | Unit | Required | Description |
|---|---|---|---|
| `area` | m² | yes | A |
| `i_y`, `i_z` | m⁴ | yes | Second moments of area about y and z |
| `i_yz` | m⁴ | no, default 0 | Product of area |
| `i_t` | m⁴ | yes | Torsion constant |
| `shear_area_y`, `shear_area_z` | m² | no | Shear areas. *(Proposed)* If missing, the Section has no shear deformation, and a Beam with element type `beam` gives a `warning`. |
| `i_w` | m⁶ | no | Warping constant |
| `shear_centre` | [y, z] in m | no, default [0, 0] | Position of the shear centre relative to the centroid |

### 3.3 Parametric

*(Proposed)* `shape` has a `kind` and dimensions in m:

| `kind` | Dimensions |
|---|---|
| `rectangle` | `h`, `b` |
| `circle` | `d` |
| `circular_hollow` | `d`, `t` |
| `rectangular_hollow` | `h`, `b`, `t` (optional `r` outer corner radius) |
| `i_section` | `h`, `b`, `t_w`, `t_f` (optional `r` root radius); doubly symmetric |
| `t_section` | `h`, `b`, `t_w`, `t_f` |
| `channel` | `h`, `b`, `t_w`, `t_f` |
| `angle` | `h`, `b`, `t` |

`h` is measured along local z and `b` along local y. Other shapes use `geometry`.

### 3.4 Library profile

A library Section is copied in with its explicit `properties` and its `provenance` [M5]. Its shape MAY be stored in `shape` for drawing. A library Section cannot taper; convert it to `parametric` first [B13].

### 3.5 Geometry

*(Proposed)* `geometry` holds one or more `parts`. Each part has:

| Key | Type | Required | Description |
|---|---|---|---|
| `outline` | array of [y, z] | yes | Closed polygon, counterclockwise, without repeating the first point |
| `holes` | array of polygons | no | Clockwise polygons inside the outline |
| `material` | UUID → Material | no | For composite Sections. Without it, the part uses the Material of the Beam Property. |

The coordinates MAY use any origin. The centroid is computed from the parts, weighted by elastic modulus for composite Sections, and the Beam axis passes through it (§3.1). Overlapping parts are invalid (`error`).

## 4. Taper

Taper rules are given in [`beams.md`](beams.md) §3.4 [B13].

## 5. Example

The [`examples/materials-sections.example.json`](../examples/materials-sections.example.json) file shows a steel and a concrete Material, a library HEA300, a parametric rectangle, an explicit Section and a composite geometry Section.
