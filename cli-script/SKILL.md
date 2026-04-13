---
name: cli-script
description: >
  Design and implement CLI scripts that follow CLIG (clig.dev) conventions — concise, robust,
  composable, and human-first. Use whenever the user asks to write, review, or improve a
  command-line tool, script, or utility in Python or Bash (primary), or other languages.
  Also trigger when the user mentions: argparse, click, typer, subcommands, flags, exit codes,
  stdin/stdout/stderr handling, shell scripting patterns, CLI UX, reusable script libraries,
  shell alias, automation script, task runner script, or mise task. Even for quick one-off
  scripts, apply this skill to establish good defaults from the start.
  For Python: default to argparse (stdlib); use click/typer only when complexity justifies it.
  For Bash: getopts + strict mode (set -euo pipefail). For Go/Rust: cobra/clap patterns noted
  but not the primary focus — suggest Python if the language is not yet chosen.
---

# CLI Script Skill

Build CLI scripts that are concise, robust, composable, and well-documented — grounded in
[CLIG guidelines](https://clig.dev).

---

## Core Principles (from CLIG)

| Principle | What it means in practice |
|-----------|--------------------------|
| Human-first | Default output is readable; machine output is opt-in (`--json`, `--plain`) |
| Composable | stdout = data, stderr = messages, exit codes = status |
| Discoverable | `-h/--help` always works; show examples first |
| Robust | Handle bad input gracefully; idempotent where possible |
| Conversational | Confirm destructive ops; suggest next steps; explain errors |

---

## Structure Template

```
my-tool/
├── my_tool/           # or cmd/, src/ depending on language
│   ├── __init__.py    # (Python) or main.go, etc.
│   ├── cli.py         # entry point / argument parsing
│   ├── core.py        # business logic (importable, no sys.exit)
│   └── lib/           # shared patterns (see below)
├── tests/
├── README.md
└── CHANGELOG.md
```

**Key rule:** Keep business logic out of argument parsing. `cli.py` calls `core.py`, not the other way around. This makes `core.py` importable and testable without invoking the CLI.

---

## Python Parser: `argparse` (stdlib-first)

Prefer `argparse` — no dependencies, ships with Python, sufficient for most tools.
Use `click` or `typer` only when subcommand complexity or decorator ergonomics justify the dep.

```python
import argparse, sys

def build_parser() -> argparse.ArgumentParser:
    p = argparse.ArgumentParser(
        prog="my-tool",
        description="One sentence. What it does, not how.",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
examples:
  %(prog)s file.txt
  %(prog)s --dry-run file.txt
  %(prog)s --json file.txt | jq .
        """,
    )
    p.add_argument("input", metavar="FILE", help="input file (- for stdin)")
    p.add_argument("-o", "--output", default="-", metavar="FILE", help="output file (default: stdout)")
    p.add_argument("-n", "--dry-run", action="store_true", help="preview without writing")
    p.add_argument("--json", dest="as_json", action="store_true", help="output as JSON")
    p.add_argument("-q", "--quiet", action="store_true", help="suppress non-essential output")
    p.add_argument("--no-color", action="store_true", help="disable color output")
    p.add_argument("--version", action="version", version="%(prog)s 1.0.0")
    return p

def main() -> None:
    args = build_parser().parse_args()
    # call core logic here — no sys.exit inside core
```

**For non-Python tools:** Bash → `getopts`; Go → `cobra`; Rust → `clap`.

---

## Argument & Flag Conventions

- **Prefer flags over positional args** — easier to extend, more self-documenting
- **Always provide long forms**: `-v` AND `--verbose`
- **Standard flag names** (follow these unless there's a reason not to):
  - `-h / --help`, `--version`, `-v / --verbose`, `-q / --quiet`
  - `-n / --dry-run`, `-f / --force`, `-o / --output`, `--json`, `--no-color`
- **Sane defaults** — the default behavior should work for 80% of users without flags
- **Never require interactive prompts in scripts** — always accept equivalent flags

---

## Output Rules

```
stdout  →  primary data output (piped, parsed, redirected)
stderr  →  progress, warnings, errors, log lines
exit 0  →  success
exit 1  →  general error
exit 2  →  misuse / bad args (convention)
```

- **Detect TTY** and adjust: colors/progress bars only when `isatty(stdout)`
- Respect `NO_COLOR` env var and `--no-color` flag
- Use `--json` for machine-readable structured output
- Use `--plain` if human-readable default would break piping (e.g. multi-line cells)
- Suppress non-essential output with `-q / --quiet`
- Use a pager (`less -FIRX`) only when outputting large text to a TTY

---

## Help Text Pattern

```
USAGE
  tool [flags] <required-arg>

DESCRIPTION
  One sentence. What it does, not how.

EXAMPLES
  tool file.txt               # most common case first
  tool --dry-run file.txt     # show what would happen
  tool --json file.txt | jq . # composability

FLAGS
  -h, --help       Show this help
  --version        Show version
  -n, --dry-run    Preview changes without applying
  -q, --quiet      Suppress non-essential output
  --json           Output as JSON
  --no-color       Disable color output

DOCS
  https://...
```

**Lead with examples.** Show the most common invocations before flag lists.

---

## Error Handling

- Catch expected errors; rewrite them as actionable messages:
  ```
  Error: cannot write to output.txt
  Hint: try: chmod +w output.txt
  ```
- Signal-to-noise: group repeated errors; don't dump stack traces by default
- Unexpected errors: write debug info to a log file, not stderr, unless `--verbose`
- For destructive operations:
  - **Mild** (delete file): optional prompt or just do it if explicit
  - **Moderate** (delete dir, remote resource): prompt `y/n`, support `--force`
  - **Severe** (delete entire env): require typed confirmation or `--confirm="name"`

---

## Shared Library Patterns

Extract cross-cutting concerns into a `lib/` or shared module:

```python
# lib/output.py
def print_error(msg, hint=None):      # stderr, red if TTY
def print_success(msg):               # stderr or stdout, green if TTY
def print_warning(msg):               # stderr, yellow if TTY
def is_tty() -> bool:                 # sys.stdout.isatty()
def should_use_color() -> bool:       # TTY + NO_COLOR check

# lib/io.py
def read_stdin_or_file(path):         # accept - as stdin
def atomic_write(path, content):      # write to tmp, then rename

# lib/confirm.py
def confirm(prompt, force=False):     # y/n prompt, skip if --force or not TTY
```

When multiple scripts share these, extract to a package (e.g. `myteam-cli-utils`).

---

## Documentation Checklist

- [ ] `--help` always works, leads with examples
- [ ] `README.md`: purpose, install, quickstart, all flags
- [ ] `CHANGELOG.md`: track changes with version + date
- [ ] Man page or `help` subcommand for complex tools (use `ronn` to generate)
- [ ] Web docs if publicly distributed

---

## Commit Message Convention

When changes are made to a CLI script, provide:

**Short (subject line):** `<type>(<scope>): <imperative summary>` — max 72 chars
- Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`
- Examples: `feat(cli): add --dry-run flag`, `fix(output): respect NO_COLOR env var`

**Long (body):** Explain *why*, not *what*. Include:
- What problem this solves
- Any non-obvious design decisions
- Migration notes if behavior changes

```
feat(output): add --json flag for machine-readable output

Scripts piping this tool's output had to parse human-readable text,
which broke when formatting changed. Structured JSON output via --json
makes downstream processing reliable and future-proof.

Follows CLIG recommendation: JSON output when --json is passed,
human-readable default otherwise.
```

---

## Full Python Example (stdlib only)

```python
#!/usr/bin/env python3
"""my-tool — process files and write results."""

import argparse
import json
import os
import sys
import tempfile
from pathlib import Path


# ── output helpers (or extract to lib/output.py) ─────────────────────────────

def _use_color(no_color: bool = False) -> bool:
    if no_color or os.environ.get("NO_COLOR") or os.environ.get("TERM") == "dumb":
        return False
    return sys.stderr.isatty()

def err(msg: str, hint: str = "", *, no_color: bool = False) -> None:
    color = _use_color(no_color)
    prefix = "\033[31mError:\033[0m" if color else "Error:"
    print(f"{prefix} {msg}", file=sys.stderr)
    if hint:
        print(f"  hint: {hint}", file=sys.stderr)

def warn(msg: str, *, no_color: bool = False) -> None:
    color = _use_color(no_color)
    prefix = "\033[33mWarning:\033[0m" if color else "Warning:"
    print(f"{prefix} {msg}", file=sys.stderr)

def info(msg: str, *, quiet: bool = False) -> None:
    if not quiet:
        print(msg, file=sys.stderr)


# ── i/o helpers (or extract to lib/io.py) ────────────────────────────────────

def open_input(path: str):
    """Open path or '-' for stdin."""
    if path == "-":
        return sys.stdin
    p = Path(path)
    if not p.exists():
        err(f"file not found: {path}")
        sys.exit(1)
    return p.open()

def atomic_write(path: str, content: str) -> None:
    """Write to a temp file then rename — avoids partial writes."""
    p = Path(path)
    with tempfile.NamedTemporaryFile("w", dir=p.parent, delete=False, suffix=".tmp") as f:
        f.write(content)
        tmp = f.name
    Path(tmp).replace(p)


# ── core logic ────────────────────────────────────────────────────────────────

def process(text: str) -> dict:
    """Pure business logic — no sys.exit, no print."""
    return {"lines": text.splitlines(), "count": len(text.splitlines())}


# ── CLI ───────────────────────────────────────────────────────────────────────

def build_parser() -> argparse.ArgumentParser:
    p = argparse.ArgumentParser(
        prog="my-tool",
        description="Process a file and write results.",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
examples:
  %(prog)s file.txt
  %(prog)s --dry-run file.txt
  %(prog)s --json file.txt | jq .
  cat file.txt | %(prog)s -
        """,
    )
    p.add_argument("input", metavar="FILE", help="input file (- for stdin)")
    p.add_argument("-o", "--output", default="-", metavar="FILE",
                   help="output file (default: stdout)")
    p.add_argument("-n", "--dry-run", action="store_true",
                   help="preview without writing")
    p.add_argument("--json", dest="as_json", action="store_true",
                   help="output as JSON")
    p.add_argument("-q", "--quiet", action="store_true",
                   help="suppress non-essential output")
    p.add_argument("--no-color", action="store_true",
                   help="disable color output")
    p.add_argument("--version", action="version", version="%(prog)s 1.0.0")
    return p


def main() -> None:
    # If stdin is a TTY and no args given, show help
    if len(sys.argv) == 1 and sys.stdin.isatty():
        build_parser().print_help(sys.stderr)
        sys.exit(2)

    args = build_parser().parse_args()
    nc = args.no_color

    try:
        with open_input(args.input) as f:
            text = f.read()
    except OSError as e:
        err(str(e), no_color=nc)
        sys.exit(1)

    result = process(text)

    if args.dry_run:
        info(f"[dry-run] would write {result['count']} lines", quiet=args.quiet)
        return

    output_text = json.dumps(result, indent=2) if args.as_json else "\n".join(result["lines"])

    if args.output == "-":
        print(output_text)
    else:
        try:
            atomic_write(args.output, output_text + "\n")
            info(f"wrote {args.output}", quiet=args.quiet)
        except OSError as e:
            err(str(e), hint=f"chmod +w {args.output}", no_color=nc)
            sys.exit(1)


if __name__ == "__main__":
    main()
```

---

## See Also

- `references/clig-checklist.md` — condensed CLIG do/don't checklist
- Full guide: https://clig.dev
