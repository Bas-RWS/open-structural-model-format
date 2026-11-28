# OSMF Core Objects

**Status:** Draft. All rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets. Items marked *(Proposed)* are not yet approved.

Machine-readable schema: [`schemas/core.schema.json`](../schemas/core.schema.json).

This document defines the properties that all objects share, how they are identified and numbered, Group labels and the List object. The object-specific contents of Nodes, Beams and so on follow in separate documents.

## 1. Common object properties

Every OSMF object has these properties:

| Property | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | yes | Identity of the object [O1]. |
| `type` | string | yes | Object type in PascalCase, for example `"Node"` or `"Beam"` [O11]. |
| `number` | integer ≥ 1 | yes, for types with a number sequence | Human-facing number [O2, O14]; see §3. |
| `name` | string | no | Optional name. Need not be unique. |
| `extensions` | object | no | Namespaced extension data [G7]. |

Elements (Beam, Shell, Solid, Spring, Ground Spring, Constraint) also have:

| Property | Type | Required | Description |
|---|---|---|---|
| `group` | Group label | yes | The single Group the element belongs to [O7, O12]; see §4. |

Nodes, Mass, shared definitions (Materials, Sections, Properties, Spring Curves), Loadcases, Loads, Grid Planes, Structural States and Lists do not belong to a Group [O12].

*Note:* Loads are organized by their Loadcase [L1]. Shared definitions are organized by type.

## 2. Identity (UUID)

- `id` is a UUID written as a lowercase string in canonical form `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.
- Writers SHOULD generate version 4 (random) or version 7 (time-ordered) UUIDs. Readers MUST accept any version [O13].
- A UUID MUST be unique within a project across all object types.
- References always use the UUID, never the number [O1]. The only exception is the Group label (§4), because a Group is not an object.
- Copying an object creates a new UUID [O5].

## 3. Object numbering

Numbers are positive integers without encoded meaning. Gaps are allowed [O2].

`number` is required for every object type listed below [O14]. A missing number is an `error`.

Numbers MUST be unique within their **number sequence**:

| Sequence | Object types |
|---|---|
| Element | Beam, Shell, Solid, Spring, Ground Spring, Constraint [O3] |
| Per type | Node, Material, Section, Beam Property, Spring Property, Ground Spring Property, Spring Curve, Mass, Loadcase, Grid Plane, Structural State, Analysis Task, List |
| Per Load type | Node Load, Beam Line Load and every other Load type, each with its own sequence [O15] |

*Example:* Beam 3 and Shell 3 cannot both exist. Node 3 and Beam 3 can. Node Load 1 and Beam Line Load 1 can.

When objects are copied or merged, they receive new numbers. The UUID stays leading [O5].

### 3.1 Numbering in the Analysis Model

Objects in an Analysis Model (the generated mesh) have their own number sequences within each Analysis Model [O14]. Generation keeps the numbers of the Structural Model where it can [O16]:

- An analysis node that corresponds to a Structural Model Node MUST take over that Node's number.
- An analysis node that exists only because of meshing gets a number above the highest Node number in the Structural Model.
- An analysis element that corresponds one-to-one to a Structural Model element MUST take over that element's number.
- When an element is split into several analysis elements (arc segmentation [B3], intersections, maximum length, Shell and Solid meshing), the analysis elements get numbers above the highest element number in the Structural Model.
- *(Proposed)* Every analysis node and analysis element carries a `source` reference (UUID) to the Structural Model object it was generated from, if there is one. The link is then kept even when numbers change.

*Note:* results always reference Analysis Model UUIDs, never solver numbers [P2].

## 4. Group

A Group is an organizational label on an element. It is not an object, and it does not change any element properties [O10, O12].

### 4.1 Label

- The label is `G` followed by one to three positive numbers separated by `_`, for example `G1`, `G1_2`, `G1_2_3` [O7].
- Pattern: `^G[1-9][0-9]*(_[1-9][0-9]*){0,2}$`. A fourth level is invalid (`error`).
- The hierarchy follows from the label: `G1_2` is a subgroup of `G1`. The numbers count per parent, so `G1_1` and `G2_1` are different Groups [O7].
- A Group exists as soon as an element uses its label or one of its subgroup labels. A parent Group may contain elements directly as well as subgroups [O7].
- Writers MUST write `"G1"` when the user defines no Groups [O12].

### 4.2 Group names

*(Proposed)* Names are stored in an optional map `group_names` in the Project Core, from label to name:

```json
"group_names": { "G1": "Pier", "G1_2": "Pier cap" }
```

A label without a name is allowed. A name for a label that no element uses MAY be kept, so that an empty Group keeps its name. A key that does not match the label pattern is invalid (`error`).

### 4.3 Operations

- Renumbering a Group means rewriting the label on its elements, in its subgroup labels and in Lists that reference it.
- Copying a Group creates new elements with a new label, reconnects internal relationships and drops external connections; the original is unchanged [O6].

## 5. List

A List is a manually composed selection. It does not affect Group membership or object properties [O10]. An object may be in any number of Lists.

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"List"`, `number`, `name`, `extensions` | | | See §1. |
| `terms` | array of List Term | yes | The terms that define the members. May be empty. |
| `source_text` | string | no | Original input text, for traceability only; not normative [O8]. |

### 5.1 List Term

| Property | Type | Required | Description |
|---|---|---|---|
| `mode` | `"include"` or `"exclude"` | yes | Whether the term adds or removes members. |
| `kind` | `"object"`, `"group"`, `"property"` or `"list"` | yes | What the term refers to [O8]. |
| `ref` | UUID | for `object`, `property`, `list` | The referenced object, definition or List. |
| `label` | Group label | for `group` | The referenced Group [O12]. |
| `include_subgroups` | boolean | no | Only for `kind = "group"`. Default `true` [O17]. |

Meaning of each kind [O8]:

| Kind | Resolves to | Behavior |
|---|---|---|
| `object` | the referenced object itself | static |
| `group` | all elements with this label, and with its subgroup labels if `include_subgroups` | dynamic |
| `property` | all objects that use the referenced Property, Material or Section, directly or through their Property [O18] | dynamic |
| `list` | the resolved members of the referenced List | dynamic |

*Example:* a `property` term that references Section HEA300 selects every Beam whose Beam Property uses HEA300.

### 5.2 Resolution

Members are resolved when the List is used, not when it is stored [O8]:

1. Resolve all `include` terms and take the union.
2. Resolve all `exclude` terms and take the union.
3. The members are (1) minus (2).

Resolution does not depend on term order, so an `exclude` always wins over an `include` of the same object [O19].

Rules:

- Ranges such as `1 to 100` are expanded at input into separate `object` terms [O8]. They are never stored as ranges.
- Cycles between Lists (directly or through other Lists) are invalid (`error`).
- A term whose `ref` does not exist is invalid (`error`). A `group` term whose label no element uses resolves to nothing and is reported with a `warning`.
- Mixed object types are allowed [O9]. Where a List is used, members of a type that does not apply there are skipped with a `warning` [O9].
- Any change in resolved membership marks dependent results as stale [R2].

## 6. Example

The [`examples/core-objects.example.json`](../examples/core-objects.example.json) file shows Group names, three elements in Groups and a List that selects `G1` plus `G2`, minus one element, plus one extra Node. It validates against `schemas/core.schema.json`.
