# Error Diagnosis Challenge

**Scenario chosen:** #6 - Global Variable Being Overwritten (JavaScript)
**Why this one:** Thematically matches the Task Manager focus of the whole course,
and is a genuinely instructive scoping bug (variable shadowing) rather than a
simple typo-level error.

## Error Description
Browser throws TypeError: Cannot read properties of undefined (reading 'map')
because displayTasks() calls tasks.map(...), but tasks is undefined at that moment
rather than an array.

## Root Cause
Line 15, inside addTask(): `let tasks = {...}` declares a BRAND NEW local variable
also named tasks, function-scoped to addTask(). It shadows the outer global tasks
array - it does not modify the global at all. addTask() never actually pushes the
new task object into the real global array (no tasks.push() call exists), so
clicking "add" never actually adds anything to the visible list, even setting aside
the crash.

The `undefined` crash itself points to a SEPARATE but related issue: since tasks is
initialized to [] at declaration (not undefined), displayTasks() reporting `tasks`
as undefined means it ran before initApp() completed - i.e. the add-task button
was wired up and clickable before window.onload = initApp had finished, or initApp()
failed silently before reaching the `tasks = [...]` assignment.

## Solution
1. Fix the shadowing bug - rename the local variable and actually push to the
   global array:
   function addTask(taskName) {
     const newTask = { id: Date.now(), name: taskName, completed: false };
     tasks.push(newTask);
     displayTasks();
   }
2. Guard displayTasks() against a not-yet-initialized state:
   if (!Array.isArray(tasks)) return;
   - or ensure initApp() fully completes (with visible errors, not swallowed)
   before any button can trigger addTask().

## Learning Points
- let/const inside a function always creates a new, function-scoped variable, even
  if the name matches something in an outer scope - real trap given JS's
  function-level (not just block-level) shadowing behavior
- A variable name reused for a genuinely different purpose (single task object vs
  array of tasks) is a strong signal something's conceptually wrong, not just a
  naming coincidence
- "Cannot read properties of undefined" should prompt checking INITIALIZATION ORDER,
  not just the immediate crash line - the true root cause was 3 function calls away
  from the actual crash site

## Reflection Questions

**How did the AI's explanation compare to documentation found online?**
Generic docs on this error type explain .map()'s requirements (it only exists on
arrays) but wouldn't catch the specific shadowing bug - that required reading the
actual code, not just looking up the error type.

**What aspects would be difficult to diagnose manually?**
Realizing the crash's proximate cause (displayTasks running too early) and the
code's actual logic bug (shadowing preventing real task additions) are TWO SEPARATE
issues, not one - easy to fix one and assume the bug is fully resolved.

**How would I modify the code for better error messages in the future?**
Add an explicit check at the top of displayTasks():
if (!Array.isArray(tasks)) console.error('tasks is not an array:', tasks);
Turns a cryptic crash into a self-diagnosing message.

**Did AI help understand underlying concepts, not just the fix?**
Yes - the real lesson is about JavaScript scoping (let creating function-scoped
shadows even when a same-named variable exists in an outer scope), not just this
one isolated bug.