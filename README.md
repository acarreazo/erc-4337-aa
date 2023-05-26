# ERC-4337-aa
ERC 4337 Account abstraction
ERC 4337 is the implementation of EIP 4337, deployed in the mainnet on 2023 on 0x0576a174D229E3cFA37253523E645A78A0C91B57. Has the following

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
