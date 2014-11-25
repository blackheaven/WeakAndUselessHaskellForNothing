WeakAndUselessHaskellForNothing
===============================

This is a try to explain Haskell from the point of view of the types system instead of from the expressions system.
We often consider the most important things first, also learning Haskell via its expressions system makes the types system look like an overlay.

My goal is to delay as much as possible the use of values and expressions (such as numbers, caracters, booleans but also all constructors and
functions) while staying clear and concise.

## What are types?
Types are **spaces**, they represent things - in term of values - that exists.
Type don't represent things that don't or can't exist.
For example, the type *Integer* represents all the whole numbers but not the real numbers or strings or graphs, or emails, etc.

Each time you declare a type, make sure that it cans represent **only** the meaningful values, not more, not less.
If you create an *Email* type, it should only hold a well-formed Email, not any Strings supposed to be an Email.


