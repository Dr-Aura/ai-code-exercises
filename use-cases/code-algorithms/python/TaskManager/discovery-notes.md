# Task Manager — Discovery Notes

## Exercise: Knowing Where to Start

### My initial guesses (before reading code)
- Purpose: task/todo management CLI app
- cli.py: entry point, defines commands
- models.py: Task data model
- storage.py: reads/writes tasks to disk
- task_manager.py: core business logic
- task_parser.py: parses task input
- task_priority.py: priority levels
- task_list_merge.py: unclear — merging task lists?
### Step 3: cli.py findings
- Confirmed entry point: if __name__ == '__main__': main()
- Imports TaskManager (task_manager.py) and TaskStatus/TaskPriority (models.py)
- Uses argparse with subcommands: create, list, status, priority, due, tag, untag, show, delete, stats
- cli.py is thin - just parses args and calls into task_manager.py for actual logic

### Step 4: task_manager.py findings
- Not the 'core logic' itself - it's a coordination layer
- Validates/converts input (strings to TaskPriority/TaskStatus enums, date parsing)
- Delegates actual persistence and queries to storage.py (TaskStorage class)
- get_statistics() is the exception - does real computation (counts, overdue, last-7-days)
- Minor smell: imports argparse but never uses it

### Step 6: models.py findings
- TaskPriority: int enum 1-4 (LOW-URGENT); TaskStatus: string enum (todo/in_progress/review/done)
- Task is a plain data class - no persistence logic, confirms storage.py owns that
- id auto-generated via uuid4; defaults to TODO status, MEDIUM priority
- update() is a generic setter via hasattr; mark_as_done() is separate, also stamps completed_at
- is_overdue(): due_date in past AND status != DONE. Uses naive datetime.now() - no timezone handling
- Full flow traced: cli.py -> task_manager.py -> storage.py -> models.py

### Step 5: storage.py findings
- Confirmed: JSON file persistence (tasks.json), not a database
- Entire task set loaded into memory as a dict on startup, rewritten fully on every save()
- Custom TaskEncoder/TaskDecoder convert enums and datetimes to/from JSON-safe values
- Filtering methods are simple list comprehensions - no query engine, linear scans
- Potential scale concern: full file rewrite on every single change

## Summary
Task Manager is a pure-Python CLI app for tracking tasks, using JSON file storage (no external deps, no database).
Architecture: cli.py (argparse dispatcher) -> task_manager.py (coordination/validation) -> storage.py (persistence, in-memory dict + JSON) -> models.py (Task, TaskStatus, TaskPriority).
task_parser.py, task_priority.py, and task_list_merge.py are NOT in the main call chain - likely tests, unused features, or alternate entry points.

## Questions to ask the team
1. What are task_parser.py and task_list_merge.py actually used for, if not called from cli.py?
2. Is the full-file-rewrite-on-every-save approach in storage.py a known/accepted limitation, or a planned refactor target?
3. Is there a plan to move off flat JSON storage if task volume grows?
4. Should is_overdue() account for timezones, or is naive datetime.now() intentional (e.g. single-timezone deployment)?
5. Is there a reason update_task_status() special-cases DONE instead of mark_as_done() being called generically?

## Exploration exercise to verify understanding
Run the CLI locally: create a task with a past due date, list with --overdue, mark it done, and confirm it drops off the overdue list. This validates the create -> storage -> is_overdue -> status update flow end to end.

## Exercise Part 2: Finding Feature Implementation
### Task: Add 'Task Export to CSV' feature

### Step 1: Initial search results
- grep for 'export': no matches anywhere in codebase
- grep for 'csv': no matches anywhere in codebase
- grep for 'open(': only storage.py (lines 52, 62) - sole file I/O in the project

### Step 2: Hypothesis
- No existing export functionality - this is a genuinely new feature, not extending something
- Closest reusable pattern: storage.py already handles reading/writing all tasks, and has a
  TaskEncoder that knows how to flatten Task objects (enums to values, datetimes to strings)
- Likely home: new method in storage.py (e.g. export_to_csv()), OR a new standalone
  module (e.g. task_exporter.py) if we want export logic decoupled from persistence
- Would need a new CLI subcommand in cli.py, following the existing argparse subparser pattern
- task_manager.py would likely need a thin new method (export_tasks()) to match its role
  as coordinator, delegating to storage.py or the new exporter module

### Step 3: Applied Feature Location Prompt - AI response summary
- Better search terms suggested: write, to_dict/to_json/serialize, format_task
- Recommendation: new module task_exporter.py (not storage.py) to keep persistence vs export separate
- task_manager.py gets thin export_tasks() coordinator method, matching its existing role
- cli.py gets new 'export' subcommand following existing subparser pattern
- storage.py/models.py stay untouched - only read from, via get_all_tasks()

### Step 4: Implementation plan
1. Review format_task() in cli.py for which fields are user-facing (CSV column list)
2. Reuse TaskEncoder logic from storage.py for stringifying enums/datetimes
3. Create task_exporter.py with export_to_csv(tasks, filepath) using stdlib csv module
4. Add task_manager.export_tasks(filepath, filters) reusing list_tasks() filter logic
5. Add 'export' subcommand to cli.py
6. Write test_task_exporter.py following existing tests/ pattern

### Self-check questions
- Am I duplicating stringification logic that already exists in TaskEncoder/format_task?
- Is storage.py staying JSON-only, or am I tempted to cram CSV logic in there?
- Does the new CLI subcommand match the existing argparse pattern exactly?

## Exercise Part 3: Understanding Domain Model

### Step 1: Domain model extraction
Core entities (from models.py): Task, TaskStatus (enum), TaskPriority (enum)

Business logic found in task_priority.py (NOT wired into cli.py - undiscovered feature):
- calculate_task_score(): computes an 'importance score' distinct from raw TaskPriority
- Formula: priority_weight*10, +due-date urgency bonus, -status penalty (DONE/REVIEW),
  +tag bonus (blocker/critical/urgent), +recency bonus (updated <1 day ago)
- sort_tasks_by_importance() and get_top_priority_tasks() build on the score

Domain-specific terminology noted:
- 'Priority' (raw enum, static) vs 'Importance score' (computed, dynamic) - these are DIFFERENT concepts
- 'Overdue' has a precise definition: due_date < now AND status != DONE (from is_overdue())

### Step 2: Entity relationship sketch
```
Task --has-a--> TaskPriority (enum, static, user-editable)
Task --has-a--> TaskStatus (enum, changes via workflow)
calculate_task_score(Task) --reads--> Task, --produces--> importance score (computed, not stored)
```

My initial understanding:
- Task is the single core entity - everything else (TaskStatus, TaskPriority) describes it
- TaskPriority is set by the user and rarely changes; TaskStatus changes as work progresses
- Importance score is a SEPARATE, derived concept from priority - confusingly similar name

Questions/confusion:
- Why is calculate_task_score() not exposed anywhere in cli.py? Intentional, in-progress, or dead code?
- Is 'importance score' meant to eventually replace manual priority, or supplement it?

### Step 3: Applied Domain Understanding Prompt - Q&A
1. URGENT/no-due/stale=60 vs LOW/overdue/blocker=53 - URGENT wins but margin is close,
   proving score is a genuine multi-factor trade-off, not priority-always-wins
2. Subtracting (not excluding) DONE/REVIEW keeps them visible in a unified ranked view,
   suggesting the score powers an all-tasks dashboard, not a strict active-only filter
3. Solves 'what to work on today' - static priority goes stale, score reacts to time passing

### Step 4: Glossary
- Priority: static, user-assigned enum (LOW/MEDIUM/HIGH/URGENT) - input, rarely changes
- Status: workflow state (TODO/IN_PROGRESS/REVIEW/DONE) - changes as work progresses
- Importance score: computed, dynamic ranking combining priority + due date + status + tags + recency
- Overdue: due_date in the past AND status != DONE (precise definition from is_overdue())
- Blocker tags: 'blocker'/'critical'/'urgent' tags that boost importance score by +8

### Revised understanding after Q&A
Confirmed: priority (input) and importance score (output) are distinct layers.
Score likely powers a not-yet-built 'smart view' feature - worth raising with the team
as an open question rather than assuming it's dead code.

### Reflection
- AI prompts helped most by forcing me to state assumptions BEFORE getting answers - this caught wrong guesses early, e.g. task_manager.py being 'core logic' vs actually a thin coordinator
- Still unsure about: whether task_parser.py and task_list_merge.py are dead code, in-progress features, or used by something outside cli.py entirely
- Next step to deepen understanding: read task_parser.py and task_list_merge.py directly, and check the tests/ folder to see if they're exercised there

## Exercise Part 4: Practical Application
### Scenario: Auto-mark tasks 'abandoned' if overdue >7 days, unless high priority

### Planning
Files to modify:
1. models.py - add TaskStatus.ABANDONED enum value
2. task_manager.py - new apply_abandonment_rule() method (coordination layer role)
3. storage.py - untouched, reuses get_all_tasks()/update_task()
4. cli.py - decision needed: auto-run vs explicit subcommand

Questions for the team:
1. Does 'high priority' mean only HIGH, or HIGH+URGENT?
2. Should the rule auto-run on every CLI call, or need an explicit command?
3. Is 'abandoned' terminal, or reversible if due date changes?
4. Does the existing naive datetime.now() (no timezone) matter for this rule?

### Final Discussion and Reflection
- Most helpful prompt: the Domain Understanding prompt (Part 3) - the quiz-style follow-up questions caught assumptions I hadn't tested, e.g. assuming priority always dominates the importance score, which turned out to be false
- Next time: read the full call chain before forming file-by-file guesses, rather than guessing purpose from filenames alone - would catch orphaned files earlier
- Complementary tools: actually running the code/tests to verify AI explanations, and a static call-graph tool to confirm which functions are truly unreferenced
