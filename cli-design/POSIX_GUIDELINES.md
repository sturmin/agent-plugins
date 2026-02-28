# POSIX Utility Syntax Guidelines

Source: https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html

These are the 14 normative guidelines from POSIX.1-2017. Follow all of them unless
the GNU extensions listed at the bottom deliberately override one.

---

## Guideline 1 — Utility name

Utility names should be between 2 and 9 characters, inclusive.
Names should contain only lowercase letters and digits from the portable character set.

## Guideline 2 — Option names

Each option must be a single alphanumeric character from the portable character set.
`-W` is reserved for vendor options and must not be used for anything else.

## Guideline 3 — Option prefix

All options must be preceded by the `-` delimiter.
An option without an argument may be combined with other options behind a single `-` (see Guideline 5).

## Guideline 4 — Option form

Option names consist of a single character only.
Multi-character option names are a GNU extension (long options, see below).

## Guideline 5 — Grouping flags

Options without arguments may be grouped behind a single `-` delimiter:

```
# equivalent
ls -l -a -h
ls -lah
```

At most one option with an argument may be appended at the end of the group:

```
tar -xvf archive.tar   # -x -v -f archive.tar
```

## Guideline 6 — Option-argument separation

Each option and its argument should be separate tokens (separate `argv` entries):

```
tool -o file    # preferred
tool -ofile     # also valid (adjacent form)
```

Both forms must be accepted when the option takes an argument.

## Guideline 7 — Option-arguments are not optional

Option-arguments must not be optional. If a flag requires an argument, that argument
is always required. Do not design `-o [file]`.

## Guideline 8 — Multiple option-arguments

If a single option takes multiple values, they should be separated by commas
or whitespace, passed as a single argument:

```
tool -i file1,file2,file3
```

## Guideline 9 — Options precede operands

All options should appear before operands on the command line.
Processing stops when a non-option argument is encountered.

```
# POSIX-strict: options before operands
tool -v -n file1 file2

# Not guaranteed to work in POSIX-strict parsers:
tool file1 -v file2
```

Note: GNU `getopt` deliberately relaxes this and permits options anywhere (see GNU Extensions below).

## Guideline 10 — End of options marker

The first argument consisting of exactly `--` (not an option-argument) signals
the end of options. All following arguments are treated as operands, even if they
begin with `-`.

```
# Safely pass a filename that starts with a dash
tool -- -weirdfilename.txt
```

## Guideline 11 — Option order

The order of options without arguments should not matter unless documented as
mutually exclusive or having a defined ordering effect (e.g., `-v -v -v` for
increasing verbosity). When options conflict, the last one on the command line wins
unless documented otherwise.

## Guideline 12 — Operand order

Operand order may matter. The utility defines the interpretation.
Document operand order explicitly (e.g., `cp source dest`, `mv source... dest`).

## Guideline 13 — Stdin as `-`

When a utility uses operands to specify files for reading or writing, the `-` operand
should mean standard input (for reading) or standard output (for writing).

```
# Reads from stdin, useful in pipelines
cat input.txt | tool -
```

## Guideline 14 — Unrecognized option-like arguments

If an argument looks like an option (starts with `-`) and cannot be identified as
an operand, the utility should treat it as an option. Do not silently ignore it.

---

## GNU Extensions (deliberate deviations from POSIX)

These GNU behaviors are widely expected and should be supported:

### Long options
`--word` and `--word-word` forms alongside or instead of short options.
Argument forms: `--option=value` or `--option value`.
```
grep --recursive --include='*.c' pattern .
```

### Options anywhere
GNU `getopt_long` permits options to appear after operands unless `--` has been used.
This is a GNU extension; POSIX requires all options before operands.

### `-` is always stdin/stdout
Treat `-` as a synonym for stdin (reading) or stdout (writing) at every applicable position.

### `--` is always honored
Even when intermixed options are supported, `--` must still end option processing.
