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

**Keyword list**:
An ordered list of two-element tuples whose first element is an atom. Illustrative type descriptions include `[{atom, dynamic}]` and `list(tuple(atom, dynamic))`; no particular spelling or dedicated type has been decided, and the API remains open.

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

## Time

**System clock**:
A clock whose readings correspond to externally meaningful wall time. Its readings may move forward or backward when the system clock is adjusted.

**Monotonic clock**:
A clock whose readings do not move backward and are used to measure elapsed time or deadlines. Its readings have no civil-time origin.

**Clock effect**:
A typed effect through which a computation requests a system or monotonic instant; clock reads return the instant directly. Recoverable time-zone database and conversion errors are returned as `Result` values.

**System instant**:
An absolute system-clock point represented as an arbitrary-precision integer nanosecond count from the Unix epoch using POSIX time; leap seconds are not represented as distinct instants. It can be compared or subtracted only with another system instant; a named time zone is separate data.

**Monotonic instant**:
A runtime-local monotonic-clock point represented as an arbitrary-precision integer nanosecond count, used for elapsed time and deadlines rather than civil timestamps. It can be compared or subtracted only with another monotonic instant from the same runtime and is not portable across runtimes.

**Duration**:
A signed fixed span of elapsed time represented as an arbitrary-precision integer nanosecond count; this precision does not promise nanosecond clock accuracy. Adding a duration to an instant preserves its clock domain; subtracting same-domain instants yields a duration.

**Calendar period**:
A span of calendar units applied to a civil date-time in an explicit time zone, rather than a fixed elapsed-time span. Month/year shifts clamp an invalid day to the target month's last valid day; applying a period can encounter a gap or fold and therefore yields a local-time resolution.

**Time zone**:
A named set of civil-time rules that maps local date-times to offsets or instants over time. A fixed UTC offset alone is not a time zone.

**Time-zone database**:
A provider of transition rules for named time zones, passed explicitly to conversions and calendar operations. When omitted through a default argument, the provider is UTC-only; its implementation and non-UTC data source remain open.

**Civil date-time**:
A local calendar date and clock time that does not identify an instant without an explicit time zone and daylight-saving resolution.

**Local-time resolution**:
The result of interpreting a civil date-time in a named time zone: a unique instant, an ordered pair of earlier/later instants for a fold, or a gap with adjacent valid local date-times. Time-zone database failures are reported separately as `Result` values, and conversion does not silently choose an instant.

**Virtual-time test handler**:
A test handler that supplies clock readings and coordinates scheduler timers with virtual monotonic time. Advancing virtual time advances both clock domains by the same amount by default and fires due timers; tests may move system time independently to simulate clock corrections.

## Source and Execution

**Script file**:
An Aura source file intended for direct execution without requiring a `main` entry point.