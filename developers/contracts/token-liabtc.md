# token-liabtc

## What is LiaBTC?

The `LiaBTC` token is a [SIP-010][sip010] compliant rebasing token that represents staked `aBTC`. The underlying Bitcoin backing these `aBTC` tokens is staked externally.

This particular staking process uses the [XLink Staking Manager][sm] contract to track on-chain status and inform staking actions, the [Babylon](https://babylonlabs.io/) Bitcoin staking platform for execution and [Cobo](https://www.cobo.com/) as the finality provider.

The lifecycle of the `LiaBTC` token invoves minting when `aBTC` is submitted for staking and burning upon unstaking. When users stake `aBTC` via LISA, they receive `LiaBTC` at a 1:1 ratio.

## Rebase mechanism

The rebasing nature of `LiaBTC` is implemented via the "shares" concept. The contract tracks and stores each user's share of an external reserve. By holding shares, users hold a portion of the total reserve. The reserve is tracked by the [`reserve`](#reserve) variable within the contract, which is updated externally, typically by the [`liabtc-mint-endpoint::rebase`][rebasef] function.

The `LiaBTC` balance of a specific user is calculated according to the following equation.

$$
\textrm{User Balance} = \frac{\textrm{User Shares}}{\textrm{Total Shares}} \; \cdot \:  \textrm{Reserve}
$$

Where:

- **Reserve** is the current total value of `aBTC` staked, backed by the `BTC` staked at Babylon and their corresponding staking rewards which were converted to `BTC` and restaked.

- **User Shares** represent the user's portion of the total reserve. Every time a user stakes `aBTC`, the equivalent value in shares is calculated and added to the user's shares balance. Shares increase when users deposit `aBTC` and decrease when they redeem.

- **Total Shares** is the sum of all shares held by all `LiaBTC` token holders.

The above equation can be generalized for an arbitrary amount of shares

$$
\textrm{LiaBTC Tokens} = \frac{\textrm{Shares}}{\textrm{Total Shares}} \; \cdot \:  \textrm{Reserve},
$$

while the token-to-share conversion can also be deduced by the same equation:

$$
\textrm{Shares} = \frac{\textrm{LiaBTC Tokens}}{\textrm{Reserve}} \; \cdot \:  \textrm{Total Shares}.
$$

This is how [`get-shares-to-tokens`](#get-shares-to-tokens) and [`get-tokens-to-shares`](#get-tokens-to-shares) functions work.

## Balances

The `LiaBTC` balance of each user, accessible via the [`get-balance`](#get-balance) function, is automatically adjusted on each rebase, without involving an explicit token transfer. In a scenario where the user do not perform any staking movements, their balance will increase over time as staking rewards are reinjected into the staking protocol.

On the other hand, the shares balance, which can be obtained through the [`get-share`](#get-share) function, accounts for the portion of the total `aBTC` reserve that belongs to the user. Shares behave like a regular fungible token, meaning that balances can only changes with transfers, mints or burns.

Users can freely use `LiaBTC` and its public interface just like any other token. The rebase and share mechanism is transparent from a `LiaBTC` token holder's perspective, with balance adjustments occurring automatically.

## Units

Except for some specific cases, all interface functions' input and output amounts are in `LiaBTC` token units. The exceptions are the following functions:

- [`get-share`](#get-share): returns an amount in shares.
- [`get-shares-to-tokens`](#get-shares-to-tokens): receives shares and returns tokens.
- [`get-tokens-to-shares`](#get-tokens-to-shares): receives tokens and returns shares.

Both shares and tokens use the same number of decimals, defined by [`token-decimals`](#token-decimals).

## Features

### Public

#### `transfer`

Transfers `LiaBTC` from the `sender` to the `recipient`. For authorization, the specified `sender` must either be the `tx-sender` or the `contract-caller`. Uses the [`ft-transfer?`](https://docs.stacks.co/reference/functions#ft-transfer) Clarity function, where the actual transferred assets are shares.

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

Converts a specified `LiaBTC` `amount` into its equivalent shares representation.

##### Parameters

| Name     | Type   |
| -------- | ------ |
| `amount` | `uint` |

#### `get-shares-to-tokens`

Converts a specified amount of `shares` into its equivalent value in `LiaBTC` tokens.

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

| Data     | Type              |
| -------- | ----------------- |
| Variable | `string-ascii 32` |

Intial value is `"LiaBTC"`.

### `token-symbol`

| Data     | Type              |
| -------- | ----------------- |
| Variable | `string-ascii 10` |

Intial value is `"LiaBTC"`.

### `token-uri`

| Data     | Type                         |
| -------- | ---------------------------- |
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
[rebasef]: liabtc-mint-endpoint.md#rebase-1
[sip010]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[sm]: xlink-staking.md
