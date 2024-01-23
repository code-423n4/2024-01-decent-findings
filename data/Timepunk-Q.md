This report summarizes the low/non-critical findings from the audit.

## Table of Contents
1. INFO: No event emitted on critical setter
2. LOW: No check if returned addresses are not address 0 will lead to unspecified error
3. LOW: Unchecked return values
4. LOW: Unchecked amount is not 0 can lead to action being taken regardless of need
5. LOW: Unchecked argument input to function can lead to action being taken regardless of need
6. LOW: Unchecked msg.value is higher than provided value can lead to underflow

## INFO: No event emitted on critical setter
**File**: UTB.sol, BaseAdapter.sol, DecentBridgeAdapter.sol
**Line**: UTB: L29, L37, L45, L327, L336 | BaseAdapter.sol: L18 | DecentBridgeAdapter.sol: L28, L41
**Issue**: No event emitted on critical setter
**Recommendation**: Define appropriate events and emit them in the setters. This allows off-chain services and UIs to react to the changes.

## LOW: No check if address is not 0 will lead to unspecified error
**File**: UTB.sol
**Line**: L186-L189; L192-L193; L211, L289
**Issue**: By not checking inputs to the mapping or checking that the returned address is different than 0, a call can be made to address 0 which will lead to an unspecified revert
**Recommendation**: Check if address returned by mapping is different than 0.

## LOW: Unchecked return values
**File**: UTB.sol
**Line**: L186-L189 getBridgedAmount; L192-L193 updateSwapParameters; L212 getBridgeToken
**Issue**: In many areas the return values of view functions are unchecked. It is a good practice to check the return value of these functions because otherwise it can cause problems down the line.
**Recommendation**: Check the returned values from view functions meet the desired intentions for the continuation of the process.

## LOW: Unchecked amount value
**File**: UTB.sol
**Line**: UTB: feeAmount in L239 and L243; L270 amt2Bridge
**Issue**: In many areas the amounts or values are not checked to not be 0, which can lead to unnecessary operations and gas waste down the line.
**Recommendation**: Check the returned values from view functions meet the desired intentions for the continuation of the process.

## LOW: Unchecked argument input to function can lead to action being taken regardless of need or unspecified reverts
**File**: UTB.sol; DecentBridgeAdapter.sol; StargateBridgeAdapter.sol
**Line**: UTB: L325, L334 | DecentBridgeAdapter: L26, L34, L44, L81 | StargateBridgeAdapter: L45
**Issue**: By not checking the argument to the function, invalid parameters can be registered, or an action can be taken by mistake, leading to bugs or unspecified reverted operations.
**Recommendation**: It is a good practice to check if the input to a state changing function is not 0, unless desired.

## LOW: Unchecked msg.value is higher than provided value can lead to underflow
**File**: UTB.sol; DecentEthRouter.sol;
**Line**: UTB: L248 | DecentEthRouter: L179
**Issue**: The value provided can be higher than the specified msg.value leading to an underflow that will create an unspecified error. It is advisable to check msg.value is greater than amount to increase transparency in the errors that are thrown
**Recommendation**: Check that msg.value is greater or equal than the specified value and revert otherwise