# Understanding What to Change with AI - Exercise

## Exercise 1: Code Readability Improvement (Java) - UserMgr class

Renamed: UserMgr->UserManager, U->User, u_list->users, a()->registerUser(),
f()->findUserByUsername(), db->connection, nu->newUser, res->insertSucceeded.

Extracted: the duplicate linear-search-by-username loop appeared in BOTH a() and
f() - consolidated into one shared private helper. Validation logic (3 unrelated
checks in one unnamed condition) split into named checks.

REAL FINDING (not just readability): the SQL is built via raw string
concatenation - "INSERT INTO users VALUES ('" + un + "', ...)" - a genuine SQL
injection vulnerability, not a style issue. A username/email containing a single
quote could break out of the string. Flagged this as a security bug requiring a
parameterized query fix, separate from and more important than the renaming.
Also flagged: password stored as a plain field with no indication of hashing.

Insight: renaming forced reading the logic closely enough to notice the SQL
injection - the abbreviated original names were arguably helping obscure it.

## Exercise 2: Function Refactoring (Python) - process_orders

5 responsibilities identified in one function: validation, pricing calculation,
inventory mutation, result/error aggregation, orchestration.

Decomposed into: validate_order(), calculate_shipping(), calculate_order_pricing(),
with process_orders() reduced to orchestration only.

REAL FINDING: the domestic shipping logic (`if price < 50: shipping = 5.99`) has
an IMPLICIT free-shipping-over-$50 rule - if price >= 50, shipping silently stays
at its initialized 0. Almost certainly intentional business logic, but nothing in
the original code names this as deliberate - a future reader could "fix" it as a
bug. Recommended a named constant (FREE_SHIPPING_THRESHOLD) and explicit comment.

Also flagged: process_orders mutates the inventory dict it receives as an
argument - a side effect on caller-owned data not obvious from the signature.

Also flagged: 0.9 discount and 0.08 tax are magic numbers, should be named
constants to avoid future duplication of the same hardcoded values.

## Exercise 3: Code Duplication Detection (JavaScript) - calculateUserStatistics

Identified 2 duplicated patterns, each repeated 3x (once per field: age, income,
score): sum-and-average loops, and find-maximum loops.

Consolidated into 2 generic helpers (average(), highest()) plus one data-driven
loop over a `fields` array - adding a 4th tracked statistic becomes a 1-line
change instead of copy-pasting 2 more loops.

Readability trade-off noted: Math.max(...values) is idiomatic but potentially
less immediately clear to a junior-heavy team than an explicit loop - flagged as
a genuine judgment call, not an automatic win, depending on team experience level.

## Reflection Questions

**Which prompting strategy was most useful, and why?**
Function Refactoring (Exercise 2) - decomposing by RESPONSIBILITY required
tracing actual business logic closely enough to surface the implicit
free-shipping-over-$50 rule, something a pure readability or duplication pass
wouldn't have been positioned to catch.

**Improvements I might not have thought of unprompted?**
Treating the SQL string concatenation in Exercise 1 as a first-class finding
rather than a readability nitpick - it was adjacent to, not strictly inside, the
prompt's stated scope (naming/readability), but was the most important catch.

**Anything I'd disagree with?**
The Math.max(...values) suggestion in Exercise 3 - more concise isn't
automatically better if it reduces clarity for the specific team maintaining it.
Context-dependent, not a universal improvement.

**Adapting these prompts for a specific codebase?**
Always supply the team's actual naming conventions/style guide explicitly (as
Prompt 1's template already requests) - generic "good naming" advice can
conflict with an established codebase's real conventions.

**Safeguards before applying AI-suggested refactoring to production code?**
Tests must exist and pass before AND after the refactor (explicit chapter
pitfall). Each suggested change should be understood well enough to explain WHY
it's better, not accepted just because AI proposed it. Security-adjacent findings
(like Exercise 1's SQL injection) need independent verification and fixing, not
bundling into an incidental rename.