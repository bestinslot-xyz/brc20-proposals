# BRC20 Programmable Module - Proposal for a new Precompile to return current Transaction ID

## Summary

This proposal suggests the introduction of a new precompile in the BRC20 programmable module that allows smart contracts to retrieve the current Bitcoin transaction ID. This functionality is essential for various use cases, including tracking transactions and implementing custom logic based on transaction history.

BRC2.0 transactions are activated by sending the inscription to an `OP_RETURN "BRC20PROG"` output. This calls or deploys a smart contract, but the transaction ID of the transaction that contains the `OP_RETURN "BRC20PROG"` output is not currently accessible from within the smart contract.

## Motivation

Currently, smart contracts on the BRC20 programmable module lack the ability to access the transaction ID of the current Bitcoin transaction. This limitation hinders developers from creating more complex and interactive contracts that rely on transaction data. By introducing a precompile to return the current transaction ID, we can enhance the capabilities of smart contracts and enable new use cases.

## Specification

### Precompile Address

The new precompile will be assigned to `0xfa`, which is currently unused. This address is chosen to avoid conflicts with existing precompiles and to maintain consistency in the precompile addressing scheme.

### Function Signature

The function signature for the new precompile will be as follows:

```solidity
    function getBtcTxId() external view returns (bytes32);
```

### Implementation

The implementation of the precompile will involve the following steps:

1. **Precompile Invocation**: When a smart contract calls the `getBtcTxId` function, the precompile will be invoked.
2. **Transaction ID Retrieval**: The precompile will access the context to retrieve the transaction ID of the current transaction.
3. **Return Transaction ID**: The precompile will return the transaction ID as a `bytes32` value.

### Gas Cost

The computational cost for invoking the `getBtcTxId` precompile is trivial, as it involves a simple retrieval operation. A nominal gas cost of `20` units is proposed to cover the computational overhead, and match the `getBlockHash` opcode.

## Backward Compatibility

This proposal is backward compatible, as it introduces a new precompile without modifying existing functionality. Existing smart contracts will continue to operate as before, and developers can choose to utilize the new precompile as needed.

## Example Usage

Here is an example of how to use the new precompile in a smart contract:

```solidity
pragma solidity ^0.8.0;

interface TxIdPrecompile {
    function getBtcTxId() external view returns (bytes32);
}

contract Example {
    function getTxId() external view returns (bytes32) {
        return TxIdPrecompile(0xfa).getBtcTxId();
    }
}
```

## Target Block Height

There is a pending proposal to upgrade the underlying EVM to Prague hard fork at [../003-prog-evm-upgrade/index.md](../003-prog-evm-upgrade/index.md). This precompile can be introduced alongside that upgrade. The target block height for this precompile will be determined based on the implementation timeline of the Prague update.