# CLIG Condensed Checklist

Source: https://clig.dev — read for full rationale.

## The Basics
- [x] Use an argument parsing library (don't hand-roll)
- [x] Exit 0 on success, non-zero on failure
- [x] Primary output → stdout
- [x] Logs, errors, progress → stderr

## Help
- [x] `-h` and `--help` always work, show concise help with no args if required
- [x] Lead with examples, not flag lists
- [x] Suggest corrections for typos
- [x] Link to web docs

## Output
- [x] Human-readable by default, machine-readable opt-in (`--json`, `--plain`)
- [x] Detect TTY; disable color/animation when piped
- [x] Respect `NO_COLOR` env var
- [x] `-q / --quiet` suppresses non-essential output
- [x] Tell the user when state changes
- [x] Use pager (`less -FIRX`) for large output, only on TTY

## Errors
- [x] Rewrite errors as human-actionable messages with hints
- [x] Group repeated errors under one header
- [x] Provide bug report path for unexpected errors

## Arguments & Flags
- [x] Prefer flags over positional args
- [x] Full-length versions for all flags (`-v` + `--verbose`)
- [x] Standard names: `-h`, `--version`, `-n/--dry-run`, `-f/--force`, `-q/--quiet`, `--json`
- [x] Default = right behavior for most users
- [x] Never *require* interactive prompts (always accept flag equivalent)
- [x] Confirm destructive ops; require `--force` or typed confirmation for severe ones

## Interactivity
- [x] Check TTY before prompting; skip prompts in non-interactive mode
- [x] `--no-input` disables all prompts

## Configuration (priority order)
1. Flags (highest priority)
2. Environment variables (`MYAPP_*`)
3. Project config file (`.myapp.toml` or similar)
4. User config (`~/.config/myapp/config.toml`, XDG-compliant)
5. System config (lowest priority)

## Environment Variables
- Use `MYAPP_` prefix for app-specific vars
- Document all env vars in help text
- Follow standard vars: `NO_COLOR`, `DEBUG`, `HOME`, `EDITOR`, `PAGER`

## Naming
- All lowercase, hyphen-separated: `my-tool`
- Subcommands: verb-noun pattern (`add-user`, `list-jobs`) or plain verbs (`push`, `pull`)

## Distribution
- Single binary or package-manager installable
- Provide shell completion scripts
- Include `--version` with the installed version

## NEVER
- [ ] Print to stdout what belongs on stderr (and vice versa)
- [ ] Use colors when piped or `NO_COLOR` is set
- [ ] Hang silently (no output for >2s without progress indication)
- [ ] Require positional args when flags would be clearer
- [ ] Mix log output with data output on stdout
- [ ] Auto-run dangerous corrected commands without confirmation
