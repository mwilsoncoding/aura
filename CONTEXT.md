# Aura Context

Aura is an exploratory language and compiler project. This glossary defines Aura-specific language and runtime concepts; [README.md](README.md) remains the source of project goals and non-goals.

## Processes and Dispatch

**Aura-process**:
An isolated actor that owns process-local state and a mailbox, and communicates with other Aura-processes by messages. It is a language-level actor, not an operating-system process or thread.
_Avoid_: OS process, thread

**Message**:
An immutable value sent to an Aura-process mailbox. Selective receive removes the earliest queued message matching a pattern and guard; unmatched messages remain available for later receives.

**Behaviour**:
A message-based dispatch contract implemented by actor modules. The message type is the type of the first argument to its declared functions.

**Protocol**:
A value-based dispatch contract implemented by struct modules. The struct type is the type of the first argument to its declared functions.

**Interface**:
A module-based dispatch contract that any module can implement. Its functions have no restriction on the type of their first argument.

**Struct**:
A named record type with nominal identity and declared fields. Struct values may also carry additional map fields; struct enumeration covers only declared fields.

## Values and Types

**Atom**:
An immutable named scalar value; each literal, including `nil`, denotes a singleton value.

**Tuple**:
An immutable positional value with fixed arity.

**List**:
An immutable, proper, ordered sequence of values.

**Ordinary value**:
A value with immutable, value-based semantics and no observable reference identity. Equality and program-visible behavior depend on the value, not its storage identity.
_Avoid_: pointer identity, object identity

**Record**:
A structural fixed-label map shape with required or optional fields and an open or closed row. Extra fields may satisfy an open record shape.

**Map**:
An arbitrary-key mapping described by key and value types, distinct from a fixed-label structural record.

**Option**:
A tagged value representing presence or absence. It is distinct from the `nil` atom and from whether a map key is absent.

**Result**:
An ordinary value representing success or recoverable failure. Returning a failure value does not itself fail or terminate an Aura-process.

## Pattern Matching

**Tuple pattern**:
A positional pattern that matches a tuple of the specified arity.

**List pattern**:
A structural pattern over a proper list that observes its ordered elements.

**Map pattern**:
A pattern that requires its specified keys and permits additional map entries.

**Record pattern**:
A pattern over a structural record; the row determines whether unspecified fields are permitted.

**Struct pattern**:
A pattern that requires the nominal struct type and matches declared fields. Additional map fields do not affect the match.

## Effects and Memory

**Effect**:
A named group of typed operations. Function effect rows track the unhandled operation labels a computation may perform; an unhandled operation propagates until a handler handles it.

**Effect handler**:
A scoped interpretation of selected effect operations. Aura handlers are deep and one-shot: a resumed computation re-enters its handler, and an operation the handler does not handle propagates outward.

**State**:
Mutable state available through an explicit effect and local to one Aura-process. Aura-processes do not share mutable state.

## Source and Execution

**Script file**:
An Aura source file intended for direct execution without requiring a `main` entry point.