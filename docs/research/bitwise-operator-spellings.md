# Bitwise operator spellings: Erlang and Elixir prior art

Research date: 2026-09-30. Worktree base: Aura `d88e4a4`.

This report gathers facts from first-party sources to help Aura choose spellings for bitwise operators that stay compatible with possible later syntax. It does **not** recommend a design. Section 7 contains labelled design inference; every other section states documented facts only.

## 0. Sources and revisions

All claims cite a commit-pinned GitHub permalink from the project that owns the claim. Line anchors refer to these revisions.

| Alias | Project | Revision | Base URL |
| --- | --- | --- | --- |
| `EX` | Elixir `main` | `ffdac87643303e781020ee02b5b25b64c4b936f4` | https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/ |
| `OTP` | Erlang/OTP `master` (`OTP_VERSION` = `30.0-rc0`) | `495ce0b626f24bc092196ffa46acc0f9f6ace625` | https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/ |
| `EX10` | Elixir tag `v1.0.0` | `52ff7e96867c027745d29f5d3feb77f546f22c4f` | https://github.com/elixir-lang/elixir/blob/52ff7e96867c027745d29f5d3feb77f546f22c4f/ |
| `RUST` | Rust Reference `master` | `a286e1ecb8ce193d50056e4bb042eea9b23876ef` | https://github.com/rust-lang/reference/blob/a286e1ecb8ce193d50056e4bb042eea9b23876ef/ |

Caveats:

- The `main`/`master` files were downloaded from the branch tips, and the SHAs were read immediately afterwards. A commit landing in that gap is theoretically possible but unlikely. Line anchors were checked against the downloaded copies.
- Elixir changelog citations use maintenance branches `v1.12` and `v1.14`, whose line numbers include later patch-release entries.
- Rendered documentation sites (`hexdocs.pm`, `erlang.org/doc`) were not fetched. The Markdown and Erlang sources they are generated from were read instead.

## 1. Erlang word operators (documented)

### 1.1 The set and its meaning

Erlang's reference manual lists six bitwise operators under "Arithmetic Expressions". All take integer arguments.

| Operator | Description | Arity |
| --- | --- | --- |
| `bnot` | Unary bitwise NOT | 1 |
| `band` | Bitwise AND | 2 |
| `bor` | Bitwise OR | 2 |
| `bxor` | Bitwise XOR | 2 |
| `bsl` | Bitshift left | 2 |
| `bsr` | Arithmetic bitshift right | 2 |

Source: [`OTP` system/doc/reference_manual/expressions.md#L925-L947](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/expressions.md#L925-L947). Worked examples are at [L966-L977](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/expressions.md#L966-L977). The `1 bsl (1 bsl 64)` example raises "a system limit has been reached".

### 1.2 Precedence and associativity

The Erlang precedence table, highest to lowest, has these rows relevant to bitwise operators ([expressions.md#L2342-L2360](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/expressions.md#L2342-L2360)):

1. Unary `+ - bnot not`
2. `/ * div rem band and`, left-associative
3. `+ - bor bxor bsl bsr or xor`, left-associative
4. `++ --`, right-associative
5. Comparison operators, non-associative

The grammar agrees:

- Terminals are declared in [erl_parse.yrl#L92-L94](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L92-L94).
- `bnot` is a `prefix_op` ([L623-L626](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L623-L626)).
- `band` is a `mult_op` ([L628-L633](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L628-L633)).
- `bor`, `bxor`, `bsl` and `bsr` are `add_op` ([L635-L642](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L635-L642)).

Consequence stated by the tables: `bsl`/`bsr` sit at the same level as `+`/`-`, and `band` at the same level as `*`.

### 1.3 Lexical status

- `band`, `bnot`, `bor`, `bsl`, `bsr` and `bxor` are reserved words in the scanner ([erl_scan.erl#L2291-L2318](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_scan.erl#L2291-L2318)).
- The parser also lists them in `reserved_word` productions ([erl_parse.yrl#L663-L669](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L663-L669)). These are used where a reserved word may appear as an atom in some contexts.

## 2. Elixir bitwise operators, current form (documented)

### 2.1 What exists now

- **Functions.** `Bitwise` defines `bnot/1`, `band/2`, `bor/2`, `bxor/2`, `bsl/2`, `bsr/2`. Each delegates to the `:erlang` function of the same name ([bitwise.ex#L62-L245](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L62-L245)).
- **Operators.** The moduledoc says only `band/2`, `bor/2`, `bsl/2`, `bsr/2` "also have operators": `&&&`, `|||`, `<<<`, `>>>` ([bitwise.ex#L9-L12](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L9-L12)). They are defined at [L104-L107](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L104-L107), [L140-L143](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L140-L143), [L216-L219](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L216-L219) and [L270-L273](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L270-L273).
- **No current documented symbolic operator for XOR or NOT.** The `~~~` (NOT) and `^^^` (XOR) definitions are `@doc false` ([L68-L71](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L68-L71), [L162-L165](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L162-L165)) and deprecated (section 3).
- **Guards and inlining.** All functions are allowed in guards and inlined by the compiler ([bitwise.ex#L14-L26](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L14-L26)). The guard list is also cross-referenced in the [patterns-and-guards page (L317)](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/patterns-and-guards.md#L317).
- **Import required.** Bitwise operators are not in `Kernel`. The operators reference says they are "used by the `Bitwise` module when imported" ([operators.md#L152](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L152)).
- **Word forms are ordinary functions in Elixir.** The reserved words are `true false nil when and or not in fn do end catch rescue after else` ([syntax-reference.md#L13-L19](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/syntax-reference.md#L13-L19)). `band`, `bor` etc. are not among them. `Bitwise` docs use them as ordinary calls, e.g. `bnot(2) &&& 3` ([bitwise.ex#L55-L59](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L55-L59)).

### 2.2 Elixir precedence for the symbolic forms

From the Elixir operator table ([operators.md#L72-L100](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L72-L100)), highest to lowest:

| Level | Operators |
| --- | --- |
| Unary | `+ - ! ^ not` |
| Left | `**` |
| Left | `* /` |
| Left | `+ -` |
| Right | `++ -- +++ --- .. <>` |
| Left | `in`, `not in` |
| Left | <code>\|> <<< >>> <<~ ~>> <~ ~> <~></code> (`<<<` and `>>>` share a level with `\|>`) |
| Left | `< > <= >=` |
| Left | `== != =~ === !==` |
| Left | `&&`, **`&&&`**, `and` |
| Left | `\|\|`, **`\|\|\|`**, `or` |
| Right | `=` |

The parser's `Left 120 or_op_eol`, `Left 130 and_op_eol` and `Left 160 arrow_op_eol` confirm this ([elixir_parser.yrl#L69-L97](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_parser.yrl#L69-L97)).

Derived from the table (not stated in prose): the bitwise `&&&` and `|||` share precedence levels with the boolean `&&`/`and` and `||`/`or`, both below the comparison operators. The shifts `<<<`/`>>>` share the pipe-operator level, below `+`.

## 3. Elixir history of the bitwise spellings (documented)

| Date | Event | Source |
| --- | --- | --- |
| 2012-07-06 | Commit "Add bitwise operators" introduces `Bitwise` with `use Bitwise` and options `:only_operators` and `:skip_operators`. The original skip list is `~~~ &&& \|\|\| ^^^ <<< >>>`. | [062813c8](https://github.com/elixir-lang/elixir/commit/062813c8f321d7b8532a72d14cd5db2ed8d80e6c) |
| v1.0.0 | `Bitwise` provides six word-named macros plus six operators: `~~~`, `&&&`, `\|\|\|`, `^^^`, `<<<`, `>>>`. Docs say `use Bitwise`. | [`EX10` bitwise.ex#L1-L128](https://github.com/elixir-lang/elixir/blob/52ff7e96867c027745d29f5d3feb77f546f22c4f/lib/elixir/lib/bitwise.ex#L1-L128) |
| 2019-11-13 | "Change Bitwise macros to rewrites" (inlined `:erlang` calls replace the older macro form). | [95871344](https://github.com/elixir-lang/elixir/commit/95871344297aa4f77f564b0cde88a16e4fc0fabe) |
| v1.12 (2021-03-23 commit) | `^^^` deprecated in favour of `bxor/2`. | [c9a171da](https://github.com/elixir-lang/elixir/commit/c9a171da5b25e0eb5d1da3b04c622f8b79a8aff4); [CHANGELOG v1.12 L291](https://github.com/elixir-lang/elixir/blob/v1.12/CHANGELOG.md#L291); [deprecations table L138](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/compatibility-and-deprecations.md#L138) |
| v1.14 (2021-12-09 commit) | `use Bitwise` deprecated in favour of `import Bitwise`. `~~~` deprecated in favour of `bnot`, "for clarity". | [f1b9d3e8](https://github.com/elixir-lang/elixir/commit/f1b9d3e818e5bebd44540f87be85979f24b9abfc); [CHANGELOG v1.14 L578-L579](https://github.com/elixir-lang/elixir/blob/v1.14/CHANGELOG.md#L578-L579); [deprecations table L126-L127](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/compatibility-and-deprecations.md#L126-L127) |

The tokenizer, as of the pinned `main` revision, still lexes both deprecated operators but warns:

- `~~~`: "~~~ is deprecated. Use Bitwise.bnot/1 instead for clarity" ([elixir_tokenizer.erl#L905-L910](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L905-L910)).
- `^^^`: "It is typically used as xor but it has the wrong precedence, use Bitwise.bxor/2 instead" ([L930-L934](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L930-L934)).
- Both are marked `TODO: Remove these deprecations on Elixir v2.0` (same lines).

The parser gave `^^^` its own precedence level (`Left 180 xor_op_eol`, [elixir_parser.yrl#L86](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_parser.yrl#L86)), and `~~~` is a unary operator at the `Nonassoc 300 unary_op_eol` level ([L93](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_parser.yrl#L93)). The `^^^` deprecation message identifies the precedence as the reason.

## 4. Established meanings of `&`, `^`, `~`

### 4.1 Elixir

| Token | Meaning in Elixir | Source |
| --- | --- | --- |
| `&` | Capture operator (special form, unary): `&fun/1`, `&(&1 * 2)`, `&1` placeholders. Listed as a special form that "cannot be overridden". Precedence: unary, just below `=`. | [special_forms.ex#L1785-L1852](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/kernel/special_forms.ex#L1785-L1852); [operators.md#L34-L39](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L34-L39); [operators.md#L94](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L94) |
| `&&` | Short-circuit boolean "and", not allowed in guards. | [kernel.ex#L4378](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/kernel.ex#L4378) |
| `&&&` | Bitwise AND (when `Bitwise` is imported). | [bitwise.ex#L104-L107](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L104-L107) |
| `^` | Pin operator (special form, unary): matches against an already-bound variable in a pattern. Cannot be overridden. | [special_forms.ex#L727-L757](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/kernel/special_forms.ex#L727-L757); [operators.md#L35](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L35) |
| `^^^` | Formerly bitwise XOR; deprecated (section 3). | see above |
| `~` | Sigil introducer (section 5). Also the first character of several parsed-but-unused operators (`~>`, `<~`, `~>>`, `<<~`, `<~>`) and of `=~`. | [operators.md#L87](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L87); [operators.md#L79](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L79) |
| `~~~` | Formerly bitwise NOT; deprecated (section 3). | see above |

### 4.2 Erlang

- The scanner produces `&&` and `&` tokens ([erl_scan.erl#L623-L626](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_scan.erl#L623-L626)), and `^` and `~` fallback tokens ([L764-L769](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_scan.erl#L764-L769)).
- The parser's terminal list ([erl_parse.yrl#L84-L99](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L84-L99)) includes `'&&'` but does not include `'&'`, `'^'` or `'~'`. So as of this revision, `&&` is the only one of those tokens used by the grammar.
- `&&` is used for zip generators in comprehensions: `[{P,Q} || P <:- [a,b,c] && Q <:- [1,2,3]]` ([expressions.md#L2058](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/expressions.md#L2058)). Zip generators and strict generators were introduced in OTP 28 ([L2006](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/expressions.md#L2006)).
- `~` introduces sigils since OTP 27 (section 5).
- Erlang has no symbolic bitwise operators. Bitwise operations are only the six word operators of section 1.
- The bit-syntax delimiters are `<<` and `>>` (terminals in [erl_parse.yrl#L98](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L98)).

### 4.3 Rust (comparison; Aura's `AGENTS.md` lists the Rust Reference as an upstream)

The Rust Reference ([operator-expr.md#L346-L347](https://github.com/rust-lang/reference/blob/a286e1ecb8ce193d50056e4bb042eea9b23876ef/src/expressions/operator-expr.md#L346-L347), [L391-L395](https://github.com/rust-lang/reference/blob/a286e1ecb8ce193d50056e4bb042eea9b23876ef/src/expressions/operator-expr.md#L391-L395)) defines:

- `!` is bitwise NOT on integers (logical NOT on booleans).
- Binary `&`, `|`, `^`, `<<`, `>>` are bitwise AND, OR, XOR, shift-left, shift-right, via `BitAnd`, `BitOr`, `BitXor`, `Shl`, `Shr`.
- Unary `&` is a borrow ([operator-expr.md#L58-L65](https://github.com/rust-lang/reference/blob/a286e1ecb8ce193d50056e4bb042eea9b23876ef/src/expressions/operator-expr.md#L58-L65)).
- Precedence, strong to weak: `+ -`, then `<< >>`, then `&`, then `^`, then `|`, then comparisons, then `&&`, then `||` ([expressions.md#L73-L94](https://github.com/rust-lang/reference/blob/a286e1ecb8ce193d50056e4bb042eea9b23876ef/src/expressions.md#L73-L94)).

## 5. Sigils and tokenizer/extension implications

### 5.1 Erlang sigils (OTP 27+)

- A sigil is a string-literal prefix: `~` followed by a sigil name, then the content between delimiters ([data_types.md#L613-L640](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/data_types.md#L613-L640)).
- Delimiters are `() [] {} <>`, and `/ | ' " ` #`, plus triple-quote forms.
- The defined sigils are `~` (vanilla), `~b`, `~B`, `~s`, `~S` ([L642-L665](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/data_types.md#L642-L665)). "Sigils were introduced in Erlang/OTP 27" ([L703](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/data_types.md#L703)).
- The scanner sends any `~` to `scan_sigil_prefix` ([erl_scan.erl#L635-L636](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_scan.erl#L635-L636)). The scanner comment says the sigil name set is enumerated later by `erl_parse:build_sigil/3` ([L1057-L1067](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_scan.erl#L1057-L1067)). Unknown prefixes cause "illegal sigil prefix" ([erl_parse.yrl#L1844-L1882](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L1844-L1882)).
- The grammar has `sigil -> sigil_prefix string sigil_suffix` ([erl_parse.yrl#L393](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/lib/stdlib/src/erl_parse.yrl#L393)).
- Sigils are not concatenated like adjacent strings ([data_types.md#L696-L701](https://github.com/erlang/otp/blob/495ce0b626f24bc092196ffa46acc0f9f6ace625/system/doc/reference_manual/data_types.md#L696-L701)).

### 5.2 Elixir sigils

- Sigils start with `~`, then one lowercase letter or one or more uppercase letters, then a delimiter, then optional modifiers ([sigils.md#L10](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/getting-started/sigils.md#L10)).
- They are user-extensible. `~x(...)` calls `sigil_x/2`. Custom names are a single lowercase letter, or an uppercase letter followed by uppercase letters and digits ([sigils.md#L227](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/getting-started/sigils.md#L227), [L241](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/getting-started/sigils.md#L241)). Built-ins include `sigil_S/s/C/c/r/R/D/T/N/U/w/W` ([kernel.ex#L6514-L6988](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/kernel.ex#L6514-L6988)).
- Tokenizer: the clause `tokenize([$~, H | _T], ...) when ?is_upcase(H) orelse ?is_downcase(H)` sends `~` plus an ASCII letter to `tokenize_sigil` ([elixir_tokenizer.erl#L217-L219](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L217-L219)). Name rules and the error message are at [L1735-L1759](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L1735-L1759). Sigil delimiters are `/ < " ' [ ( { |` ([elixir_tokenizer.hrl#L16-L17](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.hrl#L16-L17)).
- The sigil clause is placed before the operator clauses in the tokenizer's clause order. That order matters (section 5.3).

### 5.3 Tokenizer mechanics that constrain operator spellings (Elixir)

- **Longest-match by clause order.** Three-character operators are tried first ([elixir_tokenizer.erl#L385-L415](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L385-L415)), then the `<<` and `>>` bit-string delimiters ([L418-L424](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L418-L424)), then two-character operators, then one-character ones. `<<<` and `>>>` are in `arrow_op3` ([L42-L47](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L42-L47)) and so take priority over `<<`/`>>`.
- **Repeated-character warning.** For `&&&`, `|||`, `^^^` and `+++`/`---`, the tokenizer warns "found X followed by Y, please use a space between X and the next Y" when the operator is followed by the same character; there is a `TODO: Turn into an error on v2.0` ([L1874-L1881](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L1874-L1881)).
- **Operator as function name.** After an operator token, a following `/` makes the tokenizer emit an identifier, so `&&&/2` works as a function reference ([L894-L943](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L894-L943)). An operator followed by `: ` becomes a keyword identifier, e.g. `[&&&: 2]` in `Bitwise.__using__` ([bitwise.ex#L36-L37](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L36-L37)).
- **`&` special cases.** `&` followed by a digit becomes a `capture_int`; `& /` sequences are handled specially ([L483-L498](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L483-L498)).
- **`<<<<<<<` at column 1** is a version-control-conflict error ([L183-L186](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/src/elixir_tokenizer.erl#L183-L186)).
- **Fixed operator vocabulary.** "It's not possible to define new operators." Elixir parses a predefined set. Users may define or override functions using those spellings ([operators.md#L111-L150](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L111-L150)). The parsed-but-unused set is <code>\|\|\| &&& <<< >>> <<~ ~>> <~ ~> <~> +++ --- ...</code>. Of these, `|||`, `&&&`, `<<<` and `>>>` are used by `Bitwise` ([L152](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L152)). The docs note the community "generally discourages custom operators" ([L154](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L154)).

### 5.4 Set-theoretic types use word operators in Elixir

Elixir's gradual set-theoretic type system writes unions as `or`, intersections as `and` and negations as `not`, e.g. `atom() or integer()` ([gradual-set-theoretic-types.md#L48-L50](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/gradual-set-theoretic-types.md#L48-L50)). Aura's README lists "Elixir-like set-theoretical types" as a goal ([README.md](../../README.md)).

## 6. Documented status of candidate spellings

This table lists documented facts about each spelling in Erlang, Elixir and Rust, with no ranking.

| Operation | Erlang | Elixir now | Elixir history | Rust |
| --- | --- | --- | --- | --- |
| AND | `band` (reserved word, `*` level) | `band/2`, `&&&` | `&&&` since 2012 | `&` |
| OR | `bor` (reserved word, `+` level) | `bor/2`, `\|\|\|` | `\|\|\|` since 2012 | `\|` |
| XOR | `bxor` (reserved word, `+` level) | `bxor/2` only | `^^^` 2012 to deprecated 1.12 | `^` |
| NOT | `bnot` (reserved word, unary) | `bnot/1` only | `~~~` 2012 to deprecated 1.14 | `!` |
| Shift left | `bsl` (`+` level) | `bsl/2`, `<<<` | `<<<` since 2012 | `<<` |
| Shift right (arithmetic) | `bsr` (`+` level) | `bsr/2`, `>>>` | `>>>` since 2012 | `>>` |

Documented status of the single-character tokens across Elixir/Erlang:

- `&`: Elixir capture, unary special form. Erlang scanner token, unused by the grammar (only `&&` is used, for zip generators).
- `^`: Elixir pin, unary special form. Erlang scanner token, unused by the grammar.
- `~`: Sigil prefix in both languages. Elixir also uses it inside composite operators.
- `|`: Elixir pipe and cons separator (the `|` line in the precedence table, [operators.md#L95](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/pages/references/operators.md#L95)). Erlang `'|'` terminal.
- `<<` / `>>`: bit-string delimiters in both languages.

## 7. Design inference (not documented; for the decision-maker)

Everything in this section is inference from the facts above, not a statement in any source.

1. **Word operators are the only spelling shared by Erlang and Elixir.** Both use `band bor bxor bnot bsl bsr`. Choosing them keeps Aura's names identical to the BEAM's `erlang:band/2` etc. and to Elixir's `Bitwise` functions. It also sidesteps the `&`, `^`, `~`, `<<`, `>>` collisions below.
2. **Word forms have two possible statuses, and they differ lexically.** Erlang reserves them and parses them as infix/prefix operators. Elixir treats them as function names. A lexer that treats them as identifiers today can promote them to infix later only by reserving them, which would break any user identifier of that name. Reserving them up front, as Erlang does, avoids that later break.
3. **Erlang's precedence for word operators may surprise readers.** `bsl`/`bsr` share the `+` level and `band` shares the `*` level. Elixir's shifts sit at the pipe level below `+`, and Rust's shifts sit below `+` but above `&`. Each language gives a different reading of `a bsl 2 + 1`. Whatever Aura picks, the precedence row is a second, separable decision from the spelling.
4. **Symbolic forms with Elixir's spellings inherit Elixir's mixing with logical operators.** `&&&` and `|||` share levels with `&&`/`and` and `||`/`or`, below comparison. This is why `x &&& 1 == 0` parses as `x &&& (1 == 0)` under the table. The `^^^` removal shows that Elixir found one such precedence choice worth deprecating.
5. **Single-character C/Rust spellings collide with prior-art meanings.**
   - `&` collides with Elixir capture and Rust borrow.
   - `^` collides with Elixir's pin.
   - `~` collides with sigils in both languages (a bare-`~` prefix operator followed by a letter would be indistinguishable from a sigil start under the Elixir tokenizer clause at L218).
   - `<<`/`>>` collide with bit-string delimiters.
   - `|` collides with Elixir's cons separator and pipe-like uses. Whether Aura has these constructs is not settled by the README, so this is only a risk if Aura adopts them.
6. **Set-theoretic types may claim `and`, `or`, `not`.** Elixir already uses them for type composition (section 5.4), so a design that also adopted them for boolean operators (as Elixir does) leaves `band`/`bor`/`bnot` as visibly distinct bitwise counterparts. This depends on whether Aura's type syntax follows Elixir's.
7. **Named functions avoid syntactic commitment.** If the operations are ordinary functions (Elixir's `Bitwise` module form), no tokenizer changes are needed and symbolic or word operator forms can be layered on later. The cost is that infix operators cannot later be introduced without an additive grammar change. Elixir's history shows named-function and operator forms coexisting since 2012.
8. **Less collision-prone symbolic tokens need a rule for what "collision" means.** Elixir's own list of parsed-but-unused operators (section 5.3) is a fixed vocabulary that Aura is not bound by. Aura's macro goal ("Compile-time macros") and pipe-operator goal may consume additional token shapes, which the README does not yet specify. Two Elixir facts constrain any triple-character candidate: the tokenizer warns on repeated characters (`&&&&`), and `<<<`/`>>>` need longest-match ordering against `<<`/`>>`. Specific alternative token shapes (for example dot-delimited or letter-prefixed spellings) were not researched here because no in-scope first-party source defines them.
9. **Forward-compatibility with sigils.** In both languages a sigil starts with `~` followed by a name. A design that keeps `~` followed by a letter reserved for sigils, and does not use a leading `~` prefix operator that could be followed by an identifier, avoids ambiguity with a later sigil feature. Elixir's `~~~` avoided it only because its second character is also `~`, not a letter.
10. **Units-of-measure and set-theoretic types are Aura goals that may want `^`, `&`, `|`, `~`.** The README lists both goals but does not specify syntax. This report has no source for how those would use the characters, so it flags the overlap as an open question.

### Open questions this report cannot answer

- Which of Aura's other goals (pipe operator, units, set-theoretic type syntax, macro quoting, `&`-style capture) will claim `& ^ ~ | << >>`?
- Should bitwise operators be infix syntax at all, or named functions plus optional aliases?
- Which precedence row do bitwise operators occupy, relative to arithmetic and comparison?
- Should the four-character sequences that Elixir warns on be errors from the start in Aura?

## 8. Not researched

- Koka's bitwise spellings: no first-party source was fetched. Koka is listed in `AGENTS.md` as an upstream.
- Verse's bitwise story: not fetched.
- Rendered Erlang and Elixir doc sites, and Erlang's `erlang` module documentation for `band`/`bsl` semantics on negative shifts. The Elixir `bsl`/`bsr` docs give negative-shift examples ([bitwise.ex#L172-L245](https://github.com/elixir-lang/elixir/blob/ffdac87643303e781020ee02b5b25b64c4b936f4/lib/elixir/lib/bitwise.ex#L172-L245)).
- Any non-first-party discussion (forums, mailing lists, blog posts).