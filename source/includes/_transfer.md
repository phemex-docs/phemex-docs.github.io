# Transfer REST API

## Transfer between spot and futures

> Request format

```
POST /assets/transfer
```

```json
{
  "amountEv": 0,
  "currency": "string",
  "moveOp": 0
}
```

> Response format

```json
{
  "amountEv": 0,
  "currency": "string",
  "linkKey": "string",
  "side": 0,
  "status": 0,
  "userId": 0
}
```

| Field    | Type    | Required  | Description           | Possible Values                            |
|----------|---------|-----------|-----------------------|--------------------------------------------|
| amountEv | Long    | True      | AmountEv to transfer  | 100000 ...                                 |
| moveOp   | Integer | True      | Direction             | 1 - futures to spot, 2 - spot to futures   |
| currency | String  | True      | Currency to transfer  | BTC, ETH, USD ...                          |

## Query transfer history

> Request format

```
GET /assets/transfer?currency=<currency>
```

> Response format

```json
[
  {
    "amountEv": 0,
    "bizType": 0,
    "createTime": 0,
    "currency": "string",
    "linkKey": "string",
    "side": 0,
    "status": 0,
    "userId": 0
  }
]
```

| Field    | Type    | Required | Description               | Possible Values                            |
|----------|---------|----------|---------------------------|--------------------------------------------|
| currency | String  | True     | The currency to query     | BTC,ETH ...                                |
| start    | Long    | False    | Start time in millisecond | Default to 2 days ago from the end         |
| end      | Long    | False    | End time in millisecond   | Default to now                             |
| offset   | Integer | False    | Page start from 0         | Start from 0, default 0                    |
| limit    | Integer | False    | Page size                 | Default to 20, max 200                     |
| side     | Integer | True     | Direction                 | 1 - futures to spot, 2 - spot to futures   |
| bizType  | Integer | True     | Business type             | 11 - futures to spot, 10 - spot to futures |
| status   | Integer | True     | transfer status           | 0 - PENDING, 10 - DONE, 11 - FAILED        |


## Spot sub to main transfer (for sub-account only)

> Request format

```
POST /assets/spots/sub-accounts/transfer
```

```json
{
  "amountEv": 0,
  "currency": "string",
  "requestKey": "string"
}
```

> Response format

```json
{
  "amountEv": 0,
  "currency": "string",
  "fromUserId": 0,
  "requestKey": "string",
  "toUserId": 0
}
```

| Field      | Type    | Required | Description          | Possible Values                                       |
|------------|---------|----------|----------------------|-------------------------------------------------------|
| amountEv   | Long    | True     | AmountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | Currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | Unique request key   | Unique request Key, system will generate if its empty |

## Query transfer history of spot accounts between main and sub

> Request format

```
GET /assets/spots/sub-accounts/transfer?currency=<currency>
```

> Response format

```json
[
  {
    "amountEv": 0,
    "createAt": 0,
    "currency": "string",
    "fromUserId": 0,
    "id": 0,
    "requestKey": "string",
    "status": 0,
    "toUserId": 0
  }
]
```
* Request parameters

| Field    | Type    | Required | Description               | Possible Values                    |
|----------|---------|----------|---------------------------|------------------------------------|
| currency | String  | True     | The currency to query     | BTC,ETH ...                        |
| start    | Long    | False    | Start time in millisecond | Default to 2 days ago from the end |
| end      | Long    | False    | End time in millisecond   | Default to now                     |
| offset   | Integer | False    | Page start from 0         | Start from 0, default 0            |
| limit    | Integer | False    | Page size                 | Default to 20, max 200             |


* Response Fields 

| Field   | Type    | Description                                          |   
|---------|---------|------------------------------------------------------|
| status  | Integer | PENDING(0), DONE(10), FAILED(11)                     |

## Futures sub to main transfer (for sub-account only)

> Request format

```
POST /assets/futures/sub-accounts/transfer
```

```json
{
  "amountEv": 0,
  "currency": "string",
  "requestKey": "string"
}
```

> Response format

```json
{
  "amountEv": 0,
  "bizCode": 0,
  "currency": "string",
  "fromUserId": 0,
  "requestKey": "string",
  "toUserId": 0
}
```

| Field      | Type    | Required | Description          | Possible Values                                       |
|------------|---------|----------|----------------------|-------------------------------------------------------|
| amountEv   | Long    | True     | AmountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | Currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | Unique request key   | Unique request Key, system will generate if its empty |

## Query transfer history of contract accounts between main and sub

> Request format

```
GET /assets/futures/sub-accounts/transfer?currency=<currency>
```

> Response format

```json
[
  {
    "fromUserId": 0,
    "toUserId": 0,
    "currency": "string",
    "amountEv": 0,
    "bizCode": 0,
    "requestKey": "string",
    "createAt": 0
  }
]
```
* Request parameters

| Field    | Type    | Required | Description               | Possible Values                    |
|----------|---------|----------|---------------------------|------------------------------------|
| currency | String  | True     | The currency to query     | BTC,ETH ...                        |
| start    | Long    | False    | Start time in millisecond | Default to 2 days ago from the end |
| end      | Long    | False    | End time in millisecond   | Default to now                     |
| offset   | Integer | False    | Page start from 0         | Start from 0, default 0            |
| limit    | Integer | False    | Page size                 | Default to 20, max 200             |

* Response Fields 

| Field   | Type    | Description                                          |   
|---------|---------|------------------------------------------------------|
| bizCode | Integer | PARENT_TO_SUB_TRANSFER(3), SUB_TO_PARENT_TRANSFER(4) |


## Universal transfer (main account only) - transfer between sub to main, main to sub or sub to sub

> Request format

```
POST /assets/universal-transfer
```

```json
{
  "amountEv": 0,
  "bizType": "string",
  "currency": "string",
  "fromUserId": 0,
  "toUserId": 0
}
```

> Response format

```
{
  "requestKey"
}
```

| Field       | Type    | Required | Description                 | Possible Values                                       |
|-------------|---------|----------|-----------------------------|-------------------------------------------------------|
| fromUserId  | Long    | True     | From user id                | Will set as main account if not given or 0            |
| toUserId    | Long    | True     | To user id                  | Will set as main account if not given or 0            |
| currency    | String  | True     | The currency to query       | BTC,ETH ...                                           |
| amountEv    | Long    | True     | AmountEv to transfer        | 100000 ...                                            |
| bizType     | String  | True     | Transfer for which biz type | SPOT, PERPETUAL                                       |
| requestKey  | String  | False    | Unique request key          | Unique request Key, system will generate if its empty |

# Convert REST API

## RFQ quote

> Request format

```
GET /assets/quote
```

> Response format

```json
{
  "code": "string",
  "quoteArgs": {
    "expireAt": 0,
    "origin": 0,
    "price": "string",
    "proceeds": "string",
    "quoteAt": 0,
    "requestAt": 0,
    "ttlMs": 0
  }
}
```

| Field        | Type   | Required | Description          | Possible Values |
|--------------|--------|----------|----------------------|-----------------|
| fromCurrency | String | True     | From currency        | BTC,USD ...     |
| toCurrency   | String | True     | To currency          | BTC,USD ...     |
| fromAmountEv | Long   | True     | AmountEv to transfer | 100000 ...      |


## Convert

> Request format

```
POST /assets/convert
```

```json
{
  "code": "string",
  "fromAmountEv": 0,
  "fromCurrency": "string",
  "toCurrency": "string"
}
```

> Response format

```json
{
  "fromAmountEv": 0,
  "fromCurrency": "string",
  "linkKey": "string",
  "moveOp": 0,
  "status": 0,
  "toAmountEv": 0,
  "toCurrency": "string"
}
```

| Field        | Type   | Required | Description            | Possible Values |
|--------------|--------|----------|------------------------|-----------------|
| toCurrency   | String | True     | To currency            | BTC,USD ...     |
| fromCurrency | String | True     | From currency          | BTC,USD ...     |
| fromAmountEv | Long   | False    | AmountEv to convert    | 100000 ...      |
| code         | String | True     | Encrypted convert args | xxxxxxxx ...    |

## Query convert history

> Request format

```
GET /assets/convert
```

> Response format

```json
[
  {
    "conversionRate": 0,
    "createTime": 0,
    "errorCode": 0,
    "fromAmountEv": 0,
    "fromCurrency": "string",
    "linkKey": "string",
    "status": 0,
    "toAmountEv": 0,
    "toCurrency": "string"
  }
]
```

| Field        | Type     | Required | Description               | Possible Values                      |
|--------------|----------|----------|---------------------------|--------------------------------------|
| fromCurrency | String   | False    | From currency             | BTC,USD ...                          |
| toCurrency   | String   | False    | To currency               | BTC,USD ...                          |
| startTime    | Long     | False    | Start time in millisecond | Default to 2 days ago from the end time |
| endTime      | Long     | False    | End time in millisecond   | Default to now                          |
| offset       | Integer  | False    | Page start from 0         | Start from 0, default 0              |
| limit        | Integer  | False    | Page size                 | Default to 20, max 200                  |

