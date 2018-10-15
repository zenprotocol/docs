==================
Contract Structure
==================

.. highlight:: rst

.. role:: fstar(code)
    :language: fsharp

Each contract must contain 2 functions: :fstar:`main` and :fstar:`cf`.

The structure of the :fstar:`main` function is:

.. codeblock:: fstar

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
