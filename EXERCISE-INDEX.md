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
---

## Still to Come
_(future exercises will be added here as they're completed)_