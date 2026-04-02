---
name: clean-code
description: "review and refactor code for readability, simplicity, smaller functions, lower complexity, clearer naming, safer control flow, and better maintainability. use when asked to simplify complex code, break down long functions, reduce nesting, remove duplication, improve naming, propose clean code refactors, or turn messy code into clearer code without changing behavior."
---

# Clean Code

Apply clean, behavior-preserving refactors. Prefer small, safe improvements over large rewrites.

## Working style

Follow this order:

1. Understand the code before changing it.
2. Preserve behavior unless the user explicitly asks for behavior changes.
3. Prefer the smallest safe refactor that improves clarity.
4. Keep public APIs, external contracts, and data formats stable unless the user asks to change them.
5. If risk is high or the requirements are unclear, do an audit first and explain the safest next step.

## Default response mode

Unless the user explicitly asks for a rewrite only, respond in this structure:

### Summary
State the main maintainability problems in 2 to 5 bullets.

### Highest-impact fixes
List the most valuable clean-code improvements first.

### Refactor plan
Describe the smallest safe sequence of changes.

### Revised code
Provide the improved version or a focused patch.

### Validation checklist
List what should be tested after the refactor.

## Core principles

### Prefer clarity over cleverness
- Choose code that is easy to read quickly.
- Avoid dense, tricky, or overly compact logic unless it is clearly better.

### Keep functions small and focused
- Make each function do one clear job.
- Split long functions into well-named helpers.
- Avoid mixing validation, transformation, side effects, and formatting in one place.

### Use names that reveal intent
- Rename vague names like `data`, `item`, `tmp`, `val`, `doStuff`, `handle`.
- Prefer names that reflect domain meaning and business intent.
- Make boolean names read naturally, such as `is_active`, `has_access`, `can_retry`.

### Reduce nesting and branching
- Prefer guard clauses and early returns over deep nesting.
- Flatten conditionals when possible.
- Replace long conditional chains with lookup tables, dedicated helpers, or polymorphism when that clearly improves readability.

### Remove duplication
- Extract repeated logic into a single helper.
- Remove copy-paste variations by parameterizing behavior when appropriate.
- Do not create abstractions too early; keep them proportional to the repetition.

### Separate pure logic from side effects
- Isolate I/O, database calls, HTTP calls, logging, and UI updates from core decision logic.
- Make business rules easier to test by keeping them as pure as possible.

### Keep parameters under control
- Reduce long parameter lists.
- Group related parameters into an object or value type when it improves readability.
- Avoid boolean flag arguments when separate functions would be clearer.

### Make comments rare and useful
- Do not add comments that repeat the code.
- Prefer code that explains itself through structure and naming.
- Keep comments for non-obvious decisions, constraints, tradeoffs, or external rules.

### Handle errors consistently
- Fail fast on invalid input when appropriate.
- Avoid silent failures.
- Keep error handling predictable and easy to follow.

## Refactoring playbook

### Long or complex functions
Use one or more of these:
- extract helper functions
- introduce guard clauses
- rename intermediate values
- split orchestration from implementation details
- move repeated logic into a dedicated helper
- remove dead code and unnecessary branches

### Complex conditionals
Use one or more of these:
- invert conditions to reduce nesting
- consolidate related checks
- extract condition intent into a well-named function
- replace conditional trees with maps, strategies, or polymorphism when the logic is growing by type or mode

### Poor naming
- rename variables, parameters, functions, classes, and files to match domain intent
- remove misleading or outdated names
- prefer consistency with surrounding code and project terminology

### Mixed responsibilities
If a function does too many things:
- separate validation
- separate data fetching
- separate transformation
- separate persistence or side effects
- leave a small orchestrator that reads top-to-bottom

### Data shape problems
- simplify deeply nested structures when possible
- normalize confusing data transformations
- prefer explicit fields over magic indexes or ambiguous tuples

## Code smells to prioritize

Treat these as high priority:
- long functions
- high nesting
- repeated code
- vague names
- boolean flag arguments
- too many parameters
- hidden side effects
- large switch or if/else trees that keep growing
- comments explaining confusing code instead of improving the code
- dead code
- mixed abstraction levels in the same function

## Guardrails

- Do not change behavior unless asked.
- Do not introduce frameworks or large patterns without clear benefit.
- Do not over-abstract a small problem.
- Do not turn a simple flow into many tiny functions if that hurts readability.
- Do not rename public APIs unless the user asks for a breaking change or a migration plan.
- Do not claim code is "clean" just because it is shorter.

## Output preferences

When refactoring:
- preserve the project's existing language and style unless the user asks otherwise
- keep imports, formatting, and structure consistent with the surrounding file
- explain important tradeoffs briefly
- call out risks explicitly if tests are missing

When the user asks for review only:
- do not rewrite everything
- show the smallest examples needed to explain the issue
- prioritize the top 3 to 5 fixes

## Special instructions for generated patches

If you generate revised code:
- keep the patch focused on the requested scope
- prefer incremental changes over full rewrites
- make the final code easier to test
- mention any follow-up cleanup that should happen later but is not included now

## Example transformation patterns

### Example: nested conditionals
Prefer this direction:

- before: nested `if/else` with multiple levels
- after: guard clauses, early returns, and named helper functions

### Example: large function
Prefer this direction:

- before: one long function doing validation, transformation, API call, and formatting
- after: one small orchestrator plus focused helpers

### Example: vague naming
Prefer this direction:

- before: `processData(data, flag)`
- after: names that describe the domain action and intent

## Final check before answering

Before giving the final answer, verify:
- is the behavior preserved
- is the code easier to read
- are functions smaller or clearer
- is nesting reduced
- are names more intentional
- is duplication reduced
- is the refactor proportionate to the problem
