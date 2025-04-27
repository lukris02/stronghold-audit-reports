#### M-1 ``setFees`` functions do not have the ranges for input parameters
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L117
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L145

The owner can set unlimited values for ``gFees.referralPercent``, ``gFees.referralDiscount``, ``gFees.ethFee``, ``gFees.secondaryTokenFee``, ``gFees.secondaryTokenDiscount``, ``gFees.liquidityFee`` in ``UniswapV2Locker.sol`` and ``FEES.tokenFee``, ``FEES.freeLockingFee``, ``FEES.feeAddress``, ``FEES.freeLockingToken`` in ``TokenVesting.sol``.  
Some values may be harmful to users.

##### Recommendation
It is recommended to limit the lower and upper possible values for all input parameters.

#### M-2 User storage is not cleaned in ``splitLock``
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L305

When a user splits a lock, it may be that ``userLock.amount == _amount``. In this case it is needed to clean user storage.

##### Recommendation
It is recommended to clean user storage as in ``withdraw``: https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L257-L265.

#### L-1 Improve ``_unlock_date`` validation
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L147
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L229
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L233

The functions ``lockLPToken`` and ``relock`` have the check: 
```
require(_unlock_date < 10000000000, 'TIMESTAMP INVALID'); // prevents errors when timestamp entered in milliseconds
```
This check does not prevent the user from setting the date too large (up to 2286).  
In case of an error, the user will not be able to reduce the lock date and can be said to lose funds.

##### Recommendation
It is recommended to check that ``_unlock_date`` is less than ``(block.timestamp + N years (in seconds))`` in the function ``lockLPToken``.   
In the function ``relock``, it can be checked that ``_unlock_date`` is less than ``(userLock.unlockDate + N years (in seconds))``.

#### L-2 Owner can renounce ownership
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L11
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/Ownable.sol#L57-L60
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L51
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/Ownable.sol#L56-L59
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L10
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/%40openzeppelin/contracts/access/Ownable2Step.sol#L6
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/%40openzeppelin/contracts/access/Ownable.sol#L61-L63

``Ownable.sol`` implements ``renounceOwnership()`` function which leaves the contract without an owner and removes any functionality that is only available to them.  
If the owner mistakenly calls the ``renounceOwnership()`` function, then they will permanently lose the ability to call the ``onlyOwner`` functions.

##### Recommendation
It is recommended to reimplement the ``renounceOwnership()`` function to disable it.

#### L-3 ``Ownable.sol`` has 1-step transfer of ownership
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L11
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/Ownable.sol#L66-L70
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L51
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/Ownable.sol#L65-L69

If the owner accidentally transfers ownership to an incorrect address, protected functions will become permanently inaccessible.

##### Recommendation
It is recommended to use ``Ownable2Step`` instead of ``Ownable`` to better safeguard against accidental transfers of access control.

#### L-4 No error message
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L405
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L457

Without an error message, the user may not understand why the transaction reverted.

##### Recommendation
It is recommended to add error message in ``require``.

#### L-5 A ``safeApprove`` is deprecated
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L184
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L371
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L441
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L244
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L245
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L315
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L316

The use of ``safeApprove`` is deprecated in favor of ``safeIncreaseAllowance`` and ``safeDecreaseAllowance``.

##### Recommendation
It is recommended to replace ``safeApprove`` with ``safeIncreaseAllowance`` and ``safeDecreaseAllowance``.

#### L-6 Emit events for critical parameters change
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L101
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L105
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L112
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L117
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L137
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L141
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L145

Events help non-contract tools to track changes.

##### Recommendation
It is recommended to emit the events.

#### L-7 Unlocked pragma
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L44

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in ``pragma solidity ^0.8.0``, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs. SWC-103: https://swcregistry.io/docs/SWC-103/

##### Recommendation
It is recommended to remove ^ in ``pragma solidity ^0.8.0`` and change it to ``pragma solidity 0.8.19``.

#### L-8 Constants can be used
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L147
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L229
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L214
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L314
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L339

Some numeric values are used multiple times.

##### Recommendation
It is recommended to replace numeric values with the constants.

#### L-9 Typos
##### Description
- ``withdrawls`` to ``withdrawals``: https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L48
- ``refferer`` to ``referrer``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L67
- ``elegible`` to ``eligible``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L110
- ``WITHDRAWL`` to ``WITHDRAWAL``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L250
- ``seperate`` to ``separate``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L293
- ``overriden`` to ``overridden``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L28
- ``withdrawble`` to ``withdrawable``: https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L30
- ``minuites`` to ``minutes``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L32
- ``withdrawble`` to ``withdrawable``: https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L34
- ``raodmap`` to ``roadmap``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L37
- ``seperate`` to ``separate``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L390
- ``malicous`` to ``malicious``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L451
- ``withdrawls`` to ``withdrawals``: https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L451
- ``evalaute`` to ``evaluate``:  https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L451

##### Recommendation
It is recommended to fix the typos.

#### L-10 Null address checks
##### Description
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L94
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L101
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L105
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/locker/UniswapV2Locker.sol#L112
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L137
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L141
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/vesting/TokenVesting.sol#L145
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L71
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L81
- https://github.com/glcanvas/stronghold-2023-11-uncx/blob/master/src/proof/UNCX_ProofOfReservesV2_UniV3.sol#L85

Before setting the value, it is needed to be checked that it is not a null address.

##### Recommendation
It is recommended to add null address checks.