# token-vliabtc

- Location: `xlink-dao/contracts/liabtc/token-vliabtc.clar`
- [Deployed contract](https://explorer.stxer.xyz/txid/SP673Z4BPB4R73359K9HE55F2X91V5BJTN5SXZ5T.token-vliabtc)

## What is vLiaBTC?

The `vLiaBTC` token is a [SIP-010][sip010] compliant value-accruing token, designed as a non-rebasing wrapper for `LiaBTC`.

This token is mainly used as a layer of compatibility to integrate `LiaBTC` with other DeFi protocols[^1] since the total supply and wallet balances remain constant over time. However, its value in terms of `LiaBTC` does change over time.

Users can wrap their `LiaBTC` into `vLiaBTC` to maintain the same value in a non-rebasing format. Upon unwrapping, users receive the same amount of `LiaBTC` tokens as they would have if they had held the liquid token (`LiaBTC`) throughout.

## How it works?

The `token-vliabtc` contract can be used as a trustless wrapper that accepts `LiaBTC` tokens and mints `vLiaBTC` in return. The contract locks the wrapped `LiaBTC` in its own balance. When the user unwraps, the contract burns the user's `vLiaBTC` and sends the corresponding `LiaBTC` in return.

Internally, the `vLiaBTC` balance represents the user's share of the total `LiaBTC` held by the `token-vliabtc` contract (vaulted `LiaBTC`). This means that for a general user that holds `vLiaBTC`, the value in `LiaBTC` is given by the following equation:

$$
\textrm{LiaBTC value} = \frac{\textrm{vLiaBTC Balance}}{\textrm{vLiaBTC Total Supply}} \; \cdot \; \textrm{Vaulted LiaBTC}
$$

Unlike some other DeFi protocols, the amount of `vLiaBTC` minted when wrapping does not directly correspond to the `LiaBTC` shares exchanged. However, both approaches are equivalent and achieve the same functional outcome. For a detailed explanation of this equivalence, see the [Appendix](#appendix).

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

#### `mint`

Mints `vLiaBTC` for the specified `recipient`. The `recipient` must be either the `tx-sender` or the `contract-caller`. The amount of `vLiaBTC` minted is calculated by applying [`get-tokens-to-share`](#get-tokens-to-shares) to the provided `amount` before transferring the `LiaBTC` from the user to the contract.

##### Parameters

| Name        | Type        |
| ----------- | ----------- |
| `amount`    | `uint`      |
| `recipient` | `principal` |

#### `burn`

Burns `vLiaBTC` for the specified `sender`. The `sender` must be either the `tx-sender` or the `contract-caller`. The amount of `LiaBTC` to be transferred back to the user is calculated via the `get-shares-to-tokens` function before performing the burn.

##### Parameters

| Name     | Type        |
| -------- | ----------- |
| `amount` | `uint`      |
| `sender` | `principal` |

### Token governance

The following functions are guarded by the [`is-dao-or-extension`](#is-dao-or-extension) function. These features are resticted to the LISA DAO or enabled extensions.

#### `set-name`

Updates the [`token-name`](#token-name) variable.

##### Parameters

| Name       | Type              |
| ---------- | ----------------- |
| `new-name` | `string-ascii 32` |

#### `set-symbol`

Updates the [`token-symbol`](#token-symbol) variable.

##### Parameters

| Name         | Type              |
| ------------ | ----------------- |
| `new-symbol` | `string-ascii 10` |

#### `set-decimals`

Updates the [`token-decimals`](#token-decimals) variable.

##### Parameters

| Name           | Type   |
| -------------- | ------ |
| `new-decimals` | `uint` |

#### `set-token-uri`

Updates the [`token-uri`](#token-uri) variable.

##### Parameters

| Name      | Type              |
| --------- | ----------------- |
| `new-uri` | `string-utf8 256` |

### Supporting features

#### `is-dao-or-extension`

Standard protocol function to check whether the `contract-caller` is an enabled extension within the DAO or the `tx-sender` is the DAO itself (proposal execution scenario). The enabled extension check is delegated to the LISA's `executor-dao` contract.

#### `get-tokens-to-shares`

Converts a specified `LiaBTC` `amount` into its equivalent value in `vLiaBTC`. The term shares is utilized because `vLiaBTC` amounts represent a share of the total vaulted `LiaBTC`.

##### Parameters

| Name     | Type   |
| -------- | ------ |
| `amount` | `uint` |

#### `get-shares-to-tokens`

Converts a specified amount of `vLiaBTC` (referred to as the `shares` parameter) into its equivalent value in `LiaBTC`. This function is called within the [`burn`](#burn) function to determine the amount of `LiaBTC` to transfer to the user. The term shares is utilized because `vLiaBTC` amounts represent a share of the total vaulted `LiaBTC`.

##### Parameters

| Name     | Type   |
| -------- | ------ |
| `shares` | `uint` |

### Getters

#### `get-balance`

Returns the `vLiaBTC` balance of a specified principal (`who`) using the [`ft-get-balance`](https://docs.stacks.co/reference/functions#ft-get-balance) Clarity function.

##### Parameters

| Name  | Type        |
| ----- | ----------- |
| `who` | `principal` |

#### `get-total-supply`

Returns the total supply of `vLiaBTC` using the [`ft-get-supply`](https://docs.stacks.co/reference/functions#ft-get-supply) Clarity function.

#### `get-share`

<!-- This name utilizes "shares" term in a different manner than the tokens-shares conversion functions. -->

Returns the value in `LiaBTC` of a user's total balance of `vLiaBTC`. It is calculated by retrieveing the `LiaBTC` amount and converting it to tokens with the [`get-shares-to-tokens`](#get-shares-to-tokens).

##### Parameters

| Name  | Type        |
| ----- | ----------- |
| `who` | `principal` |

#### `get-total-shares`

<!-- This name utilizes "shares" term in a different manner than the tokens-shares conversion functions. -->

Returns the vaulted `LiaBTC` balance. This is the `LiaBTC` held by the `token-vliabtc` contract, which are the tokens from the users that currently hold the wrapped token.

#### `get-name`

Returns the [`token-name`](#token-name) variable.

#### `get-symbol`

Returns the [`token-symbol`](#token-symbol) variable.

#### `get-token-uri`

Returns the [`token-uri`](#token-uri) variable.

#### `get-decimals`

Returns the [`token-decimals`](#token-decimals) variable.

## Storage

### `token-name`

| Data     | Type              |
| -------- | ----------------- |
| Variable | `string-ascii 32` |

Intial value is `"vLiaBTC"`.

### `token-symbol`

| Data     | Type              |
| -------- | ----------------- |
| Variable | `string-ascii 10` |

Intial value is `"vLiaBTC"`.

### `token-uri`

| Data     | Type                         |
| -------- | ---------------------------- |
| Variable | `optional (string-utf8 256)` |

Initial value is `some u"https://cdn.alexlab.co/metadata/vtoken-liabtc.json"`.

### `token-decimals`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

Initial value is `u8`.

## Contract calls

- [`token-liabtc`][liabtc]: Core external calls are made to perform `LiaBTC` transfers when wrapping/unwrapping and to retrieve the balance of `LiaBTC` held by the `token-vliabtc` contract (vaulted `LiaBTC`).

- `'SM26NBC8SFHNW4P1Y4DFH27974P56WN86C92HPEHH.lisa-dao`: This contract is exclusively called by the [`is-dao-or-extension`](#is-dao-or-extension) function for authorizing governance operations.

## Errors

| Error Name         | Value         |
| ------------------ | ------------- |
| `err-unauthorised` | `(err u3000)` |

## Appendix

The amount of `vLiaBTC` minted during wrapping does not directly correspond to the `LiaBTC` shares exchanged. In some other DeFi protocols, these two values are exactly the same. Why are these different approaches to wrapping equivalent?

To understand this, let's revisit the equation from the [How it works?](#how-it-works) section, pasted down here to facilitate the explanation:

$$
\begin{equation} \textrm{LiaBTC value} = \frac{\textrm{vLiaBTC Balance}}{\textrm{vLiaBTC Total Supply}} \; \cdot \; \textrm{Vaulted LiaBTC}. \end{equation}
$$

This equation shows the **LiaBTC value** for a general user holding a specific **vLiaBTC Balance**.

Now, consider the relationship between `LiaBTC` shares and tokens, given by [equation (1)][eq1] of the `token-liabtc` document. Using that equation, the **Vaulted LiaBTC** can be expressed as:

$$
\textrm{Vaulted LiaBTC} = \frac{\textrm{Vaulted Shares}}{\textrm{Total Shares}} \; \cdot \:  \textrm{Reserve},
$$

where the **Vaulted Shares** are the `LiaBTC` shares representation of the vaulted `LiaBTC`. Similarly, we can think of the **LiaBTC value** expressed as a fraction of the **Reserve** using a hypothetical shares term:

$$
\textrm{LiaBTC value} = \frac{\textrm{Shares}_h}{\textrm{Total Shares}} \; \cdot \:  \textrm{Reserve}.
$$

Assuming the **LiaBTC value** remain unchanged since the user wrapped their tokens, the hypothetical shares _are_ the shares representation of the `LiaBTC` amount that the user wrapped. So, from now on, we will refer them as **User Shares**, the `LiaBTC` shares that the user exchanged during the wrap operation. Replacing the expressions for **Vaulted LiaBTC** and **LiaBTC value** into equation (1) and cancelling the **Reserve** factor from both sides, we get:

$$
\frac{\textrm{User Shares}}{\textrm{Total Shares}} = \frac{\textrm{vLiaBTC Balance}}{\textrm{vLiaBTC Total Supply}} \; \cdot \; \frac{\textrm{Vaulted Shares}}{\textrm{Total Shares}}.
$$

After cancelling **Total Shares** on both sides, we obtain

$$
\begin{equation} \textrm{User Shares} = \frac{\textrm{vLiaBTC Balance}}{\textrm{vLiaBTC Total Supply}} \; \cdot \; \textrm{Vaulted Shares} \end{equation}
$$

as the general equation that relates the `LiaBTC` shares (representing the `LiaBTC` amount that the user initially wrapped) and the corresponding `vLiaBTC` balance.

In many DeFi protocols, the amount of wrapped tokens minted equals the rebasing token's shares exchanged. In such cases, the **Vaulted Shares** equal the total circulating supply of the wrapped token (**vLiaBTC Total Supply**), simplifying the equation to:

$$
\textrm{User Shares} = \textrm{vLiaBTC Balance}.
$$

This scenario is a specific case of equation (2), showing that both approaches are functionally equivalent.

[sip010]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[liabtc]: token-liabtc.md
[eq1]: token-liabtc.md#rebase-mechanism

[^1]: Note that [`get-balance`](#get-balance) and [`get-total-supply`](#get-total-supply) functions are implemented like standard Clarity fungible tokens.
