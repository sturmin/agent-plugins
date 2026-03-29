---
name: cli-design
description: Design and update command-line interface specifications — subcommands, short/long options, and operands — following POSIX and GNU standards. Use when asked to design, create, plan, specify, review, or update a CLI tool's interface, commands, flags, or argument structure, like git, ls, wc, or curl.
---

# CLI Design

This skill operates in two modes:

- **Create** — produce a full CLI Specification Document from a tool description
- **Update** — apply a change request to an existing CLI Specification Document

---

## Mode: Create

Triggered when the user describes a tool idea without an existing spec.

### Process

1. **Name the tool** — 2–9 chars, lowercase letters and digits only
2. **Identify operations** — distinct operations become subcommands; single-purpose tools stay flat
3. **Identify modifiers** — what changes behavior? → options (short + long pairs)
4. **Identify operands** — what does it act on? files, IDs, URLs?
5. **Write synopses first** — forces precision before filling in details
6. **Fill in the tables** — options, then operands, then exit codes
7. **Write examples** — validate ergonomics by writing real invocations
8. **Run the checklist** — catch gaps before finalizing

---

## Mode: Update

Triggered when the user provides an existing spec and a change request.

### Process

1. **Read the existing spec** — understand the current structure fully before touching anything
2. **Apply the change** — add, remove, or rename the requested element
3. **Propagate consistency** — check every affected area:
   - New option added to one subcommand → should it appear in sibling subcommands?
   - Option renamed → update synopsis, table, and all examples that reference it
   - Subcommand added/renamed → update the subcommand tree and any cross-references
   - New exit code → add to the exit codes table
   - New operand → update synopsis and operands table together
4. **Re-run the checklist** — validate the full document after changes, not just the changed section
5. **Report what changed** — briefly summarize every section that was modified

---

## Output artifact: CLI Specification Document

All sections are required. Produce them in this order.

---

### 1. Synopsis block

One synopsis line per command/subcommand, using man-page notation:

```
tool [GLOBAL-OPTIONS] <subcommand> [OPTIONS] <OPERAND> [OPERAND ...]
tool [GLOBAL-OPTIONS] <subcommand> [OPTIONS] -o <file> <OPERAND>
tool --help | --version
```

| Notation | Meaning |
|----------|---------|
| `[-x]` | optional flag |
| `<operand>` | required positional (placeholder) |
| `[operand ...]` | zero or more operands |
| `{a\|b}` | mutually exclusive, choose one |
| `--` | end of options; everything after is treated as an operand |

---

### 2. Subcommand tree (omit if flat tool)

```
tool
├── subcommand-a      One-line description
│   └── sub-sub       Only if genuinely needed
├── subcommand-b      One-line description
└── subcommand-c      One-line description
```

Rules:
- Global options come **before** the subcommand; subcommand options come **after** — never permuted
- Each subcommand supports `--help`
- Document aliases (`rm`/`remove`, `ls`/`list`) on the same line

---

### 3. Options table — one table per command/subcommand

| Short | Long | Arg | Required | Default | Description |
|-------|------|-----|----------|---------|-------------|
| `-o` | `--output` | `<file>` | No | stdout | Write output to `<file>` instead of stdout |
| `-v` | `--verbose` | — | No | off | Print additional diagnostic information |

Column rules:
- **Short** — `-x` form, or blank if none
- **Long** — always present; `--word` or `--word-word`
- **Arg** — metavar (`<file>`, `<n>`, `<format>`) or `—` for boolean flags
- **Required** — Yes / No
- **Default** — value when absent; `—` if not applicable
- **Description** — one sentence; state what it does. Inline constraints: allowed enum values, whether repeatable, mutual exclusions

Standard options every tool must include:

| Short | Long | Description |
|-------|------|-------------|
| `-h` | `--help` | Print usage information and exit 0 |
| | `--version` | Print version string and exit 0 |

Common standard options — include as appropriate:

| Short | Long | Description |
|-------|------|-------------|
| `-v` | `--verbose` | Increase verbosity; repeatable (`-vvv`) |
| `-q` | `--quiet` | Suppress all non-error output; mutually exclusive with `--verbose` |
| `-n` | `--dry-run` | Print what would happen without executing |
| `-f` | `--force` | Skip confirmation prompts |
| `-o` | `--output` | Write output to a file instead of stdout |
| `-r` / `-R` | `--recursive` | Recurse into directories |
| `-y` | `--yes` | Assume yes to all prompts |
| | `--no-color` | Disable colored output (also honor `NO_COLOR` env var) |
| | `--config` | Path to config file |

See [GNU_CONVENTIONS.md](GNU_CONVENTIONS.md) for the full canonical list.

---

### 4. Operands table — one table per command/subcommand

| # | Name | Type | Required | Description |
|---|------|------|----------|-------------|
| 1 | `source` | path | Yes | File to read; use `-` for stdin |
| 2+ | `dest` | path | Yes | Destination path; repeatable |

Column rules:
- **#** — position; use `N+` for repeatable operands starting at position N
- **Type** — `path`, `file`, `dir`, `string`, `integer`, `url`, or domain-specific
- Note when `-` is accepted as stdin/stdout substitute
- Omit table if the command takes no operands

---

### 5. Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Operational error — the command ran but the operation failed |
| 2 | Misuse — bad invocation: unknown option, missing required argument, wrong type |

Add additional codes only when needed. Document each one — scripts depend on them.

---

### 6. Environment variables (omit if none)

| Variable | Default | Description |
|----------|---------|-------------|
| `TOOL_CONFIG` | `~/.toolrc` | Path to the default config file |
| `NO_COLOR` | unset | Disable color output when set to any value |

---

### 7. Examples

3–6 annotated examples covering common use cases and important edge cases:

```bash
# Basic usage
tool input.txt

# Specify output explicitly
tool -o result.txt input.txt

# Read from stdin, write to stdout (pipeline-friendly)
cat input.txt | tool -

# Dry run to preview changes
tool --dry-run --verbose input.txt

# Subcommand usage
tool subcommand --flag <operand>
```

---

## Naming conventions

- **Subcommands** — lowercase, hyphens for multi-word: `add-remote`, `push-all`
- **Long options** — `--verb` or `--verb-noun`: `--set-upstream`, `--sort-by`
- **Metavars** — angle-bracket lowercase throughout: `<file>`, `<format>`, `<n>`
- **Consistency rule** — if one subcommand uses `--format`, all must use `--format`, not `--output-format`

## I/O conventions

- Results → **stdout**
- Errors and diagnostics → **stderr**
- Error message format: `progname: descriptive message` (not `Error: ...`)
- Machine-readable output: `--output=json` or `--porcelain` pattern
- Respect `NO_COLOR` env var for colored output
- Interactive prompts only when stdin is a tty; suppress in pipelines or with `--yes`/`--force`

## Validation checklist

- [ ] Every command/subcommand has a written synopsis
- [ ] Every option has a long form; frequently-used ones also have a short form
- [ ] Mutually exclusive options are noted in the options table
- [ ] `--help` and `--version` present at every level
- [ ] Exit codes 0, 1, 2 defined; any extras documented
- [ ] Errors go to stderr; results go to stdout
- [ ] `-` accepted as stdin/stdout wherever files are accepted
- [ ] Examples cover happy path, piped input, and at least one edge case
- [ ] Naming is consistent across all subcommands and options

---

For all 14 POSIX utility guidelines: [POSIX_GUIDELINES.md](POSIX_GUIDELINES.md)
For real-world annotated examples (git, ls, wc, curl): [EXAMPLES.md](EXAMPLES.md)
