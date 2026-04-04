# Cephalon ClojureScript Implementation

> Canonical home: `orgs/octave-commons/cephalon/packages/cephalon-cljs`

A ClojureScript implementation of the Cephalon "always-running mind" based on the specification in `docs/notes/cephalon/`.

## Reading order

1. `../../README.md`
2. `../../docs/INDEX.md`
3. `docs/INDEX.md`
4. `specs/ecs-runtime-and-effects.md`
5. `docs/notes/cephalon/cephalon-mvp-spec.md`
6. `docs/notes/cephalon/cephalon-nexus-index-v01.md`
7. `docs/notes/cephalon/cephalon-daimoi-v01.md`

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Cephalon Runtime                             │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────────────────┐   │
│  │ ECS World   │──▶│ Event Bus   │──▶│ Session Manager         │   │
│  │ (tick loop) │   │ (in/out)    │   │ [related, persistent,   │   │
│  └─────────────┘   └─────────────┘    │  recent] assembly       │   │
│         │                               └─────────────────────────┘   │
│         │                                       │                     │
│         ▼                                       ▼                     │
│  ┌─────────────┐                       ┌─────────────────────────┐   │
│  │ Systems     │                       │ LLM Provider            │   │
│  │ - Route     │                       │ (qwen3-vl-2b)           │   │
│  │ - Cephalon  │                       └─────────────────────────┘   │
│  │ - Sentinel  │                                               │      │
│  └─────────────┘                                               │      │
│                                                                ▼      │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │                        Memory Store                           │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │   │
│  │  │ Memories │  │ Events   │  │ Vectors  │  │ Nexus Index  │  │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘  │   │
│  └───────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
orgs/octave-commons/cephalon/packages/cephalon-cljs/
├── deps.edn              ;; Clojure dependencies
├── shadow-cljs.edn       ;; Shadow-CLJS build config
├── package.json          ;; Node.js package config
├── externs.js            ;; JS externs for compilation
│
├── src/promethean/
│   ├── main.cljs              ;; Entry point + tick loop
│   │
│   ├── ecs/
│   │   ├── world.cljs         ;; ECS World structure
│   │   └── tick.cljs          ;; System execution
│   │
│   ├── event/
│   │   └── types.cljs         ;; Event type definitions
│   │
│   ├── memory/
│   │   └── types.cljs         ;; Memory schema (from spec)
│   │
│   ├── context/
│   │   └── assembler.cljs     ;; [related, persistent, recent] assembly
│   │
│   ├── sessions/
│   │   └── types.cljs         ;; Session types
│   │
│   ├── normalization/
│   │   └── discord_message.cljs ;; Message normalization + SimHash
│   │
│   ├── policy/
│   │   ├── types.cljs         ;; Policy types (from spec section 7)
│   │   └── loader.cljs        ;; EDN policy loader
│   │
│   └── debug/
│       └── log.cljs           ;; Logging utilities
│
└── test/promethean/
    └── (tests)
```

## Key Components

### ECS World (`ecs/world.cljs`)

The core data structure for entity-component-system architecture:

```clojure
{:tick 0                      ;; current tick number
 :time-ms 0                   ;; current time in ms
 :entities {eid {...}}        ;; entity map
 :events-in []                ;; incoming events
 :events-out []               ;; emitted events
 :effects []                  ;; side effects queue
 :env {:config {}             ;; configuration
       :clients {}            ;; LLM, Discord clients
       :adapters {}}          ;; FS, persistence adapters}}
```

### Memory Schema (from `cephalon-mvp-spec.md`)

Canonical memory record supporting all memory kinds:

```clojure
{:memory/id "uuid"
 :memory/timestamp 0
 :memory/cephalon-id "Duck"
 :memory/session-id "janitor"
 :memory/event-id "uuid"
 :memory/role "user|assistant|system|developer|tool"
 :memory/kind "message|tool_call|tool_result|think|image|summary|admin|aggregate"
 :memory/content {:text "" :normalized-text "" :snippets []}
 :memory/source {:type "discord|cli|timer|system|admin|sensor" :channel-id "" :author-id ""}
 :memory/retrieval {:pinned false :locked-by-admin false :locked-by-system false :weight-kind 1.0}
 :memory/usage {:included-count-total 0 :included-count-decay 0.0 :last-included-at 0}
 :memory/embedding {:status "none|ready|stale|deleted" :model "" :vector-id "" :vector []}
 :memory/lifecycle {:deleted false :deleted-at 0 :replaced-by-summary-id ""}
 :memory/schema-version 1}
```

### Context Assembly (`context/assembler.cljs`)

Assembles messages in the order specified by the MVP spec:

1. `system` (hard-locked)
2. `developer` (contract)
3. `system` (session personality)
4. **persistent** (pinned memories)
5. **related** (retrieved, scored)
6. **recent** (last N events)
7. `user` (current input)

Token budgets (from spec section 2):
- system+developer: 6%
- persistent: 8%
- recent: 18%
- related: 42% (min 1.6× recent)
- safety: 3%

### Session Management

Cephalons have multiple sessions (facets/aspects):

```clojure
{:session/id "uuid"
 :session/cephalon-id "Duck"
 :session/name "janitor"
 :session/priority-class :interactive|:operational|:maintenance
 :session/credits 100
 :session/recent-buffer []
 :session/subscriptions {:hard-locked true :filters [{:event/type :discord.message/new :discord/channel-id "..."}]}
 :session/status :idle|:ready|:blocked
 :session/queue []}
```

### Forced Discord Channels (from spec section 6.2)

| Channel | ID | Embed Raw | Embed Aggregates |
|---------|-----|-----------|------------------|
| bots | 343299242963763200 | false | true |
| duck-bots | 450688080542695436 | true | true |
| general | 343179912196128792 | false | true |
| memes | 367156652140658699 | false | true |

## Building and Running

```bash
# Install dependencies
cd orgs/octave-commons/cephalon/packages/cephalon-cljs
npm install

# Build
npm run build

# Test
npm test
```

## Running the TS bridge locally as OpenHax

If you want the TypeScript Cephalon bridge to run all 8 circuits using the OpenHax bot identity locally, set:

```bash
CEPHALON_TS_BRIDGE=true CEPHALON_BOT_ID=openhax OPENHAX_DISCORD_TOKEN=xxx npm run build && node dist/cephalon.js
```

`CEPHALON_BOT_ID=openhax` switches token resolution to `OPENHAX_DISCORD_TOKEN` while keeping the shared eight-circuit runtime active.

To run those same 8 circuits on the personal model routed through `proxx`, set:

```bash
CEPHALON_TS_BRIDGE=true \
CEPHALON_BOT_ID=openhax \
OPENHAX_DISCORD_TOKEN=xxx \
CEPHALON_MODEL=blongs-definately-legit-model \
OLLAMA_BASE_URL=http://127.0.0.1:8789 \
OLLAMA_API_KEY=${OPEN_HAX_OPENAI_PROXY_AUTH_TOKEN:-$PROXY_AUTH_TOKEN} \
npm run build && node dist/cephalon.js
```

`CEPHALON_MODEL=blongs-definately-legit-model` forces the shared eight-circuit TS runtime to use the personal model alias for every circuit.

## Reference Implementation

This implementation now lives alongside the TypeScript path in the canonical `orgs/octave-commons/cephalon/` repo and follows the ClojureScript/Shadow-CLJS architecture from `docs/notes/cephalon/brain-daemon-skeleton.md`.

## Specification Documents

- `docs/notes/cephalon/cephalon-mvp-spec.md` - Core model and MVP spec
- `docs/notes/cephalon/cephalon-concrete-specs.md` - Normalization, dedupe, schemas
- `docs/notes/cephalon/cephalon-storage-schema.md` - Storage layout
- `docs/notes/cephalon/cephalon-mvp-contracts.md` - Tool contracts, janitor session
- `docs/notes/cephalon/cephalon-context-assembly.md` - Context assembly algorithm
- `docs/notes/cephalon/cephalon-nexus-index-v01.md` - Nexus index design
- `docs/notes/cephalon/cephalon-embedding-scheduler-v01.md` - Embedding scheduler
- `docs/notes/cephalon/brain-daemon-skeleton.md` - Shadow-CLJS skeleton

## Next Steps

1. **Effects Runner** - Execute LLM, FS, Discord effects
2. **Memory Store** - MongoDB adapter for persistence
3. **Vector Store** - ChromaDB integration for embeddings
4. **Discord Integration** - Gateway connection and event ingestion
5. **Tool Validator** - JSON schema validation + repair loop
6. **Janitor Session** - Spam cleanup state machine
7. **Nexus Index** - Metadata graph for retrieval
8. **Daimoi Walkers** - Graph-walking retrieval algorithm
