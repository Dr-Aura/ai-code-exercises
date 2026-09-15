# Learning a New Programming Language with AI (R, coming from Python)

## Part 1: Learning Journey Plan

**Goals:** (1) read/manipulate tabular data in R, (2) compute descriptive stats
and simple hypothesis tests, (3) produce a basic visualization.

4-phase plan created: R Fundamentals -> Data Frames -> Statistical Functions ->
Visualization and Reporting, each with prerequisites, learning steps, and a
verification activity (mirroring the workshop's Java example structure).

## Part 2: Four-Step Prompting Strategy - applied to Vectors and Vectorization

**Step 1 (Conceptual):** Key difference - R is built AROUND the vector as its
fundamental unit; even a scalar is technically a length-1 vector. `+` on two
vectors means element-wise math by default in R, vs list concatenation in
Python - the single most common early confusion. Common misconception: reaching
for for-loops as the default, when vectorization is the idiomatic default in R.

**Step 2 (Breakdown) - RAN REAL CODE:** Demonstrated vector creation, 1-indexing,
vectorized arithmetic, vectorized comparison producing a logical vector, and
using that logical vector DIRECTLY as a filter (v[v > 25]) - no loop, no
filter() call needed. This pattern has no direct Python equivalent outside
NumPy/pandas.

**Step 3 (Guided implementation) - RAN REAL CODE:** Implemented a two-group
mean comparison using R's built-in t.test() - one function call producing a
full statistical test (t=4.09, p=0.0038) that would require importing scipy
in Python.

**Step 4 (Verification) - RAN REAL CODE, FOUND A REAL BUG:** Wrote a first
draft using a Python-loop mindset (growing a vector with c() in a loop,
looping via 1:length(x)). Verification caught TWO real issues, not just style:
1. Growing a vector with c() in a loop is an R anti-pattern (O(n^2) reallocation)
2. GENUINE BUG: 1:length(x) on an EMPTY vector produces `1 0` (R's `:` counts
   DOWN from 1 to 0), meaning the loop runs TWICE on empty input instead of
   zero times - confirmed empirically with empty_values <- c(). This is the
   exact gotcha a Python developer expecting range(len([])) behavior would hit.

Fixed with values^2 (pure vectorization) - verified identical output on real
data AND correct behavior on empty input (length 0, not 2).

## Part 3: Advanced Prompting Techniques (2 applied)

**Using Context Effectively:** Compared sapply() to Python's map()/list
comprehensions - but concluded pure vectorization (values^2) is MORE idiomatic
than sapply() for simple math, the same way NumPy vectorized ops beat Python
list comprehensions for numeric work.

**Learning Through Teaching:** Explained vectorization in own words ("even 5
is really a vector of length 1"), then VERIFIED this claim with real code:
length(5) returns 1 and class(5) returns "numeric" - confirming the claim
empirically rather than taking the explanation on faith.

## Part 4: Mini-Project

Built and RAN a complete data-processing script: student exam scores data
frame, summary() statistics, colMeans() (vectorized), boolean-vector filtering
(students scoring >80 in both subjects), cor() correlation calculation
(0.888), and a real scatter plot saved to PNG and visually confirmed.

## Reflection Questions

**Which prompting strategies were most effective?**
Step 4 (Understanding Verification) - specifically because it caught a
genuine BUG (the 1:length() empty-vector gotcha), not just a style
preference. Writing code with old habits first and THEN having it reviewed
surfaced something writing "correct" code from the start would have hidden.

**What surprised me about R?**
That a logical vector can be used directly as an index/filter (v[v > 25]) -
this isn't just sugar for a filter() call, it's a fundamentally different way
of thinking about selecting data than Python's approach.

**How did Python mental models help or hinder?**
Helped: general programming concepts (variables, functions, conditionals)
transferred immediately. Hindered: the instinct to loop-and-append, and
assuming 1:length(x) behaves like Python's range(len(x)) on empty input -
it doesn't, and this is a real, not just stylistic, source of bugs.

**What would I do differently next session?**
Write the "Python habit" version FIRST and deliberately look for edge cases
(empty input, single-element input) before ever seeing the idiomatic version
- the empty-vector bug would not have surfaced without explicitly testing
that boundary.

**What gaps remain?**
Data frame manipulation beyond basic subsetting (dplyr-style piping), and
R's handling of missing data (NA) - noted as Phase 2/3 items in the learning
plan, not yet covered here.