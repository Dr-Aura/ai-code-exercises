# Design Pattern Implementation Challenge

**Pattern/example chosen:** Strategy Pattern - JavaScript shipping cost calculator
**Why this one:** Fully self-contained and runnable, allowing a real before/after
comparison test rather than just describing the refactor.

## Prompt 1 applied: Pattern Opportunity Identification

Structure identified: three parallel if/else-if blocks (one per shipping method),
each with its own nested country-based pricing AND its own dimensional-weight
surcharge rule. Adding a new method means adding a whole new block; adding a new
country means touching EVERY block. Classic Strategy pattern shape - each
shipping method is really an interchangeable pricing algorithm sharing the same
interface (package details + country -> cost).

Drawback flagged BEFORE refactoring: the overnight branch returns a STRING
("Overnight shipping not available...") for unsupported countries, while every
other path returns a numeric cost (via .toFixed(2), also technically a string,
but a numeric one). This return-type inconsistency needed to be preserved
exactly for a behavior-preserving refactor, even though it's arguably a
pre-existing design flaw.

## Prompt 2 applied: Pattern Implementation Guidance

Implemented 3 strategy objects (standardStrategy, expressStrategy,
overnightStrategy), each with a calculate(packageDetails, destinationCountry)
method - matching interface, different pricing logic. Main
calculateShippingCost() reduced to: look up the strategy by method name, call
it, and format the result - no per-method conditional logic left in the main
function at all.

Explicitly preserved two edge-case behaviors that a naive refactor might have
"fixed" away:
1. overnightStrategy still returns a raw string (not a number) for unsupported
   countries - documented with a comment explaining this is intentional
   behavior-preservation, not an oversight
2. An unrecognized shippingMethod value falls through to "0.00", matching the
   original's implicit `cost = 0` initialization behavior, made explicit here
   rather than left as an accident of variable initialization

## Verification: comparison testing (original vs refactored)

Built a comparison harness running 18 test cases through both implementations,
checking both VALUE and TYPE equality (important given the string/number
inconsistency being tested). Cases covered: all 3 methods x multiple countries
(including the "any other country" fallback branches), dimensional/large-package
surcharge boundaries, the overnight-unavailable string-return case, an
unrecognized shipping method, and a zero-weight edge case.

RESULT: 18/18 test cases produced IDENTICAL output (value AND type) between
original and refactored versions. Strategy pattern refactor verified to
preserve behavior exactly, including its pre-existing quirks.

## Benefits gained from the Strategy pattern refactor

- Adding a new shipping method (e.g. "economy") now means adding ONE new
  strategy object and one line in the shippingStrategies lookup - zero risk of
  accidentally modifying an unrelated existing method's logic
- Each strategy is independently unit-testable in isolation
  (standardStrategy.calculate(...) can be tested without touching express or
  overnight logic at all)
- The main calculateShippingCost() function is now trivial to read - it's
  purely "look up and delegate," with all the actual pricing complexity pushed
  into named, single-purpose strategy objects

## Reflection Questions

**How did implementing the pattern improve maintainability?**
The biggest change is BLAST RADIUS - in the original, a bug fix or rate change
to "standard shipping to Canada" required editing inside a large function
containing all 3 methods' logic, with real risk of an accidental edit bleeding
into an adjacent branch. In the refactored version, that same change is
isolated entirely inside standardStrategy, with zero possibility of touching
express or overnight code.

**What future changes will be easier because of this pattern?**
Adding new shipping methods (the core promise of Strategy), but also: a
future requirement like "let customers choose their shipping provider at
runtime" becomes trivial - shippingStrategies is already a lookup table keyed
by method name, ready to be extended or made pluggable/configurable.

**Unexpected challenges?**
Deciding whether to "fix" the overnight string-return inconsistency during the
refactor. The exercise's goal was behavior PRESERVATION, so the right call was
keeping the inconsistency and documenting it explicitly with a comment,
rather than silently improving it - a genuinely different task (fixing a bug)
was mixed into what looked like a pure structural refactor. Recognizing this
distinction, and choosing to defer the fix rather than bundle it in, was the
real work here.