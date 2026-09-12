# Function Decomposition Challenge

**Function chosen:** validateUserData (JavaScript) - the only one of the 3 sample
functions fully self-contained with no external dependencies (DB, logger,
repository), making it possible to actually run and verify tests, per the
exercise's explicit instruction.

## Prompt 1 applied: Function Responsibility Analysis

9 distinct responsibilities identified in the single 150+ line function:
1. Required-field checking for registration
2. Username format/availability validation
3. Password strength + confirmation validation
4. Email format/availability validation
5. Date of birth validation (age boundaries)
6. Address validation (structure + country-specific postal codes)
7. Phone number validation
8. Custom/pluggable validation
9. Orchestration - deciding which checks apply and aggregating errors

## Decomposition

Extracted 10 focused helper functions (validateRequiredFields,
validateProfileRequiredFields, validateUsername, validatePassword,
validateEmail, validateDateOfBirth, validatePostalCode, validateAddress,
validatePhone, validateCustomFields), each taking only the specific data it
needs rather than the whole userData/options objects. Main validateUserData()
reduced to pure orchestration - calling helpers and concatenating their error
arrays.

## Verification: comparison testing (original vs refactored)

Built a test harness running BOTH the original and refactored implementations
against 31 diverse inputs (valid cases, one error per field, multiple
simultaneous errors, edge cases per field type) and comparing outputs exactly,
including crash-for-crash comparison (not just successful-output comparison).

RESULT: 31/31 test cases produced IDENTICAL output between original and
refactored versions, including 6 cases where BOTH versions crash with the
exact same error message. Confirms the refactoring is a pure structural change
with zero behavioral difference - correct AND buggy behavior both preserved
exactly.

## MAJOR FINDING: real, previously-hidden crash bug in the original code

6 of the 31 test cases crashed - not a test-writing mistake, a genuine bug
present in the ORIGINAL, shipped function. In profile-update mode
(isRegistration=false), requiredForProfile includes 'address'. The generic
required-field loop blindly calls `.trim()` on every field in that list,
assuming all of them are strings. But `address` is explicitly validated
elsewhere in the SAME function as an OBJECT (`typeof userData.address ===
'object'`), not a string.

Result: ANY profile update where address is provided as a non-empty object (its
own documented, expected shape) crashes with "userData[field].trim is not a
function" - before the function ever reaches its own dedicated address
validation logic. This means the profile-update path is fundamentally broken
for its own intended address format whenever the address is provided and
non-empty. This is a severe, production-impacting bug that testing surfaced
directly - it would not have been obvious from reading the code alone, since
the two conflicting assumptions about address's type are ~80 lines apart in
the original.

Because the refactored version preserves this bug identically (both crash the
same way), the decomposition is verified correct - but the underlying bug
still needs a real fix (separate from this decomposition exercise, since the
goal here was preserving behavior, not fixing it).

## Benefits gained from decomposition

- Each extracted function is independently testable (as proven by being able
  to unit-test validateUsername, validatePassword, etc. in isolation, rather
  than only through the full orchestrating function)
- validatePostalCode and validatePhone are genuinely reusable in any other
  form/validation context in the codebase - no dependency on the larger
  userData/options shape
- The bug found above became MUCH easier to isolate and explain precisely
  because validateProfileRequiredFields is now a small, nameable function -
  "the bug is in validateProfileRequiredFields" is a much clearer statement
  than pointing at a specific line number inside a 150-line function

## Reflection Questions

**How did breaking down the function improve readability and maintainability?**
Each helper function fits on one screen and has one clear name describing
exactly what it checks - a new developer can understand "what does username
validation do" by reading validateUsername() alone, without needing to hold
the entire original function's control flow in their head.

**What was the most challenging part of decomposing the function?**
Deciding exactly what each helper needed as parameters, rather than just
passing the entire userData/options objects everywhere (which would have been
easier but would have just moved the tangled coupling into more functions
instead of removing it). Password validation, for example, only needed
`password` and `confirmPassword` as two plain strings - not the whole userData
object.

**Which extracted function would be most reusable in other contexts?**
validatePostalCode(zip, country) - it has zero dependency on the "user" domain
at all and could validate a postal code anywhere in the codebase an address is
collected (billing, shipping, etc.), not just user profile addresses.