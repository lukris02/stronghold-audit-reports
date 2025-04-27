#### M-1 No limits for variables
There are no limits for the following variables:
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L143 (``_rewardsDuration``)
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L160 (``_rewardsDuration``)

##### Recommendation
It is recommended to add ranges for variables.

#### M-2 A possibility to lose access to protocol
##### Description
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L4
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L4
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L5
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L4

``Ownable.sol`` has 1-step transfer of ownership. If the owner ``transferOwnership`` to non-active address, the owner loses access to the contract.

##### Recommendation
It is recommended to use ``Ownable2Step.sol``.

#### M-3 Reentrancy risk
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L405-L408

Changing state variables after the external call is not safe.

##### Recommendation
It is recommended to use Checks Effects Interactions pattern.

#### L-1 Unsafe cast
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L444
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L449

In the ``setVestingCategory`` function, the argument ``maxAllocation`` is cast to ``int256`` without overflow checks.

##### Recommendation
It is recommended to check ``if (maxAllocation > 0)``.

#### L-2 Owner can renounce ownership
##### Description
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L4
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L4
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L5
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L4

``Ownable.sol`` implements ``renounceOwnership()`` function which leaves the contract without an owner and removes any functionality that is only available to them.  
If the owner mistakenly calls the ``renounceOwnership()`` function, then they will permanently lose the ability to call the ``onlyOwner`` functions.

##### Recommendation
It is recommended to reimplement the ``renounceOwnership()`` function to disable it.

#### L-3 A ``SafeApprove`` is deprecated.
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L136
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L137
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L157
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L189
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L238

The use of ``SafeApprove()`` is deprecated in favor of ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()``. 

##### Recommendation
It is recommended to replace ``safeApprove()`` with ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()``.

#### L-4 Null address checks 
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L43-L45
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L153
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufMigrator.sol#L28
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/VotingEscrowTruf.sol#L72-L73
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/VotingEscrowTruf.sol#L75

The checks for zero address is missing.

##### Recommendation
It is recommended to add null address checks.

#### L-5 Event emission is missing
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L152-L154

The function ``setRewardsDistribution`` does not emit ``RewardsDistributionUpdated`` event.

##### Recommendation
It is recommended to add the emission of ``RewardsDistributionUpdated(_rewardsDistribution)``.

#### L-6 Typos
##### Description

- ``Interal`` to ``Internal``: https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/VotingEscrowTruf.sol#L289
- ``montlhy`` to ``monthly``: https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L102
- ``Calcualte`` to ``Calculate``: https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufVesting.sol#L166

##### Recommendation
It is recommended to fix the typos.

#### L-7 No NatSpec
##### Description
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol

The NatSpec is missing for all function in above contracts.

##### Recommendation
It is recommended to add NatSpec.

#### L-8 ``_rewardsDuration`` can be set to zero value
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L143

##### Recommendation
It is recommended to add the check as in ``VirtualStakingRewards.sol`` https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L164-L166.

#### L-9 Sanity checks
##### Description
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L67
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/TrufPartner.sol#L86
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L42
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/StakingRewards.sol#L152
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L65
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/staking/VirtualStakingRewards.sol#L172
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/TrufMigrator.sol#L27
- https://github.com/truflation/truflation-contracts/blob/010448ea069c0a1b762104a75aad345219c3f96e/src/token/VotingEscrowTruf.sol#L68

##### Recommendation
It is recommended to check if the contracts exist at the specified address.