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


## Part 1.2: Test Planning (all three functions)

AI's questions surfaced: sort_tasks_by_importance should be tested for ORDERING
only (using known/pre-computed scores), not re-testing calculate_task_score's
internal logic. get_top_priority_tasks needs a limit > len(tasks) case (should
return everything, not error) and an empty-list case.

3 tests written and run: descending-sort verification, limit-exceeds-available,
empty list. ALL PASSED.

## Part 2.1: Improving a Single Test

Before: `assertTrue(calculate_task_score(task) > 0)` - passes even if weights are
completely wrong, verifies almost nothing.
After: `assertEqual(calculate_task_score(task), 20)` - exact value, would catch
any change to the weight/scoring logic.
Both versions run and confirmed the before/after difference in what they actually verify.

## Part 2.2: Learning From Examples (due-date logic) - REAL BUG FOUND

Wrote precise boundary tests for due-date thresholds (overdue, exactly 7 vs 8
days). The "due exactly today" test FAILED: expected 40, got 55.

Root cause (confirmed via direct investigation): task.due_date is set once at
task creation; calculate_task_score() calls datetime.now() again later. ANY
elapsed time between the two calls makes (due_date - now()) slightly NEGATIVE,
and Python's timedelta.days FLOORS toward negative infinity - so even a
microsecond of elapsed time turns "due today" into "-1 days" (treated as overdue,
+35, instead of due-today, +20).

This is a genuine, previously undiscovered bug in calculate_task_score() - not a
test-writing mistake. Confirmed independently: timedelta(microseconds=-500).days
== -1. Corrected the test to document this instability explicitly rather than
asserting a single "correct" value that the function cannot reliably produce.

Checked whether the same flooring quirk affects days_since_update too: it DOES
exist there as well (updated_at even slightly in the future produces -1 days),
but does NOT produce a wrong OUTCOME there, because the comparison (`< 1`) treats
-1 and 0 identically (both grant the recency bonus). Same underlying language
quirk, different downstream impact depending on how the value is used - documented
as an honest finding rather than forcing a fake bug to match the exercise's
JS/Java-specific premise (Python's version doesn't have the ms-division bug
described, since it already uses .days directly).

## Part 3.1: TDD for New Feature - assignee score boost (+12)

Before writing any test, checked the Task model: NO assignee/current-user concept
exists at all. This is itself a finding worth documenting, not skipping past -
the feature requires a model change, not just a function change.

RED: wrote 2 tests referencing task.assigned_to and a current_user_id parameter
that didn't exist yet. Ran them - failed with TypeError (unexpected keyword
argument), confirming a genuine RED state for the right reason.

GREEN: added current_user_id=None parameter to calculate_task_score() and a
getattr(task, "assigned_to", None) == current_user_id check granting +12. Ran
tests again - both passed.

REFACTOR SAFETY CHECK: ran the full existing suite after the change - no
regressions (16/16 passing at that point, before Part 4 was added).

## Part 3.2: TDD for Bug Fix - days_since_update calculation

The exercise's described bug (ms-division in JS, wrong ChronoUnit in Java)
doesn't apply to this Python implementation, which already correctly uses
`.days`. Rather than forcing an artificial bug to match the prompt, investigated
whether the REAL timedelta-flooring bug found in Part 2.2 also affects this
calculation - confirmed it exists here too, but does not change the outcome,
since the `< 1` comparison treats a floored -1 the same as 0.

## Part 4.1: Integration Testing

2 integration tests written exercising calculate_task_score, sort_tasks_by_importance,
and get_top_priority_tasks together on realistic task lists:
1. Confirmed a DONE URGENT task never outranks active tasks despite raw priority,
   because the -50 penalty dominates - validates a multi-factor tradeoff finding
   from an earlier exercise (Domain Model, code-algorithms)
2. Confirmed get_top_priority_tasks respects COMPUTED score order, not input list order

Both passed.

## Final verification
Ran the complete test suite across all parts together: 18 tests, ALL PASSED,
zero regressions from the new assignee-boost feature.

## Overall Reflection

The guided-questioning approach (AI asking questions rather than generating tests)
consistently surfaced gaps in my first-pass understanding before any code was
written - the invalid-priority fallback, the case-sensitive tag matching, and
critically, checking whether the Task model even SUPPORTED the requested feature
before attempting to test it.

The single most valuable moment was Part 2.2's test FAILURE - writing a precise
boundary test (rather than a vague "due soon scores higher" test) is what
surfaced a genuine, real bug in calculate_task_score() that none of the previous
exercises' code-reading had caught. This validates the module's core principle:
guided practice that makes you write and RUN the actual tests finds things that
discussion and analysis alone do not.