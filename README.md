# Week 01 2026-02-17
- Forked dotty, created [feature branch](https://github.com/OzairFaizan/scala3/tree/feature/termination-check).
- Added `@terminates` annotation.
- Added `-Ycheck-termination` flag.
- Added `CheckTermination` `MiniPhase`
    - Inspired by `TailRec`.
    - Checks termination syntactically:
        - Looks for "destructors" in match cases to find smaller symbols.
        - A function terminates if for each recursive call at least one of its parameter is smaller.
- Added simple tests:
    - `tests/neg/ycheck-termination-error`
    - `tests/pos/ycheck-termination`

# Week 02 2026-02-24
- Made the first two commits and created the [PR](https://github.com/scala/scala3/pull/25404).
    - First for the infrastructure.
    - Second implements simple structural recursion checks:
        - Checks that type is indeed immutable and "can decrease".
        - Checks for matches or class parameters to find sub-term.
        - Checks that arguments of recursive calls satisfy:
            - At least one argument is smaller than parameters.
            - No arguments are of unknown size.
- Fixed how sub-terms were stored, using maps to store the hierarchy.
- Hacky implementation for `l.tail`.
- Works for when `this` is decreasing.
- Added tests for:
    - If type is not a case class.
    - If type is a case class with `unapply` overridden.
    - Non-decreasing arguments.
    - Decreasing argument.
    - Decreasing `this`

# Week 2026-03-03
## Features:
- Can track same sizes for `l @ ...` and `val x = ...`.
- Checks for synthetic `unapply` only when matched over term. Only a warning is
  given when not synthetic however sub-terms are not considered.
- Lexicographic order is used to compare arguments of recursive calls with
  parameters.
- Checks if type can decrease (is well-founded) only when needed:
  If it's the type first argument that decreases and all previous arguments are
  non-increasing
- More meaningful error messages.
## Bug fixes:
- `canTypeDecrease` was not sound, we also need to check:
    - The base cases do not have any self references.
    - There exists a base case (case object), so the parameter should have a
      type which is a sealed trait (or enum).

