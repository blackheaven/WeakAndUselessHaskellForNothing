WeakAndUselessHaskellForNothing
===============================

This is a try to explain Haskell from the point of view of the types system instead of from the expressions system.
We often consider the most important things first, also learning Haskell via its expressions system makes the types system look like an overlay.

My goal is to delay as much as possible the use of values and expressions (such as numbers, caracters, booleans but also all constructors and
functions) while staying clear and concise.

## What are types?
Types are *spaces*, they represent existing things in term of values.
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
For example the *Kind* of ``A`` is ``*`` meaning it is concrete.
While the *Kind* of parameterized types -like ``C``, ``D``, and ``E``-
are a ``*`` preceded by as many ``* ->`` as they have parameters.

Providing types to parameterized types change their *Kind*:

```haskell
Prelude> :k E A
E A :: * -> *
Prelude> :k E A A
E A A :: *
```
*Applying a type*, by adding a type after an other, changes the *Kind* which
needs of one less parameter.
Applying two types ``A`` to ``E`` make it a concrete type.

Type application enforces the *Kind* of the applied type.
For example ``E`` needs two concrete types and ``A`` is one of them.
We aren't able to apply a parameterized type when a concrete type is needed:

```haskell
Prelude> :k E D

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

### Types manipulation
Type expressions are sometimes complex and deface type's intent.

*Types functions* create an other way to call the same type expression.
Spaces and types remain unchanged.

Here are some examples:
```haskell
Prelude> :k E (E A A) (D A (E A A))
E (E A A) (D A (E A A)) :: *
Prelude> type Eaa = E A A
Prelude> :kind! E Eaa (D A Eaa)
E Eaa (D A Eaa) :: *
= E (E A A) (D A (E A A))
Prelude> type Double e a = e a a
Prelude> :kind! E (Double E A) (D A (Double E A))
E (Double E A) (D A (Double E A)) :: *
= E (E A A) (D A (E A A))
Prelude> type Ead = Double E A
Prelude> :kind! E Ead (D A Ead)
E Ead (D A Ead) :: *
= E (E A A) (D A (E A A))
Prelude> type MapSnd x a f = x a (f a)
Prelude> :kind! MapSnd E (Double E A) (D A)
MapSnd E (Double E A) (D A) :: *
= E (E A A) (D A (E A A))

```

*Type families* allow us to do operations on types, such as comparisons, called *pattern matching*.
Here is an implementation of the ``Boolean`` operator ``And``:
```haskell
data True
data False

type family And a b where
  And True  d     = d
  And False c     = False
```

Play with it:
```haskell
Prelude> :kind! And True False
And True False :: *
= False
Prelude> :kind! And True False
And True False :: *
= False
Prelude> :kind! And True True
And True True :: *
= True
```

### Kinds creating and enforcing
The definition of ``And`` is weak, it possible to apply a non-``Boolean`` type
and get a non-``Boolean`` as resulting type:
```haskell
Prelude> :kind! And True A
And True A :: *
= A
```

We need to create a new *Kind* which only holds ``True`` and ``False`` and
enforce it at *type family* level.
then enforce it at *type family* level.

To create a new *Kind*, we define a value-level
[Algebraic data type](#algebraic-data-types) with two values:
```haskell
data Bool = True | False
```

We indicate the *Kind* of each parametric type of the *type family* and leave
the rest unchanged:
```haskell
type family And (a :: Bool) (b :: Bool) :: Bool where
  And True  d     = d
  And False c     = False
```

We are able to use [Algebraic data type](#algebraic-data-types) values, also
called *constructors*, at type-level.

We are now only able to manipulate ``Booleans`` with this *type family*:
```haskell
Prelude> :kind! And True True
And True True :: Bool
= 'True
Prelude> :kind! And True False
And True False :: Bool
= 'False
Prelude> :kind! And False True
And False True :: Bool
= 'False
Prelude> :kind! And False A

    The second argument of ‘And’ should have kind ‘Bool’,
      but ‘A’ has kind ‘*’
    In a type in a GHCi command: And False A
```
``Bool`` is a new *Kind*, as ``*`` and ``* -> *``and ``Left`` and ``Right`` are new types.
It's called *types promotion*.

## Algebraic data-types
### Operations
Types are spaces, they are composable via two operations:

* ``Sum``: the resulting space is the sum of its components's space. Only one of its parametric types set the space at a time.
* ``Product``: the resulting space is the product of its components's space. Both of its parametric types set the space, everytime.

Here are their definition:
```haskell
data Product a b

data Branch = Left | Right
type family Sum v a b where
  Sum Left  l r = l
  Sum Right l r = r
```
``Product`` is often called ``(,)`` and ``Sum`` is often called ``Either``.
``Product`` is a cartesian product of spaces.
``Sum`` is more complicated because we need to check the selected type and *types families* do the job well.
Let's try:

```haskell
Prelude> :kind! Sum Left A B
Sum Left A B :: *
= A
Prelude> :kind! Sum Right A B
Sum Right A B :: *
= B
```

#### Kind polymorphism
Our previous declaration of ``Sum`` doesn't precise the *Kinds*, let's see
what *Kinds* are deduced by the compiler:

```haskell
Prelude> data A
Prelude> data B
Prelude> data C a
Prelude> :kind! Sum Left A B
Sum Left A B :: *
= A
Prelude> :kind! Sum Right A B
Sum Right A B :: *
= B
Prelude> :kind! Sum Right A C

    Expecting one more argument to ‘C’
    The third argument of ‘Sum’ should have kind ‘*’,
      but ‘C’ has kind ‘* -> *’
    In a type in a GHCi command: Sum Right A C
```

The compiler infers the ``*`` *Kind*, in order to change that we have to use
*Kind polymorphism*:

```haskell
data Branch = Left | Right

type family Sum (v :: Branch) (a :: k) (b :: k) :: k where
  Sum Left  l r = l
  Sum Right l r = r
```

Let's see what happen:

```haskell
Prelude> :kind! Sum Left A B
Sum Left A B :: *
= A
Prelude> :kind! Sum Right A B
Sum Right A B :: *
= B
Prelude> :kind! Sum Right C D
Sum Right C D :: * -> *
= D
Prelude> :kind! Sum Right C B

    The third argument of ‘Sum’ should have kind ‘* -> *’,
      but ‘B’ has kind ‘*’
    In a type in a GHCi command: Sum Right C B
```


We aren't able to mix up *Kinds*.
*Kind polymorphism* act like parameterized types: it forces the *Kind* unicity.

### Values
Apart these operations there are also type-level values:

* ``Void``: it holds no value, its space is empty.
* ``Unit``: it holds one value, its space is equal to one.

Here are their definition:
```haskell
data Void
data Unit
```
``Unit`` is often called ``()``.

Here are some examples of the evolution of space during composition:
```haskell
Product Unit Unit
-- Resulting space = 1 (1 from Unit * 1 from Unit)
Sum Unit Unit
-- Resulting space = 2 (1 from Unit + 1 from Unit)
Product a Unit
-- Resulting space = 1 * a's space
Sum a Unit
-- Resulting space = a's space + 1
Product (Sum Unit (Sum Unit Unit)) (Sum Unit Unit)
-- Resulting space = (1 + (1 + 1)) * (1 + 1) = 3 * 2 = 6
```

## Maybe: ability to (not) have a type
The possibility to don't have a type is represented by ``Maybe``:
```haskell
data IsMaybe = Just | Nothing

type family Maybe v a where
  Maybe Just    x = x
  Maybe Nothing x = Void
```

An alternative of this implementation is to use ``Product``:
```haskell
type Just = Right
type Nothing = Left
type Maybe v x = Product v Void x
```

This one avoid the creation a new *Kind* and of new types, it decreases
type-safety but improves the reusability.

## Parametric types manipulation
Our [``Sum``](#algebraic-data-types) type forces its types to have the same
*Kind*:
```haskell
Prelude> :kind! Sum Left A C

    Expecting one more argument to ‘C’
    The third argument of ‘Sum’ should have kind ‘*’,
      but ‘C’ has kind ‘* -> *’
    In a type in a GHCi command: Sum Left A C
```

### Const: add one
We can add a parametric just to match the expected *Kind* and do nothing with it:
```haskell
type Const t a = t
```

Our ``Sum`` type works again:
```haskell
Prelude> :kind! Sum Left (Const A) C
Sum Left (Const A) C :: * -> *
= Const A
```

### Flip: change the order
We are also able to change the order of any ``Sum`` type:
```haskell
type Flip t a b = t b a
```

Applied to ``Sum``:
```haskell
Prelude> :kind! Flip (Sum Right) A B
Flip (Sum Right) A B :: *
= A
```

#### Currying and partial application
Each time you apply a type to a parameterized type its
[*Kind* changes](#kinds-of-types-and-kinds).
A parameterized type take only one parametric type and generate either a
concrete type or an other parameterized type.
Multi-parameterized types are a syntactic sugar:
```haskell
Prelude> :k D
D :: * -> * -> *
Prelude> :k D A
D A :: * -> *
Prelude> :k D A B
D A B :: *
Prelude> :k C
C :: * -> *
Prelude> type C' = C
Prelude> :k C'
C' :: * -> *
```

It can also be parenthesized:
```haskell
Prelude> :k D
D :: * -> (* -> *)
Prelude> :k D A B
(D A) B :: *
```

Partial application is the generation of a parameterized type resulting of the application of a strict subset of its parameters.
## Indexed structures
Parameterized types take an arbitrary number of various types.
*Indexed structures* fill the same goal: providing a flexible way to manage a
set of types.

### ``List``s
#### Structure
A list is a [``Sum``](#algebraic-data-types) type of a list-ending and
a [``Product``](#algebraic-data-types) type of an element and an other list,
the next element:
```haskell
type Cons (e :: k1) (n :: k2) = Product e n
type End = Void

Prelude> :kind! Cons A (Cons B (Cons C (Cons D (Cons (E A) End))))
Cons A (Cons B (Cons C (Cons D (Cons (E A) End)))) :: *
= Product
    A (Product B (Product C (Product D (Product (E A) Void))))
```

An alternative with [``Maybe``](#algebraic-data-types):
```haskell
type Cons e n = Maybe Just (Product e n)
type End = Nothing

Prelude> :kind! Cons A (Cons B (Cons C (Cons D (Cons (E A) End))))
Cons A (Cons B (Cons C (Cons D (Cons (E A) End)))) :: *
= forall (k :: BOX) (k :: BOX) (k :: BOX) (k :: BOX).
  Product
    A (Product B (Product C (Product D (Product (E A) 'Nothing))))
```
