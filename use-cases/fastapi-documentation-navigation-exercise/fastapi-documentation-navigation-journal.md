# Documentation Navigation for FastAPI

## Part 1: Documentation Summarization

Reading order: Tutorial (First Steps) -> Path Parameters -> Query Parameters ->
Request Body (Pydantic) -> Dependencies -> Security -> Bigger Applications
(multi-file) -> Background Tasks -> Testing.

5 most important sections for shipping quickly: Request Body/Pydantic,
Path & Query Params, Dependencies, Security/OAuth2, Error Handling.

Dependency injection summary: a callable FastAPI calls before your route runs,
injecting the result as a parameter - shared logic (auth, DB sessions,
pagination) written once, reused declaratively.

## Part 2: Documentation Deep Dive - Depends()

When to use: setup/teardown needs (DB session), shared across multiple routes
(auth check), needs independent test-time override. When NOT to: trivial
values with no reuse/testing need - just use a normal function call.

## Part 3: Concept to Code Translation

Tested the exercise's provided sample code AS WRITTEN first.

REAL FINDING (3rd occurrence of the SAME issue across this course's
exercises): `class Config: schema_extra = {...}` in UserCreate is Pydantic V1
syntax - confirmed with the same PydanticDeprecatedSince20 error as the
previous two FastAPI exercises. This pattern is clearly baked into the course
material's example code repeatedly, not a one-off typo.

REAL FINDING (4th unique dependency issue across the FastAPI exercises):
EmailStr requires the separate `email-validator` package, not mentioned
anywhere in this exercise's instructions. Confirmed with a real ImportError.

FIXED both issues (ConfigDict, installed email-validator) and built out all
5 documentation concepts fully, testing each with real requests (9 test cases
via TestClient):
1. Dependency injection (API key header check) - valid (200) and invalid (403)
2. Pydantic validation - valid user created (200), invalid user rejected (422)
3. Background tasks - VERIFIED the task actually executed (checked a log list
   populated by the background function, not just that the endpoint returned
   200 - confirms real execution, not just a successful-looking response)
4. Path/query/header/cookie parameters combined in one request - all correctly
   parsed and returned
5. Exception handling - 404 (not found), 400 (bad request), and 422
   (validation error via custom handler) - all three distinct error paths verified

ALL 9 PASSED.

## Part 4: Comprehensive Documentation Challenge - Mini Blog API

Built a full blog API applying everything verified across all three FastAPI
exercises: JWT auth (reusing the Part 3 pattern from the Contextual Learning
exercise), CRUD for posts, comments, and search - all using corrected
Pydantic V2 syntax throughout from the start (no deprecated patterns
reintroduced).

Full end-to-end test (13 real requests via TestClient):
- Login -> 200, valid JWT
- Create 2 posts (authenticated) -> 201 each
- Create post WITHOUT auth -> 401 (correctly rejected)
- List all posts -> 200, 2 posts
- Search "fastapi" -> 200, 1 correct match (case-insensitive, title+body)
- Get single post -> 200
- Get nonexistent post -> 404
- Add comment to a post -> 201
- List comments for that post -> 200, 1 comment
- Update own post -> 200, title updated
- Delete own post -> 204
- Confirm deletion (get again) -> 404

ALL 13 PASSED. Also implemented but not separately re-tested: ownership
checks (403 if a non-author tries to edit/delete another user's post) -
implemented via `if posts_db[post_id]["author"] != username` checks mirroring
the pattern, following the same "explicit per-route authorization" style
learned from the auth exercise.

## Reflection

This is now THREE FastAPI exercises in a row where real, verified
version/dependency issues were found in provided sample code - and the SAME
Pydantic V1 Config pattern specifically has now appeared 3 separate times
across this course's material. This stopped feeling like isolated bad luck
and started looking like the course material itself was written against an
older Pydantic version and never updated - a genuinely useful meta-lesson:
even within one course/tutorial series, the same deprecated pattern can
recur repeatedly, so recognizing it once (as we did in the first FastAPI
exercise) made every subsequent occurrence much faster to catch and fix.