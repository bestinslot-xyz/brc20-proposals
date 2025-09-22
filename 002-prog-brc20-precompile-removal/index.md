# BRC20 Programmable Module - Proposal to Remove BRC20 Balance Precompile at 0xFF

This document proposes the removal of the BRC20 balance precompile from BRC2.0 protocol. The BRC20 balance precompile was initially introduced to facilitate token balance queries for BRC20 tokens. However, it has become evident that this precompile introduces unnecessary complexity and potential security vulnerabilities to the protocol.

> [!NOTE]
> This is not a critical security patch, but a proposal to remove an unnecessary feature that adds complexity and potential security risks.

## What we're proposing to remove:

- BRC20_Balance precompile (at address 0xff) that allows smart contracts to check an account's balance outside BRC2.0

## Impact

As of block 915800, no smart contracts are known to be using the BRC20 balance precompile (verified using transaction traces). However, if any contracts are using it, they will need to be updated to remove this dependency.

### On Developers

- Smart contracts will no longer be able to query BRC20 token balances outside the BRC2.0 context using the precompile. They can still query balances of tokens held within the BRC2.0 module.

### On Indexers

- BRC2.0 package will need to be updated to the latest version that does not include the precompile. It isn't a breaking change, as the precompile will simply not be available.
- As a cleanup afterwards: Indexers will need to update their implementations to remove support for the BRC20 balance precompile which involves removing the server-side component.

## Rationale

### Complexity

The BRC20 balance precompile adds an additional layer of complexity to the protocol, making it harder for indexers to integrate BRC2.0 into their systems. It requires indexers to host a server-side component to allow BRC2.0 to read this data, hence creating a circular dependency, as in, BRC2.0 relies on a BRC20 indexer, and the BRC20 indexer relies on BRC2.0 which makes this harder to manage and implement correctly. Removing this precompile will reduce indexing to a one-way interaction, simplifying the integration process for indexers.

### Security

The precompile requires a HTTP API to function correctly and it doesn't support authentication at the moment. This exposure increases the risk of exploitation, as it opens up a potential attack vector for malicious actors. By removing the precompile, we can reduce the attack surface of the protocol and enhance its overall security.

### Maintenance

The precompile requires ongoing maintenance and support from indexers, which can be burdensome and resource-intensive. Removing the precompile will reduce the maintenance overhead for indexers and simplify their operations.

### Performance

The precompile introduces latency in balance queries, as it requires a network request to fetch the balance data. This can lead to slower transaction processing times.

### Redundancy

BRC2.0 allows smart contracts to manage token balances internally. This capability makes the balance precompile redundant, as contracts can handle balance queries without relying on an external precompile.

The intention is for people to deposit their tokens within the module and use them there, BRC20 tokens outside the BRC2.0 context are irrelevant to the module, as they can't be spent within the module.

### Usefulness

BRC20 tokens can move in the same block, so the balance precompile can return stale data. This makes it less useful for contracts that require real-time balance information.

## Conclusion

For these reasons, we propose the removal of the BRC20 balance precompile from the BRC2.0 protocol. This change will simplify the protocol, enhance its security and performance, reduce the maintenance burden for indexers. We believe that this change is in the best interest of the BRC2.0 community and will help to ensure the long-term success of the protocol.