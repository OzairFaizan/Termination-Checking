# Week 01 2026-02-17
- Forked dotty, created [feature branch](https://github.com/OzairFaizan/scala3/tree/feature/check-termination).
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


