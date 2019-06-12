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
     - :fsharp:`: string`
     - The name of the issued token
   * - 2. **amount**
     - :fsharp:`: uint64`
     - The issed amount of the token
   * - 3. **returnAddress**
     - :fsharp:`: Zen.Types.lock`
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

where :fsharp:`n` is the cost of the function and equal to :fsharp:`cf txSkel context command sender messageBody wallet state`
(notice that :fsharp:`cf` doesn't take the :fsharp:`contractId` as an argument, since the cost shouldn't depend on it).

In practice we usually don't actually have to specify the types of the parameters, as they would be inferred by the compiler.

It should look like this:

.. code-block:: fsharp

    let main txSkel context contractId command sender messageBody wallet state =
        ...

We haven't supplied the body of the function yet, which should go below that line (instead of the ellipsis).

The first thing we need to do is to parse the data - to extract the name, amount, and return address out of it.

The data should be sent to the contract through the :fsharp:`messageBody` parameter, in the form of a dictionary,
which will contain the specified data as *(key, value)* pairs, where each key corresponds to one of the specified fields
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

        ...

:fsharp:`dict` will either contain a :fsharp:`Some d` (where :fsharp:`d` is a dictionary) or :fsharp:`None`.

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

The :fsharp:`let!` usage strips the cost out of the declared variable (using the cost monad), so it would be easier to work with -
the type of :fsharp:`returnAddress` will be :fsharp:`option lock`, instead of ``option lock `cost` m``.

Now the whole :fsharp:`main` function should look like this:

.. code-block:: fsharp

    let main txSkel context contractId command sender messageBody wallet state =

        let dict = messageBody >!= tryDict in

        let! returnAddress =
            dict
            >?= Zen.Dictionary.tryFind "returnAddress"
            >?= tryLock
        in

        ...

To extract the **"amount"** and **"name"** keys we'll do something similar
(using :fsharp:`tryU64` and :fsharp:`tryString`, respectively, instead of :fsharp:`tryLock`):

.. code-block:: fsharp

    let main txSkel context contractId command sender messageBody wallet state =

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

        ...

Now that we have all of the data, we can use it assuming everything was provided by the issuer.

To consider both the case where the issuer has provided everything and the case where there is missing information,
we pattern match on the data, like this:

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        ...
    | _ ->
        ...

The 1st case will be executed when all the data was provided, and the 2nd case will be executed if any of the required parameters wasn't provided.

Let's throw an error when some of the parameters are missing.

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        ...
    | _ ->
        Zen.ResultT.autoFailw "parameters are missing"

The function :fsharp:`autoFailw` in :fsharp:`Zen.ResultT` throws an error (within a :fsharp:`ResultT`) and infers the cost automatically.

If all the parameters were provided - we need to check that the provided name of the token is at most 32 characters, because that's the
maximum size an asset subidentifier can have.

If the name is longer than 32 characters - we throw an error:

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        if FStar.String.length name <= 32 then
            ...
        else
            Zen.ResultT.autoFailw "name is too long"
    | _ ->
        Zen.ResultT.autoFailw "parameters are missing"

In Zen Protocol assets are defined by 2 parts:

    1. Main Identifier - The contract ID of the contract which have minted the asset.
    2. Subidentifier - The unique ID of the asset, given by 32 bytes.

If the name is short enough to fit as an asset subidentifier - we can define a token with the given name as the subidentifier and the
contract ID of this contract as the main identifier (using the :fsharp:`fromSubtypeString` function from :fsharp:`Zen.Asset`):

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        if FStar.String.length name <= 32 then
            begin
              let! token = Zen.Asset.fromSubtypeString contractId name in
              ...
            end
        else
            Zen.ResultT.autoFailw "name is too long"
    | _ ->
        Zen.ResultT.autoFailw "parameters are missing"

(Notice that we're using :fsharp:`begin` and :fsharp:`end` here instead of parentheses, to make the code cleaner)

Now that we have defined the named token - we **mint** the specified amount of it,
and then **lock** the minted tokens to the specified return address -
this is done by modifying the supplied transaction (:fsharp:`txSkel`) with :fsharp:`mint`,
and then modifying the result with :fsharp:`lockToAddress` (both are defined in :fsharp:`Zen.TxSkeleton`):

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        if FStar.String.length name <= 32 then
            begin
              let! token = Zen.Asset.fromSubtypeString contractId name in

              let! txSkel =
                Zen.TxSkeleton.mint amount token txSkel
                >>= Zen.TxSkeleton.lockToAddress token amount returnAddress in

              ...
            end
        else
            Zen.ResultT.autoFailw "name is too long"
    | _ ->
        Zen.ResultT.autoFailw "parameters are missing"

Notice the syntax we're using here - both :fsharp:`mint` and :fsharp:`lockToAddress` return a costed :fsharp:`txSkeleton`,
so to chain them we're using the :fsharp:`(>>=)` operator (bind) of the cost monad, and then we name the result using a
:fsharp:`let!` so we can use it as a "pure" :fsharp:`txSkeleton` (instead of a **costed** :fsharp:`txSkeleton`).

Now that we've prepared the transaction - all that is left is to return it (using :fsharp:`ofTxSkel` from :fsharp:`Zen.ContractResult`),
and the contract is done:

.. code-block:: fsharp

    match returnAddress,amount,name with
    | Some returnAddress, Some amount, Some name ->
        if FStar.String.length name <= 32 then
            begin
              let! token = Zen.Asset.fromSubtypeString contractId name in

              let! txSkel =
                Zen.TxSkeleton.mint amount token txSkel
                >>= Zen.TxSkeleton.lockToAddress token amount returnAddress in

              Zen.ContractResult.ofTxSkel txSkel
            end
        else
            Zen.ResultT.autoFailw "name is too long"
    | _ ->
        Zen.ResultT.autoFailw "parameters are missing"

The whole file should now look like this:

.. code-block:: fsharp

    module NamedToken

    open Zen.Base
    open Zen.Cost
    open Zen.Data

    let main txSkel context contractId command sender messageBody wallet state =

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

        match returnAddress,amount,name with
        | Some returnAddress, Some amount, Some name ->
            if FStar.String.length name <= 32 then
                begin
                  let! token = Zen.Asset.fromSubtypeString contractId name in

                  let! txSkel =
                    Zen.TxSkeleton.mint amount token txSkel
                    >>= Zen.TxSkeleton.lockToAddress token amount returnAddress in

                  Zen.ContractResult.ofTxSkel txSkel
                end
            else
                Zen.ResultT.autoFailw "name is too long"
        | _ ->
            Zen.ResultT.autoFailw "parameters are missing"

Now we can verify the validity of this file with:

.. code-block:: bash

    zebra -v NamedToken.fst

It should verify successfully, returning:

.. code-block:: bash

    > zebra -v NamedToken.fst
    SDK:	Verified

But hold on - **we aren't done yet!**

We have finished with the :fsharp:`main` function, but we still need to define the :fsharp:`cf` function.

The type signature of :fsharp:`cf` is:

.. code-block:: fsharp

    cf
        ( txSkel      : Zen.Types.txSkeleton  )
        ( context     : Zen.Types.context     )
        ( command     : string                )
        ( sender      : Zen.Types.sender      )
        ( messageBody : option Zen.Types.data )
        ( wallet      : Zen.Types.wallet      )
        ( state       : option Zen.Types.data )
        : nat `cost` n

So we should add the :fsharp:`cf` function to the end of the file, like this:

.. code-block:: fsharp

    let cf txSkel context command sender messageBody wallet state =

To start - let's give assign it to :fsharp:`0` and then lift it into the cost monad with :fsharp:`Zen.Cost.ret`:

.. code-block:: fsharp

    let cf txSkel context command sender messageBody wallet state =
        0
        |> Zen.Cost.ret

Let's try to **elaborate** the contract, to make sure the cost is correct.

.. code-block:: bash

    zebra -e NamedToken.fst

You should get the following error:

.. code-block:: bash

    (Error 19) Subtyping check failed; expected type
    _: Zen.Types.Realized.txSkeleton ->
    context: Zen.Types.Main.context ->
    command: Prims.string ->
    _: Zen.Types.Main.sender ->
    messageBody: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    _: Zen.Types.Realized.wallet ->
    state: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    Prims.Tot (Zen.Cost.Realized.cost Prims.nat (0 + 2)); got type
    txSkel: Zen.Types.Realized.txSkeleton ->
    context: Zen.Types.Main.context ->
    command: Prims.string ->
    sender: Zen.Types.Main.sender ->
    messageBody: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    wallet: Zen.Types.Realized.wallet ->
    state: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    Prims.Tot (Zen.Cost.Realized.cost Prims.int (0 + 2))

Notice how it infers that :fsharp:`cf` returns an :fsharp:`int`, while it should return a :fsharp:`nat`.

To solve it we need to **cast** the value of :fsharp:`cf` into a :fsharp:`nat`, using the :fsharp:`cast` function:

.. code-block:: fsharp

    let cf txSkel context command sender messageBody wallet state =
        0
        |> cast nat
        |> Zen.Cost.ret

Let's elaborate it again, now we get the following error:

.. code-block:: bash

    (Error 19) Subtyping check failed; expected type
    txSkel: Zen.Types.Realized.txSkeleton ->
    context: Zen.Types.Main.context ->
    _: Zen.Types.Extracted.contractId ->
    command: Prims.string ->
    sender: Zen.Types.Main.sender ->
    messageBody: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    wallet: Zen.Types.Realized.wallet ->
    state: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    Prims.Tot
    (Zen.Cost.Realized.cost Zen.Types.Main.contractResult
      (Zen.Cost.Realized.force (CostFunc?.f (Zen.Types.Main.CostFunc NamedToken.cf)
              txSkel
              context
              command
              sender
              messageBody
              wallet
              state))); got type
    txSkel: Zen.Types.Realized.txSkeleton ->
    context: Zen.Types.Main.context ->
    contractId: Zen.Types.Extracted.contractId ->
    command: Prims.string ->
    sender: Zen.Types.Main.sender ->
    messageBody: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    wallet: Zen.Types.Realized.wallet ->
    state: FStar.Pervasives.Native.option Zen.Types.Data.data ->
    Prims.Tot
    (Zen.Cost.Realized.cost Zen.Types.Main.contractResult
      (4 + 64 + 2 + (4 + 64 + 2 + (4 + 64 + 2 + (64 + (64 + 64 + 3)))) + 54))

Look at the number at the bottom - this is the cost that was **inferred** by the compiler,
so let's try to paste it into the function:

.. code-block:: fsharp

    let cf txSkel context command sender messageBody wallet state =
        (4 + 64 + 2 + (4 + 64 + 2 + (4 + 64 + 2 + (64 + (64 + 64 + 3)))) + 54)
        |> cast nat
        |> Zen.Cost.ret

Let's try to elaborate again:

.. code-block:: bash

    > zebra -e NamedToken.fst
    SDK:	Elaborating NamedToken.fst ...
    SDK:	Wrote elaborated source to NamedToken.fst
    SDK:	Verified

**Congratulations!**

You have written, elaborated, and verified your very first contract.

This time we were lucky - we didn't have to explicitly type our terms and the code was simple enough for the compiler to infer its cost.

With more complex contracts it might not be so easy - in many cases you'll have to explicitly type your terms to convince the compiler
that the cost of the contract is what you claim it is.
