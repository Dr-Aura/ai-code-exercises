# Code Readability Challenge

**Example chosen:** Example 1 - Cryptic Variable Names (JavaScript, inventory processing)

## Step 1: Understanding the original code (before renaming anything)

Ran the original unit tests first - all 3 passed against the unmodified original.
Traced what p(i, a, q) actually does before touching names: takes a list of
requested items, an inventory array, and a fixed quantity to request of EVERY
item in the batch. For each requested item, searches inventory by ID; if found
with sufficient stock, adds to a results list, adds to running total cost, and
decrements inventory. Logs a message for items not found. Returns
{s: successfulItems, t: totalCost}.

## Prompt 1 applied: Code Readability Improvement

Renamed:
- p(i, a, q) -> processInventoryRequest(requestedItems, inventory, quantityPerItem)
- r -> fulfilledItems, t -> totalCost
- c -> requestedItem, f -> itemFound
- Return shape {s, t} -> {fulfilledItems, totalCost}

Kept unchanged: loop counters j, k - these are only ever used as array indices,
never referenced for their own meaning, so renaming them (e.g. to
requestedItemIndex) would add length without adding clarity. This follows the
chapter's own "Excessive Abstraction" pitfall guidance: aim for clarity over
cleverness, and sometimes the simpler original is fine.

Added a JSDoc comment documenting parameters, return shape, and the important-
but-non-obvious behavior that `inventory` is mutated in place (a real side
effect a caller needs to know about, easy to miss in the unnamed original).

## Step 2: Verification - re-ran tests against refactored version

Since the return object's OWN keys were renamed (s->fulfilledItems, t->totalCost)
as part of the readability improvement, the test assertions needed corresponding
updates to check the new key names - this is an intentional API surface change,
not a bug. Adapted the test file accordingly and re-ran.

RESULT: All 3 tests PASSED against the refactored version, with identical
underlying logic and identical numeric results (same totals, same inventory
mutations, same "not available" logging) - only the naming changed.

## Reflection Questions

**How much easier is the code to understand now?**
Significantly - `processInventoryRequest(requestedItems, inventory,
quantityPerItem)` reads as a sentence describing what's happening; `p(i, a, q)`
requires reading the entire function body just to guess what's being processed.

**What readability issues did AI-style analysis catch that might be missed?**
The `inventory` mutation side effect - the original code silently modifies its
second argument in place (`a[k].q -= q`), which is easy to miss when scanning
a function using single-letter names, since there's no visual/semantic cue that
`a` is being changed rather than just read.

**What did AI analysis miss that a human might notice?**
The function name choice itself - "processInventoryRequest" is one reasonable
name, but a reviewer familiar with the domain might prefer something like
"fulfillBulkOrder" if this represents an order-fulfillment step rather than
generic "processing." Naming that depends on business context beyond what's
in the code itself is something no amount of code-reading alone resolves.

**Which improvement had the biggest impact?**
Renaming the return object's own keys (s/t -> fulfilledItems/totalCost) - this
matters more than the internal variable names, because the return shape is the
function's actual PUBLIC interface; anyone calling this function benefits from
clear keys regardless of whether they ever read the function's internals.

**How did the improved names change understanding of the code's purpose?**
The original gave zero indication this was about inventory/fulfillment at all -
`p`, `i`, `a`, `q` could be literally any domain. The renamed version is
self-describing enough that the accompanying JSDoc comment mostly restates
what the code already makes clear, rather than being the only source of that
information.

**Readability patterns to apply going forward:**
1. Prioritize renaming a function's PARAMETERS and RETURN VALUE keys first -
   these form the actual contract other code depends on, more so than internal
   variables.
2. Don't rename loop counters used purely as array indices - clarity has a
   ceiling, and past that point renaming adds noise, not signal.
3. Always re-run existing tests after a rename that touches a return value's
   shape, since API-surface renames require updating callers/tests
   deliberately, not just relying on "renaming can't break anything."