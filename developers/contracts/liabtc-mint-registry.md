# liabtc-mint-registry

- Location: `xlink-dao/contracts/liabtc/liabtc-mint-registry.clar`
<!-- - [Deployed contract](link-to-explorer) -->

The `liabtc-mint-registry` is the data and treasury counterpart to the [`liabtc-mint-endpoint`][mint] contract. It manages the storage of burn requests data and serves as the `aBTC` treasury during burn operations.

Although it is primarly designed to be called by the `liabtc-mint-endpoint`, this registry can function as a general vault accesible to governance roles (LISA DAO or enabled extensions).

What actions can this contract do?

- Create new burn requests, each tracked using a unique nonce.
- Update any field of a burn request, including its status.
- Transfer a specified token[^1] from its balance to a designated recipient. The caller specifies the token, recipient and transfer amount.

## Features

All features are resticted to the LISA DAO or enabled extensions.

### Governance

The following functions are guarded by the [`is-dao-or-extension`](#is-dao-or-extension) function. These features are resticted to the LISA DAO or enabled extensions.

#### `set-burn-request`

Creates or modifies burn requests in the [`burn-requests`](#burn-requests) map as specified in the `details` parameter. It is called by the functions [`request-burn`](liabtc-mint-endpoint.md#request-burn), [`revoke-burn`](liabtc-mint-endpoint.md#revoke-burn) and [`finalize-burn`](liabtc-mint-endpoint.md#finalize-burn) in the [`liabtc-mint-endpoint`][mint] contract.

New requests are created by passing `u0` as the `request-id` parameter. On each request creation, the [`burn-request-nonce`](#burn-request-nonce) variable is incremented by one, resulting in the id of the new request.

##### Parameters

| Name         | Type                                                                              |
| ------------ | --------------------------------------------------------------------------------- |
| `request-id` | `uint`                                                                            |
| `details`    | `{ requested-by: principal, amount: uint, requested-at: uint, status: (buff 1) }` |

#### `transfer`

Calls the `transfer` function of the `token-trait` passed with `as-contract` privilege. The caller has the ability to send tokens from the registry's balance to a designated `recipient`.

##### Parameters

| Name        | Type        |
| ----------- | ----------- |
| `amount`    | `uint`      |
| `recipient` | `principal` |
| `token-trait` | `<sip-010-trait>` |

### Supporting features

#### `is-dao-or-extension`

Standard protocol function to check whether the `contract-caller` is an enabled extension within the DAO or the `tx-sender` is the DAO itself (proposal execution scenario). The enabled extension check is delegated to the LISA's `executor-dao` contract.

### Getters

#### `get-burn-request-nonce`

Returns the [`burn-request-nonce`](#burn-request-nonce) variable.

#### `get-burn-request-or-fail`

Returns the burn request on the [`burn-requests`](#burn-requests) map at a given key. If there is no entry for the provided `request-id`, throws.

#### Parameterss

| Name         | Type   |
| ------------ | ------ |
| `request-id` | `uint` |

## Storage

### `burn-request-nonce`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

Indicates the `request-id` of the last burn request created. This variable can only monotonically increase. It initialized as `u0`, the key that will always have an empty value. 

### `burn-requests`

| Data | Type                                                                                   |
| ---- | -------------------------------------------------------------------------------------- |
| Map  | `uint { requested-by: principal, amount: uint, requested-at: uint, status: (buff 1) }` |

Map that stores the burn requests used by the [`liabtc-mint-endpoint`][mint].

### Relevant constants

#### `PENDING`

| Type     | Value  |
| -------- | ------ |
| `buff 1` | `0x00` |

Burn request pending status. When created, burn requests start with this status.

#### `FINALIZED`

| Type     | Value  |
| -------- | ------ |
| `buff 1` | `0x01` |

Burn request finalize status.

#### `REVOKED`

| Type     | Value  |
| -------- | ------ |
| `buff 1` | `0x02` |

Burn request revoked status.

## Contract calls

- `<sip-010-trait>`: Interaction with potentially any contract implementing the [official SIP-010][sip010] occurs when the [`transfer`](#transfer) function is called.

<!-- TODO: LiaBTC DAO will switch to LISA's DAO when going live. -->
- `'SP2XD7417HGPRTREMKF748VNEQPDRR0RMANB7X1NK.executor-dao`: This contract is exclusively called by the [`is-dao-or-extension`](#is-dao-or-extension) function for authorizing governance operations.

## Errors

| Error Name               | Value         |
| ------------------------ | ------------- |
| `err-unauthorised`       | `(err u1000)` |
| `err-unknown-request-id` | `(err u1008)` |

[mint]: liabtc-mint-endpoint.md
[sip010]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^1]: The token just needs to comply with the [official SIP-010][sip010].
