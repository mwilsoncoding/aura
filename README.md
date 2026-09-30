# Aura

The BEAM, if it were reference counted and had an effect system.

An idea borne out of the following key influences: 

- Erlang BEAM / OTP
- Perceus reference counting
- Koka effect handling
- Verse

## Goals

All goals are planned, but feasibility may vary per goal. This list may change over time.

- Having fun
- Exploring
- Compiler
- Deep modules applied to language design: the language constructs and features should be deep.
- Modules
- Functions
- Args
- Compile-time macros
- Build system
- Perceus reference counting
- BEAM / OTP / Actor model style concurrency
  - actors are aura-processes
  - aura-processes have their own stack, heap, and message mailbox
  - actors have isolated memory and must communicate via message passing
  - Cooperative multi-threading via custom scheduling algorithm
- Koka-style effect handling system
  - most aura programs will be written as cooperative aura-processes that run in supervision trees as handled effects
- Behaviours
  - Message-based dispatch contracts
  - Modules that encode actors (which respond to messages) can implement behaviours
  - Interfaces centered around message passing / handling
  - The message datatype is the type of the first argument to the functions declared in the interface
- Protocols
  - Value-based dispatch contracts
  - Struct modules (modules that define structs) can implement protocols
  - Interfaces centered around data types
  - The struct is the type of the first argument to the functions declared in the interface
- Interfaces
  - Module-based dispatch contracts
  - All modules can implement interfaces
  - Interfaces centered around modules
  - There is no restriction on the type of the first argument of functions declared in this type of interface
- Transactional memory
- Memory safety
- JIT compilation
- REPL
- Lightweight builds
- Statically linked binaries
- Units of measure
- Support for scripting in-the-language
- Parallel compilation
- Modules
- Non-blocking IO
- Pipe operator support
- Functional syntax
- Elixir-like set-theoretical types
- Lexical scoping

### Stretch Goals

- BEAM
- CLR
- JVM
- LLVM
- WASM
- Multi-arch

## Non-Goals

- Backward compatibility (yet)
- Market penetration
- Production support (in any capacity, whatsoever)
