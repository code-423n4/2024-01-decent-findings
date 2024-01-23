## [G-01] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtructions act the same way.

There is 1 instance of this issue in 1 file:

```
File: lib/decent-bridge/src/DecentEthRouter.sol	

56: balanceOf[msg.sender] += amount;
```
    diff --git a/src/DecentEthRouter.sol b/src/DecentEthRouter.sol
    index eb90549..5c4e73f 100644
    --- a/src/DecentEthRouter.sol
    +++ b/src/DecentEthRouter.sol
    @@ -53,7 +53,7 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
         }

         modifier userDepositing(uint256 amount) {
    -        balanceOf[msg.sender] += amount;
    +        balanceOf[msg.sender] = balanceOf[msg.sender] + amount;
             _;
         }


https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |