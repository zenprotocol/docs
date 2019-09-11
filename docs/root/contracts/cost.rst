Cost Analysis
=============

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

The Cost Model
--------------

Central to the analysis of contracts cost is the notion of a *cost model*.

A *cost model* for a language assigns a natural number to each closed term in the language in
such a way that the term should take **at most** this number of "abstract steps" to evaluate,
this natural number is called *the cost* of that term (relative to that cost model).

The abstract steps could be, for example, beta-reductions, so if the cost of a term :fsharp:`M`
is :fsharp:`n` it would take at most :fsharp:`n` beta-reductions to reduce :fsharp:`M` to a
normal form with the chosen evaluation strategy of the language.

The cost is defined **recursively** on the structure of the term, and it could depend on
the **values** of some of its subterms, so we can think of the cost as a term in the language
which **evalutes** to a natural number.

We use the notation :math:`c\left(M\right)` for the cost of the term :fsharp:`M`.

.. list-table::
   :header-rows: 1

   * -
     - Term
     - Cost
   * - Function Abstraction
     - :math:`c\left(\color{black}{ \lambda x.M }\right)`
     - :math:`0`
   * - Function Application
     - :math:`c\left(\color{black}{ M\:N }\right)`
     - :math:`1 + c\left(M\right) + c\left(N\right)`
   * - Let Expression
     - :math:`c\left(\begin{array}{cc}\textbf{let} & L_{k}\\\textbf{in} & M\end{array}\right)`
     - :math:`c\left(M\right) + \sum_{k=1}^{n}c\left(L_{k}\right)`
   * - Nullary Let Clause
     - :math:`c\left(x = N\right)`
     - :math:`c\left(N\right)`
   * - Applied Let Clause
     - :math:`c\left(f\:x_{1}\ldots x_{n}=N\right)`
     - :math:`0`
   * - Conditional
     - :math:`c\left( \textbf{if}\:M\:\textbf{then}\:N_{1}\:\textbf{else}\:N_{2} \right)`
     - :math:`3+c\left(M\right)+\textbf{if}\:M\:\textbf{then}\:c\left(N_{1}\right)\:\textbf{else}\:c\left(N_{2}\right)`
   * - Pattern Matching
     - :math:`c\left( \begin{array}{c}\textbf{match}\:M\:\textbf{with}\\|\:P_{k}\rightarrow N_{k}\end{array} \right)`
     - :math:`c\left(M\right)+\sum_{k=1}^{n}c\left(P_{k}\right)+\begin{array}{c}\textbf{match}\:M\:\textbf{with}\\|\:P_{k}\rightarrow c\left(N_{k}\right)\end{array}`
   * - Record Definition
     - :math:`c\left(\color{black}{ \left\{ F_{k}=M_{k};\right\} }\right)`
     - :math:`n+\sum_{k=1}^{n}c\left(\color{black}{M_{k}}\right)`
   * - Record Projection
     - :math:`c\left(\color{black}{ M.F }\right)`
     - :math:`1 + c\left(\color{black}{M}\right)`

Notice how the cost of both **function abstraction** and **applied let clause** is **0** -
that's because function abstraction is in a **weak head normal form** - it's not evaluated further until
it is applied on an argument, so as far as the execution environment is concerned it is **already a value**,
so it won't be evaluated further;
The cost of applied let clause is **0** as well, since an applied let expression if equivalent to a beta redex
where the argument is itself a lambda (:math:`\textbf{let} \: f \: x = N \: \textbf{in} \: M` is equivalent to
:math:`\left(\lambda f.M\right) \: \left(\lambda x.N\right)`), so it is considered the same;
In both cases the cost of the body of the function is **internalized**, through the use of :fsharp:`inc`
(which would be explained later).

In practice the actual cost (relative to the cost model) is **not computed directly**,
instead - the cost is decomposed into 2 components:

  1. **Compositional Cost** - which is **declared** by the **developer** and
     **verified** by the **type system**
  2. **Syntactic Cost** - which is **computed** by the **elaborator**

:math:`\textbf{total cost}=\textbf{compositional cost}+\textbf{syntactic cost}`

In fact - given general recursion it is **impossible** to automatically compute the cost
of any arbitrary term in the language.

Another deviation from the cost model is that the cost we give in practice is
an **upper bound** on the cost given by the cost model, because it is easier to compute,
but since the cost model itself is an upper bound on the number of steps the approximation
we use in practice still gives a valid upper bound.

The **syntactic cost** is given to terms **by virtue of their form**, and acts as a **seed** which
the **compositional cost** later **propogates** into the program through the **type system**.


The Cost Type
-------------

Given a cost model we define a special type constructor :fsharp:`cost` to indicate that
a term of type :fsharp:`cost A m`, where :fsharp:`A` is a type and :fsharp:`m` is a natural number,
would take at most :fsharp:`m` steps to produce a **value** of type :fsharp:`A`.

Dually - we use the type :fsharp:`cost A m` to indicate that a term of this type **represents
a value** of type :fsharp:`A` which **has taken** at most :fsharp:`m` steps to evaluate.

When a **term** :fsharp:`M` has the type :fsharp:`cost A m` we say that :fsharp:`M` is a
*costed term*, with a *cost* of :fsharp:`m`.

When a **function** :fsharp:`f` has the type :fsharp:`A -> cost B m` we say that
:fsharp:`f` is a *costed function*, with a *cost* of :fsharp:`m`, which means that when
given an input of type :fsharp:`A` it takes :fsharp:`f` at most :fsharp:`m` steps to produce
an output of type :fsharp:`B`.


Compositional Cost
------------------

The :fsharp:`cost` type constructor behaves as a an **indexed monad**, indexed over the
additive monoid of the natural numbers, so whenever you compose (using Kleisli composition)
2 costed functions :fsharp:`f : B -> cost C n` and :fsharp:`g : A -> cost B m`
you get a function of type :fsharp:`A -> cost C (m + n)` where the cost is the sum of the costs
of :fsharp:`f` and :fsharp:`g`.

The **bind** operator (:fsharp:`>>=`) has the type signature:

.. code-block:: fsharp

    bind: cost a m -> (a -> cost b n) -> cost b (m+n)

In practice - the :fsharp:`cost` monad in *ZF\** is implemented as **the identity monad**,
where the index could be set arbitrarily, so as far as *ZF\** is concerned - **it is up to
the developer to honestly declare the costs of terms**.

**The validity of the cost of a term is not fully enforced by the compiler!**

The compiler only makes sure that the costs are **composed correctly**.

To enforce the validity of the costs we combine the compiler with an **elaborator**,
which would be explained in detail later on.

To lift a term into the monad we use the :fsharp:`ret` function, which is the unit of the
monad and has the type :fsharp:`ret : a -> cost a 0`.
Since the :fsharp:`ret` function gives a term a cost of :fsharp:`0`, we use the function
:fsharp:`inc : (m:nat) -> cost a n -> cost a (n+m)` to increase the declared cost of a term.

Syntactic Cost
--------------

The cost monad can only ensure that costs are composed correctly, but it cannot
enforce the declared costs to conform to the cost model - it completely trusts
the developer to declare costs honestly.

In order to actually **enforce** the cost model, we use a device called *the elaborator*.

The elaborator scans the syntax trees of the terms and recursively sums up the cost of
each branch, adding additional constant cost with each clause and primitive operation.

Eventually, when the elaborator reaches either a **lambda expression**,
or a **let expression** (which could be a **top-level let**) - it embeds the accumulated cost of the body of the expression
into the body, by replacing it with an application of the :fsharp:`inc` function, along
with the accumulated cost, on the body; this ensures 2 things:

    1. That the term returns an output which is wrapped in the :fsharp:`cost` monad.
    2. That all the syntactic cost is accounted for.

.. list-table::
   :header-rows: 1

   * - Notation
     - Meaning
   * - :math:`\color{red}{s\left(\color{black}{M}\right)}`
     - Syntactic cost of :math:`M`
   * - :math:`\color{blue}{\left[ \color{black}{M}\right]}`
     - Modified (elaborated) term :math:`M`
   * - :math:`\underline{\color{red}{n}}`
     - The number :math:`\color{red}{n}` **as a term**

.. list-table::
   :header-rows: 1

   * -
     - Term
     - Cost
   * - Function Abstraction
     - :math:`\color{red}{s\left(\color{black}{ \lambda x.M }\right)}`
     - :math:`\color{red}{0}`
   * - Let Expression
     - :math:`\color{red}{s\left(\color{black}{\begin{array}{cc}\textbf{let} & L_{k}\\\textbf{in} & M\end{array}}\right)}`
     - :math:`\color{red}{s\left(\color{black}{M}\right)} + \color{red}{\sum_{k=1}^{n}s\left(\color{black}{L_{k}}\right)}`
   * - Nullary Let Clause
     - :math:`\color{red}{s\left(\color{black}{x = N}\right)}`
     - :math:`\color{red}{s\left(\color{black}{N}\right)}`
   * - Applied Let Clause
     - :math:`\color{red}{s\left(\color{black}{f\:x_{1}\ldots x_{n}=N}\right)}`
     - :math:`\color{red}{0}`
   * - Function Application
     - :math:`\color{red}{s\left(\color{black}{ M\:N }\right)}`
     - :math:`\color{red}{1 + s\left(\color{black}{M}\right) + s\left(\color{black}{N}\right)}`
   * - Conditional
     - :math:`\color{red}{s\left(\color{black}{ \textbf{if}\:M\:\textbf{then}\:N_{1}\:\textbf{else}\:N_{2} }\right)}`
     - :math:`\color{red}{3 + s\left(\color{black}{M}\right) + \max\left(s\left(\color{black}{N_{1}}\right),s\left(\color{black}{N_{2}}\right)\right)}`
   * - Pattern Matching
     - :math:`\color{red}{s\left(\color{black}{ \begin{array}{c}\textbf{match}\:M\:\textbf{with}\\|\:P_{k}\rightarrow N_{k}\end{array} }\right)}`
     - :math:`\color{red}{s\left(\color{black}{M}\right)+\sum_{k=1}^{n}s\left(\color{black}{P_{k}}\right)+\max\left\{ s\left(\color{black}{N_{k}}\right)\right\} _{k=1}^{n}}`
   * - Record Definition
     - :math:`\color{red}{s\left(\color{black}{ \left\{ F_{k}=M_{k};\right\} }\right)}`
     - :math:`\color{red}{n+\sum_{k=1}^{n}s\left(\color{black}{M_{k}}\right)}`
   * - Record Projection
     - :math:`\color{red}{s\left(\color{black}{ M.F }\right)}`
     - :math:`\color{red}{1 + s\left(\color{black}{M}\right)}`

.. list-table::
   :header-rows: 1

   * -
     - Term
     - Elaborated Term
   * - Function Abstraction
     - :math:`\color{blue}{\left[\color{black}{ \lambda x.M }\right]}`
     - :math:`\lambda x . \textbf{inc}\:\underline{\color{red}{s\left(\color{black}{M}\right)}}\:\text{(}\color{blue}{\left[ \color{black}{M}\right]}\text{)}`
   * - Function Application
     - :math:`\color{blue}{\left[\color{black}{ M\:N }\right]}`
     - :math:`\color{blue}{\left[ \color{black}{M}\right]}\:\color{blue}{\left[ \color{black}{N}\right]}`
   * - Let Expression
     - :math:`\color{blue}{\left[\color{black}{\begin{array}{cc}\textbf{let} & L_{k}\\\textbf{in} & M\end{array}}\right]}`
     - :math:`\begin{array}{cc}\textbf{let} & \color{blue}{\left[\color{black}{L_{k}}\right]}\\\textbf{in} & \color{blue}{\left[\color{black}{M}\right]}\end{array}`
   * - Nullary Let Clause
     - :math:`\color{blue}{\left[\color{black}{x = N}\right]}`
     - :math:`x = \color{blue}{\left[\color{black}{N}\right]}`
   * - Applied Let Clause
     - :math:`\color{blue}{\left[\color{black}{f\:x_{1}\ldots x_{n}=N}\right]}`
     - :math:`f\:x_{1}\ldots x_{n}=\textbf{inc}\:\underline{\color{red}{s\left(\color{black}{N}\right)}}\:\text{(}\color{blue}{\left[\color{black}{N}\right]}\:\text{)}`
   * - Conditional
     - :math:`\color{blue}{\left[\color{black}{ \textbf{if}\:M\:\textbf{then}\:N_{1}\:\textbf{else}\:N_{2} }\right]}`
     - :math:`\textbf{if}\:\color{blue}{\left[ \color{black}{M}\right]}\:\textbf{then}\:\color{blue}{\left[ \color{black}{N_{1}}\right]}\:\textbf{else}\:\color{blue}{\left[ \color{black}{N_{2}}\right]}`
   * - Pattern Matching
     - :math:`\color{blue}{\left[\color{black}{ \begin{array}{c}\textbf{match}\:M\:\textbf{with}\\|\:P_{k}\rightarrow N_{k}\end{array} }\right]}`
     - :math:`\begin{array}{c}\textbf{match}\:\color{blue}{\left[\color{black}{M}\right]}\:\textbf{with}\\|\:P_{k}\rightarrow \color{blue}{\left[\color{black}{N_{k}}\right]}\end{array}`
   * - Record Definition
     - :math:`\color{blue}{\left[\color{black}{ \left\{ F_{k}=M_{k};\right\} }\right]}`
     - :math:`\left\{ F_{k}=\color{blue}{\left[ \color{black}{M_{k}}\right]};\right\}`
   * - Record Projection
     - :math:`\color{blue}{\left[\color{black}{ M.F }\right]}`
     - :math:`\color{blue}{\left[ \color{black}{M}\right]}.F`

-----

Proceed to the examples_

.. _examples: examples.html
