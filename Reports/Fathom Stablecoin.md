#### M-1. Centralization issues
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L325
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L330
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L64
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L146
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L152
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L164
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L47
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L112
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L127

Having a single EOA as the only owner of contracts is a large centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary, or the single owner can become malicious and perform damaging operations. For example, an owner can manipulate prices by changing ``PriceOracle`` and ``CollateralPoolConfig`` contract.
##### Recommendation
It’s recommended to use multisig or DAO for the onlyGovernor operations and consider adding the ability to revoke the governor or decreasing the number of governor-controlled variables.

#### M-2. No check that ``_to != address(0)`` in ``emergencyWithdraw``
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L161

In the event of emergency, the user may accidentally transfer funds to address(0).
##### Recommendation
It’s recommended to add ``require(_to != address(0), "zero destination address")``.

#### L-1. Unrestricted access to the ``settleSystemBadDebt`` function in ``BookKeeper``
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L439

The [comment](https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L434) for the ``settleSystemBadDebt`` function says: 
```
    * @dev This function can only be called by the SystemDebtEngine, which incurs the system debt. The BookKeeper contract must not be paused.
```
But the [function](https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L439) doesn't have a modifier that restricts access:
```
    function settleSystemBadDebt(uint256 _value) external override nonReentrant whenNotPaused {
```
Anyone can call the ``settleSystemBadDebt`` function in ``BookKeeper`` contract to change the values of the ``totalUnbackedStablecoin`` and ``totalStablecoinIssued`` variables.
##### Recommendation
It’s recommended to protect the function with a modifier.

#### L-2. No compatibility with tokens with fees
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L193
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L225
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L234
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L173
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L189
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L214

Smart contracts do not check token balances before and after transfers.
##### Recommendation
It’s recommended to compare token balances before and after transfers.

#### L-3. Timelock to critical functions
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L325
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L330
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L146
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L152
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L164
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L454

Giving users time to react and adjust to critical changes in the protocol provides more guarantees and increases the transparency of the protocol.
##### Recommendation
It’s recommended to add a timelock.

#### L-4. The function does not have a modifier
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L42

The function ``allowManagePosition`` does not have a modifier ``onlyDelegateCall`` like the other functions.
##### Recommendation
It’s recommended to add a modifier.

#### L-5. Typos
##### Description
There are typos:
- ``recevied`` => ``received`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L228
- ``neeeded`` => ``needed`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L455
- ``stalbecoin`` => ``stablecoin`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L38
- ``postion`` => ``position`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L414
- ``systemDebyEngine`` => ``systemDebtEngine`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L427
- ``withdrawl`` => ``withdrawal`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L168

##### Recommendation
It’s recommended to correct them.

#### L-6. Check null input data
##### Description
- ``_destination`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L246
- ``_dst`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L234
- ``_usr`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L227
- ``_dst`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L267
- ``_to`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L458
- ``_usr`` https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L214

##### Recommendation
It’s recommended to check that the function argument ``!= address(0)``.

#### L-7. The event is emitted even if no withdrawal occurs.
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L216

When ``_amount`` == 0, the event ``LogWithdraw`` is emitted.
##### Recommendation
It’s recommended place the event emission inside if-statement as here https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L198.

#### L-8. Use flags 1/2 instead of 0/1
##### Description
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L42
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L44
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L40
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L21

For whitelisting and for the ``live`` variable, two flags are used: 0 and 1. It is better to use ``uint256(1)`` and ``uint256(2)`` for ``true``/``false`` to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ``false`` to ``true``, after having been ``true`` in the past.
##### Recommendation
It’s recommended to use ``uint256(1)`` and ``uint256(2)``.

#### L-9. Some functions are missing a NatSpec
##### Description
These functions are not annotated using NatSpec:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L75
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L325
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L330
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L343
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L347
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L128
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L146
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L152
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L164
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L173
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L491
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L495
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L74
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L112
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L127
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol (In this contract, all functions except one are not annotated)

##### Recommendation
It’s recommended to fully annotated contracts using NatSpec.

#### L-10. Lack of event emission
##### Description
These ``initialize`` and setter functions do not emit any events:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L75
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L128
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L74
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L325
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L330
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L127

##### Recommendation
It’s recommended to add ``emit event`` statements within the listed functions

#### L-11. Add more information in the events
##### Description
These events only store information about new values:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L149
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L160
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L168

##### Recommendation
It’s recommended to include information about previous values.

#### L-12. ``require`` check without message
##### Description
This ``require`` check has no message:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L30

##### Recommendation
It’s recommended to add necessary message to the mentioned check.

#### L-13. ``Public`` functions can be declared ``external``
##### Description
These functions are not called by the contract and can be declared ``external``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L233
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L353
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L378
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/proxy-actions/FathomStablecoinProxyActions.sol#L409

##### Recommendation
It’s recommended to make the listed functions ``external``

#### L-14. Cache some state variables
##### Description
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. State variables that are used more than once in a function can be cached instead of being re-read from storage:
``bookKeeper``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L112
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L118

``lastPositionId``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L118
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L119
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L120
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L121
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L125
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L128
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L129
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L131
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L134
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L136

``bookKeeper``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L257
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L258

``bookKeeper``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L277
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L278

``bookKeeper``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L297
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/managers/PositionManager.sol#L298

``collateralPoolConfig``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L293
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L300

``collateralPoolConfig``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L413
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L419

``collateralPoolConfig``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L478
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/BookKeeper.sol#L481

``collateralPoolId``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L163
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L169

``collateralToken``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L189
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L195

``collateralPoolId``:
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L207
- https://github.com/Into-the-Fathom/fathom-stablecoin-smart-contracts/blob/a7d2ff13fde5beab1a677bfc7171641b4023d9c8/contracts/main/stablecoin-core/adapters/CollateralTokenAdapter/CollateralTokenAdapter.sol#L208

##### Recommendation
It’s recommended to cache the listed variables.