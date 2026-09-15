# Applying AI to Deepen Programming Language Understanding (Python)

## Activity 1: Idiomatic Code Transformation

**Before (21 lines):** get_top_scorers() used a manual index loop
(`for i in range(len(students))`), manual dict-building field by field, and a
hand-written insertion sort.

**After (6 lines):** Replaced with enumerate() in a list comprehension for
filtering, and sorted(key=..., reverse=True) for ordering.

**Verified:** ran both versions against 5 test cases including empty input,
boundary min_score, and a TIE case (two equal scores) specifically to check
that the manual insertion sort's stability matched Python's built-in stable
sort. ALL 5 MATCHED exactly.

**3 key learnings:**
1. Manual index loops (`for i in range(len(x))`) are almost always a smell
   when both index and value are needed - enumerate() exists for exactly this.
2. Reimplementing sorting by hand is a red flag, not a feature - I'd written
   an accidental O(n^2) insertion sort without registering that's what it was;
   sorted() is both shorter AND asymptotically better (Timsort, O(n log n)).
3. Stability guarantees are easy to get wrong by hand and must be verified,
   not assumed - required a deliberate tie-case test to confirm behavior
   actually matched, not just "looks the same."

## Activity 2: Code Quality Detective

**Code reviewed:** fetch_user_orders() - representative "3+ months old" style
code with: mutable default argument (orders=[]), bare except:, global mutable
cache with no invalidation, manual sum loop, magic numbers (100, 0.1), and
undocumented input mutation.

**Empirically demonstrated the most dangerous smell** (not just described it):
built an isolated example showing a list default argument persisting across
calls - `target=[]` is evaluated ONCE at function definition time, so a
second, independent-looking call inherited state from the first
(['a'] -> ['a', 'b']). Confirmed this exact landmine exists in
fetch_user_orders()'s orders=[] parameter.

**Checklist created for future code reviews:**
- Mutable default argument on any function signature?
- Bare except: with no exception type?
- Global mutable state with no invalidation strategy?
- Unnamed numeric literal in business logic?
- Function mutates its input arguments without that being obvious?
- Manual loop duplicating a built-in (sum, max, sorted)?

**3 key learnings:**
1. The mutable default argument bug is invisible until demonstrated - reading
   `orders=[]` alone didn't make the danger click; watching ['a'] survive into
   an unrelated second call did.
2. Bare except: isn't caution, it's a bug generator - it feels defensive but
   actively converts real bugs into silent wrong answers, strictly worse than
   a loud crash.
3. A checklist built from a bug I personally demonstrated sticks far better
   than one copied from a style guide - I'll remember "mutable default
   arguments" because I watched it fail, not because a linter flagged it.

## Activity 3: Understanding Language Feature - Decorators

**Implemented:** a @retry(max_attempts, delay_seconds) decorator with a
practice function (flaky_data_fetch) that fails twice then succeeds - ran it
and confirmed exactly 3 attempts were made before returning success.

**Verified functools.wraps matters, not just theoretically:** checked
flaky_data_fetch.__name__ and __doc__ after decoration - correctly preserved
as "flaky_data_fetch" (not "wrapper") specifically because @functools.wraps
was included in the inner wrapper.

**3 practical use cases identified:** retry logic for flaky network/DB calls,
timing/profiling for identifying slow pipeline steps, caching/memoization for
expensive repeated computations (directly relevant to future data-science work).

**Practice project idea:** a small data pipeline (load -> clean -> aggregate)
with each step wrapped in both @retry and @timing decorators.

**3 key learnings:**
1. A decorator is just a function returning a function - mentally expanding
   `@retry(...)` to `flaky_data_fetch = retry(...)(flaky_data_fetch)` made the
   mechanism stop feeling mysterious.
2. A "decorator factory" (one that takes arguments) needs one extra layer of
   nesting than expected - confirming the layer count required an actual test
   run, not just reading about the pattern.
3. Skipping functools.wraps silently breaks function metadata - confirmed
   directly: without it, __name__ would report "wrapper" instead of the real
   function name, a genuinely confusing bug to hit later in stack traces or logs.

## Cross-activity theme

All three activities surfaced the same pattern: a bug or improvement that
looks obvious/theoretical when just described becomes genuinely convincing
only once empirically demonstrated - the tie-order test (Activity 1), the
isolated mutable-default reproduction (Activity 2), and the __name__ check
(Activity 3) were each the moment understanding actually solidified, not the
initial explanation.