# Cephalon Implementation Backlog

## Goal

Turn the consolidated Cephalon canon into a legible, convergent program without destroying the distinct insights carried by each package.

## Phase 0 — keep the dossier honest

- [ ] keep root docs and package-level dossier indexes in sync
- [ ] add one package-status matrix showing what is active, precursor, or archival
- [ ] explicitly mark `packages/cephalon-ts/src/app.ts` as the preferred TS assembly seam
- [ ] explicitly mark `packages/cephalon-ts/src/main.ts` as experimental/legacy unless promoted again
- [ ] keep `recovered/cephalon-clj` clearly labeled as archaeology rather than runnable source

## Phase 1 — harden each living stratum

### TypeScript head runtime
- [ ] smoke-test the `createCephalonApp` path
- [ ] verify the eight-circuit scheduler and output-channel routing in one reproducible local profile
- [ ] test `DiscordIntegration`, `ToolExecutor`, and `TurnProcessor` as a single runtime slice
- [ ] decide whether `main.ts` should be absorbed into `app.ts` or remain an explicit lab path

### ClojureScript mind runtime
- [ ] smoke-test the shadow-cljs build and test targets from the canonical repo
- [ ] verify the ECS system order still matches the note/spec doctrine
- [ ] test the TS bridge path deliberately instead of leaving it as a mostly-implicit option
- [ ] identify which note-derived concepts are now executable and which are still only conceptual

### JVM Clojure precursor
- [ ] verify the JVM runtime still boots cleanly from `deps.edn`
- [ ] run the sentinel note-tagging path against a sandbox note set
- [ ] document what parts of eidolon retrieval still work and what parts are only scaffolding

## Phase 2 — boundary contracts between strata

Drafted contract set:
- `specs/boundary-contract.md`
- `specs/contracts/event-envelope.md`
- `specs/contracts/memory-record.md`
- `specs/contracts/tool-surface.md`
- `specs/contracts/runtime-state-and-handoff.md`

- [x] draft one shared event-envelope contract across TS, CLJS, and precursor CLJ paths
- [x] draft one shared memory-record contract with clear required vs optional fields
- [x] draft one shared tool-surface vocabulary for note, memory, Discord, browser, and runtime actions
- [x] draft one runtime-state contract for graph/field/prompt summaries
- [x] draft a revived candidate runtime handoff flow from `packages/cephalon-ts/docs/runtime-handoff.md`

Implemented so far:
- `packages/cephalon-ts/src/contracts/event-envelope.ts`
- `packages/cephalon-cljs/src/promethean/contracts/event_envelope.cljs`
- `packages/cephalon-cljs/src/promethean/bridge/cephalon_ts.cljs` boundary-envelope bridge helpers
- `packages/cephalon-clj/src/promethean/contracts/event_envelope.clj`
- `packages/cephalon-clj/src/promethean/runtime/eventbus.clj` boundary-edge bus helpers

Implementation and adapter work still remain after the draft, especially for memory, tool, and runtime-state surfaces.

## Phase 3 — choose convergence lines

- [ ] decide whether the long-term live runtime is primarily TS with absorbed CLJS ideas, CLJS with TS service adapters, or a stricter mixed architecture
- [ ] extract shared doctrine and schemas into repo-level contracts instead of duplicating them per package
- [ ] choose where the canonical implementation of eidolon, nexus, and prompt-field logic should live
- [ ] choose whether the control plane belongs inside the TS runtime only or as a cross-package contract

## Phase 4 — package-family polish

- [ ] add one root matrix relating Cephalon to `graph-runtime`, `graph-weaver`, `myrmex`, `daimoi`, and `simulacron`
- [ ] add reproducible dev profiles for `duck`, `openhax`, `openskull`, and `error`
- [ ] define which repo-local docs are doctrine, which are historical, and which are implementation notes
- [ ] add small architecture diagrams for the TS runtime, CLJS ECS loop, and recovered two-process branch

## Sharp warnings

Bad convergence moves would be:
- deleting the CLJS note corpus because the TS runtime is more runnable
- pretending the recovered CLJ archive is a runnable package
- keeping both `src/app.ts` and `src/main.ts` as equal truths without naming the split
- flattening the family into one language before the shared contracts are explicit

The dossier already tells us something important:
- TS currently speaks best
- CLJS still thinks best in architecture
- JVM CLJ still explains the small loop best
- recovered CLJ still remembers a distributed topology the others keep rediscovering
