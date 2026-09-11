# Algorithm Deconstruction Challenge — Task List Merging

**Algorithm chosen:** Task list merging (two-way sync), `task_list_merge.py`
**Why this one:** task_priority.py's scoring algorithm was already deeply explored in a
previous exercise. task_list_merge.py and task_parser.py were unexplored - merge logic
had genuine branching complexity worth deconstructing.

## Step-by-step breakdown

Purpose: two-way sync between a local and remote task store. Given both sides' task
dictionaries, produces one merged view plus four "what needs to happen" dictionaries
(create/update on each side).

Walkthrough with concrete values - task T1 exists on both sides:
- Local: title="Buy milk", updated_at=10:00, status=TODO
- Remote: title="Buy milk and eggs", updated_at=10:05, status=DONE

1. merge_task_lists() finds T1 in both -> routes to resolve_task_conflict()
2. merged_task starts as a DEEP COPY OF LOCAL (not remote) - this default matters
3. Remote is newer (10:05 > 10:00) -> title/description/priority/due_date overwritten
   with remote's -> should_update_local = True
4. Status check: remote is DONE, local isn't -> completion always wins, REGARDLESS OF
   TIMESTAMP -> status forced to DONE, should_update_local = True
5. Tags: union of both sets, never a removal
6. updated_at set to max() of both

## Control flow diagram
resolve_task_conflict(local, remote)
|
|- merged = deepcopy(local) <- default: local wins unless overridden
|
|- [1] Which is newer by updated_at?
| remote newer -> copy remote's title/desc/priority/due_date -> flag update_local
| local newer -> keep local's (already in merged) -> flag update_remote
|
|- [2] Status - THREE separate branches, timestamp-independent for the first two:
| remote=DONE, local!=DONE -> force DONE (ignores which is "newer")
| local=DONE, remote!=DONE -> keep local DONE
| both!=DONE but differ -> THIS branch alone re-checks timestamp
|
|- [3] Tags = union(local.tags, remote.tags) <- monotonic, never shrinks
|
`- [4] updated_at = max(local.updated_at, remote.updated_at)


## Insights and learning points

**MAJOR finding: deletions are never handled.** merge_task_lists() computes
all_task_ids = local keys UNION remote keys. There is no concept of a task having been
deleted on one side. If a task is deleted locally, it simply disappears from the local
dict - but is still sitting in the remote dict. On the next merge, that task ID exists
ONLY in remote from the algorithm's point of view, which routes into "Case 2: exists
only in remote -> add to local." A deleted task gets silently resurrected. Classic
sync-algorithm gap: without a tombstone/deletion marker, a merge can't distinguish
"never existed here" from "existed here and was removed."

**Secondary finding: completion status ignores recency entirely.** Every other field
(title, description, priority, due_date) is resolved by "most recent update wins." But
status has a special carve-out: if either side is DONE, DONE wins outright, even if the
non-DONE side was updated more recently. This is a deliberate business rule (finishing
a task shouldn't be "undone" by an older sync), but it's undocumented in the code and a
new reader could easily assume all fields follow the same recency rule.

**Secondary finding: tags only ever grow.** Union-based merging means a tag removed on
either side is never actually removed after a sync - it gets re-added from whichever
side still has it. No way to represent "I removed this tag" without also tracking removals.

**Process learning:** Prompt 1 (step-by-step, concrete example) was essential for
untangling resolve_task_conflict()'s field-by-field logic. But the deletion-handling gap
was only found by asking a level up - not "what does this function do" but "what INPUT
scenario does this function never consider" - i.e. testing the function's assumptions
against a case it doesn't handle, not just tracing the cases it does.

## Reflection Questions

**How did the AI's explanation change my understanding?**
Initially read resolve_task_conflict() as one big "most recent wins" function. Tracing it
step-by-step revealed it's actually three SEPARATE conflict-resolution rules bolted
together (recency for most fields, completion-always-wins for status regardless of
recency, and set-union for tags) - not one consistent rule.

**What was still difficult after the explanation?**
Confirming the deletion gap wasn't something the algorithm's own logic pointed to - it
only became visible by asking "what input case does this never consider," not by
reading more carefully.

**How would I explain this to another junior developer?**
"It's not really merging - it's letting the newer side win field-by-field, except
finishing a task always beats an older 'still in progress' state, and tags only ever
accumulate, never shrink."

**Did I test this understanding against AI?**
Yes - walked through a concrete two-sided conflict example line-by-line to confirm the
field-by-field behavior before trusting the summary.

**How might I improve the algorithm?**
Add a tombstone/deletion marker so removed tasks don't get resurrected on sync;
document the completion-overrides-recency rule explicitly as a comment, since it's a
deliberate but easily-missed exception to the rest of the function's logic.