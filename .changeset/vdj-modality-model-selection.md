---
'@platforma-open/milaboratories.sequence-embeddings.model': minor
'@platforma-open/milaboratories.sequence-embeddings': minor
---

Offer antibody and TCR models for VDJ amplicon-profiling input (MILAB-6668).

`synthetic-repertoire-profiler` tags its whole-variant amino-acid sequence with the
`amplicon-sequence` feature in both of its modalities, and this block mapped that
feature onto the `peptide` scope unconditionally. Since model compatibility is gated
on the scope feature, a DMS **VDJ** run was offered only ESM-2 and PeptideCLM-2, with
the peptide specialist as the default — no antibody or TCR model was reachable.

The block now reads the producer's `pl7.app/modality` declaration off the entity
axis. On a `vdj` run the whole-variant column becomes a `VDJRegion` scope and the
profiler's CDR3 region column becomes a `CDR3` scope (a new selector discovers the
region columns, which are keyed by region name in `pl7.app/feature` rather than on
`pl7.app/vdj/sequence`). That puts CurrAb, AbLang2, VHHBERT, H3BERTa and TCR-BERT in
reach. Amplicon runs, projects predating the declaration, and peptide-extraction
input are unaffected — all keep the peptide scope and the PeptideCLM-2 default.

Receptor handling: the profiler declares VDJ data but emits neither
`pl7.app/vdj/receptor` nor `pl7.app/vdj/chain`, because germline auto-detection
builds a custom reference from the user's own parent sequences and has no library
locus to read. `ScopeReceptor` gains an explicit `"unknown"` value for that case, and
model filtering relaxes its receptor and heavy-chain gates rather than defaulting to
`IG` — which would have silently hidden the TCR specialists from TCR data. The trade
is deliberate: the dropdown can now offer a TCR model for antibody input, so the user
picks the model that matches their library. With the receptor unknown the default
model is the universal one (ESM-2) rather than the highest-priority specialist, so no
receptor-specific model is chosen on the user's behalf from data that does not state
a receptor.

Inputs that do carry a receptor or chain key are unchanged, as are inputs with no
modality declaration — both keep the historical `IG` fallback and full gating.
