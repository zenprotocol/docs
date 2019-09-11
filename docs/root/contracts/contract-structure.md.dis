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
    : Zen.Types.contractResult `Zen.Cost.t` n
```

where ```n``` must be an expression which evaluates to a natural number (```nat```).

### Parameters

- ```txSkel : Zen.Types.txSkeleton```
    Details about the function.

- ```context : Zen.Types.context```
    Details about the function.

- ```contractId : Zen.Types.contractId```
    Details about the function.

- ```command : string```
    - String that the contract may use.
    - Contains a command which tells the contract what to do.
    - For example:
        ```F*
        match command with
        | "redeem" -> redeem txSkeleton contractId returnAddress wallet
        | "buy"    -> buy txSkeleton contractId returnAddress
        ```

- ```sender : Zen.Types.sender```
    Details about the function.

- ```messageBody : option Zen.Types.data```
    Details about the function.

- ```wallet : Zen.Types.wallet```
    Details about the function.

- ```state : option Zen.Types.data```
    Details about the function.
