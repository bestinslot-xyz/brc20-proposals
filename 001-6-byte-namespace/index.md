# BRC-20 - 6-byte Tickers with Limited Charset and Snipe Protection

## Introduction

This proposal introduces a new namespace extension for BRC20 tokens that allows 6-byte tickers. Previously, the standard supported only 4- and 5-byte tickers. The goal is to expand the expressive capacity of BRC20 while maintaining full compatibility with the Programmable Module and future composable tooling.

To ensure predictable behavior, especially in contexts requiring case normalization or cross-protocol matching, we propose restricting valid 6-byte tickers to a constrained character set: uppercase and lowercase English alphanumerics (A-Z, a–z, 0–9) and dash (-) only. The namespace is always treated as case-insensitive.

We're also introducing a snipe protection mechanism to prevent malicious actors from preemptively claiming 6-byte tickers before their deployment.

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

- Symbols or extended Unicode (foo✓ar, foøbar) except for the dash (-)
- Tickers longer than 6 characters

Tickers of 4 or 5 bytes remain governed by existing rules and are not affected.

## Self Mint Rules

Following [self-mint rules from 5-byte tickers](https://github.com/brc20-devs/brc20-proposals/blob/main/bp04-self-mint/proposal.md), BRC20 tokens with 6-byte tickers can be specified in the deploy operation as self-minted tokens.

If the self-mint field is set to `"true"`, it allows everyone to mint the token. If it's unset, or set to any other value other than `"true"`, it defaults to false, meaning anyone can mint the token.

## Snipe Protection via Pre-deploy Inscription

To prevent sniping attacks on 6-byte tickers, we introduce a snipe protection mechanism with a pre-deploy command.

Pre-deployment allows the creator to deploy a double sha256 of the ticker, concatenated with a salt and the pre-deployer's pkscript, without revealing the ticker. This hash is then used to verify the ticker and pre-deployer during the actual deployment. This prevents anyone else from claiming a ticker by simply guessing it and/or frontrunning the deployment on blockchain.

Including the pre-deployer's pkscript in the hash prevents malicious actors from replaying the pre-deploy inscription, as the hash would need to change if the pre-deployer's pkscript changes.

Deploy inscription should be a child of the pre-deploy inscription, so only the owner of the pre-deploy inscription can deploy it.

Deploy inscriptions require a 3-block delay to ensure that the pre-deploy inscription is processed before the deploy inscription, so any deploy inscriptions that are earlier than 3 blocks after the pre-deploy should be rejected by the indexers. This delay allows the network to confirm the pre-deploy inscription and increases the costs of preemptively pre-deploying and deploying a ticker in-between pre-deployment and deployment.

Pre-deployment is done by creating a BRC20 inscription with the following JSON:

```json
{
  "p": "brc-20",
  "op": "predeploy",
  "hash": "sha256(sha256(ticker bytes + salt bytes + pre-deployer pkscript))"
}
```

Then deployment is done by adding a "salt" field to the deploy operation:

```json
{
  "p": "brc-20",
  "op": "deploy",
  "tick": "ordi",
  "max": "21000000",
  "lim": "1000",
  "self_mint": "true | false",
  "salt": "<salt as hex string i.e. 1234567890abcdef>",
}
```

The `salt` field is a hex string (Can only contain 0-9 and A-F) that is used to create a unique hash for the ticker. The `hash` field in the pre-deploy inscription must match the double SHA256 of the ticker concatenated with the salt.

For the pre-deployer with pkscript `"5120fcdc5a7bd66b4d3a8c91f1a1cf94ad7d561f3a304bf18faf5678b1ee47e783b7"`, the ticker `ticker` (`"7469636b6572"`), and the salt `salt` (`"73616c74"`) would result in the following hash:

```plaintext
> sha256(sha256(ticker + salt + pkscript))

e87852f99ef17cae507d1eda1ddd29c2271812e05d9af33abf1e6301eba83618
```

JavaScript function to create a pre-deploy hash:

```js
function predeploy_hash(ticker, salt, deployer_pkscript) { // All parameters are strings
  // Build concatenated buffer
  let predeploy_buffer = Buffer.concat([
      Buffer.from(ticker, 'utf8'),
      Buffer.from(salt, 'hex'),
      Buffer.from(deployer_pkscript, 'hex')
  ])
  
  // Calculate double SHA256
  let first = require('crypto').createHash('sha256').update(predeploy_buffer).digest()
  let second = require('crypto').createHash('sha256').update(first).digest('hex')

  return second
}

let ticker = "ticker"
let salt = "73616c74"
let deployer_pkscript = "5120fcdc5a7bd66b4d3a8c91f1a1cf94ad7d561f3a304bf18faf5678b1ee47e783b7"

console.log(predeploy_hash(ticker, salt, deployer_pkscript))
```

Predeploy inscription would look like this:

```json
{
  "p": "brc-20",
  "op": "predeploy",
  "hash": "e87852f99ef17cae507d1eda1ddd29c2271812e05d9af33abf1e6301eba83618"
}
```

And the deploy inscription would look like this:

```json
{
  "p": "brc-20",
  "op": "deploy",
  "tick": "ticker",
  "salt": "73616c74",
  ...other fields...
}
```

Parent of the deploy inscription must be the pre-deploy inscription, and the deploy inscription must be created at least 3 blocks after the pre-deploy inscription. This allows the indexers to verify that the pre-deploy inscription exists and has been processed before the deploy inscription is created.

## Compatibility
- BRC20 indexers currently reject 6-byte tickers, so they should index them after block height `909969`.
- BRC20 indexers must reject invalid 6-byte tickers during mint, transfer, or interaction.

## Conclusion
6-byte tickers offer a meaningful namespace expansion for BRC20, enabling broader token design and future growth. The restricted charset is a deliberate constraint to preserve compatibility, security, and simplicity, especially as BRC20 expands into programmable and composable territory.

Snipe protection via pre-deploy inscriptions ensures that the namespace remains fair and accessible, preventing malicious actors from front-running legitimate deployments.