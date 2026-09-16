# Understanding FastAPI Code Patterns

## Real finding before starting: 4th occurrence of Pydantic V1 syntax

Checked `class Config: orm_mode = True` from the sample code against installed
Pydantic 2.13.5 - confirmed TWO stacked warnings: the same class-based Config
deprecation found 3 times previously in this course, PLUS a SEPARATE
UserWarning specific to this case: "'orm_mode' has been renamed to
'from_attributes'". 4th distinct occurrence of outdated Pydantic syntax
across this course's FastAPI material.

## Part 1: Analyzing Complex Code

Repository pattern: separates data-access from business logic - swapping
storage backends only requires changing the Repository, not every caller.
Generic[T]: lets Repository be written once, reused for any model with full
type-checker support (UserRepository(Repository[User]) gets typed returns).
DI layers: get_db() -> get_current_user() (depends on oauth2_scheme AND
get_db()) -> route handlers (depend on get_current_user()) - each layer only
knows the layer directly below it.

RBAC decorator (requires_role): wraps a route, injecting its OWN
Depends(get_current_user) and checking is_superuser before calling through.

## Part 2: Tracing Execution Flow - with empirical verification, not just theory

Traced the full /admin/users/ request flow, then tested 3 specific questions
the trace raised rather than just describing them:

TEST 1: Does the decorator+FastAPI-Depends stacking pattern actually work
correctly? VERIFIED YES - built an isolated test, ran it, got correct 403 for
a non-admin user.

TEST 2 (non-obvious finding): does get_current_user get called TWICE per
request, since BOTH the requires_role decorator's own Depends() AND the
route's own Depends() parameter reference it? VERIFIED: called only ONCE.
This reveals FastAPI's dependency CACHING behavior - Depends() results are
cached per-request by default, so referencing the same dependency twice
doesn't re-execute it twice. Debunked an inefficiency concern I initially
suspected, and surfaced a genuinely useful, non-obvious fact about how
FastAPI's DI system actually works internally.

## Part 3: Simplifying Complex Concepts

asynccontextmanager + lifespan: replaces the older @app.on_event
startup/shutdown pattern - code before yield runs once at startup, after
yield runs once at shutdown, like try/finally around the app's whole lifetime.
TimingMiddleware: call_next(request) IS "continue processing" - code before
it runs pre-route, after runs post-route.
JWT flow: login exchanges credentials for a signed, self-contained token;
subsequent requests present the token, server verifies signature+expiry
without a session-store lookup.

## Part 4: Building Understanding Through Implementation - Audit Logging

Built a full audit-logging feature EXTENDING the same architectural patterns
already present (Repository, Service layer, DI) - not bolted on separately:
AuditLogRepository (same Repository shape), AuditService (same Service
shape), and a NEW `audited()` decorator following the exact same
decorator-with-Depends pattern as the existing requires_role().

Used an in-memory store instead of real SQLAlchemy/DB (the original sample's
get_db() is illustrative/non-functional as written - no real engine wired up)
to keep the implementation genuinely RUNNABLE and testable while preserving
the same architecture.

Full test (5 real requests via TestClient):
- Login as non-admin (alice) -> 200, audited (1 log entry)
- Login as admin (admin_bob) -> 200, audited (2 log entries total)
- Non-admin attempts admin endpoint -> 403, correctly NOT audited (confirms
  requires_role blocks BEFORE audited ever runs - decorator stacking order
  verified correct)
- Admin accesses admin endpoint -> 200, correctly audited (3rd log entry:
  "admin_bob list_all_users")
- Retrieve audit logs via new endpoint -> 200, all 3 entries returned correctly

ALL PASSED.

## Reflection Questions

**How did implementing this feature help understand the architecture?**
Building audited() as a near-mirror of requires_role() made the DECORATOR
pattern itself click - both wrap a function, inject their own dependency via
Depends(), and conditionally call through to the wrapped function. Once one
was understood deeply (via the empirical caching test in Part 2), writing
the second was mechanical, not mysterious.

**Which design patterns were most useful in the original code?**
The Repository pattern's separation of concerns - being able to write
AuditLogRepository as a near-parallel structure to UserRepository, with a
completely different backing store (in-memory list vs SQL), without touching
any of the Service or route layers, is the whole point of the pattern
actually paying off in practice.

**How would I explain Repository pattern and DI to a colleague?**
Repository: "your business logic never talks to the database directly, it
talks to a Repository object, so you can swap what's actually storing the
data without touching business logic." DI: "instead of a function creating
the things it depends on, FastAPI hands them in as parameters - and it's
smart enough to reuse the same result if two things in the same request ask
for the same dependency" (this last point specifically discovered, not just
read about, via the call-count test).

**How did tracing execution flow help find where to add code?**
Directly - seeing that login() was the natural point for a "login" audit
event, and that requires_role already established the pattern of
"decorator wraps route, injects Depends(get_current_user)," made audited()
an obvious, low-risk addition rather than a redesign.