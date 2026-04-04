# cephalon

**Cephalon** is the head of the agent system: the thing the user is actually speaking to.

This repo is now the intended **single source of truth** for the Cephalon family inside the `octave-commons` line.

## Reading order

1. `docs/INDEX.md`
2. `docs/CONSOLIDATION_MAP.md`
3. `specs/head-of-agent-system.md`
4. `specs/package-lattice.md`
5. package READMEs under `packages/`
6. `specs/recovered-clj-absorption.md`

## Package lattice

- `packages/cephalon-ts` — TypeScript head/runtime/hive/control-plane path
- `packages/cephalon-cljs` — ClojureScript always-running mind / ECS / eidolon path
- `packages/cephalon-clj` — JVM Clojure skeleton and precursor runtime
- `recovered/cephalon-clj` — recovered two-process experiment archive and path archaeology

## Why this repo exists

Cephalon was previously scattered across:
- `packages/cephalon-ts`
- `packages/cephalon-clj`
- `services/cephalon-cljs`
- `recovered/cephalon-clj`
- multiple specs and session artifacts

This repo is the act of making that head legible again.
