# Transfer REST API

## Transfer Between Spot and Futures 

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
| amountEv | Long    | True      | amountEv to transfer  | 100000 ...                                 |
| moveOp   | Integer | True      | direction             | 1 - futures to spot, 2 - spot to futures   |
| currency | String  | True      | currency to transfer  | BTC, ETH, USD ...                          |

## Query Transfer History

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

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |

## Spot Sub To Main Transfer (for sub-account only)

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
| amountEv   | Long    | True     | amountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | unique request key   | Unique request Key, system will generate if its empty |

## Query Spot Sub To Main Transfer

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

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |


## Futures Sub To Main Transfer (for sub-account only)

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
| amountEv   | Long    | True     | amountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | unique request key   | Unique request Key, system will generate if its empty |

## Query Futures Sub To Main Transfer

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

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |

## Universal Transfer (Main account only) - Transfer between sub to main, main to sub or sub to sub

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
| fromUserId  | Long    | True     | from user id                | Will set as main account if not given or 0            |
| toUserId    | Long    | True     | to user id                  | Will set as main account if not given or 0            |
| amountEv    | Long    | True     | amountEv to transfer        | 100000 ...                                            |
| bizType     | String  | True     | transfer for which biz type | SPOT, PERPETUAL                                       |
| requestKey  | String  | False    | unique request key          | Unique request Key, system will generate if its empty |

# Convert REST API

## RFQ Quote

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
| fromCurrency | String | True     | from currency        | BTC,USD ...     |
| toCurrency   | String | True     | to currency          | BTC,USD ...     |
| fromAmountEv | Long   | True     | amountEv to transfer | 100000 ...      |


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
| toCurrency   | String | True     | to currency            | BTC,USD ...     |
| fromCurrency | String | True     | from currency          | BTC,USD ...     |
| fromAmountEv | Long   | False    | amountEv to convert    | 100000 ...      |
| code         | String | True     | encrypted convert args | xxxxxxxx ...    |

## Query Convert History 

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
| fromCurrency | String   | False    | from currency             | BTC,USD ...                          |
| toCurrency   | String   | False    | to currency               | BTC,USD ...                          |
| startTime    | Long     | False    | start time in millisecond | default 2 days ago from the end time |
| endTime      | Long     | False    | end time in millisecond   | default now                          |
| offset       | Integer  | False    | page start from 0         | start from 0, default 0              |
| limit        | Integer  | False    | page size                 | default 20, max 200                  |

