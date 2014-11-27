WeakAndUselessHaskellForNothing
===============================

This is a try to explain Haskell from the point of view of the types system instead of from the expressions system.
We often consider the most important things first, also learning Haskell via its expressions system makes the types system look like an overlay.

My goal is to delay as much as possible the use of values and expressions (such as numbers, caracters, booleans but also all constructors and
functions) while staying clear and concise.

## What are types?
Types are **spaces**, they represent existing things - in term of values.
Types must not represent outlier values.
For example, the type ``Integer`` represents all the whole numbers but not the real numbers or strings or graphs, or emails, etc.

Each time you declare a type, make sure it is able to **only** represent meaningful values, not more, not less.
If you create an ``Email`` type, it should only hold a well-formed Email, not any Strings supposed to be an Email.

### Types' genesis

Here are the two ways to declare new types:

```haskell
data A
newtype B
```

``data A`` create an [Algebraic data type](#algebraic-data-types) named ``A``.

``newtype B`` allows the compiler to distinguish some values of an existing type.
The original type and the new type -named ``B``- have same values but these values are either of original type or of new type.
Types are differents even if they hold same values.

## Kinds of types and Kinds
There are two kinds of types:

 * *Concrete types* like ``A`` declared via ``data A``. They have a fixed space.
 * *Parameterized types* declared like any other types followed by at least a lower case letter. For example ``data C x``. Their space depend on their parametric types' space.

With [ghci](https://downloads.haskell.org/~ghc/7.8.3/docs/html/users_guide/ghci.html) you should have this output:
```haskell
Prelude> data A
Prelude> :k A
A :: *
Prelude> data C x
Prelude> :k C
C :: * -> *
Prelude> data D x y
Prelude> :k D
D :: * -> * -> *
Prelude> data E a b
Prelude> :k E
E :: * -> * -> *
```
``:k`` is the command to print the *Kind* of a type.
``A`` is concrete and its *Kind* is ``*``.

However ``C``, ``D``, and ``E`` are parametrized their *Kind* are a ``*`` preceded by as many ``* ->`` as they have parameters.
Providing types to these kinds of types change their *Kind*:

```haskell
Prelude> :k E A
E A :: * -> *
Prelude> :k E A A
E A A :: *
```
Placing a type -``A`` here- after an other is called **applying a type**.
Applying a type changes the *Kind* which need of one less parameter.
So we need to apply two types to ``E`` to have a concrete type.

They are different types of *Kinds* and each time you apply a type this type have to have the right *Kind*.
For example ``E`` needs two concrete types and ``A`` is one of them.
But if we try to apply a parameterized type, we get an error:

```haskell
Prelude> :k E D

<interactive>:1:3:
    Expecting two more arguments to ‘D’
    The first argument of ‘E’ should have kind ‘*’,
      but ‘D’ has kind ‘* -> * -> *’
    In a type in a GHCi command: E D

```

An expected *Kind* can be filled with either a type with the right *Kind* -like ``A``- or a type-expression having the right *Kind* -like ``C A``, ``D A A``, and so on.
The following type is valid:

```haskell
Prelude> :k E (C A) (D A (E A A))
E (C A) (D A (E A A)) :: *
```
