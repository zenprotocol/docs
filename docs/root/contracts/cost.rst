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

The :fsharp:`cost` Type
-----------------------

Given a cost model we can define a special type constructor :fsharp:`cost` to indicate that
a term of type :fsharp:`cost m A`, where :fsharp:`m` is a natural number and :fsharp:`A` is a
type, would take at most :fsharp:`m` steps to produce a **value** of type :fsharp:`A`.

Dually - we use the type :fsharp:`cost m A` to indicate that a term of this type **represents
a value** of type :fsharp:`A` which **has taken** at most :fsharp:`m` steps to evaluate.

When a **term** :fsharp:`M` has the type :fsharp:`cost m A` we say that :fsharp:`M` is a
*costed term*, with a *cost* of :fsharp:`m`.

When a **function** :fsharp:`f` has the type :fsharp:`A -> cost m B` we say that
:fsharp:`f` is a *costed function*, with a *cost* of :fsharp:`m`, which means that when
given an input of type :fsharp:`A` it takes :fsharp:`f` at most :fsharp:`m` steps to produce
an output of type :fsharp:`B`.

The :fsharp:`cost` type constructor should behave as a an **indexed monad**, indexed over the
additive monoid of the natural numbers, so whenever you compose (using Kleisli composition)
2 costed functions :fsharp:`f : B -> cost n C` and :fsharp:`g : A -> cost m B`
you get a function of type :fsharp:`A -> cost (m + n) C` where the cost is the sum of the costs
of :fsharp:`f` and :fsharp:`g`.

In practice - the :fsharp:`cost` monad in *ZF\** is implemented as **the identity monad**,
where the index could be set arbitrarily, so as far as *ZF\** is concerned - **it is up to
the developer to honestly declare the costs of terms**.

**The validity of the cost of a term is not fully enforced by the compiler!**

The compiler only makes sure that the costs are **composed correctly**.

To enforce the validity of the costs we combine the compiler with an **elaborator**,
which would be explained in detail later on.

To lift a term into the monad we use the :fsharp:`ret` function, which is the unit of the
monad and has the type :fsharp:`ret : 'a -> cost 0 'a`.
Since the :fsharp:`ret` function gives a term a cost of :fsharp:`0`, we use the function
:fsharp:`inc : (m:nat) -> cost n 'a -> cost (n+m) 'a` to increase the declared cost of a term.

Syntactic Cost
--------------

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

Elaboration
-----------

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
