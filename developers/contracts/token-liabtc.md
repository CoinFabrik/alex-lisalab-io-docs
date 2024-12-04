# token-liabtc

Implementation of the `LiaBTC` SIP-010 rebasing token, representing staked `aBTC`. The underlying Bitcoin that backs these `aBTC` tokens is staked at Babylon. Tokens are minted when `aBTC` is submitted to the protocol and burned upon redemption.

The sole public feature is transferring `LiaBTC` tokens from a sender to a specified `recipient`. For authorization, the sender must either be the `tx-sender` or the `contract-caller`.

## Rebase Mechanism

The rebasing mechanism is implemented via the "shares" concept. The protocol tracks and stores each user's share of the total reserve. The token balance of a specific principal $p$ is calculated following:

$$
\begin{equation} \textrm{Balance}^{(p)} = \frac{\textrm{Shares}^{(p)}}{\textrm{Total Shares}} \; \cdot \:  \textrm{Reserve} \end{equation}
$$

Where:

- **Reserve** is the total BTC managed by the protocol, composed by
  - BTC staked at Babylon;
  - staking rewards already converted to BTC but not yet staked; and
  - newly deposited BTC but not yet staked.
- **Shares** represent the user's portion of the total reserve. Every time a user deposits `aBTC` to the protocol, it is converted to shares and added to the current user shares amount. Shares increase when users deposit `aBTC` and decrease when they redeem.
- **Total Shares** is the sum of all shares held by all token holders.

The reserve in the contract it is tracked by the `reserve` variable, which is updated externally. Note that token balances of all holders adjust when the reserve changes without an explicit token transfers.

<!-- However, the rebalancing mechanism is transparent from a token holder perspective rather than their balance will change due to the rebalance adjustments. -->

All interface amounts are in token units. Both shares and tokens use the same number of decimals, defined by `token-decimals`.

| Name              | Type   |
| ----------------- | ------ |
| `amount`          | `uint` |
| `message`         | `a`    |
| `signature-packs` | `b`    |

## Features

### Public

#### `transfer`

Transfers `LiaBTC` from the `sender` to the `recipient`. For authorization, the specified `sender` must either be the `tx-sender` or the `contract-caller`. Uses the [`ft-transfer?`](https://docs.stacks.co/reference/functions#ft-transfer) Clarity function and the actual transferred assets are the shares.

##### Parameters

| Name        | Type                   |
| ----------- | ---------------------- |
| `amount`    | `uint`                 |
| `sender`    | `principal`            |
| `recipient` | `principal`            |
| `memo`      | `optional (buff 2048)` |

### Token management

The following functions are guarded by the [`is-dao-or-extension`](#is-dao-or-extension) function. These features are resticted to the LISA DAO or enabled extensions.

#### `set-reserve`

Updates the [`reserve`](#reserve) variable. It is primarly called by the [`liabtc-mint-endpoint`][mint] contract.

##### Parameters

| Name          | Type   |
| ------------- | ------ |
| `new-reserve` | `uint` |

#### `add-reserve`

Increments the reserve.

##### Parameters

| Name        | Type   |
| ----------- | ------ |
| `increment` | `uint` |

#### `remove-reserve`

Decrements the reserve.

##### Parameters

| Name        | Type   |
| ----------- | ------ |
| `decrement` | `uint` |

#### `dao-mint`

Mints `LiaBTC`. This function uses the [`ft-mint?`](https://docs.stacks.co/reference/functions#ft-mint) Clarity native function. The actual minted amount is first converted to shares before minting given the rebasing nature of `LiaBTC`. It is primarly called by the [`liabtc-mint-endpoint`][mint] contract.

##### Parameters

| Name        | Type        |
| ----------- | ----------- |
| `amount`    | `uint`      |
| `recipient` | `principal` |

#### `dao-burn`

Burns `LiaBTC`. This function uses the [`ft-burn?`](https://docs.stacks.co/reference/functions#ft-burn) Clarity native function. The actual burned amount is first converted to shares before burning given the rebasing nature of `LiaBTC`. It is primarly called by the [`liabtc-mint-endpoint`][mint] contract.

##### Parameters

| Name     | Type        |
| -------- | ----------- |
| `amount` | `uint`      |
| `sender` | `principal` |

#### `burn-many`

Performs bulk burning of `LiaBTC`.

##### Parameters

| Name      | Type                                           |
| --------- | ---------------------------------------------- |
| `senders` | `list 200 { amount: uint, sender: principal }` |

### Token governance

The following functions are guarded by the [`is-dao-or-extension`](#is-dao-or-extension) function. These features are resticted to the LISA DAO or enabled extensions.

#### `dao-set-name`

Updates the [`token-name`](#token-name) variable.

##### Parameters

| Name       | Type              |
| ---------- | ----------------- |
| `new-name` | `string-ascii 32` |

#### `dao-set-symbol`

Updates the [`token-symbol`](#token-symbol) variable.

##### Parameters

| Name         | Type              |
| ------------ | ----------------- |
| `new-symbol` | `string-ascii 10` |

#### `dao-set-decimals`

Updates the [`token-decimals`](#token-decimals) variable.

##### Parameters

| Name           | Type   |
| -------------- | ------ |
| `new-decimals` | `uint` |

#### `dao-set-token-uri`

Updates the [`token-uri`](#token-uri) variable.

##### Parameters

| Name      | Type              |
| --------- | ----------------- |
| `new-uri` | `string-utf8 256` |

### Supporting features

#### `is-dao-or-extension`

Standard protocol function to check whether the `contract-caller` is an enabled extension within the DAO or the `tx-sender` is the DAO itself (proposal execution scenario). The enabled extension check is delegated to the LISA's `executor-dao` contract.

#### `get-tokens-to-shares`

TODO:

##### Parameters

| Name     | Type   |
| -------- | ------ |
| `amount` | `uint` |

#### `get-shares-to-tokens`

TODO:

##### Parameters

| Name     | Type   |
| -------- | ------ |
| `shares` | `uint` |

### Getters

#### `get-balance`

Returns the `LiaBTC` balance of a specified principal (`who`). The balance is calculated by retrieving the user's shares and converting them into their equivalent value in tokens. This conversion depends on the current share supply and the total value of the reserve.

##### Parameters

| Name  | Type        |
| ----- | ----------- |
| `who` | `principal` |

#### `get-total-supply`

Returns the total supply of `LiaBTC`, which is the [`reserve`](#reserve).

#### `get-share`

Returns the amount of shares held by a specified principal (`who`). This function uses the [`ft-get-balance`](https://docs.stacks.co/reference/functions#ft-get-balance) Clarity function.

##### Parameters

| Name  | Type        |
| ----- | ----------- |
| `who` | `principal` |

#### `get-total-shares`

Returns the total supply of shares, which is the value returned by the [`ft-get-supply`](https://docs.stacks.co/reference/functions#ft-get-supply) Clarity function.

#### `get-reserve`

Returns the [`reserve`](#reserve) variable.

#### `get-name`

Returns the [`token-name`](#token-name) variable.

#### `get-symbol`

Returns the [`token-symbol`](#token-symbol) variable.

#### `get-token-uri`

Returns the [`token-uri`](#token-uri) variable.

#### `get-decimals`

Returns the [`token-decimals`](#token-decimals) variable.

## Storage

### `reserve`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

Tracks the reserve of `LiaBTC`. It represents the current total value of `aBTC` staked through the [`liabtc-mint-endpoint`][mint], including the restaked rewards.

### `token-name`

| Data     | Type   |
| -------- | ------ |
| Variable | `string-ascii 32` |

Intial value is `"LiaBTC"`.

### `token-symbol`

| Data     | Type   |
| -------- | ------ |
| Variable | `string-ascii 10` |

Intial value is `"LiaBTC"`.

### `token-uri`

| Data     | Type   |
| -------- | ------ |
| Variable | `optional (string-utf8 256)` |

Initial value is `some u"https://cdn.alexlab.co/metadata/token-liabtc.json"`.

### `token-decimals`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

Initial value is `u8`.

## Contract calls

<!-- TODO: LiaBTC DAO will switch to LISA's DAO when going live. -->
- `'SP2XD7417HGPRTREMKF748VNEQPDRR0RMANB7X1NK.executor-dao`: This contract is exclusively called by the [`is-dao-or-extension`](#is-dao-or-extension) function for authorizing governance operations.

## Errors

| Error Name           | Value         |
| -------------------- | ------------- |
| `err-unauthorised`   | `(err u3000)` |
| `err-invalid-amount` | `(err u3001)` |

[mint]: liabtc-mint-endpoint.md
