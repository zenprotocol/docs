==================
Contract Structure
==================

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

Each contract must contain 2 functions: :fsharp:`main` and :fsharp:`cf`.

The :fsharp:`main` function is run whenever a contract is used within a transaction.

The function will run both at the validation and at the generation of the transaction.

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
    The partial transaction supplied as input to the contract.

* :fsharp:`context : Zen.Types.context`
    The blockchain context of the transaction, given by :fsharp:`blockNumber` (unsigned 32 bit integer), and :fsharp:`timestamp` - which is the UNIX Epoch time (unsigned 64 bit integer) of the block creation.

* :fsharp:`contractId : Zen.Types.contractId`
    The contract identifier.
    :fsharp:`(version, hash)`

* :fsharp:`command : string`
    String that the contract may use.
    Contains a command which tells the contract what to do.
    For example:
      .. code-block:: fsharp

        match command with
        | "redeem" -> redeem txSkeleton contractId returnAddress wallet
        | "buy"    -> buy txSkeleton contractId returnAddress


* :fsharp:`sender : Zen.Types.sender`
    The sender identity.
    Can be any of the following:

    .. list-table::
       :header-rows: 0

       * - :fsharp:`Contract`
         - :fsharp:`of Zen.Types.contractId`
         - A contract, given by its ID
       * - :fsharp:`PK`
         - :fsharp:`of Zen.Types.publicKey`
         - Public key
       * - :fsharp:`Anonymous`
         -
         - An anonymous sender

* :fsharp:`messageBody : option Zen.Types.data`
    The transaction may carry a message which can be any of the following:

    .. list-table::
       :header-rows: 0

       * - :fsharp:`Byte`
         - :fsharp:`of FStar.UInt8.t`
         - 8 bit unsigned integer
       * - :fsharp:`U32`
         - :fsharp:`of FStar.UInt32.t`
         - 32 bit unsigned integer
       * - :fsharp:`U64`
         - :fsharp:`of FStar.UInt64.t`
         - 64 bit unsigned integer
       * - :fsharp:`I64`
         - :fsharp:`of FStar.Int64.t`
         - 64 bit signed integer
       * - :fsharp:`ByteArray`
         - :fsharp:`of Zen.Array.t FStar.UInt8.t`
         - Byte array
       * - :fsharp:`String`
         - :fsharp:`of string`
         - String
       * - :fsharp:`Hash`
         - :fsharp:`of Zen.Types.hash`
         - 256-bit hash value
       * - :fsharp:`Lock`
         - :fsharp:`of Zen.Types.lock`
         - Lock
       * - :fsharp:`Signature`
         - :fsharp:`of Zen.Types.signature`
         - Signature
       * - :fsharp:`PublicKey`
         - :fsharp:`of Zen.Types.publicKey`
         - Public key
       * - :fsharp:`Collection`
         - :fsharp:`of Zen.Types.dataCollection`
         - Data collection


* :fsharp:`wallet : Zen.Types.wallet`
    Contains all the transaction inputs that were previously locked to the contract.
    In order for a contract to spend its own funds they need to come from contract wallet.

* :fsharp:`state : option Zen.Types.data`
    The contract current state.
    Can be either :fsharp:`Some` data (as the message body) or :fsharp:`None`.

Output
------
The output of the contract is of the record type :fsharp:`Zen.Types.contractReturn` which has 3 fields:


* :fsharp:`state : Zen.Types.stateUpdate`
  State update.
  Can be any of the following:

  .. list-table::
    :header-rows: 0

    * - :fsharp:`Delete`
      -
      - Delete the current state, resetting it to :fsharp:`None`
    * - :fsharp:`NoChange`
      -
      - Keeping the current state as it is, with no change.
    * - :fsharp:`Update`
      - :fsharp:`of Zen.Types.data`
      - Change the state to be the new given :fsharp:`data`.

* :fsharp:`tx : Zen.Types.txSkeleton`
  The generated transaction structure.

* :fsharp:`message : option Zen.Types.message`
  An optional message for invoking another contract.
  This is a record type which has 3 fields:

  .. list-table::
    :header-rows: 0

    * - :fsharp:`recipient: Zen.Types.contractId`
      - The recipient contract.
    * - :fsharp:`command: string`
      - The command given to the recipient contract.
    * - :fsharp:`body: option Zen.Types.data`
      - The message body of given to the recipient contract.
