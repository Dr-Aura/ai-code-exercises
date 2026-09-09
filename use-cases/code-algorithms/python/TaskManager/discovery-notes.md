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
