---
'@platforma-open/milaboratories.sequence-embeddings': patch
'@platforma-open/milaboratories.sequence-embeddings.model': patch
---

Add the block's `kind` component

Every block must now declare a kind — a fourth component next to model, workflow
and ui. It carries the block's identity and its init-params contract, and
`block-tools structure check` fails without it.

This block's contract is `{ inputAnchor?: PlRef; embedding?: EmbeddingSelection }`.
A project template can therefore seed a new instance with both the dataset to
embed and the (scope, model) combination to embed it with. Everything else in
the block's data still defaults: `embeddingInitializedForAnchor` is the UI's
re-seed guard, `mem` and `cpu` are opt-in overrides the workflow otherwise sizes
itself, and `defaultBlockLabel` is written by the UI from the chosen input.

The two fields travel as a pair. A scope's `columns` are anchored ids that
resolve against `inputAnchor`, so an `embedding` seeded without the matching
`inputAnchor` points at nothing. Both stay optional, and the args projection —
which already refuses an incomplete selection — is what catches the mismatch
when the block runs.

The model's `init` now reads those params, and a new `templateParams` projection
hands the same two fields back out, so seeding and exporting are inverses. A
block created without params starts exactly as before.

The types the contract is built from (`EmbeddingSelection`, `SelectedScope`,
`EmbeddingModelId`, `Fidelity`, `ScopeFeature`, `ScopeReceptor`,
`WorkflowReceptor`) moved into the kind package, which owns the contract. The
model re-exports them, so every existing import path keeps working.
