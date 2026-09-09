# Task Manager — Code Comprehension Exercise Discoveries
### (code-comprehension-001, Python starter code)

**Repo:** `Dr-Aura/ai-code-exercises` (forked from `wethinkcode/ai-code-exercises`)
**Path:** `use-cases/code-comprehension-001/python/TaskManager/`

---

## 1. Initial Understanding (before opening files)

**Guess at structure:** Same layered pattern expected as the earlier Task Manager
project: `cli.py` = entry point/argparse, `models.py` = data definitions,
`storage.py` = persistence, `task_manager.py` = business logic coordinator
(renamed from `app.py`). New addition: a `tests/` folder, not present in the
previous version explored.

**Note from README:** explicitly states this starter code is "by no means
brilliant or optimized... intentionally created to be improved" via AI prompts —
so some rough edges were expected going in.

---

## 2. Structural Comparison vs. Previously-Explored Task Manager

| Aspect | Previous version | This version (code-comprehension-001) |
|---|---|---|
| Business logic file name | `app.py` | `task_manager.py` |
| Import style | Relative (`from .models import ...`) — requires package context | Absolute (`from models import ...`) — run standalone from inside the folder |
| `models.py` | — | Identical, word-for-word |
| `storage.py` | — | Identical logic; only import style changed |
| `cli.py` | — | Identical logic; only import style changed |
| Tests | Not present/not explored | `tests/test_task_manager.py` present — 25 tests |

## 3. Confirmed Real Difference: `get_statistics()` Key Convention

```python
status_counts   = {status.value: 0 for status in TaskStatus}     # "todo", "in_progress", ...
priority_counts = {priority.name: 0 for priority in TaskPriority} # "LOW", "MEDIUM", ...
```

Initially flagged as a possible inconsistency/bug. **Confirmed by the test suite to
be intentional, tested behavior** — `test_get_statistics_1` and
`test_get_statistics_with_empty_task_list` both explicitly assert this exact
asymmetric shape. Lesson: an apparent inconsistency in code isn't always a bug —
always check the tests before assuming.

## 4. Test Suite Findings (`tests/test_task_manager.py`)

**Coverage:** 25 tests across `TaskManagerTest`, covering create/list/update/delete/
tag management/statistics, including several negative/edge cases (invalid date
format, invalid priority/status values, operating on nonexistent tasks).

**Mocking patterns used:** Mixed — `Mock()`, `MagicMock()`, and `Mock(spec=Task)`
(the most disciplined, since it restricts the mock's attributes to only those that
really exist on `Task`, catching typos other mock styles wouldn't).

**Assertion style:** Mixed — both `self.assertTrue(...)` (unittest style) and bare
`assert ...` (pytest style) appear in the same file. Minor consistency issue, but
functionally harmless since `unittest` tolerates plain `assert`.

**Real issue found — test isolation:**
Several tests instantiate a real `TaskManager()` (which runs the actual
`TaskStorage("tasks.json")` constructor, touching the real filesystem) *before*
replacing `.storage` with a mock — and a few tests never replace storage at all
(e.g. `test_delete_nonexistent_task`, `test_create_task_invalid_date_format`,
`test_get_task_details_nonexistent_task`). These tests will read/write a real
`tasks.json` file in whatever directory the suite runs from, with no temp-directory
isolation. Risk: stray leftover files between runs, and potential test
interference if ever run in parallel or against a pre-existing `tasks.json`.

**Confirms behavior we'd flagged earlier as a minor style gap:**
`update_task_status()`'s implicit `None` return (when status is `DONE` but the task
isn't found) is explicitly tested and passes — `assertFalse(None)` succeeds since
`None` is falsy. Not a functional bug, just an inconsistent return style versus the
`else` branch's explicit `return`.

---

## 5. Summary — Misconceptions Corrected

- Expected more substantial functional changes given the "not optimized, meant to
  be improved" README warning — in practice, the only genuine differences were
  import style, the file rename, and the added test suite. The actual business
  logic is unchanged from the earlier version.
- Initially treated the `.value`/`.name` asymmetry as a likely bug — the test
  suite proved it's deliberate, tested behavior instead.

## 6. Candidates for Improvement (via later AI-assisted refactoring exercises)

1. Fix test isolation — use `tempfile.TemporaryDirectory()` or an in-memory
   fake/mock consistently for `TaskStorage`, so no test touches the real filesystem.
2. Standardize on one assertion style (`self.assertX` or bare `assert`) throughout
   the test file.
3. Either document *why* `by_status` and `by_priority` use different key
   conventions, or unify them if there's no real reason for the difference —
   currently the asymmetry is untelegraphed to a new reader.
4. Standardize `update_task_status()`'s return type — explicit `return False` in
   the `DONE`-but-not-found branch, matching the `else` branch's explicit return.

## Exercise Part 1: Understanding Task Creation and Status Updates

### Main components
- models.py: Task class (auto-generates UUID, defaults to TODO), TaskStatus/TaskPriority enums
- task_manager.py: create_task() validates/converts input; update_task_status() coordinates the transition
- storage.py: add_task() and update_task() persist changes, full JSON rewrite on every save

### Execution flow
Create: cli.py -> task_manager.create_task() -> Task.__init__() -> storage.add_task() -> storage.save()
Mark done: cli.py -> task_manager.update_task_status() -> special DONE branch -> task.mark_as_done() -> storage.save()

### How data is stored/retrieved
Full task set held in-memory as a dict, rewritten entirely to tasks.json on every single change - no incremental writes

### Design pattern discovered
update_task_status() looks like a generic status-transition function but has a hidden special case: DONE
bypasses the generic Task.update() path entirely because generic update() cannot set completed_at.
Risk: a future new status requiring its own side effect (e.g. ABANDONED needing an abandoned_at timestamp)
could easily be added without realizing it needs the same special-case treatment.
