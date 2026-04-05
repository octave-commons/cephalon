# cephalon

**Cephalon** is the head of the agent system: the thing the user is actually speaking to.

This repo is the intended **single source of truth** for the Cephalon family inside the `octave-commons` line.

## Reading order

1. `docs/INDEX.md`
2. `docs/CONSOLIDATION_MAP.md`
3. `docs/FORK_TALES_SOURCE_MAP.md`
4. `docs/OPENCODE_SESSION_PROVENANCE.md`
5. `docs/ANNOTATED_SOURCE_EXCERPTS.md`
6. `specs/head-of-agent-system.md`
7. `specs/adjacent-systems-matrix.md`
8. `specs/graph-workbench-adapter.md`
9. `specs/cephalon-openplanner-graph-query-contract.md`
10. `specs/package-lattice.md`
11. `specs/boundary-contract.md`
12. `specs/recovered-clj-absorption.md`
13. `specs/implementation-backlog.md`
11. package dossier indexes under `packages/*/docs/`

## Package lattice

- `packages/cephalon-ts` — TypeScript head/runtime/hive/control-plane path
- `packages/cephalon-cljs` — ClojureScript always-running mind / ECS / eidolon path
- `packages/cephalon-clj` — JVM Clojure skeleton and precursor runtime
- `recovered/cephalon-clj` — recovered two-process experiment archive and path archaeology

## Package dossiers

- `packages/cephalon-ts/docs/INDEX.md` — operational runtime dossier
- `packages/cephalon-cljs/docs/INDEX.md` — always-running mind / ECS dossier
- `packages/cephalon-clj/docs/INDEX.md` — precursor runtime dossier
- `recovered/cephalon-clj/docs/INDEX.md` — archaeology dossier

## Why this repo exists

Cephalon was previously scattered across:
- `packages/cephalon-ts`
- `packages/cephalon-clj`
- `services/cephalon-cljs`
- `recovered/cephalon-clj`
- multiple specs and session artifacts

This repo is the act of making that head legible again.
