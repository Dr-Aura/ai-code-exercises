# Using AI to Help With Testing - Exercise

## Part 1.1: Behavior Analysis 

Initial understanding (before AI questioning): calculate_task_score() computes an
urgency number from priority, due-date proximity, completion status, tags, and
recency of update.

AI's questions surfaced 3 gaps:
1. Priority weights (LOW=1, MEDIUM=2, HIGH=4, URGENT=6) are NOT evenly spaced -
   URGENT isn't proportionally double HIGH the way MEDIUM is double LOW
2. "Due soon" is actually 4 separate threshold bands (overdue/+35, today/+20,
   within 2 days/+15, within a week/+10) - nothing beyond a week gets any bonus
3. No due date at all silently skips the entire due-date block - no bonus, no
   error, nothing

Additional behaviors missed entirely:
- Status affects score in 2 directions (DONE: -50, REVIEW: -15) but TODO/
  IN_PROGRESS get no adjustment
- Tag matching is a hardcoded, case-sensitive exact-match list
- Recency bonus uses `< 1` day (strictly "today"), not `<= 1`
- Invalid/unrecognized priority silently contributes 0 via .get() fallback,
  no error raised

Edge cases identified (mine + AI's additions):
- No due date at all
- Exactly due today (boundary)
- Multiple matching tags (double-counting?)
- DONE + overdue simultaneously (do both penalty and bonus apply?)
- Exact boundary values (days_until_due == 2 vs 3, == 7 vs 8)
- Negative days_since_update (clock skew / future updated_at)
- Empty tags list
- Timezone-naive vs aware datetime mismatch (would raise TypeError, not wrong answer)

First test to write, and why: the base case with NO modifiers active (MEDIUM
priority, no due date, TODO, no tags, stale update) - isolates the priority_weights
lookup alone before any other rule can interfere, giving a stable foundation.

## Deliverable: 6 real, executed test cases

Implemented and RAN against the actual calculate_task_score() function (not just
described) using Python's unittest:

1. Base case (no modifiers) -> expected exactly 20 -> PASS
2. Due-date boundary: 2 days vs 3 days out -> expected 35 vs 30 -> PASS
3. DONE + overdue simultaneously -> expected 20+35-50=5 (BOTH bonus and penalty
   apply, confirmed - this matches an earlier finding from a prior exercise about
   status handling) -> PASS
4. Invalid/unrecognized priority -> expected 0 (silent fallback via .get()) -> PASS
5. Tag case sensitivity: "blocker" vs "Blocker" -> expected +8 bonus only on exact
   lowercase match -> PASS
6. No due date at all -> expected 20, no crash -> PASS

ALL 6 TESTS PASSED - every prediction from the behavior-analysis conversation was
empirically confirmed against the real function, not just theorized.

## What was learned from making this "legit" (actually running tests)

The behavior-analysis conversation alone would have produced a plausible-sounding
list of test cases - but actually RUNNING them against the real function is what
turned "I think this is how it behaves" into "I've confirmed this is how it
behaves." Test 3 in particular (DONE + overdue) was a genuine open question during
analysis - both interpretations (only one modifier applying vs both applying) were
plausible from reading the code alone, and only execution settled it definitively
(both apply, additively).