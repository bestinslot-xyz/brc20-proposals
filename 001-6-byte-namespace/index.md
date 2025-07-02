# BRC-20 - 6-byte Tickers with Limited Charset

## Introduction

This proposal introduces a new namespace extension for BRC20 tokens that allows 6-byte tickers. Previously, the standard supported only 4- and 5-byte tickers. The goal is to expand the expressive capacity of BRC20 while maintaining full compatibility with the Programmable Module and future composable tooling.

To ensure predictable behavior, especially in contexts requiring case normalization or cross-protocol matching, we propose restricting valid 6-byte tickers to a constrained character set: uppercase and lowercase English alphanumerics (A-Z, a–z, 0–9) and dash (-) only. The namespace is always treated as case-insensitive.

## Rationale

Expanding to 6 bytes without affecting 4 and 5 bytes increases combinatorial capacity without breaking existing semantics or bloating the protocol.

The Programmable Module is the first consumer of this 6-byte namespace, and limiting the launch 6-byte initially will allow us to avoid affecting existing BRC20 namespaces.

Enforcing a strict subset of ASCII characters ensures cross-system compatibility and simplifies downstream parsing.

## New Limitation

6-byte tickers must:

- Be exactly 6 characters long
- Match the regex: ^[A-Za-z0-9-]{6}$
- Be treated case-insensitively
 
Rejected inputs include:

- Symbols or extended Unicode (foo✓ar, foøbar)
- Tickers longer than 6 characters

Tickers of 4 or 5 bytes remain governed by existing rules and are not affected.

## Self Mint Rules

Following [self-mint rules from 5-byte tickers](https://github.com/brc20-devs/brc20-proposals/blob/main/bp04-self-mint/proposal.md), BRC20 tokens with 6-byte tickers can be specified in the deploy operation as self-minted tokens.

If the self-mint field is set to false, it allows everyone to mint the token.

## Compatibility
- BRC20 indexers currently reject 6-byte tickers, so they should index them after block height `909969`.
- BRC20 indexers must reject invalid 6-byte tickers during mint, transfer, or interaction.

## Conclusion
6-byte tickers offer a meaningful namespace expansion for BRC20, enabling broader token design and future growth. The restricted charset is a deliberate constraint to preserve compatibility, security, and simplicity, especially as BRC20 expands into programmable and composable territory.