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
---

## Still to Come
_(future exercises will be added here as they're completed)_