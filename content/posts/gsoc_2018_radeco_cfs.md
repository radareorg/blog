+++
date = 2018-08-12T21:42:12-04:00
draft = false
title = "GSoC 2018: Control Flow Structuring for Radeco-lib"
slug = "gsoc_2018_radeco_cfs"
aliases = [
      "gsoc 2018 radeco cfs" 
]
+++

# GSoC 2018: Control Flow Structuring for Radeco-lib
## Introduction
This summer, I implemented the control flow structuring algorithm described in [*No More Gotos*](https://doi.org/10.14722/ndss.2015.23185). The algorithm takes a program represented as a control flow graph and converts it into a semantically equivalent program but with all control flow represented with C-like control flow statements (e.g. `if`-statements, `while`-loops, etc.) and **zero `goto` statements**.
### Example
![example cfg](/blog/images/example_cfg.png)
```cpp
bool c0 = test0();
if (!c0) {
    run1();
}
if (c0 && test2()) {
    run4();
} else {
    run3();
}
run5();
```
## Algorithm Overview
This section is essentially a (very brief) summary of *No More Gotos*.
We traverse every node in the graph in post-order. If the current node is the target of a backedge, we're at the head of a loop, so we funnel all entries/exits to a single loop header/successor, structure the loop body, and structure the whole loop. Otherwise, if the set of nodes dominated by the current node has exactly one or zero successors, we structure that region. Otherwise, we just continue the graph traversal.
To structure an acyclic region, we compute the reaching condition for each node, move the whole region to a new graph, perform refinements on that graph, and finally make an AST node containing all the nodes in topological order.
To structure a loop, we repeatedly apply a set of rewrite rules to the loop body's AST until no more apply.
To funnel abnormal loop entries/exits, we first associate a number to each node that has an incoming abnormal edge. We then introduce a structuring variable and redirect each node's incoming abnormal edges to a new node that assigns the value we associated with that node to the structuring variable. We connect these new nodes to a condition cascade that checks the value of the structuring variable and directs control flow to the appropriate destination.
## Implementation Overview
All of the code for this algorithm lives in `backend::ctrl_flow_struct`. The correctness-critical parts live in the main module and the refinements live in the submodule `refinement`. `condition` contains the implementation of conditions; in particular it contains the code to simplify conditions which is necessary for `refinement` to be able to do anything useful. All of the code is generic over the particular concrete AST implementation; all operations on the underlying AST (such as introducing new assignment statements) are done through the trait `ast_context::AstContextMut`.
The code that interfaces between this algorithm and the rest of the decompiler lives in the modules `backend::lang_c::c_cfg::ctrl_flow_struct` and `backend::ctrl_flow_struct::export`.
## Further improvements
- There could be options to switch off various refinements.
- Loop membership/successor refinement could be tweakable by the user (and could be moved to the `refinement` module).
- Conditions *should* probably be interned (like types in rustc). That way, comparing two conditions for equality can be done by a single pointer comparison instead of walking the whole expression graph.
- The algorithm will never produce a `continue` statements or a labeled `break` or `continue` statement. There are probably cases where using these would make the output simpler.
- The semantic model for conditions in condition nodes in control flow graphs (as described in the paper) appears to be "a pure predicate on the program state evaluated when control flow enters the node". However, this is obviously violated when we effectively "sink" conditions to each node during acyclic region structuring. Condition-based refinement appears to partly resolve this by "rehoisting" repeated conditions into bigger `if`-statements, but that doesn't happen for all conditions. The paper briefly touches on this problem (section IV.D on pg. 9), but the authors never conclusively show that these semantics are preserved.
    - This could be fixed by having the algorithm convert every condition node into an assignment of its value to a boolean variable and using only that variable in the reaching conditions. There would also need to be a pass that inlines that variable when appropriate.
    - Also, there exist cases where conditions in the output AST are evaluated "eagerly" (i.e. before they "should" be evaluated according to the control flow graph). [example](https://hackmd.io/x_lOEIIyR9etkCecbpLFeg#)
# Some Other Things I Also Did
- (before GSoC) [Implemented a parser for textual IR](https://github.com/radareorg/radeco-lib/pull/126)
- [Made a test driver for `radeco-lib` that we never got around to using](https://github.com/HMPerson1/radeco-csmith-tester)
- [Added a pass that determines how a function uses each register and also a pass that turns `(x+4)-4` into `x+0` and then into `x` all in the same pull request for some reason](https://github.com/radareorg/radeco-lib/pull/140)
- [Brutally annihilated the dependency on capstone](https://github.com/radareorg/radeco-lib/pull/194)
# Links
- [GSoC project page](https://summerofcode.withgoogle.com/projects/#4679353015730176)
- [Pull Requests](https://github.com/radareorg/radeco-lib/pulls?q=is%3Apr+author%3AHMPerson1+created%3A2018-05-01..2018-08-14)
- [Commits](https://github.com/radareorg/radeco-lib/commits?author=HMPerson1&since=2018-05-01&until=2018-08-14)
