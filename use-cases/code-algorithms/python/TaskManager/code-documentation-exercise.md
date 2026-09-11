# Code Documentation Exercise — Task List Merging

**Code documented:** `merge_task_lists()` and `resolve_task_conflict()` from `task_list_merge.py`
**Why this code:** Already deeply understood from the Algorithm Deconstruction exercise
(including the deletion-gap finding) - documenting it accurately, not from a cold read.

## Prompt 1 output: Comprehensive function documentation

### merge_task_lists()
Merges two task dictionaries from separate sources with per-field conflict resolution.
Computes the union of task IDs across local_tasks and remote_tasks. Tasks present in
only one source are copied to the other side unchanged. Tasks present in both are
reconciled field-by-field via resolve_task_conflict().

Args:
- local_tasks (dict[str, Task]): Tasks from the local store, keyed by task ID.
- remote_tasks (dict[str, Task]): Tasks from the remote store, keyed by task ID.

Returns: tuple[dict, dict, dict, dict, dict] - (merged_tasks, to_create_remote,
to_update_remote, to_create_local, to_update_local)

Raises: Does not raise directly; assumes valid dict[str, Task] inputs. Malformed
values surface as AttributeErrors inside resolve_task_conflict().

Important notes: Does NOT handle deletions. A task removed from one source but still
present in the other is treated as "new" and recreated on the side it was deleted
from. Callers needing deletion sync must track tombstones separately.

### resolve_task_conflict()
Reconciles two versions of the same task into a single merged Task, using a different
resolution rule per field group:
- title/description/priority/due_date: most-recently-updated version wins
- status: completion takes precedence over recency (DONE always wins if only one
  side is DONE); falls back to recency only if both are non-DONE but differ
- tags: unioned from both sides, never removes a tag present on either side
- updated_at: set to the later of the two input timestamps

Args:
- local_task (Task), remote_task (Task)

Returns: tuple[Task, bool, bool] - (merged_task, should_update_local, should_update_remote)

Raises: Assumes both tasks have non-None updated_at; a None value raises TypeError
on comparison.

Important notes: merged_task starts as a deep copy of local_task, so untouched fields
silently keep LOCAL's value. Status resolution has 3 branches; only the third
re-checks timestamps - easy to misread as symmetric with the other rules.

## Prompt 2 output: Intent and logic explanation

High-level purpose: Two-way sync between independently-editable task stores,
reconciling divergent state into one agreed version plus a work list of what to
push where.

Step-by-step logic: Union all IDs -> classify each as local-only/remote-only/in-both
-> trivial copy for the first two -> delegate to field-specific rules for the third.

Assumptions/edge cases identified:
- Assumes updated_at is always populated and directly comparable - no clock-skew
  handling between sources
- Assumes presence-in-only-one-dict always means "created there," never "deleted
  from the other" - the deletion/tombstone gap
- Not a neutral 50/50 merge - local is the base (deepcopy), so untouched fields
  default to local's value

Suggested inline comments:
- Above merged_task = copy.deepcopy(local_task): "# Base case: local wins unless
  overridden below"
- Above the status branches: "# NOTE: completion overrides even a more recently-
  updated non-done status"
- Above the tags union line: "# Tags only ever grow via merge - a removed tag
  reappears from whichever side still has it"

Potential improvements: Add deletion tombstones; make "local wins by default"
explicit; document the three different per-field-group rules in a top-level comment.

## Comparing the two approaches

Prompt 1 produced structurally complete docs (params, returns, exceptions, example) -
good for a reference/API-lookup use case. Prompt 2 surfaced the assumptions and hidden
behavior (local-as-base, non-symmetric resolution) that Prompt 1's format doesn't have
a natural place for. Neither alone was sufficient - Prompt 1 without Prompt 2 would
have shipped accurate-but-shallow docs that don't warn a future developer about the
implicit local-wins default or the deletion gap.

## Final combined documentation

Kept Prompt 1's structure (Args/Returns/Raises/Example) as the skeleton, and folded
in Prompt 2's assumptions and inline-comment suggestions under "Important notes" in
each docstring above - this is the version that would actually ship in the codebase.

## What was learned

Most challenging for the AI: accurately flagging the deletion gap and the
local-wins-by-default behavior required ALREADY knowing about them from prior
analysis. A generic first-pass Prompt 1 run on this code, with no prior context
supplied, would likely describe surface behavior only and miss both.

Additional info needed in prompts: had to explicitly supply the earlier analysis
(three-branch status logic, deletion gap) since Prompt 1's phrasing alone tends
toward structural completeness, not deeper hidden-assumption surfacing - that's
specifically what Prompt 2 is for.

How I'd use this in my own projects: Prompt 1 for fast docstring scaffolding on
unfamiliar or new functions, but never trust it alone for anything with hidden
business rules - always follow with Prompt 2 (or manual review) before publishing
documentation for code with non-obvious behavior.