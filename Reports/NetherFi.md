#### H-1 Less NLP tokens can be minted
##### Description
The amount of tokens for minting in ``NlpManager.sol`` can be calculated incorrectly because tokens can not be removed from the ``allWhitelistedTokens``.
1. The governance adds ``someToken`` to whitelist. ``whitelistedTokens[`someToken] = true``, and ``someToken`` is added to``allWhitelistedTokens`` array .
2. The governance removes ``someToken`` from whitelist. ``whitelistedTokens[`someToken] = false``, but ``someToken`` stays in ``allWhitelistedTokens`` array .
3. The governance adds ``someToken`` to whitelist **again**. ``whitelistedTokens[`someToken] = true``, and ``someToken`` is added to``allWhitelistedTokens`` array **a second time**.
4. The function ``getAum()`` calculates the same token **twice**.
5. ``_addLiquidity`` function will mint less NLP tokens because ``aumInUsdg`` will increase and the ``mintAmount`` will decrease.

- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L360-L379
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L393-L405
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L136-L179
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L209-L229

##### Recommendation
It is recommended to implement token removal from ``allWhitelistedTokens``.

#### H-2 Chainlink’s deprecated API
##### Description
The ``VaultPriceFeed.sol`` uses Chainlink’s deprecated API. Deprecated functions may no longer work if Chainlink stops supporting deprecated APIs. Then prices cannot be obtained and contracts have to be redeployed.
Also ``latestRoundData()`` should be used instead of ``latestAnswer()`` because``latestAnswer()`` may return stale data and does not allow to check if the data is fresh. Usage of ``latestRoundData()`` allows to run some extra validations. For example:
```
(uint80 roundID, int256 price,,uint256 timestamp, uint80 answeredInRound) = priceFeed.latestRoundData();
require(timestamp != 0, "round not complete");
require(answeredInRound >= roundID, "stale data");
require(price != 0, "invalid price");
```

- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/VaultPriceFeed.sol#L287-L316

##### Recommendation
It is recommended to use V3 (https://docs.chain.link/data-feeds/api-reference).

#### M-1 ``Vault`` contract cannot be upgraded
##### Description
``Timelock`` contract does not have an access to the function ``upgradeVault()``. If it is necessary to update the ``Vault`` contract, the team will need to deploy a new ``Timelock`` contract.
Also, it is better to use the UUPS proxy pattern.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L436-L440
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/peripherals/Timelock.sol

##### Recommendation
It is recommended to add the ability to call the ``upgradeVault()`` function in the ``Timelock`` contract and consider using the UUPS proxy pattern.

#### M-2 Some ``onlyGov`` functions are not available in ``Timelock.sol``
##### Description
The functions listed below cannot be called from ``Timelock.sol``. To use them the team will have to deploy a new ``Timelock`` contract.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L245
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L259
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L393
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L93

##### Recommendation
It is recommended to add the ability to call listed functions in the ``Timelock`` contract or remove them if they are not needed.

#### M-3 The first ``gov`` does not lose admin rights
##### Description
The first ``gov`` continues to have admin rights obtained in the constructor after passing ``gov`` rights through the ``setGov()`` function.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L49
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L53-L55

##### Recommendation
It is recommended to revoke admin rights in ``setGov()``.

#### M-4 Tokens cannot be removed from ``allWhitelistedTokens``
##### Description
There is no functionality in the contract to remove tokens from ``allWhitelistedTokens``. This causes the frequently used ``getAum()`` function to waste extra gas iterating over all tokens. Moreover, this may eventually lead to DoS due to an out of gas error.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L373
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L137-L142

##### Recommendation
It is recommended to implement a function to remove tokens  from allWhitelistedTokens.

#### M-5 The ``setDepositFee`` function does not have a range for ``_depositFee``
##### Description
Users may be charged very high fees of up to 100%. The function ``setDepositFee`` can be called by ``admin``, not ``gov``.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L110

##### Recommendation
It is recommended to have an upper bound for the ``_depositFee``.

#### M-6 The ``setMinExecutionFee`` function does not have a range for ``_minExecutionFee``
##### Description
Users may be charged very high fees. The function ``setMinExecutionFee`` can be called by ``admin``, not ``gov``.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionRouter.sol#L200

##### Recommendation
It is recommended to have an upper bound for the ``_minExecutionFee``.

#### M-7 Unlimited position sizes in ``setMaxGlobalSizes()``
##### Description
Not limiting ``_longSizes`` and ``_shortSizes`` to a sane lower limit
can make certain tokens unusable if set to economically very low values.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L127-L128

##### Recommendation
It is recommended to have an lower limit for the ``_longSizes`` and ``_shortSizes``.

#### L-1 No check of the result of the ``send()`` function
##### Description
``address.send()`` never throws an exception but will return false if the call encounters an exception. Using this function it is necessary to always handle the possibility that the call will fail by checking the return value. If ``_receiver`` is unable to accept the Ether, the user will lose their funds.
1. User calls ``decreasePositionETH()`` or ``decreasePositionAndSwapETH()`` function and as parameter ``_receiver`` specifies the address of a contract that cannot accept ether or whose fallback function requires more than 2,300 gas.
2. ``_receiver.send()`` fails but does not throw an exception.
3. The user's funds are lost and cannot be retrieved.

- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L285
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L145-L158
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L177C14-L194

##### Recommendation
It is recommended to check the return value of ``.send()``, handle failures, and use withdrawal pattern.

#### L-2 Price manipulation
##### Description
It is not secure to read pricing information from an exchange or other protocol. Pair reserves can be easily manipulated. An attacker can take a flashloan and set any price in the pair: either very high or extremely low. Once an asset’s price has been driven up, the attacker can then exchange their artificially inflated holdings for other tokens with greater liquidity and a more consistent value, or use them as collateral to borrow assets, never to be repaid. 
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/VaultPriceFeed.sol#L372

##### Recommendation
It is recommended to use the service of a robust decentralized oracle or by aggregating many different price feeds.

#### L-3 ``_name`` and ``_symbol`` of the token can be changed
##### Description
Using ``setInfo()``, a``gov`` can change ``_name`` and ``_symbol`` of the token. It may lead to unexpected behavior with other contracts or platforms.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L57
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/BaseToken.sol#L56

##### Recommendation
It is recommended to remove this functionality.

#### L-4 DoS due to unlimited iterations
##### Description
Length of the ``yieldTrackers`` array is not limited. Some functions loop through the yieldTrackers array. Too many iterations can cause out of gas error.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L102
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L109
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/YieldToken.sol#L215
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/BaseToken.sol#L101
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/BaseToken.sol#L108
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/tokens/BaseToken.sol#L218

##### Recommendation
It is recommended to limit ``yieldTrackers`` size.

#### L-5 Unsafe addition
##### Description
``increasePositionBufferBps`` is not limited, and unsafe addition operation can cause overflow.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L115
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L344

##### Recommendation
It is recommended to use safe addition operation and/or consider having upper and lower limit for ``increasePositionBufferBps``.

#### L-6 Use of Old Solidity Version
##### Description
Newer versions of Solidity not only address security issues, but may also introduce implicit security enhancements, such as the addition of implicit underflow / overflow checks in Solidity >=0.8.0.
- All contracts

##### Recommendation
It is recommended to update all contracts to the latest major solidity version.

#### L-7 Emit events for critical parameters change
##### Description
Events help non-contract tools to track changes.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L264
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L279
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L294
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L299 
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L304
- and other setters

##### Recommendation
It is recommended to emit the events.

#### L-8 Update some events
##### Description
Include information about both old and new values in events.
-  https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L112
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L117
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L107
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L72
-  https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionRouter.sol#L202
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionRouter.sol#L207
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionRouter.sol#L197

##### Recommendation
It is recommended to update the events.

#### L-9 Null address checks
##### Description
Before setting the value, it is needed to be checked that it is not a null address.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L232-L237 
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L64-L68 
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L92-L96 
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L52
- and all setters

##### Recommendation
It is recommended to add null address checks.

#### L-10 Unlocked pragma
##### Description
Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in ``pragma solidity ^0.6.0``, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs. ([SWC-103](https://swcregistry.io/docs/SWC-103/))
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/BasePositionManager.sol#L3
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L3
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionRouter.sol#L3

##### Recommendation
It is recommended to remove ^ in ``pragma solidity ^0.6.0`` and change it to ``pragma solidity 0.6.12`` to be consistent with the rest of the contracts.

#### L-11 Gas optimizations
##### Description
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. State variables that are used more than once in a function can be cached instead of being re-read from storage:
``usdg``:
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L452
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/Vault.sol#L484

``vault``:
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L136
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L209
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L232
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L80
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/PositionManager.sol#L106

##### Recommendation
It is recommended to create and use a memory variable.

#### L-12 No NatSpec
##### Description
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces.
- All contracts

##### Recommendation
It is recommended to add NatSpec.

#### L-13 Glp to nlp
##### Description
Change ``glp`` to ``nlp`` in variables:
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L23
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L30
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L47
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L55
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L57

##### Recommendation
It is recommended to rename variables.

#### L-14 Public function can be external
##### Description
The function is not called by the contract where it is defined, so it can be declared external.
- https://github.com/nether-labs/nether.fi-contracts/blob/e2531589be47e5530cc2157e6eb2af9119a03c2b/contracts/core/NlpManager.sol#L124

##### Recommendation
It is recommended to change the visibility of the function.