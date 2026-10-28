# Opt-in Termination Checker

## Summary
Adds a `MiniPhase` (`CheckTermination`) that verifies termination of methods annotated with `@terminates`.
The checker is gated behind `-Ycheck-termination` (also requires `-Yretain-trees`) and requires no external tools.

The code is available on [GitHub](https://github.com/OzairFaizan/scala3/tree/feature/termination-check).

## Overview of the algorithm
The checker performs structural decrease analysis over well-founded Algebraic Data Types.

- The checker maintains a `valMap` to track immutable values and a `varMap` to track mutable variables. Values are assigned a relative size of `Smaller`, `Same`, or `Unknown` compared to the method parameters.
- A size decrease is established primarily through pattern matching on ADTs. Binders are recorded as `Smaller` than the selector. Alternatively, selecting a field that corresponds to a case class constructor parameter also registers as a decrease.
- During recursive calls, the arguments passed are lexicographically compared to the original method parameters. The call is accepted if the first differing argument is `Smaller` and its type is verified as well-founded.
- To handle dynamic dispatch, the algorithm enumerates all sealed strict descendants of a polymorphic type and independently verifies every overridden concrete implementation.
- The checker supports mutual recursion by maintaining a callStack of currently analyzed methods and inlining callees. Size information propagates through argument-to-parameter mappings, allowing size inequalities to be composed across call chains. Higher-order functions are verified similarly by resolving and inlining the passed closures.
- While loops are checked using two traversals. The first pass disables call checking and find a mutable variable that is assigned a `Smaller` value in every path. If a decreasing variable is found, a second traversal checks recursive calls in the loop body.
- Because the syntactic analysis is conservative, it may reject semantically terminating programs. The PR introduces annotations like `@assumeTerminates`, `@assumeDecreases`, and `@assumeWellFounded` to manually assert termination properties, shifting the verification to the developer when automatic checker fails.

## Examples
Simple structural recursion
```scala
@terminates
def sum(l: List[Int]): Int =
  l match
    case Nil => 0
    case hd :: tl => hd + sum(tl)
```

Mutual recursion
```scala
@terminates
def isEven(n: Nat): Boolean =
  n match
    case Zero => true
    case Succ(pn) => isOdd(pn)
@terminates
def isOdd(m: Nat): Boolean =
  m match
    case Zero => false
    case Succ(pm) => isEven(pm)
```

`@decreases` can be used to override the default left-to-right order:
```scala
@terminates
def f(logger: List[Nat], n: Nat): (Nat, List[Nat]) @decreases(n) =
  n match
    case Zero => (Zero, Zero :: logger)
    case Succ(pn) => f(n :: logger, pn)
```

More positive and negative examples along with detailed explanations are given in the [report](report/report.pdf).


## Standard Library changes
Some annotations were added to the standard library:
- Added `@assumeWellFounded` on `List`, because `::` has a `var` field therefore failing the automatic check.
- Added `@assumeDecreases` on the `head` and `tail` methods of `List`.
- Added `@assumeTerminates` to primitive operators (e.g., `+`, `-`, `<`) on types like `Int` because their implementations are not available for analysis.


