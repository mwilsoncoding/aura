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
- Perceus reference counting
- Koka-style effect handling system
- BEAM / OTP / Actor model style concurrency
- Behaviours
  - Message-based dispatch contracts
  - Modules that encode actors (which respond to messages) can implement behaviours
- Transactional memory
- Memory safety
- JIT compilation
- REPL
- Protocols
  - Value-based dispatch contracts
  - Struct modules (modules that define structs) can implement protocols
- Interfaces
  - Module-based dispatch contracts
  - All modules can implement interfaces
- Units of measure
- Support for scripting in-the-language
- Parallel compilation
- Non-blocking IO
- Pipe operator support
- Functional syntax
- Elixir-like set-theoretical types

### Stretch Goals

- BEAM
- CLR
- JVM
- LLVM
- WASM

## Non-Goals

- Backward compatibility (yet)
- Market penetration
- Production support (in any capacity, whatsoever)
