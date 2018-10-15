==================
Contract Structure
==================

.. highlight:: rst

.. role:: fsharp(code)
    :language: fsharp

Each contract must contain 2 functions: :fsharp:`main` and :fsharp:`cf`.

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
    Details about the function.

* :fsharp:`context : Zen.Types.context`
    Details about the function.

* :fsharp:`contractId : Zen.Types.contractId`
    Details about the function.

* :fsharp:`command : string`
    String that the contract may use.
    Contains a command which tells the contract what to do.
    For example:
      .. code-block:: fsharp

        match command with
        | "redeem" -> redeem txSkeleton contractId returnAddress wallet
        | "buy"    -> buy txSkeleton contractId returnAddress


* :fsharp:`sender : Zen.Types.sender`
    Details about the function.

* :fsharp:`messageBody : option Zen.Types.data`
    Details about the function.

* :fsharp:`wallet : Zen.Types.wallet`
    Details about the function.

* :fsharp:`state : option Zen.Types.data`
    Details about the function.
