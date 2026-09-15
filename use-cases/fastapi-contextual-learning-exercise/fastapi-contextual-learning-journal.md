# Contextual Learning with FastAPI

## Part 1: Framework Comparison - translation table

| Flask/Django concept | FastAPI equivalent | Key difference |
|---|---|---|
| Flask @app.route | @app.get/@app.post etc. | Split by HTTP method, not one generic route |
| Flask manual request.get_json() + validation | Pydantic model as function parameter | Validation automatic from type hint, no manual parsing |
| Flask Blueprints | APIRouter | Included via app.include_router(), closer to Django's include() |
| Django middleware | Depends() dependency injection | Middleware runs on EVERY request; Depends() is per-route, composable |
| Django Forms/serializers | Pydantic models | Django's is framework-specific; Pydantic is general-purpose, reusable outside web context |

## Part 2: Design Philosophy Summary

Pydantic chosen over custom validation: reuses an independently-maintained,
general-purpose library - same models work outside any web context.
Automatic docs generated directly from type hints/models - can't silently
drift out of sync with actual behavior the way hand-written docs can (directly
relevant given the LAST exercise's finding of stale/deprecated sample code).
Type hints serve 3 jobs at once (docs, runtime validation, static analysis)
instead of 3 separate mechanisms. Async-first (Starlette/ASGI) for
high-concurrency I/O-bound workloads vs Flask's sync WSGI model.

## Part 3: Applied Contextual Learning - JWT Authentication

Built and tested the full JWT auth flow, per the exercise's provided sample
code - and found THREE real, verified issues along the way, continuing the
pattern from the previous FastAPI exercise:

REAL FINDING 1 (missing dependency): the exercise's documented install
command (`pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt]`)
is missing `python-multipart`, which OAuth2PasswordRequestForm requires
internally for form-data parsing. Confirmed via a real RuntimeError when
omitted.

REAL FINDING 2 (well-known ecosystem compatibility bug, not exercise-specific):
passlib 1.7.4 (last updated ~2020) is incompatible with bcrypt 4.1+, which
removed the `__about__` attribute passlib depends on internally. Confirmed
with a real AttributeError -> cascading ValueError ("password cannot be
longer than 72 bytes"). This is a widely-hit issue across the Python ecosystem
for anyone following JWT-auth tutorials today, not something specific to this
course's exercise.

FIX VERIFIED: pinning `bcrypt<4.1` resolves it - re-ran verify_password()
after the pin and got the correct True result.

Full end-to-end auth flow tested (5 real requests via TestClient):
- POST /token with correct credentials -> 200, valid JWT returned
- POST /token with wrong password -> 401, correct error
- GET /users/me/ with no token -> 401 "Not authenticated"
- GET /users/me/ with valid token -> 200, correct user data returned
- GET /users/me/ with garbage/invalid token -> 401 "Could not validate credentials"

ALL PASSED.

## Part 4: Mental Model Translation

Django views -> FastAPI path operation functions (direct equivalent)
Django models (ORM) -> NO direct equivalent - FastAPI is unopinionated about
persistence entirely; Pydantic models describe request/response SHAPE, not
storage
Django middleware -> FastAPI dependencies (opt-in per-route) + Starlette
middleware (for genuinely global concerns like CORS)
Django's automatic admin panel -> NO FastAPI equivalent - a real, meaningful
gap, not just a naming difference

## Reflection Questions

**How does FastAPI's auth approach compare to frameworks used before?**
The dependency-injection model (Depends(get_current_active_user)) makes auth
an explicit, visible, per-route opt-in rather than a global middleware config
- you can see directly in a route's signature that it requires
authentication, without checking a separate middleware registration file.

**What advantages does dependency injection provide for authentication?**
Composability - get_current_active_user itself depends on get_current_user,
which depends on oauth2_scheme - each layer is independently testable and
reusable (e.g. a route needing just "is this a valid token" vs "is this an
active, non-disabled user" can depend on different layers of the same chain).

**How does type hinting make security implementation clearer?**
`current_user: User = Depends(get_current_active_user)` documents, at the
function signature level, exactly what identity data a route has access to -
no separate lookup into a middleware-populated request context object needed.

**What patterns from other frameworks are visible in the JWT implementation?**
OAuth2PasswordBearer/OAuth2PasswordRequestForm follow the OAuth2 spec's
standard flow shape, which is framework-agnostic - the same conceptual flow
(exchange credentials for a token, present the token on subsequent requests)
appears in Django REST Framework's token auth and countless other systems;
FastAPI's version is just more explicit about the dependency chain.

## Cross-exercise theme

This is now the SECOND FastAPI exercise in a row where real, verified
version/dependency issues were found in provided sample code before it would
run correctly - reinforcing that "cross-reference against the actual
installed environment" isn't a one-time caution, it's a standing practice
needed every time tutorial code meets a real, current environment.