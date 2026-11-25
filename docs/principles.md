# OSMF Design Principles

These principles guide the development of OSMF. They represent the current design baseline and should not be changed implicitly.

## 1. Open

The format should be openly documented and implementable without proprietary dependencies.

## 2. Vendor-neutral

The core format must not depend on a specific software vendor, solver or platform.

## 3. Modular domains

Domains are independently defined and optional. A file or project may contain any combination of domains.

## 4. No mandatory workflow

OSMF must not require a prescribed sequence such as geometry, structural model, analysis model and results. Models may be created independently or linked when appropriate.

## 5. Semantic clarity

Data should explicitly describe what it represents. Meaning should not depend solely on application-specific conventions or naming.

## 6. Stable identifiers

Entities and datasets should have identifiers that enable reliable references across models, domains and revisions.

## 7. Traceability and provenance

Relationships, derivations, transformations, source information and relevant history should be representable.

## 8. Extensibility

The format should support domain extensions and application-specific information without compromising the core specification.

Unknown extensions should be preservable by applications that do not interpret them.

## 9. Long-term preservation

The format should be documented, versioned and designed to remain interpretable over time.

## 10. AI compatibility

Explicit semantics, structured data, stable identifiers and traceable relationships should make OSMF suitable for automated processing and AI-assisted engineering.

## 11. External large datasets

Large datasets such as point clouds, measurements, media, meshes and other data collections should be stored externally and referenced from OSMF.

External references should support portability, integrity verification and clear identification of missing data.

## 12. External computational results

All computational result values shall be stored in external standardized result datasets, regardless of dataset size.

The OSMF project shall contain result metadata, descriptions, indexes, provenance and references, but not actual computational result arrays or values.

A dedicated result file extension using HDF5 as its underlying storage technology is a candidate design. The extension and internal layout remain to be decided.

## 13. Separation of data and implementation

OSMF describes data, meaning and relationships. It should not prescribe how an application internally stores, processes or displays that information.

## 14. Explicit decisions

Design decisions should be documented and classified as:

- Agreed
- Proposed
- Open question

Proposals must not be treated as final decisions without explicit agreement.
