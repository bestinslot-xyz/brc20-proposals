# BRC20 Programmable Module - Proposal for EVM Upgrade to Prague

This document proposes an upgrade to the BRC20 Programmable Module to support the Prague hard fork. The Prague upgrade introduces several improvements and optimizations to the Ethereum Virtual Machine (EVM), which can enhance the performance and functionality of the BRC20 module.

https://ethereum.org/roadmap/pectra/ outlines the key features of the Prague upgrade, including:

- Opcodes and gas cost adjustments
- Improved handling of certain operations
- Enhanced support for cryptographic operations

Blobs and access lists are not relevant to BRC2.0 as it does not use them.

## What we're proposing to upgrade:

- Upgrade the BRC20 Programmable Module to be compatible with the Prague hard fork.
- Inherit the new EVM features and optimizations introduced in Prague.
- Precompile changes to support new opcodes and gas cost adjustments.

## Impact

### On Developers

- Developers will have access to new EVM features, enabling more complex and efficient smart contract designs and cryptographic operations such as BLS12-381.
- Gas costs for certain operations may change, which could affect the cost of deploying and interacting with BRC2.0 smart contracts.
- Smart contracts may need to be reviewed and potentially updated to ensure compatibility with the new EVM features and gas cost structure.

### On Indexers

- Indexers will need to update their implementations to support the changes introduced in the Prague upgrade. For most, this will be a matter of updating the Programmable Module to the latest version that supports Prague.

## Rationale

### Alignment with Ethereum

The BRC20 module aims to be compatible with the Ethereum ecosystem. Upgrading to Prague ensures that the module remains aligned with the latest developments in the Ethereum network, providing users with a consistent experience.

### Precompile Improvements

The Prague upgrade introduces new precompiles, which can enhance the performance of cryptographic operations. This is particularly relevant for BRC20, as it may involve operations that can benefit from these optimizations such as staking pools, restaking, light clients, bridges but also zero-knowledge.

## When?

The upgrade will be scheduled on Signet first, and then on Mainnet after sufficient testing and validation.

Block heights will be announced in advance to allow developers and indexers to prepare for the upgrade.

## Conclusion

Upgrading the BRC20 Programmable Module to support the Prague hard fork is a crucial step in ensuring its continued relevance and performance in the evolving Ethereum ecosystem. By implementing the proposed changes, we can enhance the functionality of the module, provide better tools for developers, and maintain alignment with Ethereum's advancements.