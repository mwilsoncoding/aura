# Transactional Memory Without Shared Actor State

Research resolution for [issue 46](https://github.com/mwilsoncoding/aura/issues/46), reviewed 2026-09-29. This report informs the scope question; it does not decide whether Aura should keep transactional memory or recommend an implementation.

## Findings

- With process-private mutable state and one active executor per process, STM's isolation between Aura-processes protects no shared actor state. Its local all-or-nothing update, exception, and transaction-composition semantics can still be meaningful. State plus a result/effect policy can also express rollback; STM's distinct feature is the built-in transaction and wait/choice model, not an automatic increase in expressiveness.
- GHC's `retry` is not a general pause: it waits for a `TVar` read by the transaction to change. If no concurrently runnable local or runtime writer can change that variable, there is no wake-up source. Aura's process-local concurrency and message-wakeup rules are not specified well enough to decide whether this condition holds.
- Runtime-private STM is technically separable from actor-visible state, but that alone does not prove observational equivalence or better performance. Scheduler ordering, wake-up, and fairness can be visible to actors. GHC and BEAM/OTP scheduler sources show established owner/lock/atomic approaches; they are examples, not evidence that those mechanisms are faster for Aura.
- Perceus makes transaction-log ownership a specific open constraint: a log that retains old and speculative values may affect uniqueness and reuse. The Perceus paper and the examined STM implementations do not establish whether, or at what cost, such an integration works.

## Scope and Assumptions

Aura's README describes each Aura-process as owning a stack, heap, and mailbox, with isolated memory, message passing, and cooperative scheduling; it lists transactional memory as a goal. Issue 46 states the no-shared-mutable-actor-state premise. Neither source specifies whether multiple local tasks can access one process's state concurrently, whether STM may yield, or how a retry would be awakened. Conclusions below that assume one active executor are conditional on that model, not a settled Aura fact. ([README at the research base commit][aura-readme]; [issue 46][issue])

The source facts below are separated from analysis. GHC STM and Clojure STM are concrete, different designs; their exact retry, exception, and conflict semantics should not be read as Aura requirements.

## 1. Actor-Local Semantics

### Source-established behavior

Harris, Marlow, and Peyton Jones present STM as a compositional concurrency model, specifically describing modular blocking and choice in addition to the familiar transaction properties. The paper is relevant evidence for composition, not a claim that all STM systems have identical semantics. ([Publisher record and abstract][harris]; [paper PDF][harris-pdf])

In GHC 9.12.2, `atomically` executes an `STM` action; `retry` abandons the current attempt and may block until a `TVar` read by that attempt is updated. `orElse` tries its second action if the first retries, and the combined action retries if both do. An uncaught `throwSTM` aborts the transaction and rolls back its STM changes; when caught with `catchSTM`, the documented rollback is limited to changes in the caught computation. ([GHC 9.12.2 `Sync.hs`][ghc-sync]) The GHC RTS describes a transaction record (`TRec`) with an entry per accessed `TVar`, including expected and proposed values; nested records allow a nested transaction to abort without aborting its enclosing transaction. Commit validates the values read, and waiters are linked to the TVars on which they depend. ([GHC 9.12.2 `STM.c`][ghc-stmc]; [GHC 9.12.2 `STM.h`][ghc-stmh])

There is a version-specific exception caveat: `stm` 2.5.3.1 contains a note that `newTVar` allocation is not rolled back on `throwSTM`, but that note is in a legacy `base < 4.3` compatibility branch. GHC 9.12.2's current `throwSTM` documentation describes rollback of the uncaught STM transaction without repeating that allocation note. This report therefore does not generalize allocation rollback semantics beyond the cited version and operation. ([`stm` 2.5.3.1 source][stm-exception]; [GHC 9.12.2 `Sync.hs`][ghc-sync])

Clojure documents a different STM model: `dosync` transactions over shared Refs are atomic and automatically retried after conflicts; its STM uses MVCC/history, and the documentation warns that side effects and I/O are unsafe in retryable transactions. Its implementation records a read point, locks references for writes, establishes a commit point, and catches its internal retry signal to rerun the transaction. This is evidence about Clojure's shared-Ref design, not a universal STM contract. ([Clojure Refs and Transactions][clojure-docs]; [Clojure 1.12.0 `LockingTransaction.java`][clojure-stm])

### Analysis under Aura's constraint

If an Aura-process has one active executor and its mutable state is inaccessible to other processes, inter-process isolation and conflict detection over that state have no competing actor access to control. That makes the isolation part observationally redundant under this assumption; it does not remove the semantics of grouping local writes, discarding them on transaction abort, or composing actions that may retry. Those semantics could still help express an invariant across several local updates or ensure a failed message handler does not leave partial state. Whether that is more convenient or valuable than other Aura constructs is not established by the sources.

The alternatives are capable of expressing comparable local policies, but do not supply the same behavior automatically:

| Construct | Source-established shape | Analysis for process-local state |
| --- | --- | --- |
| State effect | `StateT s m a` threads state as `s -> m (a, s)`. [Haskell `transformers` 0.6.1.1][state-t] | Sequencing local state is direct; whether failure preserves or discards the candidate state depends on the surrounding effect/handler semantics. |
| Result value | `ExceptT e m a` wraps `m (Either e a)`. [Haskell `transformers` 0.6.1.1][except-t] | A pure update can return `(new_state, value)` only on success, with the caller installing it only for `Right`; a result value alone neither mutates state nor waits for a state change. |
| Effect handler | Koka's specification identifies effect typing and effect handlers as language concepts. [Koka specification at pinned revision][koka-spec] | A handler can define local state, failure, and suspension policy, but handlers do not imply transactional isolation or rollback unless those semantics are explicitly implemented by the handler. |
| STM | `atomically`, abort, `retry`, and `orElse` have the GHC semantics above. [GHC STM primitives][ghc-sync] | Supplies a first-class transaction boundary and read-set-based waiting/choice; its conflict-isolation benefit depends on actual concurrent access. |

The transformer types make failure policy concrete: `StateT s (Either e) a` has shape `s -> Either e (a, s)`, so a `Left` exposes no final state; `ExceptT e (State s) a` has shape `s -> (Either e a, s)`, so it can return state alongside a `Left`. This is an example of selectable semantics in a functional encoding, not a claim about Aura's not-yet-defined State effect. ([`StateT` source][state-t]; [`ExceptT` source][except-t])

`retry` is the sharpest distinction. Under the one-executor assumption, a transaction that retries gives up its current attempt; its own uncommitted writes cannot wake it. With no other local executor and no runtime event changing a read TVar, it can remain blocked indefinitely. If Aura permits concurrent local tasks, or a runtime-owned signal updates a watched variable, that conclusion changes. Receiving a message does not by itself wake STM unless the message path is integrated with the transaction's watched state. ([GHC `retry` documentation][ghc-sync]; [GHC STM wait contract][ghc-stmh])

## 2. Runtime-Internal Concurrent State

### Source-established behavior

GHC's scheduler gives the task holding a `Capability` exclusive access to that capability's run queue, and comments that the common scheduler path is lock-free. A `Mutex` on the capability protects other shared fields, including the running task and inbox. ([GHC 9.12.2 `Capability.h`][ghc-capability]) BEAM/OTP 28.5.0.7 uses a per-run-queue mutex and atomic operations for run-queue flags; its run-queue traversal macro locks and unlocks each queue. ([OTP `erl_process.h`][otp-process-h]; [OTP `erl_process.c`][otp-process-c]) These sources show concrete implementations, not comparative benchmark results.

GHC's STM source separately describes per-TVar transaction records and validation. It also documents an `STM_UNIPROC` mode whose caller serializes invocations, suitable only for a non-threaded RTS build. This demonstrates that GHC can run its STM interface with serialized callers; it does not establish a benefit for Aura's actor-local case. ([GHC 9.12.2 `STM.c`][ghc-stmc])

### Analysis and constraints

STM could be used privately for runtime invariants spanning several scheduler fields without exposing transactional objects to Aura programs. That can leave the language-level state model unchanged only if the runtime preserves the externally observable scheduling contract. Queue choice, message-processing order, wake-up, and fairness may be actor-visible; Aura's README does not yet define those guarantees, so equivalence cannot be demonstrated from the available sources. This is a possibility, not an argument to adopt it.

Compared with that option, a lock can protect a multi-field critical section, while atomics can handle narrower flag/state transitions; ownership-local queues can avoid shared mutation on a common path. GHC and OTP demonstrate these mechanisms in their scheduler implementations. STM offers a compositional commit/abort model across multiple transactional variables, but the GHC implementation shows the accompanying per-access log, validation, locking, allocation, and waiter bookkeeping. Clojure's documentation also makes clear that transaction bodies may be rerun. These are concrete integration and work costs, not evidence of a net performance result for Aura. ([GHC `STM.c`][ghc-stmc]; [GHC scheduler][ghc-capability]; [OTP run queues][otp-process-h]; [Clojure STM docs][clojure-docs])

Retry and retryable work also interact with scheduler progress: a blocked transaction needs a writer to a watched variable, and a runtime transaction may run again after a conflict. A scheduler must not lose the ability to perform that wake-up or accidentally repeat non-transactional effects such as I/O or message sends. Clojure explicitly warns about repeated side effects; GHC's API confines ordinary actions to `STM` and provides an explicitly unsafe I/O escape. The particular lock ordering, blocking rules, and scheduler integration Aura would need are not specified by these examples. ([Clojure STM docs][clojure-docs]; [GHC STM primitives][ghc-sync]; [GHC STM wait contract][ghc-stmh])

### Perceus interaction

The Perceus extended paper describes precise reference counting and reuse analysis that can enable guaranteed in-place updates when reuse is safe; it reports an implementation in Koka. ([Perceus paper and versioned PDF][perceus]) GHC's STM transaction record, by contrast, keeps expected and proposed value pointers while a transaction is active, and Clojure's documented design favors persistent values for speculative updates. ([GHC `STM.c`][ghc-stmc]; [Clojure STM docs][clojure-docs])

**Analysis, not published compatibility evidence:** if an Aura transaction log retains Perceus-managed old and proposed values, those references must participate in ownership accounting and preserve the old value for abort. Such retained references could prevent a uniqueness proof from permitting in-place reuse until commit/abort, or require transaction-aware ownership handling. The exact effect depends on what runtime values are logged and how Aura's ownership model works. None of the cited STM implementations uses Perceus, and the Perceus paper does not study STM; these sources establish neither incompatibility nor a performance penalty for Aura.

## What the Sources Do Not Decide

- Whether Aura-processes have one executor or can run multiple local tasks concurrently; whether transactions may yield; and how mailbox delivery, effects, and `retry` would interact.
- Whether local rollback/composition would be more useful or maintainable than a State effect, explicit result values, or a specified effect handler in Aura.
- Whether runtime-only STM can preserve Aura's eventual scheduling/fairness/message-order contract, or how its throughput and latency compare with locks, atomics, or ownership-local structures.
- Whether a Perceus-aware transaction log can be correct and efficient, especially if it retains Aura heap objects. No measured Aura implementation or comparable benchmark is available in these sources.

The ACM endpoint for Shavit and Touitou's 1995 paper is not accessible from this research environment (HTTP 403); its title, year, and DOI are included below for bibliographic context only, and no detailed claim here relies on it. The publisher record for Harris et al. was accessible, but no PDF text extractor is installed; claims about that paper are limited to its publisher abstract's statements about composition, blocking, and choice. Detailed formal results or implementation claims from the full text were not independently verified. ([Shavit and Touitou, 1995][shavit]; [Harris et al.][harris])

## Sources and Revisions

- [Harris, Marlow, and Peyton Jones, "Composable Memory Transactions," PPoPP 2005][harris] ([publisher PDF][harris-pdf]).
- [Shavit and Touitou, "Software Transactional Memory," PODC 1995, DOI 10.1145/224964.224987][shavit].
- GHC 9.12.2 release source at commit `71791bc3284756a960a3367afa2c0aef07f09353`: [`Sync.hs`][ghc-sync], [`STM.c`][ghc-stmc], [`STM.h`][ghc-stmh], and [`Capability.h`][ghc-capability].
- Haskell `stm` 2.5.3.1 at commit [`ff8f8ceeceb14ac59accd53dd82a5d32c7e08626`][stm-exception].
- Haskell `transformers` 0.6.1.1 at commit [`88f5db9f559960c7f45ebb8bdfa8ba1201a67384`][state-t].
- Clojure 1.12.0 at commit [`d4bb93f0d1ab2004f89c6ead1b32449fd7ed1a6d`][clojure-stm], plus the [official Refs and Transactions guide][clojure-docs].
- Erlang/OTP 28.5.0.7 at commit [`09fb046150ab94104e03ee816b95c8b7e6b04683`][otp-process-h].
- Koka specification source at commit [`8b803da69d3941ce1e03173578093aefc41ee548`][koka-spec].
- Perceus, MSR-TR-2020-42, extended version v4 (2021-06-07), [publisher record and PDF][perceus].
- Aura premise: [`README.md` at base commit `36baa4e07bdcf3449d4f0936979b2909dbb66308`][aura-readme]; [issue 46][issue].

[aura-readme]: https://github.com/mwilsoncoding/aura/blob/36baa4e07bdcf3449d4f0936979b2909dbb66308/README.md#L25-L31
[issue]: https://github.com/mwilsoncoding/aura/issues/46
[harris]: https://www.microsoft.com/en-us/research/publication/composable-memory-transactions/
[harris-pdf]: https://www.microsoft.com/en-us/research/wp-content/uploads/2005/01/2005-ppopp-composable.pdf
[shavit]: https://doi.org/10.1145/224964.224987
[ghc-sync]: https://gitlab.haskell.org/ghc/ghc/-/blob/ghc-9.12.2-release/libraries/ghc-internal/src/GHC/Internal/Conc/Sync.hs#L781-810
[ghc-stmc]: https://gitlab.haskell.org/ghc/ghc/-/blob/ghc-9.12.2-release/rts/STM.c#L8-72
[ghc-stmh]: https://gitlab.haskell.org/ghc/ghc/-/blob/ghc-9.12.2-release/rts/STM.h#L126-164
[ghc-capability]: https://gitlab.haskell.org/ghc/ghc/-/blob/ghc-9.12.2-release/rts/Capability.h#L66-87
[stm-exception]: https://github.com/haskell/stm/blob/ff8f8ceeceb14ac59accd53dd82a5d32c7e08626/Control/Monad/STM.hs#L106-135
[state-t]: https://github.com/haskell/transformers/blob/88f5db9f559960c7f45ebb8bdfa8ba1201a67384/Control/Monad/Trans/State/Strict.hs#L163-174
[except-t]: https://github.com/haskell/transformers/blob/88f5db9f559960c7f45ebb8bdfa8ba1201a67384/Control/Monad/Trans/Except.hs#L128-169
[clojure-docs]: https://clojure.org/reference/refs
[clojure-stm]: https://github.com/clojure/clojure/blob/d4bb93f0d1ab2004f89c6ead1b32449fd7ed1a6d/src/jvm/clojure/lang/LockingTransaction.java#L223-389
[koka-spec]: https://github.com/koka-lang/koka/blob/8b803da69d3941ce1e03173578093aefc41ee548/doc/spec/book.kk.md#L66-L75
[otp-process-h]: https://github.com/erlang/otp/blob/09fb046150ab94104e03ee816b95c8b7e6b04683/erts/emulator/beam/erl_process.h#L473-475
[otp-process-c]: https://github.com/erlang/otp/blob/09fb046150ab94104e03ee816b95c8b7e6b04683/erts/emulator/beam/erl_process.c#L521-531
[perceus]: https://www.microsoft.com/en-us/research/publication/perceus-garbage-free-reference-counting-with-reuse/