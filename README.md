WeakAndUselessHaskellForNothing
===============================

I conduct an experiment that try to explain Haskell from a types system point of view instead of from the expression system.

The idea behind this is that we should do the most important things before, in this context we are more focused on these ones. Test-Driven
Development suffers from the same issue, some ones have told us how to develop without tests, so, when we discover tests we tend to value them less
than the production code even if they are more valuable (and we should take more care of them).

My goal is simple delay the more that I can the use of value-expressions (such as numbers, caracters, booleans but also all constructors and
functions).


## What are types?
Types are _spaces_, types represent things, in term of values, that exists not things that don't or can't exist. For example, the type *Integer*
represents all the whole numbers but not the real numbers or strings or graphs, or emails, and so on.

Each time you declare a type make sur that it cans represent _only_ the meaningful values, not more, not less. If you create an *Email* type, make
sure that it's a well-formed Email, not a lambda String supposed to be an Email.
