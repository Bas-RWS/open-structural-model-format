# OSMF Core Objects

**Status:** Draft. Items marked *(Proposed)* are new in this document and are not yet in the decision log. All other rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets.

Machine-readable schema: [`schemas/core.schema.json`](../schemas/core.schema.json).

This document defines the properties that all objects share, how they are identified and numbered, and the two organizational objects Group and List. The object-specific contents of Nodes, Beams and so on follow in separate documents.

## 1. Common object properties

Every OSMF object has these properties:

| Property | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | yes | Identity of the object [O1]. |
| `type` | string | yes | Object type, for example `"Node"` or `"Beam"`. *(Proposed)* |
| `number` | integer ≥ 1 | depends on type | Human-facing number [O2]; see §3. |
| `name` | string | no | Optional name. Need not be unique. |
| `extensions` | object | no | Namespaced extension data [G7]. |

Structural objects also have:

| Property | Type | Required | Description |
|---|---|---|---|
| `group` | UUID → Group | yes | The single Group the object belongs to [O7]. |

*(Proposed)* `group` applies to Nodes, all elements (Beam, Shell, Solid, Spring, Ground Spring, Constraint) and Mass. Shared definitions do not belong to a Group: Materials, Sections, Properties and Spring Curves. Neither do Loadcases, Loads, Grid Planes, Structural States, Lists or Groups themselves.

*Note:* Loads are organized by their Loadcase [L1]. Shared definitions are organized by type.

## 2. Identity (UUID)

- `id` is a UUID written as a lowercase string in canonical form `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.
- *(Proposed)* Writers SHOULD generate version 4 (random) or version 7 (time-ordered) UUIDs. Readers MUST accept any version.
- A UUID MUST be unique within a project across all object types.
- References always use the UUID, never the number [O1].
- Copying an object creates a new UUID [O5].

## 3. Object numbering

Numbers are positive integers without encoded meaning. Gaps are allowed [O2].

Numbers MUST be unique within their **number sequence**:

| Sequence | Object types |
|---|---|
| Element | Beam, Shell, Solid, Spring, Ground Spring, Constraint [O3] |
| Per type | Node, Material, Section, Beam Property, Spring Property, Ground Spring Property, Spring Curve, Mass, Loadcase, each Load type, Grid Plane, Structural State, Analysis Task, List |
| Per parent | Group (see §4) |

*Example:* Beam 3 and Shell 3 cannot both exist. Node 3 and Beam 3 can.

*(Proposed)* Each Load type has its own sequence (Node Load, Beam Line Load and so on), as in most programs. The alternative is one sequence for all Loads.

When objects are copied or merged, they receive new numbers. The UUID stays leading [O5].

*(Proposed)* `number` is required for every object type listed above. Analysis Model objects (the generated mesh) also have numbers, but in their own sequences within each Analysis Model.

## 4. Group

A Group is an organizational container. It does not change any object properties [O10].

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"Group"`, `name`, `extensions` | | | See §1. |
| `number` | integer ≥ 1 | yes | Sequential number **within the parent**. |
| `parent` | UUID → Group or `null` | yes | Parent Group; `null` for a top-level Group. |

Rules:

- Hierarchy is at most three levels deep [O7]. A Group at level 3 (for example `G1_2_3`) MUST NOT have subgroups; a fourth level is invalid (`error`).
- A Group MUST NOT be its own ancestor (`error`).
- A parent Group may contain objects directly as well as subgroups [O7].
- The **label** is derived, not stored: `G` plus the chain of numbers, for example `G1`, `G1_2`, `G1_2_3`.
- *(Proposed)* Group numbers are unique among siblings, not project-wide. This is the only exception to the per-type numbering of §3. It follows directly from the agreed label notation, where `G1_1` and `G2_1` both have number 1.
- Every structural object references exactly one Group (§1). *(Proposed)* A project with structural objects MUST contain at least one Group. Writers SHOULD create a default Group `G1` when the user defines none.
- *(Proposed)* A Group that still contains objects or subgroups cannot be deleted without first reassigning them. This is a rule for applications; the format only requires that every reference resolves.

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
| `ref` | UUID | yes | The referenced object, Group, Property or List. |
| `include_subgroups` | boolean | no | Only for `kind = "group"`. Default *(Proposed)* `true`. |

Meaning of each kind [O8]:

| Kind | Resolves to | Behavior |
|---|---|---|
| `object` | the referenced object itself | static |
| `group` | all objects in the Group, and in its subgroups if `include_subgroups` | dynamic |
| `property` | all objects that reference this Property, Material or Section *(Proposed: Material and Section also allowed)* | dynamic |
| `list` | the resolved members of the referenced List | dynamic |

### 5.2 Resolution

Members are resolved when the List is used, not when it is stored [O8]:

1. Resolve all `include` terms and take the union.
2. Resolve all `exclude` terms and take the union.
3. The members are (1) minus (2).

*(Proposed)* Resolution is independent of term order, so an `exclude` always wins over an `include` of the same object.

Rules:

- Ranges such as `1 to 100` are expanded at input into separate `object` terms [O8]. They are never stored as ranges.
- Cycles between Lists (directly or through other Lists) are invalid (`error`).
- A term whose `ref` does not exist is invalid (`error`).
- Mixed object types are allowed [O9]. Where a List is used, members of a type that does not apply there are skipped with a `warning` [O9].
- Any change in resolved membership marks dependent results as stale [R2].

## 6. Example

The [`examples/core-objects.example.json`](../examples/core-objects.example.json) file shows two Groups with a subgroup and a List that selects `G1` plus `G2`, minus one element, plus one extra Node. It validates against `schemas/core.schema.json`.
