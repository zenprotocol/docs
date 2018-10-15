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
