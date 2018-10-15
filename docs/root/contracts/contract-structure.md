# Contract Structure

Each contract must contain 2 functions: ```main``` and ```cf```.

The structure of the ```main``` function is:

```F*
main
    ( txSkel      : Zen.Types.txSkeleton  )
    ( context     : Zen.Types.context     )
    ( contractId  : Zen.Types.contractId  )
    ( command     : string                )
    ( sender      : Zen.Types.sender      )
    ( messageBody : option Zen.Types.data )
    ( wallet      : Zen.Types.wallet      )
    ( state       : option Zen.Types.data )
    : contractResult `Zen.Cost.t` n
```

where ```n``` must be an expression which evaluates to a natural number (```nat```).
