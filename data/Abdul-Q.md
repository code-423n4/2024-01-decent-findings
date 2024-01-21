issue#1
getId() function should be defined once because both contracts(DecentBridgeAdapter.sol, StargateBridgeAdapter.sol ->  at src/bridge_adapters)have same functions at line 30-32 and 41-43 repectively. 

issue#2
getId() function is not pure function as it is interacting with contract's variables.

issue#3
mark these functions(getBridgeToken(), getBridgedAmount(), bridge() ) as override function located at mentioned contracts in issue#1