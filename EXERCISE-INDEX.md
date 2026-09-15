# AI Code Exercises — Submission Index

**Repo:** Dr-Aura/ai-code-exercises (fork of wethinkcode/ai-code-exercises)

This index links to every completed exercise and its discovery journal, so there's a single entry point for grading.

---

## Completed Exercises

### 1. Task Manager Code Comprehension
**Location:** `use-cases/code-algorithms/python/TaskManager/`
**Discovery journal:** [discovery-notes.md](use-cases/code-algorithms/python/TaskManager/discovery-notes.md)
**Submission summary (PDF):** [task-manager-exercise-submission.pdf](use-cases/code-algorithms/python/TaskManager/task-manager-exercise-submission.pdf)

Parts completed:
- Part 1: Knowing Where to Start
- Part 2: Finding Feature Implementation Locations
- Part 3: Understanding Domain Model
- Part 4: Practical Application

### 2. Codebase Exploration Challenge
**Location:** `use-cases/code-comprehension-001/python/TaskManager/`
**Discovery journal:** [code-comprehension-001-discoveries.md](use-cases/code-comprehension-001/python/TaskManager/code-comprehension-001-discoveries.md)
**Presentation (PPTX):** [codebase-exploration-reflection.pptx](use-cases/code-comprehension-001/python/TaskManager/codebase-exploration-reflection.pptx)

Parts completed (all 4 — exercise complete):
- Part 1: Understanding a Specific Feature (task creation and status updates)
- Part 2: Deepen Understanding Through Guided Questions (task priority)
- Part 3: Mapping Data Flow (task completion)
- Part 4: Reflection and Presentation

### 3. Algorithm Deconstruction Challenge
**Location:** `use-cases/code-algorithms/python/TaskManager/`
**Journal:** [algorithm-deconstruction-journal.md](use-cases/code-algorithms/python/TaskManager/algorithm-deconstruction-journal.md)

Algorithm chosen: Task list merging (two-way sync conflict resolution)

Covers:
- Step-by-step breakdown with a worked example
- Control flow diagram
- Major finding: deleted tasks get silently resurrected on merge (no tombstone/deletion tracking)
- Reflection questions

### 4. Code Documentation Exercise
**Location:** `use-cases/code-algorithms/python/TaskManager/`
**Journal:** [code-documentation-exercise.md](use-cases/code-algorithms/python/TaskManager/code-documentation-exercise.md)

Code documented: `merge_task_lists()` and `resolve_task_conflict()` (task list merging)

Covers:
- Prompt 1 output: full structural documentation (params, returns, exceptions, example)
- Prompt 2 output: intent/logic explanation, assumptions, suggested inline comments
- Comparison: Prompt 1 alone would have shipped accurate-but-shallow docs; Prompt 2 surfaced the hidden local-wins-by-default behavior and deletion gap
- Final combined documentation version

### 5. API Documentation Exercise
**Location:** `use-cases/code-algorithms/python/TaskManager/`
**Journal:** [api-documentation-exercise.md](use-cases/code-algorithms/python/TaskManager/api-documentation-exercise.md)

Endpoint documented: POST /api/users/register (Flask starter example)

Covers:
- Prompt 1: full endpoint documentation (params, responses, error codes)
- Genuine bug found: email-uniqueness check runs before lowercasing, so a different-case duplicate email bypasses the check
- Prompt 2: OpenAPI 3.0 conversion
- Prompt 3: developer usage guide with a working Python example

### 6. README and User Guide Documentation Exercise
**Location:** `use-cases/code-algorithms/python/TaskManager/`
**Journal:** [readme-user-guide-exercise.md](use-cases/code-algorithms/python/TaskManager/readme-user-guide-exercise.md)

Project documented: The Task Manager CLI itself

Covers:
- Genuine bug found: the project's existing README documents command names that don't match the actual code (`update-status` vs. real `status`, etc.) - every example in the shipped README would fail as written
- Prompt 1: corrected README.md
- Prompt 2: step-by-step beginner guide for creating/completing a task
- Prompt 3: FAQ grounded in prior exercises' verified findings (silent save failures, merge 
deletion gap, priority vs. importance score)

### 7. Error Diagnosis Challenge
**Location:** `use-cases/error-diagnosis-challenge/`
**Journal:** [error-diagnosis-journal.md](use-cases/error-diagnosis-challenge/error-diagnosis-journal.md)

Scenario chosen: Global Variable Being Overwritten (JavaScript) - variable shadowing bug

Covers:
- Root cause tracing: distinguished the crash's proximate cause (displayTasks running before initialization) from the actual logic bug (let shadowing preventing real task additions)
- Fix and initialization guard
- Reflection on JS scoping concepts (function-level shadowing)

### 8. Performance Optimization Challenge
**Location:** `use-cases/performance-optimization-challenge/`
**Journal:** [performance-optimization-journal.md](use-cases/performance-optimization-challenge/performance-optimization-journal.md)

Scenario chosen: Slow Code Analysis (Python) - find_product_combinations()

Covers:
- Profiled with cProfile to find the real bottleneck: an O(matches²) duplicate-check, not the O(n²) loop it appeared to be at first read
- Fix: restructured the loop to generate each pair once, eliminating the dedup check entirely
- Real measured results: ~1,015x speedup at n=500 (25.78s → 0.025s); original did not complete at n=1,000 within 250s, optimized handled the full n=5,000 target in 6.28s
- Correctness verified: identical output before and after

### 9. AI Solution Verification Challenge
**Location:** `use-cases/ai-solution-verification-challenge/`
**Journal:** [verification-challenge-journal.md](use-cases/ai-solution-verification-challenge/verification-challenge-journal.md)

Problem chosen: Buggy merge sort (JavaScript) - infinite loop from a mis-incremented counter

Covers:
- Empirically confirmed the bug (input-dependent hang, verified via `timeout` + exit code) before trusting any fix
- Applied all three verification techniques: Collaborative Verification, Alternative Approaches, Developing a Critical Eye
- Final fix tested against 8 cases including a 100,000-element stress test
- Key insight: the bug didn't manifest on all inputs, only when `right` exhausts before `left` -
easy to miss with a narrow test suite

### 10. Using AI to Help with Testing
**Location:** `use-cases/ai-assisted-testing-exercise/`
**Journal:** [testing-exercise-journal.md](use-cases/ai-assisted-testing-exercise/testing-exercise-journal.md)

All parts complete (1.1 through 4.1) - guided-questioning approach throughout,
not AI-generated tests.

Covers:
- 1.1/1.2: Behavior analysis and test planning for calculate_task_score, sort_tasks_by_importance, get_top_priority_tasks
- 2.1/2.2: Improved a weak assertion into a precise one; boundary testing uncovered a REAL, previously undiscovered bug - timedelta.days floors negative deltas, so "due today" tasks can silently score as overdue depending on microsecond timing
- 3.1: Full TDD cycle (red/green/refactor-safety-check) for a new assignee-boost feature - discovered the Task model had no assignee concept at all before writing any test
- 3.2: Investigated a described bug that didn't apply to this Python implementation; found the real timedelta quirk from 2.2 also exists here but doesn't change the outcome
- 4.1: Integration tests confirming the full scoring→sorting→top-N workflow, including a DONE/URGENT task correctly losing to active tasks
- 18 tests total, all passing, zero regressions

### 11. Understanding What to Change with AI
**Location:** `use-cases/refactoring-what-to-change-exercise/`
**Journal:** [refactoring-exercise-journal.md](use-cases/refactoring-what-to-change-exercise/refactoring-exercise-journal.md)

All 3 parts complete: Code Readability (Java), Function Refactoring (Python),
Code Duplication Detection (JavaScript)

Covers:
- Java UserMgr: renamed for clarity, but the real finding was a SQL injection vulnerability surfaced while reading closely enough to rename things
- Python process_orders: decomposed into 3 focused functions; surfaced an implicit, undocumented "free shipping over $50" business rule
- JavaScript calculateUserStatistics: consolidated 6 near-identical loops into 2 generic helpers + a data-driven loop
- Reflection on when to disagree with AI suggestions (readability vs. team experience level trade-offs) and safeguards before applying refactors to production code

### 12. Function Decomposition Challenge
**Location:** `use-cases/function-decomposition-challenge/`
**Journal:** [decomposition-challenge-journal.md](use-cases/function-decomposition-challenge/decomposition-challenge-journal.md)

Function chosen: validateUserData (JavaScript, 150+ lines, 9 responsibilities)

Covers:
- Decomposed into 10 focused helper functions; main function reduced to pure orchestration
- Built a real comparison test harness: ran 31 diverse inputs through BOTH original and refactored versions, comparing outputs exactly (including crash-for-crash)
- All 31 identical - refactoring verified to preserve behavior exactly
- MAJOR FINDING: a real, previously-hidden crash bug in the original code - profile updates with a non-empty address object crash the function, because a generic field-required loop assumes all fields are strings while address is documented elsewhere as an object

### 13. Code Readability Challenge
**Location:** `use-cases/code-readability-challenge/`
**Journal:** [readability-challenge-journal.md](use-cases/code-readability-challenge/readability-challenge-journal.md)

Example chosen: Cryptic Variable Names (JavaScript) - inventory processing function

Covers:
- Ran original unit tests first to understand behavior before renaming anything
- Renamed function, parameters, internal variables, and the return object's own keys for clarity
- Deliberately kept loop counters unrenamed (used purely as indices) - clarity has a ceiling
- Adapted and re-ran the test suite against the refactored version - all 3 tests passed, confirming behavior preservation
- Reflection on which renames matter most (return-value keys, since they're the actual public contract)

### 14. Design Pattern Implementation Challenge
**Location:** `use-cases/design-pattern-implementation-challenge/`
**Journal:** [pattern-challenge-journal.md](use-cases/design-pattern-implementation-challenge/pattern-challenge-journal.md)

Pattern chosen: Strategy Pattern - JavaScript shipping cost calculator

Covers:
- Identified 3 parallel conditional blocks as interchangeable pricing algorithms sharing one interface
- Refactored into 3 strategy objects + a lookup-based dispatcher, eliminating per-method conditional logic from the main function
- Deliberately preserved two pre-existing quirks (a string return for unavailable overnight destinations, and a "0.00" fallback for unrecognized methods) rather than silently fixing them mid-refactor
- Built a comparison harness: 18 test cases, checking both value AND type equality - all identical between original and refactored versions
- Reflection on distinguishing behavior-preserving refactoring from bug-fixing, and not mixing the two

### 15. Deepening Knowledge of Your Current Programming Language
**Location:** `use-cases/deepen-language-knowledge-exercise/`
**Journal:** [language-deepening-journal.md](use-cases/deepen-language-knowledge-exercise/language-deepening-journal.md)

Language: Python - all 3 activities complete

Covers:
- Activity 1 (Idiomatic Code): 21 lines -> 6 lines using enumerate/comprehension/sorted; verified against 5 cases including a tie-order edge case
- Activity 2 (Code Quality Detective): identified 6 real code smells including a mutable default argument - EMPIRICALLY DEMONSTRATED with an isolated reproduction, not just described; built a reusable review checklist
- Activity 3 (Language Feature - Decorators): implemented and tested a @retry decorator; confirmed functools.wraps preserves function metadata by checking __name__ directly
- Cross-activity theme: understanding solidified through empirical demonstration, not just explanation, in all three activities

### 16. Learning a New Programming Language with AI
**Location:** `use-cases/learning-new-language-exercise/`
**Journal:** [learning-r-journal.md](use-cases/learning-new-language-exercise/learning-r-journal.md)

Target language: R, coming from Python - all 4 parts complete

Covers:
- 4-phase learning journey plan (Fundamentals -> Data Frames -> Statistics -> Visualization)
- Four-step prompting strategy applied to vectors/vectorization, with REAL R code run throughout (not just described)
- GENUINE BUG FOUND: 1:length(x) on an empty vector produces "1 0" in R (counts down), meaning a Python-habit loop runs twice on empty input instead of zero times - confirmed empirically
- 2 advanced prompting techniques practiced (Context Effectively, Learning Through Teaching) with claims verified via actual code, not taken on faith
- Mini-project: full data summary + stats + visualization script, run end-to-end with a real saved plot

### 17. Getting Started with FastAPI
**Location:** `use-cases/fastapi-getting-started-exercise/`
**Journal:** [fastapi-getting-started-journal.md](use-cases/fastapi-getting-started-exercise/fastapi-getting-started-journal.md)

All 4 parts complete, with real running code tested via FastAPI's TestClient
(not just described)

Covers:
- Part 2: basic API built and tested (5 requests) - including automatic 422 validation demonstrating FastAPI's core type-hints-as-validation advantage
- Part 3: TWO real, verified deprecation bugs found in the EXERCISE'S OWN sample code (Pydantic V1 `class Config` syntax, and `HTTP_422_UNPROCESSABLE_ENTITY`) - both confirmed with actual Python tracebacks, both fixed and re-verified
- Part 4: full To-Do List CRUD API built and tested (9 requests) - create, list, filter by completion status, complete, delete, 404 handling, validation
- Direct validation of this chapter's own core warning: AI/tutorial-provided code can already be outdated - always cross-reference against the actual installed version
---

## Still to Come
_(future exercises will be added here as they're completed)_