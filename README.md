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
If we create an ``Email`` type, it should only hold a well-formed Email, not any Strings supposed to be an Email.

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
*Applying a type*, by adding a type after another, changes the *Kind* which
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

*Types functions* create another way to call the same type expression.
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

*Closed type families* allow us to do operations on types, such as comparisons, called *pattern matching*.
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

To create a new *Kind*, we define a value-level
[Algebraic data type](#algebraic-data-types) with two values:
```haskell
data Boolean = True | False
```

We indicate the *Kind* of each parametric type of the *type family* and leave
the rest unchanged:
```haskell
type family And (a :: Boolean) (b :: Boolean) :: Boolean where
  And True  d     = d
  And False c     = False
```

We are able to use [Algebraic data type](#algebraic-data-types) values, also
called *constructors*, at type-level.

We are now only able to manipulate ``Booleans`` with this *closed type family*:
```haskell
Prelude> :kind! And True True
And True True :: Boolean
= 'True
Prelude> :kind! And True False
And True False :: Boolean
= 'False
Prelude> :kind! And False True
And False True :: Boolean
= 'False
Prelude> :kind! And False A

    The second argument of ‘And’ should have kind ‘Boolean’,
      but ‘A’ has kind ‘*’
    In a type in a GHCi command: And False A
```
``Boolean`` is a new *Kind*, as ``*`` and ``* -> *``and ``Left`` and ``Right`` are new types.
It's called *types promotion*.

## Algebraic data-types
### Operations
Types are spaces, they are composable via two operations:

* ``Sum``: the resulting space is the sum of its components's space. Only one of its parametric types set the space at a time.
* ``Product``: the resulting space is the product of its components's space. Both of its parametric types set the space, everytime.

Here are their definition:
```haskell
data Product a b

data Sum a b = Left a | Right b
```
``Product`` is often called ``(,)`` and ``Sum`` is often called ``Either``.
``Product`` is a cartesian product of spaces.
``Sum`` is a new *Kind* using the *type promotion*.
Let's try:

```haskell
Prelude> :kind! Left A
Left A :: Sum * k
= forall (k :: BOX). 'Left A
Prelude> :kind! Right B
Right B :: Sum k *
= forall (k :: BOX). 'Right B
Prelude> :kind! Right C
Right C :: Sum k (* -> *)
= forall (k :: BOX). 'Right C
```

The compiler infers *Kinds* from from used parametric types.
When ``Left`` is used only the first parametric type is used so the second stay
to ``k``.

*Promoted types* are prefixed by a ``'``, this is useful when a constructor
shares the same name of its type:
```haskell
data Unit = Unit

Prelude> :k Unit
Unit :: *
Prelude> :k 'Unit
'Unit :: Unit'
```

#### Sorts and Kind polymorphism
Every *Kind* have the same *Sort* named *BOX*, a *Sort* is a type of *Kind*.
All the *Kinds* are not equals.

We can't use a type of a *Kind* when another is expected:
```haskell
Prelude> :k Product
Product :: * -> * -> *
Prelude> :k Product A C

    Expecting one more argument to ‘C’
    The second argument of ‘Product’ should have kind ‘*’,
      but ‘C’ has kind ‘* -> *’
    In a type in a GHCi command: Product A C
```

The compiler infers a ``*`` *Kind* by default.
To change this, we have to specify that we accept all the *Kinds*:
```haskell
data Product (a :: k0) (b :: k1)
Prelude> :k Product A C
Product A C :: *
```

We are also able to declare cross-parametric constraints:
```haskell
type Application (t :: k0 -> k) (a :: k0) = t a
Prelude> :kind! Application C A
Application C A :: *
= C A
```

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
data Maybe a = Nothing | Just a
```

An alternative of this implementation is to use ``Sum``:
```haskell
type Nothing = Left Void
type Just a = Right a
type Maybe a = Sum Void a
```

This one avoid the creation a new *Kind* and of new types, it decreases
type-safety but improves the reusability.

## Parametric types manipulation
We represent *type application* like this:

```haskell
type Application t a = t a
```

Its first argument must be ``* -> *`` *Kinded*:
```haskell
Prelude> :kind! Application A A

    The first argument of ‘Application’ should have kind ‘* -> *’,
      but ‘A’ has kind ‘*’
    In a type in a GHCi command: Application A A
```

### Const: add one
We can add a parametric just to match the expected *Kind* and do nothing with it:
```haskell
type Const t a = t
```

Our ``Application`` type works again:
```haskell
Prelude> :kind! Application (Const A) A
Application (Const A) A :: *
= A
```

### Flip: change the order
We are also able to change the order of any ``Product`` type:
```haskell
type Flip t a b = t b a
```

Applied to ``Product``:
```haskell
Prelude> :kind! Flip Product A B
Flip Product A B :: *
= Product B A
```

#### Currying and partial application
Each time we apply a type to a parameterized type its
[*Kind* changes](#kinds-of-types-and-kinds).
A parameterized type take only one parametric type and generate either a
concrete type or another parameterized type.
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
A list is a [``Sum``](#algebraic-data-types) type of a list-ending and
a [``Product``](#algebraic-data-types) type of an element and another list,
the next element:
```haskell
type Cons (e :: k1) (n :: k2) = Product e n
type Nil = Void

Prelude> :kind! Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil))))
Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil)))) :: *
= Product
    A
    (Product
       B (Product (C A) (Product (D A B) (Product (E A B) Void)))
```

An alternative with [``Maybe``](#algebraic-data-types):
```haskell
type Cons e n = Just (Product e n)
type Nil = Nothing

Prelude> :kind! Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil))))
Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil)))) :: Maybe
                                                                    *
= forall (k :: BOX).
  'Just
    (Product
       A
       ('Just
          (Product
             B
             ('Just
                (Product
                   (C A)
                   ('Just (Product (D A B) ('Just (Product (E A B) 'Nothing)))))))))
```

Our reference implementation:
```haskell
data List a = Nil | Cons a (List a)

Prelude> :kind! Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil))))
Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil)))) :: List *
= 'Cons
    A ('Cons B ('Cons (C A) ('Cons (D A B) ('Cons (E A B) 'Nil))))
```

The value space is ``L = 1 + A * L``.

For readability purposes we will use these notations:
```haskell
Prelude> :kind! A ': B ': (C A) ': (D A B) ': (E A B) ': '[]
A ': B ': (C A) ': (D A B) ': (E A B) ': '[] :: [*]
= '[A, B, C A, D A B, E A B]
Prelude> :kind! '[A, B, C A, D A B, E A B]
'[A, B, C A, D A B, E A B] :: [*]
= '[A, B, C A, D A B, E A B]
```

### ``Tree``s
A tree is a [``Sum``](#algebraic-data-types) type of a
leaf and a [``Product``](#algebraic-data-types) type of a
value and a [``Product``](#algebraic-data-types) type of
two trees:
```haskell
type Node x l r = Product x (Product l r)
type Tip = Void

:kind! Node A (Node B (Node A Tip Tip) Tip) Tip
Node A (Node B (Node A Tip Tip) Tip) Tip :: *
= Product
    A
    (Product
       (Product B (Product (Product A (Product Void Void)) Void)) Void)
```

An alternative with [``Maybe``](#algebraic-data-types):
```haskell
type Tree x l r = Maybe (Product x (Product l r))
type Node (x :: k0) (l :: k1) (r :: k1) = Just (Product x (Product l r))
type Tip = Nothing

Prelude> :kind! Node A (Node B (Node A Tip Tip) Tip) Tip
Node A (Node B (Node A Tip Tip) Tip) Tip :: Maybe *
= forall (k :: BOX).
  'Just
    (Product
       A
       (Product
          ('Just
             (Product
                B
                (Product
                   ('Just (Product A (Product 'Nothing 'Nothing))) 'Nothing)))
          'Nothing))
```

Our reference implementation:
```haskell
data Tree a = Tip | Node a (Tree a) (Tree a)

Prelude> :kind! Node A (Node B (Node A Tip Tip) Tip) Tip
Node A (Node B (Node A Tip Tip) Tip) Tip :: Tree *
= 'Node A ('Node B ('Node A 'Tip 'Tip) 'Tip) 'Tip
```

The value space is ``L = 1 + A * L²``.

### ``Zipper``s
``Zipper``s are a kind of data structures that track traversing:
```haskell
data Zipper a = Zipper [a] a [a]
```

It supports forward and backward traversing:
```haskell
type family ShiftLeft a where
  ShiftLeft ('Zipper (x ': xs) y ys) = 'Zipper xs x (y ': ys)

type family ShiftRight a where
  ShiftRight ('Zipper xs x (y ': ys)) = 'Zipper (x ': xs) y ys

Prelude> :kind! 'Zipper '[A, B, C A] (D A B) '[E A B]
'Zipper '[A, B, C A] (D A B) '[E A B] :: Zipper *
= 'Zipper '[A, B, C A] (D A B) '[E A B]
Prelude> :kind! ShiftLeft ('Zipper '[A, B, C A] (D A B) '[E A B])
ShiftLeft ('Zipper '[A, B, C A] (D A B) '[E A B]) :: Zipper *
= 'Zipper '[B, C A] A '[D A B, E A B]
Prelude> :kind! ShiftLeft (ShiftLeft ('Zipper '[A, B, C A] (D A B) '[E A B]))
ShiftLeft (ShiftLeft ('Zipper '[A, B, C A] (D A B) '[E A B])) :: Zipper *
= 'Zipper '[C A] B '[A, D A B, E A B]
Prelude> :kind! ShiftLeft (ShiftRight ('Zipper '[A, B, C A] (D A B) '[E A B]))
ShiftLeft (ShiftRight ('Zipper '[A, B, C A] (D A B) '[E A B])) :: Zipper *
= 'Zipper '[A, B, C A] (D A B) '[E A B]
```

``Zipper``s can be defined with [Algebraic data type](#algebraic-data-types):
```haskell
Prelude> :kind! Zipper '[A, B, C A] (D A B) '[E A B]
Zipper '[A, B, C A] (D A B) '[E A B] :: *
= Product (D A B) (Product '[A, B, C A] '[E A B])
Prelude> :kind! ShiftLeft (Zipper '[A, B, C A] (D A B) '[E A B])
ShiftLeft (Zipper '[A, B, C A] (D A B) '[E A B]) :: *
= Product A (Product '[B, C A] '[D A B, E A B])
Prelude> :kind! ShiftLeft (ShiftLeft (Zipper '[A, B, C A] (D A B) '[E A B]))
ShiftLeft (ShiftLeft (Zipper '[A, B, C A] (D A B) '[E A B])) :: *
= Product B (Product '[C A] '[A, D A B, E A B])
Prelude> :kind! ShiftRight (Zipper '[A, B, C A] (D A B) '[E A B])
ShiftRight (Zipper '[A, B, C A] (D A B) '[E A B]) :: *
= Product (E A B) (Product '[D A B, A, B, C A] '[])
Prelude> :kind! ShiftLeft (ShiftRight (Zipper '[A, B, C A] (D A B) '[E A B]))
ShiftLeft (ShiftRight (Zipper '[A, B, C A] (D A B) '[E A B])) :: *
= Product (D A B) (Product '[A, B, C A] '[E A B])
```

### Isomorphisms
*Morphisms* are ways to transform expressions into others expressions.
At the type level they can be represented like this:
```haskell
data A
data B
data D a b
data E a b

type family Morphism (a :: k0) (b :: k1) (c :: k2) :: *
type instance Morphism t t t = t

type instance Morphism A B A = B
type instance Morphism B A B = A

type instance Morphism D E (D a b) = E a b

type instance Morphism ('Left Void) 'Nothing ('Left Void) = 'Nothing
type instance Morphism 'Right 'Just ('Right a) = 'Just a

type instance Morphism 'Nothing ('Left Void)'Nothing = 'Left Void
type instance Morphism 'Just 'Right ('Just a)= 'Right a
```

We have used *opened type families* because there is not a closed set of
*morphisms.*
```haskell
Prelude> :kind! Morphism A B A
Morphism A B A :: *
= B
Prelude> :kind! Isomorphism A B A
Isomorphism A B A :: *
= A
Prelude> :kind! Morphism D E (D A B)
Morphism D E A B :: *
= forall (k :: BOX) (k :: BOX). E A B
```

An *isomorphism* between two expressions is the existence of a *morphism*
from ``A`` to ``B`` and from ``B`` to ``A`` such that, if composed, give the
same expression. It is also called *Identity morphism*:
```haskell
type Isomorphism (a :: k0) (b :: k1) (v :: k2) = Morphism b a (Morphism a b v)
type IsIsomorphic (a :: k0) (b :: k1) (v :: k2) = Isomorphism a b v ~ v

Prelude> :kind! Morphism B (Morphism A B)
Morphism B (Morphism A B) :: *
= B
Prelude> :kind!  Isomorphism B A
Isomorphism B A :: *
= B
```

Here's an attempt to show the *isomorphisms* between the different implementations of
the [indexed structures](#indexed-structures).

#### ``List``'s isomorphisms
```haskell
type Cons1 (e :: k1) (n :: k2) = Product e n
type Nil1 = Void

type Cons2 e n = Just (Product e n)
type Nil2 = Nothing

data List3 a = Nil3 | Cons3 a (List3 a)

type family List1_List2 a where
  List1_List2 (Cons1 a b) = Cons2 a (List1_List2 b)
  List1_List2 Nil1 = Nil2

type family List2_List1 a where
  List2_List1 (Cons2 a b) = Cons1 a (List2_List1 b)
  List2_List1 Nil2 = Nil1

type family List2_List3 a where
  List2_List3 (Cons2 a b) = Cons3 a (List2_List3 b)
  List2_List3 Nil2 = Nil3

type family List3_List2 a where
  List3_List2 (Cons3 a b) = Cons2 a (List3_List2 b)
  List3_List2 Nil3 = Nil2

type List1_List3 a = List2_List3 (List1_List2 a)
type List3_List1 a = List2_List1 (List3_List2 a)
```

#### ``Tree``'s isomorphisms
```haskell
type Node1 x l r = Product x (Product l r)
type Tip1 = Void

type Tree2 x l r = Maybe (Product x (Product l r))
type Node2 (x :: k0) (l :: k1) (r :: k1) = Just (Product x (Product l r))
type Tip2 = Nothing

data Tree3 a = Tip3 | Node3 a (Tree3 a) (Tree3 a)

type family Tree1_Tree2 x where
  Tree1_Tree2 (Node1 x l r) = Node2 x (Tree1_Tree2 l) (Tree1_Tree2 r)
  Tree1_Tree2 Tip1 = Tip2

type family Tree2_Tree1 x where
  Tree2_Tree1 (Node2 x l r) = Node1 x (Tree2_Tree1 l) (Tree2_Tree1 r)
  Tree2_Tree1 Tip2 = Tip1

type family Tree2_Tree3 x where
  Tree2_Tree3 (Node2 x l r) = Node3 x (Tree2_Tree3 l) (Tree2_Tree3 r)
  Tree2_Tree3 Tip2 = Tip3

type family Tree3_Tree2 x where
  Tree3_Tree2 (Node3 x l r) = Node2 x (Tree3_Tree2 l) (Tree3_Tree2 r)
  Tree3_Tree2 Tip3 = Tip2

type Tree1_Tree3 x = Tree2_Tree3 (Tree1_Tree2 x)
type Tree3_Tree1 x = Tree2_Tree1 (Tree3_Tree2 x)
```

#### Fix
There a common pattern between these types: they are composed of a recursion
part and a "data" part.
The recursion part is called *Fix* point and can be defined as this:
```haskell
newtype Fix f a = Fix (f (Fix f a))
```

Due to the polymorphic nature of it's constructor, ``Fix`` is not promotable
to the type-level.
We are forced to declare a weaker type [isomorphic](#isomorphisms) to ``List``:
```haskell
data Fix (f :: k1 -> k0) (n :: k1)
```

The data part is always or a terminaison term or the value which is
[isomorphic](#isomorphisms) to ``Maybe``:
```haskell
type family SN x where
  SN ('Left Void) = 'Nothing
  SN ('Right a) = 'Just a

type family NS x where
  NS 'Nothing = 'Left Void
  NS ('Just a) = 'Right a
```

We have abstracted this part like this:
```haskell
data Proxy (a :: k) = Proxy
newtype StructureF f a n = StructureF (Proxy ('Just (f a n)))
```
``newtype`` waits a type of *Kind* ``*`` and ``'Just`` has the *Kind* ``Maybe k``.
We use ``Proxy`` to transform any *Kind* into a ``*`` *Kind*.

This allow us to rewrite [``List``](lists-isomorphisms) and
[``Tree``](#trees-isomorphisms) like that:
```haskell
data Const (t :: k0) (a :: k1)

type Cons a n = Fix (StructureF Product a) n
type Nil = Fix (Const 'Nothing) 'Nothing

newtype NodeF a n = NodeF (Product a (Product n n))
type Node a n = Fix (StructureF Product a) n
type Tip = Fix (Const 'Nothing) 'Nothing

Prelude> :kind! Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil))))
Cons A (Cons B (Cons (C A) (Cons (D A B) (Cons (E A B) Nil)))) :: *
= Fix
    (StructureF Product A)
    (Fix
       (StructureF Product B)
       (Fix
          (StructureF Product (C A))
          (Fix
             (StructureF Product (D A B))
             (Fix
                (StructureF Product (E A B)) (Fix (Const 'Nothing) 'Nothing)))))
Prelude> :kind! Node A (Node B (Node A Tip))
Node A (Node B (Node A Tip)) :: *
= Fix
    (StructureF Product A)
    (Fix
       (StructureF Product B)
       (Fix (StructureF Product A) (Fix (Const 'Nothing) 'Nothing)))
```

##### Learning how to count: Peano numbers
``Integer``s don't exist àt type-level.
To surpass that limitation, we can use the Peano numbers.
They are generally defined like this:
```haskell
data Peano = Zero | Succ Peano

Prelude> :kind! Zero
Zero :: Peano
= 'Zero
Prelude> :kind! Succ (Succ Zero)
Succ (Succ Zero) :: Peano
= 'Succ ('Succ 'Zero)
```

Using [``Fix`` point](#fix), they are defined as following:
```haskell
type Succ n = Fix 'Just n
type Zero = Fix (Const 'Nothing) 'Nothing

Prelude> :kind! Zero
Zero :: *
= Fix (Const 'Nothing) 'Nothing
Prelude> :kind! Succ (Succ Zero)
Succ (Succ Zero) :: *
= Fix 'Just (Fix 'Just (Fix (Const 'Nothing) 'Nothing))
```

## Indexed computations
Having an unique recursion abstraction enable to define a set of computation
around it and apply them to all the abstractions we are able to
represent with this abstraction.

### ``Functor``: keep the container, change the content
A ``Functor`` is a ``Map`` function that take a container and and a morphism that
changes its content.
```haskell
type family MapFix (f :: (k0 -> k1) -> k0 -> k1) (a :: k2) :: * where
  MapFix f (Fix a n) = Fix (f a) (MapFix f n)

type family MapMaybe (f :: k0 -> k1) (a :: Maybe k0) :: Maybe k1 where
  MapMaybe f ('Just a) = 'Just (f a)
  MapMaybe f 'Nothing = 'Nothing
```

### ``ZipWith``
```haskell
type family ZipWith (f :: k0 -> k1 -> k2) (a :: k0) (b :: k1) :: k2 where
  ZipWith f (Fix a (Fix x xs)) (Fix b (Fix y ys)) = Fix (f a b) (ZipWith f (Fix x xs) (Fix y ys))
  ZipWith f (Fix a na) (Fix b nb) = Fix (f a b) 'Nothing
```

## From value-level to type-level
We will see to transform weak types into strong types.

