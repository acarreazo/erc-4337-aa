# ERC-4337-aa
ERC 4337 Account abstraction
ERC 4337 is the implementation of EIP 4337, deployed in the mainnet on 2023 on 0x0576a174D229E3cFA37253523E645A78A0C91B57. <br>
_`According to EIP 2938 Ethereum has two types of accounts: Externally Owned Account (EOA) and Contract Account (CA) (Smart Contract). Only EOAs can start transaction execution by paying gas.`_


ERC 4337 give smart contract capabilities to EOA, The implementation of EIP 4337 has the following
| Contracts | Description |
| --- | --- |
| EntryPoint.sol |  |
| Helpers.sol |  |
| SenderCreator.sol  |  |
| StakeManager.sol |  |
| IAccount.sol |  |
| IAggregator.sol |  |
| IEntryPoint.sol | |
| IPaymaster.sol |  |
| IStakeManager.sol |  |
| UserOperation.sol |  |
| Exec.sol |  |

These are the main definition for this specification

* **UserOperation** - a structure that describes a transaction to be sent on behalf of a user. To avoid confusion, it is not named "transaction".
  * Like a transaction, it contains "sender", "to", "calldata", "maxFeePerGas", "maxPriorityFee", "signature", "nonce"
  * unlike a transaction, it contains several other fields, described below
  * also, the "nonce" and "signature" fields usage is not defined by the protocol, but by each account implementation
* **Sender** - the account contract sending a user operation.
* **EntryPoint** - a singleton contract to execute bundles of UserOperations. Bundlers/Clients whitelist the supported entrypoint.
* **Bundler** - a node (block builder) that bundles multiple UserOperations and create an EntryPoint.handleOps() transaction. Note that not all block-builders on the network are required to be bundlers
* **Aggregator** - a helper contract trusted by accounts to validate an aggregated signature. Bundlers/Clients whitelist the supported aggregators.

 Aditionally , there is a smart contract called **Paymasters** that can sponsor transactions for other users

EntryPoint has struct UserOperation
```solidity
   struct UserOperation {
        address sender;
        uint256 nonce;
        bytes initCode;
        bytes callData;
        uint256 callGasLimit;
        uint256 verificationGasLimit;
        uint256 preVerificationGas;
        uint256 maxFeePerGas;
        uint256 maxPriorityFeePerGas;
        bytes paymasterAndData;
        bytes signature;
    }
 ```
 
`EntryPoint` receives in **handleOps** from `Bundler` bundles of userOperations, this methods validate userOperations and then execute UserOperations. In the case Agregator exists, EntryPoint run function **handleAggregatedOps**

```solidity
function handleOps(UserOperation[] calldata ops, address payable beneficiary) public {

        uint256 opslen = ops.length;
        UserOpInfo[] memory opInfos = new UserOpInfo[](opslen);

    unchecked {
        for (uint256 i = 0; i < opslen; i++) {
            UserOpInfo memory opInfo = opInfos[i];
            (uint256 validationData, uint256 pmValidationData) = _validatePrepayment(i, ops[i], opInfo);
            _validateAccountAndPaymasterValidationData(i, validationData, pmValidationData, address(0));
        }

        uint256 collected = 0;

        for (uint256 i = 0; i < opslen; i++) {
            collected += _executeUserOp(i, ops[i], opInfos[i]);
        }

        _compensate(beneficiary, collected);
    } //unchecked
 }
 ```

When handleOps validate userOperations validate account and paymaster (if is the case), also make sure total validation doesn't exceed verificationGasLimit

```solidity
    function _validatePrepayment(uint256 opIndex, UserOperation calldata userOp, UserOpInfo memory outOpInfo)
    private returns (uint256 validationData, uint256 paymasterValidationData) {
 ```
