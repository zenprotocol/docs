# Contract Structure

Each contract must contain 2 functions: ```main``` and ```cf```.

The structure of the ```main``` function is:

```F*
main (txSkel: txSkeleton) _ (contractId: contractId) (command: string)
         (sender: sender) (messageBody: option data) (wallet: wallet)
         (state: option data)
```
