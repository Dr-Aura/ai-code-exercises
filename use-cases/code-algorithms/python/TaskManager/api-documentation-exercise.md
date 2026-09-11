# API Documentation Exercise — User Registration Endpoint

**Endpoint chosen:** POST /api/users/register (Flask example, provided starter code)

## Prompt 1 output: Comprehensive endpoint documentation
See full table of params, response format, and error codes.

Important finding: registration succeeds even if the confirmation email fails to
send (exception caught, logged, swallowed) - intentional, not a bug.

GENUINE BUG FOUND: email-uniqueness check runs against raw-case input
(User.query.filter_by(email=data['email'])), but stored emails are lowercased
(data['email'].lower()) only AFTER that check. A different-case variant of an
existing email (e.g. User@Example.com vs stored user@example.com) bypasses the
duplicate check entirely, allowing a second account for the same address.

No rate limiting visible on this public registration endpoint - flagged as a gap.

## Prompt 2 output: OpenAPI conversion
Converted to a full OpenAPI 3.0 spec: paths, request body schema (required fields,
minLength on password), response schemas per status code (201/400/409/500), and
reusable User/Error component schemas.

## Prompt 3 output: Developer usage guide
Written for junior devs integrating client-side. Covered: no-auth requirement,
request format, the "first missing field only" 400 behavior (a debugging gotcha),
handling 201/409/500 responses distinctly, an explicit warning about the
case-sensitivity bug found in Prompt 1 (don't rely on server-side dedup catching
case variants), and a working Python example.

## What was learned

Most challenging to document: the case-sensitivity duplicate-check bug. Prompt 1's
structured format (params/response/errors) doesn't have a natural home for a subtle
ordering bug like this - it only surfaced by reading the validation sequence
carefully (check-then-lowercase, in that order) rather than just listing what each
line does.

Prompt adjustments: had to explicitly ask Prompt 1 to note "important notes or edge
cases developers should be aware of" - without that, structured docs would have
listed the error codes correctly but missed the underlying bug entirely, since it's
not a distinct error path, just an ordering flaw in existing logic.

Most effective format: Markdown (Prompt 1 output) for the day-to-day contributor
reference; OpenAPI for anything that needs to be machine-readable/importable into
tooling (Postman, codegen); the usage guide (Prompt 3) for onboarding a new
integrator quickly - each serves a different audience, not different quality tiers
of the same thing.

How I'd use this in my own projects: run Prompt 1 first, since its "edge cases"
field consistently forces a re-read of the logic (not just the happy path) - that's
where real bugs get found, not from the OpenAPI or usage-guide passes.