WeakAndUselessHaskellForNothing
===============================

This is a try to explain Haskell from the point of view of the types system instead of from the expressions system.
We often consider the most important things first, also learning Haskell via its expressions system makes the types system look like an overlay.

My goal is to delay as much as possible the use of values and expressions (such as numbers, caracters, booleans but also all constructors and
functions) while staying clear and concise.

## What are types?
Types are **spaces**, they represent existing things in term of values.
Types must not represent outlier values.
For example, the type ``Integer`` represents all the whole numbers but not the real numbers or strings or graphs, or emails, etc.

Types must **solely** represent meaningful values, not more, not less.
If you create an ``Email`` type, it should only hold a well-formed Email, not any Strings supposed to be an Email.

### Types' genesis

Here are the two ways to declare new types:

```haskell
data A
newtype B
```

``data A`` create an [Algebraic data type](#algebraic-data-types) named ``A``.

``newtype B`` allows the compiler to distinguish some values of an existing type.
The original type and the new type, named ``B``, have same values but these values are either of original type or of new type.
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

The command ``:k`` prints the *Kind* of an type expression.
For example the *Kind* of ``A`` is ``*`` meaning it is concrete while parameterized types' *Kind* of ``C``, ``D``, and ``E`` are a ``*`` preceded by as many ``* ->`` as they have parameters.
Providing types to parameterized types change their *Kind*:

```haskell
Prelude> :k E A
E A :: * -> *
Prelude> :k E A A
E A A :: *
```
**Applying a type**, by adding a type after an other, changes the *Kind* which needs of one less parameter.
Applying two types ``A`` to ``E`` make it a concrete type.

Type application enforces the *Kind* of the applied type.
For example ``E`` needs two concrete types and ``A`` is one of them.
We can't apply a parameterized type when a concrete type is needed:

```haskell
Prelude> :k E D

<interactive>:1:3:
    Expecting two more arguments to ‘D’
    The first argument of ‘E’ should have kind ‘*’,
      but ‘D’ has kind ‘* -> * -> *’
    In a type in a GHCi command: E D

```

Types and types expression, with the same *Kind*, fill without distinction any type parameters with this expected *Kind*.
The following type is valid:

```haskell
Prelude> :k E (C A) (D A (E A A))
E (C A) (D A (E A A)) :: *
```

## Algebraic data-types
Types are spaces, they are composable via two operations:


* ``Sum``: the resulting space is the sum of its components's space. Only one of its parametric types set the space at a time.
* ``Product``: the resulting space is the product of its components's space. Both of its parametric types set the space, everytime.

Here are their definition:
```haskell
data Sum a b
data Product a b
```
``Sum`` is often called ``(,)`` and ``Product`` is often called ``Either``.

Apart these operations there are also type-level values:

* ``Void``: it holds one value, its space is equal to one
* ``Identity``: it holds the same space of its parametric type, like ``newtype``.

Here are their definition:
```haskell
data Void
data Identity a
```
``Void`` is often called ``()``.

Here are some examples of the evolution of space during composition:
```haskell
Product Void Void
-- Resulting space = 1 (1 from Void * 1 from Void)
Sum Void Void
-- Resulting space = 2 (1 from Void + 1 from Void)
Product a Void
-- Resulting space = 1 * a's space
Sum a Void
-- Resulting space = a's space + 1
Product (Sum Void (Sum Void Void)) (Sum Void Void)
-- Resulting space = (1 + (1 + 1)) * (1 + 1) = 3 * 2 = 6
```
