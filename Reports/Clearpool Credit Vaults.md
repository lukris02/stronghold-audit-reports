#### M-1 No limits for variables
There are no limits for the following variables:
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/Auction.sol#L337 (``bidIncrementMaxAmount_``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/BondNFT.sol#L84 (``_exchangeRate``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L385 (``_newGracePeriod``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L410 (``_newProtocolFee``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L438 (``_rate``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L457 (``period``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L986 (``_newDepositCap``)
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L1030 (``period``)

##### Recommendation
It is recommended to add ranges for variables.

#### M-2 A possibility to lose access to protocol
##### Description
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/WhitelistControl.sol#L175-L180
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/Auction.sol#L7
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L6

The owner of ``WhitelistControl`` can ``transferOwnership`` to non-active address. If this happens, the owner loses access to the contract. ``OwnableUpgradeable.sol`` also has 1-step transfer of ownership.

##### Recommendation
It is recommended to add a two-step procedure for ``transferOwnership`` function and use ``Ownable2StepUpgradeable.sol``.

#### L-1 A division by zero in the library ``RewardAsset``
##### Description
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/libraries/RewardAsset.sol#L94
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/libraries/RewardAsset.sol#L107

The ``totalSupply`` input value in the ``updateMagnifiedRewardPerShare`` and ``updateMagnifiedRoundRewardPerShare`` functions in the ``RewardAsset``
contract could be equal to zero. Since the library can be used in other contracts, to prevent forgetting the zero value check, it is better to place this check inside the library.

##### Recommendation
It is recommended to add zero checks for the ``totalSupply`` value in the library functions.

#### L-2 Owner can renounce ownership
##### Description
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/Auction.sol#L7
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L6

``OwnableUpgradeable.sol`` implements ``renounceOwnership()`` function which leaves the contract without an owner and removes any functionality that is only available to them.  
If the owner mistakenly calls the ``renounceOwnership()`` function, then they will permanently lose the ability to call the ``onlyOwner`` functions.

##### Recommendation
It is recommended to reimplement the ``renounceOwnership()`` function to disable it.

#### L-3 Reentrancy risk
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/BondNFT.sol#L137
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/BondNFT.sol#L160

``safeTransferFrom`` and ``safeBatchTransferFrom`` functions can potentially allow a reentrancy attack when transferring tokens to an untrusted contract, when invoking {onERC1155Received} on the receiver.
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/IERC1155.sol#L83
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/IERC1155.sol#L103

##### Recommendation
It is recommended to use reentrancy guards.

#### L-4 Unused errors
##### Description
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/Auction.sol#L96
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L75

##### Recommendation
It is recommended to remove unused errors.

#### L-5 No NatSpec in ``RewardAsset``
##### Description
- https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/libraries/RewardAsset.sol

The NatSpec is missing for all function in``RewardAsset.sol``.

##### Recommendation
It is recommended to add NatSpec.

#### L-6 Typos
##### Description

- ``borrowwer`` to ``borrower``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L230
- ``accross`` to ``across``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L302
- ``penality`` to ``penalty``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L392
- ``penality`` to ``penalty``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolFactory.sol#L394
- ``supplyed`` to ``supplied``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L32
- ``redeeemed`` to ``redeemed``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L34
- ``repaing`` to ``repaying``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L467
- ``penality`` to ``penalty``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L803
- ``mininum`` to ``minimum``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/PoolMaster.sol#L992
- ``occured`` to ``occurred``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/libraries/RewardAsset.sol#L19
- ``occured`` to ``occurred``: https://github.com/clearpool-finance/credit-vaults-public/blob/main/contracts/libraries/RewardAsset.sol#L25

##### Recommendation
It is recommended to fix the typos.