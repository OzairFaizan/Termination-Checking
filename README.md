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

# Week 03 2026-03-03
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

# Week 04 2026-03-10
- Remove base case checks: everything must be case classes/objects!
- Remove `l.tail` too...
- Mutual recursion, for every call to a function not in the call stack:
    - Add `called_param -> (arg, Same)` to compare map.
    - Add called method symbol to call stack.
    - Traverse the called function and undo the above afterward.
  If the function is in the call stack:
    - Compare its arguments with the saved parameters lexicographically.
- `-Yretain-trees` is required for "general" mutual recursion.
- Read "The Size-Change Principle for Program Termination" which is a generalization of the above.
    - The strengths: Very general, works even when arguments are swapped or not used.
    - The Weaknesses: Computationally inefficient PSPACE-Complete.

# Week 05 2026-03-19
- Added `@decreasesBy()` on output type which allows user to give an order in which lexicographical
  order should be checked.
- Added `assumeTerminates(body)` function, which wraps a body the user marks as terminating and
  therefore not checked.

# Week 06 2026-03-24
- Added `@assumeTerminates` annotation.
- Added a whitelist of functions (might be removed in favor of the annotation).
- Curried functions partially supported (some bugs when >2 arguments).
- Functions passed as arguments are now checked.
- The explanation for the `addAll`:
    - Keep the `op` function tree when defined in `addAll`.
    - The problem reduces to mutual recursion termination checking: `addAll` calls `foldRight` and 
      `foldRight` calls `addAll` through `op`.
    - Note we cannot prove termination for `foldRight` alone as `op` is not generally known, however
      whenever it is known we can proof its termination alongside the functions where it is called.

# Week 07 2026-03-31
- Add `@assumeTerminates` to standard library functions that terminate but don't have a body.
- Curried functions should have all their arguments to work properly.
  It also works if all but the last argument is given then the last one is given, but this is just by "accident".
  For complete generality we need to keep track of partial argument lists which is doable but not
  worth the pain for the use case.

# Easter Break 2026-04-07

# Week 08 2026-04-14
- Assume well-founded on types, for instance on `List` this allows us to match over `head :: tail` and consider both `head` and `tail` as sub-terms.
- Make selectors work for instance `l.tail` should be less than `l: List`.
  The selected term should either have `assumeDecreases` or be an immutable parameter of the case class.
- Assume decreases annotation on methods for instance `head` and `tail` of `List`.

# Week 09 2026-04-21
- Fixed select failing test.
- While loops are doable:
  - One function that takes a tree, and a boolean (whether to check or not).
      - The function matches on the tree with the cases:
          - If      recurse on both branches fold results.
          - Match   recurse on all branches fold results.
          - WhileDo recurse on body fold result with all Size.Same
          - Assign  update the `sizeMap`.
          - Others  recurse on children. 
        The "fold results" means that for each variable we output one Size, indicating whether it
        decreased across all branches.

# Week 10 2026-04-28
- Put annotation on List tail and head, instead of Iterable.
- Optimize by keeping a set of checked functions.
- While loops implementation (requires two traversals).
## Bug fix:
- When we do `l.foo(...)`, for `l: List[...]` we need to find all the possible methods `foo` can
  take and check them for termination.
  Empty trees are often due to this issue, but correctness is a bigger concern.

# Week 11 2026-05-05
- Add annotation, on `throw` rather than explicit check (reverted due to failing tests.)
- Run benchmarks.
- Ask LLM for code review.
- Start Report and Documentation.

# Week 12 2026-05-12

# Week 13 2026-05-19

# Week 14 2026-05-28

# Week 15 2026-06-05

