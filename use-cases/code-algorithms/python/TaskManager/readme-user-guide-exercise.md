# README and User Guide Documentation Exercise — Task Manager

**Project documented:** The Task Manager CLI itself (code-algorithms starter project)
**Real finding before starting:** The EXISTING README's command examples are all wrong.
It documents `update-status`, `update-priority`, `update-due-date`, `add-tag`,
`remove-tag` - but the actual argparse subcommands (verified via grep on cli.py) are
`status`, `priority`, `due`, `tag`, `untag`. Every example command in the shipped
README would fail with "invalid choice" if run as written.

## Prompt 1 output: README.md (corrected)
Generated a full README - title, features, install (Python 3.11+, no deps), corrected
usage examples using verified real command names, configuration (tasks.json, no
config file), troubleshooting (including the command-name mismatch as a known gotcha,
full-file-rewrite behavior, UUID-not-sequential task IDs), test-running instructions,
contributing/license sections.

## Prompt 2 output: Step-by-step guide - "Creating and completing your first task"
5-step beginner guide: create -> confirm via list -> move to in_progress -> mark done
-> verify via show. Included the update-status vs status naming trap as an explicit
"common mistake" callout, plus valid status value list (exact strings, case-sensitive).

## Prompt 3 output: FAQ
Built entirely from previously-verified findings rather than generic guesses:
- Priority vs computed importance score distinction (calculate_task_score not
  exposed via CLI)
- Why "done" status is handled specially (completed_at side effect)
- The command-name mismatch (linking back to the real README bug found)
- No file locking / concurrent-run race risk
- storage.save() silent failure (can report success even if write failed)
- task_list_merge.py exists but isn't CLI-wired, and doesn't track deletions

## What was learned

Most challenging to document: NOT the code itself - it was noticing that the
project's own existing README was inaccurate. Documentation exercises assume you're
writing fresh docs; this one revealed you often need to verify EXISTING docs against
the real code first, since shipped documentation can silently drift out of sync with
the implementation.

Prompt adjustments: for Prompt 1, had to explicitly grep the actual cli.py subparsers
rather than trusting the existing README as a source - a generic run of Prompt 1
using the README as its "information" input would have faithfully reproduced the
same wrong command names.

Document structure learned: FAQ format was the most natural home for the deeper
findings from earlier exercises (silent save failures, merge deletion gap) - these
don't fit naturally into README's "how to use it" framing or the step-by-step guide's
single-task focus, but are exactly what an FAQ's "why does X happen" format is for.

How I'd use this in my own projects: treat "does the README's example commands
actually run" as a first-class documentation check, not an assumption - especially
before using an existing README as a prompt's source material, since AI will
faithfully reproduce whatever inaccuracies are already there unless told to verify
against the real code first.