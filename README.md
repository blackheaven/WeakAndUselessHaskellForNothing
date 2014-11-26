WeakAndUselessHaskellForNothing
===============================

This is a try to explain Haskell from the point of view of the types system instead of from the expressions system.
We often consider the most important things first, also learning Haskell via its expressions system makes the types system look like an overlay.

My goal is to delay as much as possible the use of values and expressions (such as numbers, caracters, booleans but also all constructors and
functions) while staying clear and concise.

## What are types?
Types are **spaces**, they represent existing things - in term of values.
Types must not represent outlier values.
For example, the type *Integer* represents all the whole numbers but not the real numbers or strings or graphs, or emails, etc.

Each time you declare a type, make sure it is able to **only** represent meaningful values, not more, not less.
If you create an *Email* type, it should only hold a well-formed Email, not any Strings supposed to be an Email.

### Types' genesis

Here are the two ways to declare new types:

```haskell
data A
newtype B
```

``data A`` create an [Algebraic data type](#algebraic-data-types) named *A*.

``newtype B`` allows the compiler to distinguish some values of an existing type.
The original type and the new type -named *B*- have same values but these values are either of original type or of new type.
Types are differents even if they hold same values.
