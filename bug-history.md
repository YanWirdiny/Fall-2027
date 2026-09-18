# Bug and debugging history 

## Bug #1 
j--  return  type null error 

root cause :
1. An analyze() method that forgets to set type.
That was the JRemainderOp.analyze() issue — it validated the operands but never did type = Type.INT; before return this;. Any node whose analyze() skips assigning type will hand back null to whoever calls .type() on it.
2. The parser built the wrong AST node, so the node that does get analyzed isn't the one you fixed.
That was the deeper cause: Parser.java created a JRemAssignOp (%=) instead of a JRemainderOp (%) for a % b. The wrong node's analyze() left type null, and the NPE surfaced one level up in JAssignOp.analyze() when it called rhs.type().

Interpretation : 
How to read it when it happens again

The stack trace tells you who tripped over the null, not necessarily who's at fault. In your case it died in JAssignOp.analyze (rhs.type()), but the actual culprit was the rhs node — either its analyze() didn't set type, or the parser gave you the wrong node type in the first place. So the debugging move is:

1. Identify which sub-expression is the one returning null (the rhs/lhs/operand being queried).
2. Check that node's analyze() actually assigns type.
3. If it looks fine, confirm the parser is even constructing the node you think it is.

Short version: type() == null ⇒ "this expression didn't get analyzed properly" ⇒ look at its analyze(), and double-check the parser handed you the right node.
