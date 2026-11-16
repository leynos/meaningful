# Assistant instructions

## Code style and structure

- **Code is for humans.** Write code with clarity and empathy—assume a
  tired teammate will need to debug it at 3 a.m.
- **Comment *why*, not *what*.** Explain assumptions, edge cases, trade-offs,
  or complexity. Don't echo the obvious.
- **Clarity over cleverness.** Be concise, but favour explicit over terse or
  obscure idioms. Prefer code that's easy to follow.
- **Use functions and composition.** Avoid repetition by extracting reusable
  logic. Prefer generators or comprehensions, and declarative code to
  imperative repetition when readable.
- **Small, meaningful functions.** Functions must be small, clear in purpose,
  single responsibility, and obey command/query segregation.
- **Functions tell a story.** A function reads as a sequence of named steps;
  each step is a helper whose name says what it does and whose signature
  declares what it consumes and produces. Prefer this over long inline
  sequences, even when the inline code is straight-line.
- **Clear commit messages.** Commit messages should be descriptive, explaining
  what was changed and why.
- **Use consistent spelling and grammar.** Comments must use en-GB-oxendict
  ("-ize" / "-yse" / "-our") spelling and grammar, with the exception of
  references to external APIs. See [Spelling](#spelling).
- **Illustrate with clear examples.** Documentation for public, re-exported, or
  cross-module functions must include clear examples demonstrating their usage
  and outcome. A private helper used only within its own module needs only a
  one-line summary docstring. Test documentation should omit examples where the
  example serves only to reiterate the test logic.
- **Keep file size manageable.** No single code file may be longer than 400
  lines. Long switch statements or dispatch tables should be broken up by
  feature and constituents colocated with targets. Large blocks of test data
  should be moved to external data files.
- **Name things precisely.** Use clear, descriptive variable and function names.
  For booleans, prefer names with `is`, `has`, or `should`.
- **Structure logically.** Each file should encapsulate a coherent module. Group
  related code (for example, models + utilities + fixtures) close together.
- **Group by feature, not layer.** Colocate views, logic, fixtures, and helpers
  related to a domain concept rather than splitting by type.
- **Use clear file boundaries.** Each module, component, and package should have
  an obvious responsibility and avoid accidental coupling.

## Documentation maintenance

- **Reference:** Use the markdown files within the `docs/` directory as a
  knowledge base and source of truth for project requirements, dependency
  choices, and architectural decisions. Start with
  [documentation contents](docs/contents.md) when orienting within the project.
- **Update:** When new decisions are made, requirements change, libraries are
  added/removed, or architectural patterns evolve, **proactively update** the
  relevant file(s) in the `docs/` directory to reflect the latest state. Ensure
  the documentation remains accurate and current.
- **Design decisions:** Record design decisions in the relevant design
  document. When a decision is substantive, capture it in an Architectural
  Decision Record (ADR) following the documentation style guide, and reference
  that ADR from the design document.
- **User-facing behaviour:** Update the [users' guide](docs/users-guide.md) for
  behaviour or user-interface changes that users should know about. Add
  migration guidance for compatibility or usage changes, and keep maintained
  translations synchronized.
- **Internal interfaces:** Document internally facing interfaces in the
  relevant component architecture document. Document internally facing
  conventions and practices in [developers' guide](docs/developers-guide.md).
- **Build tooling:** When changing build tooling, document the effective
  toolchain, inherited flags, backend and linker selection, and deliberate
  differences between validation commands.
- **Style:** All documentation must adhere to the
  [documentation style guide](docs/documentation-style-guide.md).
- **Accuracy:** Verify changed examples, commands, imports, links, paths, and
  stated guarantees against the implementation. Update maintained copies and
  indexes when names or locations change. Keep transient validation results in
  the PR or execution record.
- **Decision history:** Preserve accepted ADR decisions; record subsequent
  changes in a new or amending ADR, following the
  [documentation style guide](docs/documentation-style-guide.md). Capture
  proof-driven API, representation, and invariant decisions in the design
  documentation.
- **Completed execplans:** Completed execplans are historical documents and
  should reflect the state of the repository at time of implementation. Do not
  retroactively update completed execplan documents to reflect subsequent
  changes in API, structures, or behaviour.

## Planning and verification

- For non-trivial work, prepare or update a self-contained ExecPlan using the
  available planning guidance. Identify the outcome, relevant roadmap and
  design references, prerequisites, scope exclusions, and observable acceptance
  criteria. Signpost relevant documentation and skills. Where the repository
  has no ExecPlan guidance, roadmap, or design document for the work, state the
  outcome, scope, and acceptance criteria in the PR description instead.
- Identify affected behavioural contracts, invariants, and failure modes before
  choosing the implementation. Use these to pressure-test the proposed API and
  verification strategy.
- Plan the smallest useful increment. Preserve decision gates and stopping
  conditions; defer abstractions, dependencies, macros, or published components
  until the preceding evidence justifies them.
- Where formal verification is selected, first sketch the independent model,
  premises, invariants, representation relationships, and principal proof
  obligations. Treat proposed lemmas as provisional. Defer detailed proof
  engineering until implementation exposes the verification conditions; do not
  add assumptions, admissions, or trusted proof bodies merely to make
  incomplete proofs pass. Unproved statements may remain in the plan until that
  milestone.
- Give each major milestone explicit acceptance commands, including applicable
  proof checks and the formatting, linting, and test gates. Keep commits
  coherent and useful as rollback points.
- When plan approval is required, deliver the plan for review and wait for
  approval before implementation. Keep the active plan current, and mark
  relevant roadmap entries complete only after their acceptance criteria pass.

## Change quality and committing

- **Atomicity:** Aim for small, focused, atomic changes. Each change (and
  subsequent commit) should represent a single logical unit of work.
- **Quality gates:** Before considering a change complete or proposing a
  commit, ensure all of the following are met:
  - New functionality or behaviour changes are fully validated by relevant unit
    and behavioural tests.
  - Bug fixes include a failing test before the fix and a passing test
    afterwards.
  - Code passes lint checks.
  - Formatting is correct and validated.
- **For Python files:**
  - **Testing:** Passes all relevant unit and behavioural tests (`make test`).
  - **Linting:** Passes lint checks (`make lint`).
  - **Formatting:** Adheres to formatting standards (`make check-fmt`; use
    `make fmt` to apply fixes).
  - **Typechecking:** Passes type checking (`make typecheck`).
  - **Auditing:** Passes dependency vulnerability checks (`make audit`).
  - The generated Makefile wiring for these targets is:
    - `make check-fmt` runs Ruff formatting checks with
      `ruff format --check $(PYTHON_TARGETS)`.
    - `make lint` runs `make lint-python`; `make lint-python` runs
      `ruff check $(PYTHON_TARGETS)` and
      `interrogate --fail-under 100 $(PYTHON_TARGETS)` for 100% docstring
      coverage. Git discovers tracked and untracked Python sources, including
      hidden action directories. The base Pylint sweep covers
      `$(PYLINT_TARGETS)` using the interpreter selected for the baseline:
      uv-managed PyPy 3.12.
      The minimum Python baseline is 3.12. DF12 Python Lints is pinned to
      v0.4.0; it and Ambrleaks, Skylos, Vulture, and the duplication detector
      run as isolated tools on Python 3.14. Ambrleaks checks `tests/`, Skylos
      checks the repository, and Vulture checks test helpers. The strict
      Skylos and Vulture dead-code checks, shared Nose duplication gate, and
      action validation are also part of `make lint`.
    - `make typecheck` runs `ty check $(PYTHON_TARGETS)`.
      `ty` 0.0.56 and Ruff 0.16.8 run in isolated Python 3.14 tools.
      `make lint-actions` validates workflows and actions with actionlint,
      action-validator, yamllint 1.38.0, and shellcheck.
    - `make test` runs `pytest -v -n $(PYTEST_XDIST_WORKERS)`.
    - `make audit` runs `pip-audit`.
  - Temporary lint suppressions must include a link to the planned fix or
    implementation that will remove the suppression.

  - **Markdown files (`.md` only):**
    - **Linting and spelling:** Passes Markdown linting and the shared
      en-GB-oxendict spelling gate (`make markdownlint`; spelling scope is
      `all`).
    - **Formatting:** Uses mdtablefix 0.6.1 through `make check-fmt`; use
      `make fmt` to apply fixes.
    - **Mermaid diagrams:** Passes validation using nixie (`make nixie`).
- **Before review:** Inspect the complete diff, including tests, fixtures,
  scripts, examples, and documentation. Check file-size limits, module
  documentation, doc-comment placement, assertion diagnostics, duplicated test
  setup, and applicable spelling and formatting gates. Link relevant roadmap
  entries, issues, and the ExecPlan in the PR description.
- **Committing:**
  - Only changes that meet all the quality gates above should be committed.
  - Write clear, descriptive commit messages summarizing the change, following
    these formatting guidelines:
    - **Imperative Mood:** Use the imperative mood in the subject line (e.g.,
      "Fix bug", "Add feature" instead of "Fixed bug", "Added feature").
    - **Subject Line:** The first line should be a concise summary of the change
      (ideally 50 characters or fewer).
    - **Body:** Separate the subject from the body with a blank line. Subsequent
      lines should explain the *what* and *why* of the change in more detail,
      including rationale, goals, and scope. Wrap the body at 72 characters.
    - **Formatting:** Use Markdown for any formatted text (like bullet points or
      code snippets) within the commit message body.
  - Do not commit changes that fail any of the quality gates.

## Refactoring heuristics and workflow

- **Recognizing refactoring needs:** Regularly assess the codebase for potential
  refactoring opportunities. Perform refactoring when observing:
  - **Long methods/functions:** Functions or methods that are excessively long
    or try to do too many things.
  - **Duplicated code:** Identical or very similar code blocks appearing in
    multiple places.
  - **Complex conditionals:** Deeply nested or overly complex `if`/`else` or
    `switch` statements (high cyclomatic complexity).
  - **Large code blocks for single values:** Significant chunks of logic
    dedicated solely to calculating or deriving a single value.
  - **Primitive obsession / data clumps:** Groups of simple variables (strings,
    numbers, booleans) that are frequently passed around together, often
    indicating a missing class or object structure.
  - **Excessive parameters:** Functions or methods requiring a very long list of
    parameters.
  - **Feature envy:** Methods that seem more interested in the data of another
    class/object than their own.
  - **Shotgun surgery:** A single change requiring small modifications in many
    different classes or functions.
- **Abstraction / port / helper policy:** This policy exists so that the same
  behaviour is not built twice without anyone noticing. Before implementing an
  abstraction, a port, or a shared helper (anything public, re-exported, or
  called from outside its own module), the agent must:
  1. Sweep the repository and approved dependencies for equivalent behaviour.
     Reuse it unless demonstrably unsuitable. New abstractions must simplify
     the current change or have a concrete reuse case.
  2. Document the new abstraction's intended scope and re-use policy
     (ownership boundaries, permitted call-sites, and composition rules).
  3. Record the decision in the appropriate architecture, design, or
     developers-guide document using `docs/contents.md` as the index to select
     the correct location.

  A private helper (a leading-underscore function in Python, a non-`pub` item
  in Rust) extracted so that one function tells a story needs only step 1: its
  name, its signature, and a one-line docstring or doc comment say the rest.
  When another module needs it, promote it under this policy rather than
  copying it.
- **Post-commit review:** After committing a functional change or bug fix (that
  meets all quality gates), review the changed code and surrounding areas using
  the heuristics above.
- **Separate atomic refactors:** If refactoring is deemed necessary:
  - Perform the refactoring as a **separate, atomic commit** *after* the
    functional change commit.
  - Ensure refactoring adheres to the testing guidelines (behavioural tests
    pass before and after, unit tests added for new units).
  - Ensure the refactoring commit itself passes all quality gates.

## Architecture and operational boundaries

- Keep domain policy separate from filesystem, process, network, framework,
  and telemetry adapters. Inject dependencies where needed through existing
  interfaces or narrow closures; avoid speculative layers and dependency cycles.
- Make fallibility, mutation, and side-effects explicit in APIs. Preserve I/O,
  parsing, and permission errors rather than silently converting them into
  absence, `false`, or empty success. Compose side-effects at orchestration
  boundaries; queries must not hide writes or unrelated workflow policy.
- For streams, subprocesses, retries, caches, queues, and background tasks,
  define applicable size limits, deadlines, ownership, cancellation, and
  cleanup. A timeout does not bound accumulated bytes. Include waiting and
  cleanup in the operation's budget. Define partial-failure and retry behaviour
  where updates span multiple effects; keep locks narrow and ordered, avoiding
  holds across awaits or external calls unless explicitly justified.
- Validate untrusted inputs before privileged effects, use structured APIs
  instead of executable string construction, and grant only required
  permissions.
- When performance materially motivates a change, measure realistic inputs and
  distinguish build, execution, queueing, and cache state. Avoid repeated I/O
  or unnecessary allocation in per-item loops and blocking work on async
  executors.

## Python verification and testing

- For Python work, use `pytest` for unit tests and `pytest-bdd` for behavioural
  tests. Cover happy paths, unhappy paths, and relevant edge cases.
- Keep pytest tests in the top-level `tests/` tree. Do not place unit tests in
  package module directories or `unittests/` subdirectories, because CI
  coverage uses xdist-backed SlipCover support.
- Snapshot tests (using `syrupy`) should be provided where multivariant output
  format consistency is relevant to the requirements. Snapshot tests must
  capture meaningful, reviewer-useful contracts rather than generic dumps of
  broad objects. Keep snapshots focused on stable output boundaries, pair them
  with semantic assertions for the behaviours they represent, and avoid
  snapshot-only coverage for logic that can be asserted directly. Redact or
  normalize nondeterministic fields such as timestamps, absolute paths, random
  identifiers, ordering-dependent maps, hostnames, and environment-specific
  values before snapshotting. Do not accept brittle snapshots that churn after
  harmless formatting or dependency changes; narrow the captured output until a
  snapshot failure identifies a real contract change.
- Add end-to-end tests where a change affects externally observable workflows,
  integration contracts, persistence, command-line behaviour, network
  boundaries, user interface flows, or other system-level behaviour.
- Use property tests with `hypothesis` or `CrossHair` when a change introduces
  an invariant over a range of inputs, states, orderings, or transitions.
- Run relevant unit, behavioural, property, and end-to-end suites before and
  after each change.

## Markdown guidance

- Validate Markdown files using `make markdownlint`.
- `make markdownlint` runs markdownlint-cli2 and the shared spelling gate over
  all repository content. Spelling scope is `all`.
- Run `make fmt` after any documentation changes to format all Markdown files
  and fix table markup with mdtablefix 0.6.1.
- Validate Mermaid diagrams in Markdown files by running `make nixie`.
- Markdown paragraphs and bullet points must be wrapped at 80 columns.
- Code blocks must be wrapped at 120 columns.
- Tables and headings must not be wrapped.
- Use dashes (`-`) for list bullets.
- Use GitHub-flavoured Markdown footnotes (`[^1]`) for references and
  footnotes.
- Avoid first-person prose outside README and BDD feature files; allow
  second-person prose in README, AGENTS, BDD feature files, and agent skill
  directories. These exceptions qualify the documentation style guide.
- Quoted API names and identifiers retain upstream spelling. Mark them as
  code using backticks or fenced blocks rather than adding spelling exceptions.

<!-- typos-config-builder:agents-md:start -->

## Spelling

- `make spelling` runs the pinned `typos-config-builder gate`, which
  regenerates `typos.toml` from the shared en-GB-oxendict dictionary and
  `typos.local.toml`, then checks spelling and the shared phrase corrections.
- `typos.toml` is generated: never edit it by hand. Put narrow
  repository-specific exceptions in `typos.local.toml`, as exact or full-line
  patterns rather than bare accepted words.
- When `make spelling` changes `typos.toml`, commit the regenerated file. If
  the change is unrelated to your work, commit it in a separate base pull
  request and stack your branch on it, so each review diff stays focused.

<!-- typos-config-builder:agents-md:end -->

## Python development guidelines

For Python development, refer to the detailed guidelines in the `.rules/`
directory:

- [Python code style guidelines](.rules/python-00.md) - Core Python 3.12+ style
  conventions.
- [Python context managers](.rules/python-context-managers.md) - Best practices
  for context managers.
- [Python exceptions and logging][python-exceptions] -
  Raising and handling exceptions and logging.
- [Python generators](.rules/python-generators.md) - Generator and iterator
  patterns.
- [Python project configuration](.rules/python-pyproject.md) -
  `pyproject.toml` and packaging.
- [Python return patterns](.rules/python-return.md) - Function return
  conventions.
- [Python typing](.rules/python-typing.md) - Type annotation best practices.

[python-exceptions]: .rules/python-exception-design-raising-handling-and-logging.md

Additional docs:

- [Scripting standards](docs/scripting-standards.md) - Guidance for writing
  robust scripts, including secure command execution via `cuprum`, catalogue
  allowlisting, and command mocking patterns with `cmd-mox`.
- Before adding or updating helper scripts, read the scripting standards guide
  and follow its `Cyclopts`, `cuprum`, `pathlib`, and `cmd-mox` conventions.

## Additional tooling

The following tooling is available in this environment:

- `mbake` — A Makefile validator. Run using `mbake validate Makefile`.
- `strace` — Traces system calls and signals made by a process; useful for
  debugging runtime behaviour and syscalls.
- `gdb` — The GNU Debugger, for inspecting and controlling programs as they
  execute (or post-mortem via core dumps).
- `ripgrep` — Fast, recursive text search tool (`grep` alternative) that
  respects `.gitignore` files.
- `ltrace` — Traces calls to dynamic library functions made by a process.
- `valgrind` — Suite for detecting memory leaks, profiling, and debugging
  low-level memory errors.
- `bpftrace` — High-level tracing tool for eBPF, using a custom scripting
  language for kernel and application tracing.
- `lsof` — Lists open files and the processes using them.
- `htop` — Interactive process viewer (visual upgrade to `top`).
- `iotop` — Displays and monitors I/O usage by processes.
- `ncdu` — NCurses-based disk usage viewer for finding large files/folders.
- `tree` — Displays directory structure as a tree.
- `bat` — `cat` clone with syntax highlighting, Git integration, and paging.
- `delta` — Syntax-highlighted pager for Git and diff output.
- `tcpdump` — Captures and analyses network traffic at the packet level.
- `nmap` — Network scanner for host discovery, port scanning, and service
  identification.
- `lldb` — LLVM debugger, alternative to `gdb`.
- `eza` — Modern `ls` replacement with more features and better defaults.
- `fzf` — Interactive fuzzy finder for selecting files and commands.
- `hyperfine` — Command-line benchmarking tool with statistical output.
- `shellcheck` — Linter for shell scripts.
- `fd` — Fast user-friendly `find` alternative with sensible defaults.
- `checkmake` — Linter for `Makefile`s, ensuring best practices.
- `srgn` — Structural grep and syntax-tree pattern editing.
- `difft` **(Difftastic)** — Semantic diff tool that compares code structure.

## Key takeaway

These practices help maintain a high-quality codebase and facilitate
collaboration.
