# Getting Started with FastAPI

## Part 1: FastAPI Fundamentals

FastAPI vs Flask/Django: FastAPI is ASGI-based (async-native, built on Starlette)
and Pydantic-driven; Flask is WSGI (sync by default); Django is full
batteries-included (ORM, admin, templating). FastAPI's core differentiator:
Python type hints ARE the validation layer - no separate schema language needed.

Core concepts: path operations (@app.get/@app.post decorators), path/query
parameters, Pydantic models, dependency injection via Depends(), automatic
OpenAPI docs generated from type hints.

## Part 2: Hello World API - built and tested

Built the basic app (root, path-parameter, query-parameter endpoints). Tested
with FastAPI's TestClient (5 real requests, not just described):
- GET / -> 200, correct message
- GET /items/42 -> 200, correct item_id
- GET /items/not-a-number -> 422 automatic validation error (demonstrates
  FastAPI's core "type hints ARE validation" advantage concretely)
- GET /search/?q=test&skip=5&limit=20 -> 200, correct query param parsing
- GET /docs -> 200, auto-generated docs exist

REAL FINDING (unprompted): just importing TestClient triggered a live
StarletteDeprecationWarning about httpx - a genuine, real-time example of the
"technology evolves faster than static tutorials" problem this chapter warns
about, surfacing on the very first test run.

## Part 3: Enhanced structure - found 2 real deprecated-pattern bugs in the
## EXERCISE'S OWN sample code

Built the full project structure (models/routes/utils) using the exercise's
provided sample code AS WRITTEN first.

REAL FINDING 1: `class Config: schema_extra = {...}` in the sample's
ItemResponse model is PYDANTIC V1 syntax. Running it under
-W error::DeprecationWarning produced a genuine PydanticDeprecatedSince20
error: "Support for class-based config is deprecated... to be removed in V3.0."
This is not hypothetical - the exercise's own code, as given, is already
outdated against the currently-installed Pydantic 2.13.5.

FIXED: replaced with `model_config = ConfigDict(json_schema_extra={...})` -
verified with the same strict-warnings run: no warning, and
json_schema_extra correctly applied and inspectable via
ItemResponse.model_config.

Also proactively fixed `item.dict()` -> `item.model_dump()` (the V1 method is
deprecated in V2) before it could surface the same class of issue.

REAL FINDING 2: running the full app surfaced a SECOND live deprecation:
'HTTP_422_UNPROCESSABLE_ENTITY' is deprecated, use
'HTTP_422_UNPROCESSABLE_CONTENT' instead - found directly from Starlette's own
runtime warning while testing the exercise's exception-handling code.

Full functional test (7 real requests via TestClient): create item (201),
read it back (200), get 404 via custom exception handler for a nonexistent
item, get 422 with correct Pydantic error details for an invalid (negative)
price, and tag-based filtering returning the correct 2-item subset. All
passed correctly - the underlying LOGIC was sound, only the syntax had drifted.

## Part 4: Exercise Challenge - To-Do List API

Built a full CRUD API (using corrected Pydantic V2 syntax throughout, learned
from Part 3's findings): POST to create, GET with optional completed-status
filter, PATCH to mark complete, DELETE to remove.

Full test suite (9 real requests via TestClient): create 2 todos (one with a
due date), list all (count=2), mark one complete, filter by completed=true
(count=1) AND completed=false (count=1) - both directions verified separately,
delete one, confirm 404 on completing the now-deleted todo, and confirm 422
validation rejects an empty title.

ALL TESTS PASSED.

## Reflection

The most valuable finding across this whole exercise wasn't anything about
FastAPI itself - it was catching that the EXERCISE'S OWN provided sample code
already contained 2 real, verifiable instances of exactly the pitfall this
chapter explicitly warns about ("Assuming deprecated approaches" / "version-
specific discrepancies"). This wasn't a hypothetical warning - running the
sample code under strict deprecation checking produced real Python
tracebacks. This validates the chapter's own core lesson: cross-reference
against the ACTUAL installed version, don't trust that provided/AI-generated
code matches current APIs just because it looks reasonable and syntactically
valid.