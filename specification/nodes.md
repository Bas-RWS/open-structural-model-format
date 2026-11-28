# OSMF Nodes and Supports

**Status:** Draft. All rules restate Agreed decisions from [`decisions.md`](decisions.md). The decision IDs are given in brackets. Items marked *(Proposed)* are not yet approved.

Machine-readable schema: [`schemas/nodes.schema.json`](../schemas/nodes.schema.json).

This document defines the Node and the Simple Support. Ground Springs, Springs and Constraints have their own documents.

## 1. Node

A Node is a point in the Structural Model [N1]. It has no local orientation and does not belong to a Group [N1, O12].

| Property | Type | Required | Description |
|---|---|---|---|
| `id`, `type` = `"Node"`, `number`, `name`, `extensions` | | | See [`core-objects.md`](core-objects.md) §1. `number` is required. |
| `position` | array of 3 numbers | yes | *(Proposed)* Global coordinates `[x, y, z]` in m. |
| `support` | Simple Support | no | Restrained DOFs. Without `support`, all six DOFs are free [N2]. |

Rules:

- Coordinates are finite doubles in m [U1, conventions §5].
- *(Proposed)* Two Nodes whose positions are closer than 1e-6 m are reported with a `warning`. They remain separate Nodes; the format does not merge them.

## 2. Simple Support

A Simple Support restrains DOFs of a Node in the global axes [N2]. Complex and angled supports use Ground Springs [N3].

*(Proposed)* A Simple Support is an object with one key per DOF. Each value is `"free"` or `"fixed"`. A missing key means `"free"`.

| Key | DOF |
|---|---|
| `x`, `y`, `z` | translation along Global X, Y, Z |
| `xx`, `yy`, `zz` | rotation about Global X, Y, Z |

*Example:* a pinned support is

```json
"support": { "x": "fixed", "y": "fixed", "z": "fixed" }
```

Rules:

- A Simple Support and a Ground Spring MUST NOT act on the same DOF of the same Node [N3]. *(Proposed)* This is an `error`.
- Line and area supports are not stored; postprocessing reconstructs them from nodal reactions and Constraints [N4].
- "Rigid link" is not a separate concept. Use a Rigid Point-to-Point Constraint [N5].

## 3. Results

Node displacements and reactions are always in the global axes. The reaction at a Node is the sum of its Simple Support and its Ground Springs [P5].

## 4. Example

The [`examples/nodes.example.json`](../examples/nodes.example.json) file shows a free Node, a pinned Node and a fully fixed Node.
