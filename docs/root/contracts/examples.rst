Examples
========

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

Simple Example
--------------

Let's create a simple function which takes a function and an argument and applies the function on the argument:

.. code-block:: fsharp

    val applyOnce (#a #b:Type): (a -> b) -> a -> b
    let applyOnce #_ #_ f x = f x

Elaborating the function will yield the following:

.. code-block:: fsharp

    val applyOnce: #a: Type -> #b: Type -> (a -> b) -> a -> b
    let applyOnce #_ #_ f x = inc 1 (f x)

That's because function application (:fsharp:`f x`) has a cost of 1, which is embedded into the let-expression with :fsharp:`inc`.

However - trying to compile this function will result in a typing error - because you can only apply :fsharp:`inc` on a value of a :fsharp:`cost` type.

To fix it we modify the function to use :fsharp:`ret` on the result, and we change the type signature to return a :fsharp:`cost` type
(with a cost of 0 since that is the cost :fsharp:`ret` gives):

.. code-block:: fsharp

    val applyOnce (#a #b:Type): (a -> b) -> a -> cost b 0
    let applyOnce #_ #_ f x = ret (f x)

Now elaborating the function will yield the following:

.. code-block:: fsharp

    val applyOnce (#a #b:Type): (a -> b) -> a -> cost b 0
    let applyOnce #_ #_ f x = inc 2 (ret (f x))

That's because now we have **2 applications**, one of :fsharp:`f` on :fsharp:`x`, and the other of :fsharp:`ret` on the result.

Now the compilation would **still fail** with a typing error:

.. code-block:: none

    Subtyping check failed; expected type Zen.Cost.Realized.cost _ 0; got type Zen.Cost.Realized.cost _ (0 + 2)

That's because while the declared cost is 0, the **inferred** cost (due to the addition of :fsharp:`inc 2`) is 2.

To fix that we now have to declare the **correct** cost within the type, to account for the increment:

.. code-block:: fsharp

    val applyOnce (#a #b:Type): (a -> b) -> a -> cost b 2
    let applyOnce #_ #_ f x = inc 2 (ret (f x))

Now that the syntactic cost is accounted for the program will compile.


Aggregated Syntactic Cost
-------------------------

The following function takes a function :fsharp:`f` and an argument :fsharp:`x` and applies :fsharp:`f` on :fsharp:`x` and then on the result:

.. code-block:: fsharp

    val applyTwice (#a:Type): (a -> a) -> a -> a
    let applyTwice #_ f x = f (f x)

Elaborating this function will yield the following:

.. code-block:: fsharp

    val applyTwice (#a:Type): (a -> a) -> a -> a
    let applyTwice #_ f x = inc 2 (f (f x))

The increment by 2 is due to the fact there are 2 applications.

Again - this format won't do, and to be able to compile the elaborated program we have to lift the result into the cost monad and account for the
additional costs:

.. code-block:: fsharp

    val applyTwice (#a:Type): (a -> a) -> a -> cost a 3
    let applyTwice #_ f x = ret (f (f x))

which will be elaborated as:

.. code-block:: fsharp

    val applyTwice: #a: Type -> (a -> a) -> a -> cost a 3
    let applyTwice #_ f x = inc 3 (ret (f (f x)))

Notice that this time we've participated the increased elaborated cost in advance, so the original code won't compile (since :fsharp:`ret` gives
a cost of 0) while the elaborated code will (since it will add :fsharp:`inc 3` to account for the costs of all the function applications).

In practice - **you won't be able to modify the elaborated code**, since the elaboration is done automatically when activating a contract,
so instead you need to elaborate it locally, look at the result, and then modify the **original code** so it would compile right after the elaboration.

Multiple Clauses
----------------

Now let's create another function - which takes a boolean :fsharp:`b`, a function :fsharp:`f`, and another argument :fsharp:`x`,
and applies :fsharp:`f` on :fsharp:`x` **once** if :fsharp:`b` is :fsharp:`true` or **twice** if :fsharp:`b` is :fsharp:`false`.

We'll use the previously defined functions to do so:

.. code-block:: fsharp

    val onceOrTwice (#a:Type): bool -> (a -> a) -> a -> cost a ?
    let onceOrTwice #_ b f x = if b then applyOnce f x else applyTwice f x

What should be the declared cost of this function?

:fsharp:`applyOnce` gives a cost of **2**, while :fsharp:`applyTwice` gives a cost of **3**, so we have a **collision of costs**.

To reconcile the collision we manually insert :fsharp:`inc 1` to the :fsharp:`applyOnce f x` clasue to account for the difference between the costs,
which would give both clauses of the :fsharp:`if-then-else` a cost of **3**:

.. code-block:: fsharp

    val onceOrTwice (#a:Type): bool -> (a -> a) -> a -> cost a 3
    let onceOrTwice #_ b f x = if b then inc 1 (applyOnce f x) else applyTwice f x

However - we still need to account for the syntactic cost of the :fsharp:`onceOrTwice` function itself - to do so we first elaborate the function,
which gives us:

.. code-block:: fsharp

    val onceOrTwice (#a:Type): bool -> (a -> a) -> a -> cost a 3
    let onceOrTwice #_ b f x = inc 7 (if b then inc 1 (applyOnce f x) else applyTwice f x)

That is beacuse the :fsharp:`then` clause has **4 applications**, while the :fsharp:`else` clause has **3**, and since the syntactic cost of
an :fsharp:`if-then-else` is defined as **3** + the maximal syntactic cost out of both clauses, which is **4** in this case, we get a total syntactic cost
of **7**, which is embedded with an :fsharp:`inc 7` into the let-expression.

Now to account for the additional cost of **7** we change the declared cost of the function to **10**:

.. code-block:: fsharp

    val onceOrTwice (#a:Type): bool -> (a -> a) -> a -> cost a 10
    let onceOrTwice #_ b f x = inc 7 (if b then inc 1 (applyOnce f x) else applyTwice f x)

The elaborated code will now compile.

Recursion
---------

The cost analysis becomes more complex once you introduce recursion.

The following function computes the factorial of a natural number:

.. code-block:: fsharp

    val fact: nat -> int
    let rec fact m =
        match m with
        | 0 -> 1
        | _ -> m * fact (m - 1)

Trying to elaborate this function will fail, since it doesn't return a :fsharp:`cost` type.

.. code-block:: fsharp

    val fact: nat -> int
    let rec fact m =
      inc 7
          (match m with
            | 0 -> 1
            | _ -> m * fact (m - 1))

To make sure the result is of a :fsharp:`cost` type - we apply :fsharp:`ret` on the base case, and bind the recursion step:

.. code-block:: fsharp

    let rec fact m =
        match m with
        | 0 -> ret 1
        | _ -> fact (m - 1) >>= (fun r -> ret (m * r))

We might prefer to use some syntactic sugar to make the code look nicer:

.. code-block:: fsharp

    let rec fact m =
        match m with
        | 0 -> ret 1
        | _ -> let! r = fact (m - 1)
               in ret (m * r)

What should be the return type of the new function? we can start with :fsharp:`cost int 0`:

.. code-block:: fsharp

    val fact: nat -> cost int 0
    let rec fact m =
        match m with
        | 0 -> ret 1
        | _ -> let! r = fact (m - 1)
               in ret (m * r)

Now the elaborated code will not compile, but it will give us some hints on how to specify the cost correctly:

.. code-block:: fsharp

    val fact: nat -> cost int 0
    let rec fact m =
      inc 10
          (match m with
            | 0 -> ret 1
            | _ -> let! r = fact (m - 1)
                   in ret (m * r))

We might be tempted to assign a cost of **10** to the function, but that won't do -
if the cost of :fsharp:`fact` is **10** then :fsharp:`fact (m - 1)` will have a cost of **10** and after accounting for the incremented syntactic cost
(which is added by :fsharp:`inc 10`) we'll get that the **actual** cost of the function is **20**.

In fact - assuming **any** constant cost will yield a **contradiction**, since we'll always get another 10 steps added to what we've started with,
which indicates that it is impossible to assign a constant cost to the function.

How do we resolve it? by making the cost **parametric** in the size of the input, by using **dependent types**.

First we explicitly declare the **name** of the input parameter **in the type**:

.. code-block:: fsharp

    val fact: (m:nat) -> cost int 0

Then we can use this name within the cost expression, for example like this:

.. code-block:: fsharp

    val fact: (m:nat) -> cost int m

But this won't do - we have to figure out the actual dependency of the cost on the parameter.

Let's assume for now the cost is some function :fsharp:`fact_c` on :fsharp:`m`:

.. code-block:: fsharp

    val fact: (m:nat) -> cost int (fact_c m)

Looking at the definition of :fsharp:`cost` again - we immediately see a problem - the cost of :fsharp:`ret 1` is always **0**,
while the cost of :fsharp:`fact (m-1)` is :fsharp:`fact_c (m-1)`, which **can't be 0**, so we have 2 pattern matching clauses with different costs.

This will result in a typing error when we try to compile the code, since all clauses of a :fsharp:`match-with` must return a result of the same type.

To fix that we need to add the difference to the first clause, like this (notice this is a modification of the original code, **not**
the elaborated code):

.. code-block:: fsharp

    val fact: (m:nat) -> cost int (fact_c m)
    let rec fact m =
        match m with
        | 0 -> inc (fact_c (m-1)) (ret 1)
        | _ -> let! r = fact (m-1)
               in ret (m * r)

Instead of using :fsharp:`ret` and then :fsharp:`inc` we can just use :fsharp:`incRet`, which combines the 2:

.. code-block:: fsharp

    val fact: (m:nat) -> cost int (fact_c m)
    let rec fact m =
        match m with
        | 0 -> incRet (fact_c (m-1)) 1
        | _ -> let! r = fact (m-1)
               in ret (m * r)

Now we have to figure out what :fsharp:`fact_c` is.

Since the elaborator adds **10** to the cost, and this value didn't change with the addition of the :fsharp:`incRet` to the first clause
(since the total syntactic cost for a :fsharp:`match-with` is determined only by the syntactically heaviest clause, which is still the second clause),
we get **10** added to the cost **with each call to the function**, which means that the **base case** has **at least** a cost of **10**,
and with each **recursion step** another **10** is added, so we get the following recurrence relation:

.. code-block:: fsharp

    fact_c m = 10 + fact_c (m - 1)

What is the base case (for :fsharp:`m = 0`)?

Since we know it is **at least 10** (since the cost is increased by 10 and it can never be decreased), we can try assigning it to be 10,
which gives us the solution:

.. code-block:: fsharp

    fact_c m = 10 * (m + 1)

So let's try it:

.. code-block:: fsharp

    val fact: (m:nat) -> cost int (10 * (m+1))
    let rec fact m =
        match m with
        | 0 -> incRet (10 * m) 1
        | _ -> let! r = fact (m-1)
               in ret (m * r)

Now the function is elaborated to:

.. code-block:: fsharp

    val fact: m: nat -> cost int (10 * (m + 1))
    let rec fact m =
        inc 10
          (match m with
            | 0 -> incRet (10 * m) 1
            | _ -> let! r = fact (m - 1) in ret (m * r))

The program now compiles successfully, which indicates that the cost is now correct.

The keen-sighted reader will notice that the first clause :fsharp:`incRet (10 * m) 1` is operationally equivalent to :fsharp:`ret 1`, since the only
case in which this clause is executed is when :fsharp:`m = 0`, but since all clauses must return a result of the same type, no matter if they are
executed or not, the only way to pass the type checker is by explicitly handling the cost for all possible values of :fsharp:`m`.
