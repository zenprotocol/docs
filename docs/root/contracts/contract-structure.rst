==================
Contract Structure
==================

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

Each contract must contain 2 functions: :fsharp:`main` and :fsharp:`cf`.

The :fsharp:`main` function is run whenever a contract is used within a transaction.

The structure of the :fsharp:`main` function is:

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

where :fsharp:`n` must be an expression which evaluates to a natural number (:fsharp:`nat`).

Parameters
----------

* :fsharp:`txSkel : Zen.Types.txSkeleton`
    The transaction which used the contract.

* :fsharp:`context : Zen.Types.context`
    **TODO: Details about the function.**

* :fsharp:`contractId : Zen.Types.contractId`
    **TODO: Details about the function.**

* :fsharp:`command : string`
    String that the contract may use.
    Contains a command which tells the contract what to do.
    For example:
      .. code-block:: fsharp

        match command with
        | "redeem" -> redeem txSkeleton contractId returnAddress wallet
        | "buy"    -> buy txSkeleton contractId returnAddress


* :fsharp:`sender : Zen.Types.sender`
    **TODO: Details about the function.**

* :fsharp:`messageBody : option Zen.Types.data`
    The transaction may carry a message which can be any of the following things:

    .. list-table::
       :header-rows: 0

       * -
         -
       * - :fsharp:`Byte of FStar.UInt8.t`
         - 8 bit unsigned integer
       * - :fsharp:`U32 of FStar.UInt32.t`
         - 32 bit unsigned integer
       * - :fsharp:`U64 of FStar.UInt64.t`
         - 64 bit unsigned integer
       * - :fsharp:`I64 of FStar.Int64.t`
         - 64 bit signed integer
       * - :fsharp:`ByteArray: Zen.Array.t FStar.UInt8.t -> data`
         - Byte array
       * - :fsharp:`String of string`
         - String
       * - :fsharp:`Hash of hash`
         - 256-bit hash value
       * - :fsharp:`Lock of lock`
         - Lock
       * - :fsharp:`Signature of signature`
         - Signature
       * - :fsharp:`PublicKey of publicKey`
         - Public key
       * - :fsharp:`Collection of dataCollection`
         - Data collection


* :fsharp:`wallet : Zen.Types.wallet`
    **TODO: Details about the function.**

* :fsharp:`state : option Zen.Types.data`
    **TODO: Details about the function.**

Output
------
The output of the contract is a new transaction.
