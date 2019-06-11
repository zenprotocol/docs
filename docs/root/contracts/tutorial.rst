Tutorial
========

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

.. role:: bash(code)
    :language: bash

Named Token
-----------

First we'll learn how to write a very simple contract,
called the "Named Token" contract.
The contract simply mints a token with an identifier specified by the issuer,
and locks the specified amount of the token to the specified return address.

Let's specify the contract.

First - the issuer executes the contract, giving it the following data:

.. list-table::
   :header-rows: 0

   * - 1. **name**
     - :fsharp:`: Zen.Types.lock`
     - The name of the issued token
   * - 2. **amount**
     - :fsharp:`: uint64`
     - The issed amount of the token
   * - 3. **returnAddress**
     - :fsharp:`: string`
     - The recipient of the token

Then - the contract mints the specified amount of the token using the given name as a subidentifier,
and locks it to the specified return address.

The whole process looks like this:

.. math::

    \begin{array}{ccc}
    \text{Issuer} & \xrightarrow[\left\langle \text{name},\:\text{amount},\:\text{returnAddress}\right\rangle ]{} & \text{Contract}\\
    \\
    \text{Contract} & \xrightarrow{\left[\text{name}\right]\times\text{amount}} & \text{returnAddress}
    \end{array}

Let's write the contract.

Create a text file called *"NamedToken.fst"*, the *"fst"* suffix is the standard suffix for **F\*** files.

At the top of the file put the module name - it should be identical to the file name (excluding the suffix):

.. code-block:: fsharp

    module NamedToken

We should also load (using the :fsharp:`open` directive) a couple of useful modules (:fsharp:`Zen.Base`, :fsharp:`Zen.Cost`, and :fsharp:`Zen.Data`)
into the namespace, which we'll use later on.

The file should now look like this:

.. code-block:: fsharp

    module NamedToken

    open Zen.Base
    open Zen.Cost
    open Zen.Data

Each contract should have at least 2 top-level functions: :fsharp:`main`, and :fsharp:`cf`.

The :fsharp:`main` function is the function which runs with each execution of the contract,
and :fsharp:`cf` function is the function which describes the cost of the :fsharp:`main` function.

Let's write the :fsharp:`main` function, it should always have the following type signature:

.. code-block:: fsharp

    main
        ( txSkel      : Zen.Types.txSkeleton  )
        ( context     : Zen.Types.context     )
        ( contractId  : Zen.Types.contractId  )
        ( command     : string                )
        ( sender      : Zen.Types.sender      )
        ( messageBody : option Zen.Types.data )
        ( wallet      : Zen.Types.wallet      )
        ( state       : option Zen.Types.data )
        : Zen.Types.contractResult `Zen.Cost.t` n

where :fsharp:`n` is the cost of the function and equal to :fsharp:`cf txSkel context contractId command sender messageBody wallet state`.

In practice we usually don't actually have to specify the times of the parameters, as they would be inferred by the compiler.

It should look like this:

.. code-block:: fsharp

    let main txSkel context contractId command sender messageBody wallet state =

We haven't supplied the body of the function yet, which should go below that line.

The first thing we need to do is to parse the data - to extract the name, amount, and return address out of it.

The data should be sent to the contract through the :fsharp:`messageBody` parameter, in the form of a dictionary,
which will contain the specified data as *(key,value)* pairs, where each key corresponds to one of the specified fields
(**"name"**, **"amount"**, and **"returnAddress"**).

Since we assume :fsharp:`messageBody` is a dictionary, we need to try to extract a dictionary out of it -
this is is done with the :fsharp:`tryDict` function, defined in :fsharp:`Zen.Data`.

The :fsharp:`tryDict` function has the following type signature:

.. code-block:: fsharp

    tryDict: data -> option (Dict.t data) `cost` 4

Recall that the :fsharp:`data` type is a discriminated union of the following:

.. code-block:: fsharp

    type data =
        | I64 of I64.t
        | Byte of U8.t
        | ByteArray: A.t U8.t -> data
        | U32 of U32.t
        | U64 of U64.t
        | String of string
        | Hash of hash
        | Lock of lock
        | Signature of signature
        | PublicKey of publicKey
        | Collection of dataCollection

    and dataCollection =
        | Array of A.t data
        | Dict of dictionary data
        | List of list data

So what :fsharp:`tryDict` does, is taking a value of type :fsharp:`data`, and if that value is a :fsharp:`Collection(Dict(d))`
- it returns :fsharp:`Some d`, and otherwise it returns :fsharp:`None`.

Now - since the :fsharp:`messageBody` is already an :fsharp:`option data`, we can't apply :fsharp:`tryDict` on it directly
(since it expects a :fsharp:`data`), so instead we use the :fsharp:`(>!=)` operator from :fsharp:`Zen.Data` which have the following
type signature:

.. code-block:: fsharp

    (>!=) : option a -> (a -> cost (option b) n) -> cost (option b) n

The dictionary extraction should look like this:

.. code-block:: fsharp

    messageBody >!= tryDict

Let's name the result as :fsharp:`dict`, using a :fsharp:`let` expression, so the :fsharp:`main` function should now look like this:

.. code-block:: fsharp

    let main txSkel context contractId command sender messageBody wallet state =

        let dict = messageBody >!= tryDict in

Now :fsharp:`dict` will either contain a :fsharp:`Some d` (where :fsharp:`d` is a dictionary) or :fsharp:`None`.

Now that we have the dictionary, let's extract the required fields out of it, using the :fsharp:`tryFind` function (from :fsharp:`Zen.Dictionary`).

The :fsharp:`tryFind` function has the following type signature:

.. code-block:: fsharp

    tryFind : string -> dictionary a -> option a `cost` 64

It takes a key name as an argument, and a dictionary, and if that dictionary has a value with the specified key name it returns it
(within a :fsharp:`Some`), and otherwise returns :fsharp:`None`.

Since :fsharp:`dict` is an ``option (dictionary data) `cost` 64`` we can't use :fsharp:`tryFind` on it directly,
so we'll use the :fsharp:`(>?=)` operator (defined in :fsharp:`Zen.Data`) instead.

The :fsharp:`(>?=)` operator has the following type signature:

.. code-block:: fsharp

    (>?=) : option a `cost` m -> (a -> option b `cost` n) -> option b `cost` (m+n)

To extract the value of the **"returnAddress"** key, we'll do:

.. code-block:: fsharp

    dict
    >?= Zen.Dictionary.tryFind "returnAddress"

(notice we use the full qualified name here, since we didn't load the :fsharp:`Zen.Dictionary` module into the namespace
with the :fsharp:`open` directive)

This will give us a (costed) :fsharp:`option data` value;
to extract an actual lock out of that value we'll use the :fsharp:`tryLock` function (defined in :fsharp:`Zen.Data`):

.. code-block:: fsharp

    dict
    >?= Zen.Dictionary.tryFind "returnAddress"
    >?= tryLock

Let's give a name to the extracted lock, using a :fsharp:`let!` expression.

.. code-block:: fsharp

    let! returnAddress =
        dict
        >?= Zen.Dictionary.tryFind "returnAddress"
        >?= tryLock
    in

Now the whole :fsharp:`main` function should look like this:

.. code-block:: fsharp

    let main txSkeleton context contractId command sender messageBody wallet state =

        let dict = messageBody >!= tryDict in

        let! returnAddress =
            dict
            >?= Zen.Dictionary.tryFind "returnAddress"
            >?= tryLock
        in

To extract the **"amount"** and **"name"** keys we'll do something similar
(using the :fsharp:`tryU64` and :fsharp:`tryString`, respectively, instead of :fsharp:`tryLock`):

.. code-block:: fsharp

    let main txSkeleton context contractId command sender messageBody wallet state =

        let dict = messageBody >!= tryDict in

        let! returnAddress =
            dict
            >?= Zen.Dictionary.tryFind "returnAddress"
            >?= tryLock
        in

        let! amount =
            dict
            >?= Zen.Dictionary.tryFind "amount"
            >?= tryU64
        in

        let! name =
            dict
            >?= Zen.Dictionary.tryFind "name"
            >?= tryString
        in
