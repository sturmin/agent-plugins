# CLI Design Examples

Real-world CLI tools annotated to show patterns and why they work.

---

## `wc` — flat tool, multiple counting modes

```
wc [-clmw] [file ...]
```

**What to learn from it:**
- Flat (no subcommands) — does one class of thing
- Flags are mode selectors, not modifiers: `-l` (lines), `-w` (words), `-c` (bytes), `-m` (chars)
- Multiple flags combine naturally: `wc -lw` counts both
- Operands are optional — falls back to stdin when absent
- Output columns align when multiple files are given; a `total` line is added automatically

**Options table:**

| Short | Long | Description |
|-------|------|-------------|
| `-l` | | Count lines |
| `-w` | | Count words |
| `-c` | | Count bytes |
| `-m` | | Count characters (locale-aware) |

**Key decisions:**
- No long options — predates GNU conventions; new tools should add them
- No `--help` / `--version` — predates that convention too; new tools must include both
- Stdin fallback when no operands given is the right default for filter-style tools

---

## `ls` — flat tool with rich display options

```
ls [-AaFGHhilnqRrStu1] [-D format] [file ...]
```

**What to learn from it:**
- Still flat despite many options — all options modify *how* to list, not *what operation* to perform
- Single-char flags with no long equivalents (historical); GNU `ls` adds long forms
- `-l` (long format) and `-1` (one per line) coexist — be careful with `-l` / `-1` ambiguity in your own designs
- Sort order controlled by flag combinations (`-t` time, `-S` size, `-r` reverse) rather than a `--sort=` option
- Color output is controlled by `LS_COLORS` env var and `--color` option

**Key decisions:**
- `-R` (recursive) is uppercase here — inconsistent with many tools; prefer lowercase `-r` / `--recursive` in new designs
- Output goes to stdout; errors (e.g. permission denied) to stderr — correct
- No operands = list current directory (sensible default)

---

## `git` — subcommand tool, the canonical example

```
git [--version] [--help] [-C <path>] [-c <name>=<value>] <command> [<args>]
```

**What to learn from it:**
- Global options (`-C`, `-c`, `--version`, `--help`) come before the subcommand
- Subcommand options come after the subcommand — the two namespaces never mix
- Each subcommand has its own `--help` (`git commit --help`)
- Subcommand aliases: `git st` → `git status` via config; `git checkout` ≈ `git switch` + `git restore`
- Machine-readable output pattern: `git status --porcelain`, `git log --format=...`

**Subcommand tree (partial):**
```
git
├── add           Stage changes for commit
├── commit        Record staged changes
│   └── --amend   Modify the most recent commit (flag, not subcommand)
├── push          Upload refs to remote
├── pull          Fetch and integrate remote changes
├── log           Show commit history
├── diff          Show changes between commits/working tree
├── remote        Manage remote connections
│   ├── add       Add a new remote
│   ├── remove    Remove a remote
│   └── rename    Rename a remote
└── config        Read and write config values
    └── --global  Apply to user config (flag, not subcommand)
```

**Key decisions:**
- `git remote` could have been flat (`git add-remote`, `git remove-remote`) — subcommand grouping is appropriate when operations share a noun
- Global `-c key=value` lets you override config per-invocation without a temp file
- `--` is universally honored: `git checkout -- file` disambiguates filenames from branch names

---

## `curl` — flat tool with extensive options

```
curl [options] <url> [url ...]
```

**What to learn from it:**
- Flat despite ~250 options — all options modify a single operation (transfer)
- Both short and long forms for nearly every option: `-X`/`--request`, `-H`/`--header`, `-o`/`--output`
- Repeatable options: `-H` can be given multiple times to add multiple headers
- `--data @file` and `--data @-` (stdin) — the `@` prefix convention for file/stdin arguments
- Output to stdout by default; `-o file` redirects; `-O` uses the remote filename

**Key decisions:**
- `-v` / `--verbose` prints to stderr so stdout stays clean for piping
- `--silent` / `-s` suppresses the progress meter but not errors; `--silent --show-error` is the pipeline-safe combination
- Exit codes are documented and meaningful: `6` = DNS failure, `7` = connection refused, `22` = HTTP error with `--fail`

---

## `grep` — filter-style flat tool

```
grep [-E|-F|-G] [-e pattern] [-f file] [-cilnqsvx] [pattern] [file ...]
```

**What to learn from it:**
- Filter pattern: reads from operands or stdin, writes matches to stdout, errors to stderr
- `-E` (extended regex), `-F` (fixed string), `-G` (basic regex) are mutually exclusive mode flags — document with `{-E|-F|-G}`
- `-e pattern` allows multiple patterns and removes ambiguity when pattern starts with `-`
- `-l` (list files) and `-c` (count) change output format rather than adding a subcommand
- Exit codes carry semantic meaning: `0` = match found, `1` = no match, `2` = error — this is an intentional extension beyond the standard 0/1/2

**Key decisions:**
- The pattern can be a positional operand (`grep pattern file`) or `-e pattern` — both work, `-e` is required when composing multiple patterns
- `-r` (recursive, GNU extension) not in original POSIX spec, but now universally expected

---

## Common patterns summary

| Pattern | Example | When to use |
|---------|---------|-------------|
| Flat tool | `wc`, `grep`, `curl` | One class of operation, options change how |
| Subcommand tool | `git`, `docker`, `npm` | Multiple distinct operations on a shared noun |
| Filter (stdin→stdout) | `grep`, `sed`, `wc` | Processing streams; operands optional |
| Source-dest operands | `cp src dest`, `mv src... dest` | Moving or transforming named resources |
| Verb-noun subcommands | `git remote add`, `docker image pull` | Operations on a specific resource type |
| `--porcelain` / `--format` | `git status --porcelain` | Machine-readable output variant |
| `@file` argument | `curl --data @body.json` | Passing file contents as an option value |
