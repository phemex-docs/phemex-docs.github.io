# New Transfer REST API (Universal Transfer REST API)

## Transfer between wallets within an account

> Request format

```
POST /wallets/account/transfer
```

```json
{
  "fromAccType": "SPOT",
  "toAccType": "CONTRACT",
  "amountEv": 100000000,
  "amount": "1",
  "currency": "USDT"
}
```

> Response format

```json
{
  "code": 0,
  "msg": null,
  "data": {
    "amountEv": 10000000,
    "currency": "USDT",
    "status": 10,
    "bizCode": 10,
    "createTime": 1706601733000
  }
}
```

> 1. During the transfer, the `fromAccType` and `toAccType` cannot be the same value.
> 2. The `amountEv` is the scaled-up value of the original amount `amount`. Both values are required and are used to verify the correctness of the amount.

| Field           | Type   | Required | Description               | Possible Values                             |
|-----------------|--------|----------|---------------------------|---------------------------------------------|
| `fromAccType`   | String | True     | Transfer out wallet type  | `SPOT`, `CONTRACT`, `MARGIN`, `COPY_TRADE`  |
| `toAccType`     | String | True     | Transfer in wallet type   | `SPOT`, `CONTRACT`, `MARGIN`, `COPY_TRADE`  |
| `amountEv`      | Long   | True     | AmountEv to transfer      | 100000000                                   |
| `amount`        | String | True     | Origin amount to transfer | 1                                           |
| `currency`      | String | True     | Currency to transfer      | BTC,ETH,USDT ...                            |


## Transfer between main and sub-accounts

> 1. Under main account login, transfers between sub-accounts and the main account or between sub-accounts are supported
> 2. When logged in under a non-main account: Only transfers from the current account to the main account are allowed

> Request format

```
POST /wallets/account/main-sub-transfer
```

```json
{
  "fromUid": 99900001,
  "fromAccType": "SPOT",
  "toUid": 99900002,
  "toAccType": "SPOT",
  "amountEv": 100000000,
  "amount": "1",
  "currency": "USDT"
}
```

> Response format

```json
{
  "code": 0,
  "msg": null,
  "data": {
    "amountEv": 10000000,
    "currency": "USDT",
    "status": 10,
    "bizCode": 3,
    "createTime": 1706601733000
  }
}
```

| Field         | Type        | Required | Description               | Possible Values               |
|---------------|-------------|----------|---------------------------|-------------------------------|
| `fromUid`     | Long        | True     | Uid for transfer out      |                               |
| `fromAccType` | String      | True     | Transfer out wallet type  | `SPOT`, `CONTRACT`, `MARGIN`  |
| `toUid`       | Long        | True     | Uid for transfer in       |                               |
| `toAccType`   | String      | True     | Transfer in wallet type   | `SPOT`, `CONTRACT`, `MARGIN`  |
| `amountEv`    | Long        | True     | AmountEv to transfer      | 100000000                     |
| `amount`      | String      | True     | Origin amount to transfer | 1                             |
| `currency`    | String      | True     | Currency to transfer      | BTC,ETH,USDT ...              |

## Query account wallet transfer history

> Currently, only transfer records from the past 3 months is supported for queries.

```
GET /wallets/account/transfer
```

* Request parameters

> Supports multiple bizTypes parameters separated by ','  for example: bizTypes=3,4

| Field    | Type    | Required | Description                | Possible Values                                                                                                                                                                                                            |
|----------|---------|----------|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| currency | String  | False    | The currency to query      | BTC,ETH ...                                                                                                                                                                                                                |
| bizTypes | Array   | False    | Business type for transfer | `3`: transfer to sub-accounts<br/>`4`: transfer to main account<br/>`10`: Spot transfer to Contract wallet<br/>`11`: Contract transfer to Spot wallet<br/>`90`: Copy trade transfer<br/>`213`: Margin wallet transfer<br/> |
| start    | Long    | False    | Start time in millisecond  | Default to 3 months ago from the end                                                                                                                                                                                       |
| end      | Long    | False    | End time in millisecond    | Default to now                                                                                                                                                                                                             |
| pageNum  | Integer | False    | Page start from 0          | Start from 1, default 1                                                                                                                                                                                                    |
| pageSize | Integer | False    | Page size                  | Default to 20, max 20                                                                                                                                                                                                      |


> Response format

```json
{
  "code": 0,
  "data": {
    "total": 1,
    "rows": [
      {
        "formUserId": 99900001,
        "toUserId": 99900001,
        "fromAccType": "SPOT",
        "toAccType": "CONTRACT",
        "currency": "USDT",
        "amountEv": 100000000,
        "amount": 1,
        "status": 10,
        "bizCode": 3,
        "requestKey": "13adf",
        "requestTime": 1706601733000
      }
    ]
  }
}
```
