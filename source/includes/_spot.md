# Spot REST API

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case sensitive**.

## Query product information

> Request

```
GET /public/products
```

* Spot symbols are defined in `.products[]` with **type=Spot**.
* Spot currencies are defined in `.currencies[]`.

## Query product information plus

> Request

```
GET /public/products-plus
```

* Spot symbols are defined in `.products[]` with **type=Spot**.
  * `list time` is defined in timeline[1].
  * `delist time` is defined in timeline[3].
* Spot currencies are defined in `.currencies[]`.

## Query server time

> Request

```
GET /public/time
```

* return server time

> Response sample

```json
{
    "code": 0,
    "msg": "",
    "data": {
        "serverTime": 1676172826345
    }
} 
```


## Price/Ratio/Value scales

Fields with post-fix "Ep", "Er" or "Ev" have been scaled based on symbol setting.

* Fields with post-fix "Ep" are scaled prices, `priceScale` in [products](#query-product-information)
* Fields with post-fix "Er" are scaled ratios, `ratioScale` in [products](#query-product-information)
* Fields with post-fix "Ev" are scaled values, `valueScale` of `settleCurrency` in [products](#query-product-information)

**NOTE**:<br>
1\) `ratioScale` is always scaled 8.<br>
2\) `priceScale` follows the `valueScale` of quote-currency, and must follow quote-ticksize criteria.<br> 
  e.g. `priceScale` of sBTCUSDT follows USDT-valueScale, i.e. 1e8; the scaled price of sBTCUSDT must be multiple times of `quoteTickSizeEv=1e6`. <br>
3\) `qtySale` follows the valueScale of base-currency, and must follow base-tickSize criteria.<br> 
  e.g. `qtyScale` of sETHUSDT follows ETH-valueScale, i.e. 1e8; the scaled qty of sETHUSDT must be mulitple times of  `baseTickSizeEv=10000`.<br>


## Common order fields

* Order type

| Order type | Description |
|-----------|-------------|
| Limit | -- |
| Market | -- |
| Stop | -- |
| StopLimit | -- |
| MarketIfTouched | -- |
| LimitIfTouched | -- |
| MarketAsLimit | -- |
| StopAsLimit | -- |
| MarketIfTouchedAsLimit | -- |

* Order status

| Order status | Description | 
|------------|-------------|
| Untriggered | Conditional order waiting to be triggered |
| Triggered | Conditional order being triggered|
| Rejected | Order rejected |
| New | Order placed in cross engine |
| PartiallyFilled | Order partially filled |
| Filled | Order fully filled |
| Canceled | Order canceled |

* TimeInForce

| TimeInForce | Description |
|------------|-------------|
| GoodTillCancel | -- |
| PostOnly | -- |
| ImmediateOrCancel | -- |
| FillOrKill | -- |

* Trigger source

| Trigger | Description |
|------------|-------------|
| ByLastPrice | Trigger by last price |

## Place order (HTTP PUT, *prefered*)

> Request format

```
PUT /spot/orders/create?symbol=<symbol>&trigger=<trigger>&clOrdID=<clOrdID>&priceEp=<priceEp>&baseQtyEv=<baseQtyEv>&quoteQtyEv=<quoteQtyEv>&stopPxEp=<stopPxEp>&text=<text>&side=<side>&qtyType=<qtyType>&ordType=<ordType>&timeInForce=<timeInForce>&execInst=<execInst>
```

> Response format

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "orderID": "d1d09454-cabc-4a23-89a7-59d43363f16d",
    "clOrdID": "309bcd5c-9f6e-4a68-b775-4494542eb5cb",
    "priceEp": 0,
    "action": "New",
    "trigger": "UNSPECIFIED",
    "pegPriceType": "UNSPECIFIED",
    "stopDirection": "UNSPECIFIED",
    "bizError": 0,
    "symbol": "sBTCUSDT",
    "side": "Buy",
    "baseQtyEv": 0,
    "ordType": "Limit",
    "timeInForce": "GoodTillCancel",
    "ordStatus": "Created",
    "cumFeeEv": 0,
    "cumBaseQtyEv": 0,
    "cumQuoteQtyEv": 0,
    "leavesBaseQtyEv": 0,
    "leavesQuoteQtyEv": 0,
    "avgPriceEp": 0,
    "cumBaseAmountEv": 0,
    "cumQuoteAmountEv": 0,
    "quoteQtyEv": 0,
    "qtyType": "ByBase",
    "stopPxEp": 0,
    "pegOffsetValueEp": 0
  }
}
```

| Field       | Type   | Required | Description               | Possible values |
|----------   |--------|----------|---------------------------|-----------------|
| symbol      | String | Yes      |                           | [Spot Symbols](#spot_symbols)     |
| side        | Enum   | Yes      |                           |  Sell, Buy     | 
| qtyType     | Enum   | Yes      | Set order quantity by base or quote currency | ByBase, ByQuote|
| quoteQtyEv  | Integer| --       | Required if qtyType = ByQuote|  |
| baseQtyEv   | Integer| --       |                           | Required if qtyType = ByBase   |
| priceEp     | Integer|          |                           | Scaled price            |
| stopPxEp    | Integer| --       | used in conditionalorder  |   |
| trigger     | Enum   | --       | Required in conditional order | ByLastPrice |
| timeInForce | Enum   | No       | Default GoodTillCancel    | GoodTillCancel, PostOnly,ImmediateOrCancel,FillOrKill |
| ordType     | Enum   | No       | Default to Limit          | Market, Limit, Stop, StopLimit, MarketIfTouched, LimitIfTouched|

## Place order (HTTP POST)

> Request format

```
POST /spot/orders
```

```json
{
  "symbol": "sBTCUSDT",
  "clOrdID": "",
  "side": "Buy/Sell",
  "qtyType": "ByBase/ByQuote",
  "quoteQtyEv": 0,
  "baseQtyEv": 0,
  "priceEp": 0,
  "stopPxEp": 0,
  "trigger": "UNSPECIFIED",
  "ordType": "Limit",
  "timeInForce": "GoodTillCancel"
}
```

## Amend order

> Request format

```
PUT /spot/orders?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&priceEp=<priceEp>&baseQtyEv=<baseQtyEv>&quoteQtyEv=<quoteQtyEv>&stopPxEp=<stopPxEp> 
```
| Field       | Type | Required | Description                           |
|-------------|------|----------|---------------------------------------|
| symbol      |      | Yes      | order symbol, cannot be changed       |
| orderID     |      | -        | order id, cannot be changed           |
| origClOrdID |      | -        | origClOrdID , cannot be changed       |
| priceEp     |      | -        | scaled price                          |
| baseQtyEv   |      | Yes      | scaled base-currency quantity         |
| quoteQtyEv  |      | Yes      | scaled quote-currency quantity        |
| stopPxEp    |      | Yes      | used in conditionalorder              |

1\) orderID and origClOrdID can't both be empty
2\) The quantity to be changed must be the same currency as placing order. e.g. If placing order is by baseQtyEv, amending order can be only by baseQtyEv as.

## Cancel order

> Request format

```
DELETE /spot/orders?symbol=<symbol>&orderID=<orderID>
DELETE /spot/orders?symbol=<symbol>&clOrdID=<clOrdID>
```

## Cancel all order by symbol

> Request format

```
DELETE /spot/orders/all?symbol=<symbol>&untriggered=<untriggered>
```

| Field | Type | Required | Description |
|---------|--------|-------|-------------|
| symbol  | Enum   | Yes   | The symbol to cancel |
| untriggered | Boolean | No | set false to cancel non-conditiaonal order, true to conditional order |

## Query open order by order ID or client order ID

> Request format

```
GET /spot/orders/active?symbol=<symbol>&orderID=<orderID>
GET /spot/orders/active?symbol=<symbol>&clOrDID=<clOrdID>
```

## Query all open orders by symbol

> Request format

```
GET /spot/orders?symbol=<symbol>
```

## Query wallets

| Parameter | Type   | Required | Description               | Case         |
|-----------|--------|----------|---------------------------|--------------|
| currency  | String | No       | the currency to query     | BTC, USDT... |

NOTE: GET /spot/wallets queries the wallets of all currencies

> Request format

```
GET /spot/wallets?currency=<currency>
```

> Response format

```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "currency": "BTC",
      "balanceEv": 0,
      "lockedTradingBalanceEv": 0,
      "lockedWithdrawEv": 0,
      "lastUpdateTimeNs": 0
    }
  ]
}
```

## Query orders by order ID or client order ID

> Request format

```
GET /api-data/spots/orders/by-order-id?symbol=<symbol>&oderId=<orderID>&clOrdID=<clOrdID>
```

| Field    | Type   | Required | Description           | Possible Values                                                                                                                           |
|----------|--------|----------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| symbol   | String | True     | the trade symbol to query | sBTCUSDT ...                                                                                                                              |
| orderID  | String | False    | Order id              | Either orderID or clOrdID is required. |
| clOrdID  | String | False    | Client order id       | Refer to orderID                                                                                                                          |

> Response format

```json
[
  {
    "avgPriceEp": 0,
    "avgTransactPriceEp": 0,
    "baseQtyEv": "string",
    "createTimeNs": 0,
    "cumBaseValueEv": 0,
    "cumFeeEv": 0,
    "cumQuoteValueEv": 0,
    "execStatus": "string",
    "feeCurrency": "string",
    "leavesBaseQtyEv": 0,
    "leavesQuoteQtyEv": 0,
    "ordStatus": "string",
    "ordType": "string",
    "orderID": "string",
    "priceEp": 0,
    "qtyType": "string",
    "quoteQtyEv": 0,
    "side": "string",
    "stopDirection": "string",
    "stopPxEp": 0,
    "symbol": "string",
    "timeInForce": "string"
  }
]
```

## Query order history

> Request format

```
GET /api-data/spots/orders?symbol=<symbol>
```

> Response format

```json
[
  {
    "avgPriceEp": 0,
    "avgTransactPriceEp": 0,
    "baseQtyEv": "string",
    "createTimeNs": 0,
    "cumBaseValueEv": 0,
    "cumFeeEv": 0,
    "cumQuoteValueEv": 0,
    "execStatus": "string",
    "feeCurrency": "string",
    "leavesBaseQtyEv": 0,
    "leavesQuoteQtyEv": 0,
    "ordStatus": "string",
    "ordType": "string",
    "orderID": "string",
    "priceEp": 0,
    "qtyType": "string",
    "quoteQtyEv": 0,
    "side": "string",
    "stopDirection": "string",
    "stopPxEp": 0,
    "symbol": "string",
    "timeInForce": "string"
  }
]
```

| Field     | Type    | Required | Description               | Possible Values                 |
|-----------|---------|----------|---------------------------|---------------------------------|
| symbol    | String  | True     | The trade symbol to query     | sBTCUSDT ...                    |
| start     | Integer | False    | Start time in millisecond | Default to 2 days before the end time |
| end       | Integer | False    | End time in millisecond   | Default to now                     |
| offset    | Integer | False    | Page start from 0         | Start from 0, default 0         |
| limit     | Integer | False    | Page size                 | Default to 20, max 200             |

## Query trade history

> Request format

```
GET /api-data/spots/trades?symbol=<symbol>
```

> Response format

```json
[
  {
    "action": "string",
    "baseCurrency": "string",
    "baseQtyEv": 0,
    "clOrdID": "string",
    "execBaseQtyEv": 0,
    "execFeeEv": 0,
    "execId": "string",
    "execInst": "string",
    "execPriceEp": 0,
    "execQuoteQtyEv": 0,
    "execStatus": "string",
    "feeCurrency": "string",
    "feeRateEr": 0,
    "leavesBaseQtyEv": 0,
    "leavesQuoteQtyEv": 0,
    "ordStatus": "string",
    "ordType": "string",
    "orderID": "string",
    "priceEP": 0,
    "qtyType": "string",
    "quoteCurrency": "string",
    "quoteQtyEv": 0,
    "side": "string",
    "stopDirection": "string",
    "stopPxEp": 0,
    "symbol": "string",
    "timeInForce": "string",
    "tradeType": "string",
    "transactTimeNs": 0
  }
]
```

| Field     | Type    | Required | Description               | Possible Values                 |
|-----------|---------|----------|---------------------------|---------------------------------|
| symbol    | String  | True     | The currency to query     | sBTCUSDT ...                    |
| start     | Integer | False    | Start time in millisecond | Default to 2 days before the end time |
| end       | Integer | False    | End time in millisecond   | Default to now                  |
| offset    | Integer | False    | Page start from 0         | Start from 0, default 0         |
| limit     | Integer | False    | Page size                 | Default 20, max 200             |

## Query PnL

> Request format

```
GET /api-data/spots/pnls
```

> Response format

```json
[
  {
    "collectTime": 0,
    "cumPnlEv": 0,
    "dailyPnlEv": 0,
    "userId": 0
  }
]
```

| Field    | Type   | Required | Description           | Possible Values                                                                                                                           |
|----------|--------|----------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| start     | Integer | False    | Start time in millisecond | Default to 2 days before the end time |
| end       | Integer | False    | End time in millisecond   | Default to now                     |

## Query chain information

> Request format

```
GET /exchange/public/cfg/chain-settings?currency=<currency>
```
## Query deposit address by currency

> Request format

```
GET /exchange/wallets/v2/depositAddress?currency=<currency>&chainName=<chainName>
```

> Response format

```json
{
  "address": "1Cdxxxxxxxxxxxxxx",
  "tag": null
}
```

* `chainName` shall be queried from [chain information](#query-chain-information).

| Field    | Type   | Required  | Description| Possible Values |
|----------|--------|-----------|------------|-----------------|
| currency | String | True      | the currency to query | BTC,ETH, USDT ... |
| chainName| String | True      | the chain for this currency | BTC, ETH, EOS |

## Query recent deposit history

> Request format

```
GET /exchange/wallets/depositList?currency=<currency>&offset=<offset>&limit=<limit>
```

> Response format

```json
{
  "address": "1xxxxxxxxxxxxxxxxxx",
  "amountEv": 1000000,
  "confirmations": 1,
  "createdAt": 1574685871000,
  "currency": "BTC",
  "currencyCode": 1,
  "status": "Success",
  "txHash": "9e84xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "type": "Deposit"
}
```

<aside class="notice">
Response data is limited to 3 months.
</aside>

| Field    | Type   | Required  | Description| Possible Values |
|----------|--------|-----------|------------|-----------------|
| currency | String | True      | the currency to query | BTC,ETH, ... |

## Query recent withdraw history

> Request format

```
GET /exchange/wallets/withdrawList?currency=<currency>&offset=<offset>&limit=<limit>
```

> Response format

```json
{
  "address": "1Lxxxxxxxxxxx",
  "amountEv": 200000,
  "currency": "BTC",
  "currencyCode": 1,
  "expiredTime": 0,
  "feeEv": 50000,
  "rejectReason": null,
  "status": "Succeed",
  "txHash": "44exxxxxxxxxxxxxxxxxxxxxx",
  "withdrawStatus": ""
}
```

<aside class="notice">
Response data is limited to 3 months.
</aside>

| Field    | Type   | Required  | Description| Possible Values |
|----------|--------|-----------|------------|-----------------|
| currency | String | True      | the currency to query | BTC,ETH, ... |

## Query funds history

> Request format

```
GET /api-data/spots/funds?currency=<currency>
```

> Response format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "rows":[{
      "id": 0,
      "currency": "string",
      "execId": "string",
      "amountEv": 0,
      "feeEv": 0,
      "side": "string",
      "action": "string",
      "balanceEv": 0,
      "bizCode": 0,
      "execSeq": 0,
      "transactTimeNs": 0,
      "text": "string",
      "createTime": 0,
    }]
  }
}
```

<aside class="notice">
Response data is limited to 3 months.
</aside>

| Field         | Type   | Required | Description                 | Possible Values       |
|---------------|--------|----------|-----------------------------|-----------------------|
| Currency      | String | True     | the currency to query       | USDT,TRY,BRZ,USDC, ...|

## Query fee rate by quote currency

> Request format

```
GET /api-data/spots/fee-rate?quoteCurrency=<quoteCurrency>
```

> Response format

```json
{
  "symbolFeeRates": [
    {
      "takerFeeRateEr": 80000,
      "makerFeeRateEr": 80000,
      "symbol": "sETHTRY"
    }
  ]
}
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| currency | String         | True     | the currency to query     | BTC,ETH, USDT ...               |
| start    | Integer        | False    | start time in millisecond | default 2 days ago from the end |
| end      | Integer        | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

## Query order book

> Request format

```
GET /md/orderbook?symbol=<symbol>
```

> Response format

```javascript
{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
    ]
    },
    "depth": 30,
    "sequence": <sequence>,
    "timestamp": <timestamp>,
    "symbol": "<symbol>",
    "type": "snapshot"
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| timestamp   | Integer| Timestamp in nanoseconds                   |              |
| priceEp     | Integer| Scaled book level price                    |              |
| size        | Integer| Scaled book level size                     |              |
| sequence    | Integer| current message sequence                   |              |
| symbol      | String | Spot symbol name                           |              |

> Request sample

```
GET /md/orderbook?symbol=sBTCUSDT
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          877050000000,
          1000000
        ],
        [
          877100000000,
          200000
        ]
      ],
      "bids": [
        [
          877000000000,
          2000000
        ],
        [
          876950000000,
          200000
        ]
      ]
    },
    "depth": 30,
    "sequence": 455476965,
    "timestamp": 1583555482434235628,
    "symbol": "sBTCUSDT",
    "type": "snapshot"
  }
}
```

## Query full order book

> Request format

```
GET /md/fullbook?symbol=<symbol>
```

> Request sample

```
GET /md/orderbook?symbol=sBTCUSDT
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          877050000000,
          1000000
        ],
        [
          877100000000,
          200000
        ]
      ],
      "bids": [
        [
          877000000000,
          2000000
        ],
        [
          876950000000,
          200000
        ]
      ]
    },
    "depth": 0,
    "sequence": 455476965,
    "timestamp": 1583555482434235628,
    "symbol": "sBTCUSDT",
    "type": "snapshot"
  }
}
```

<aside class="notice">
The depth value is 0 in full book response.
</aside>

## Query recent trades

> Request format

```
GET /md/trade?symbol=<symbol>
```

> Response format

```javascript
{
  "error": null,
  "id": 0,
  "result": {
    "type": "snapshot",
    "sequence": <sequence>,
    "symbol": "<symbol>",
    "trades": [
      [
        <timestamp>,
        "<side>",
        <priceEp>,
        <size>
      ],
      ...
      ...
      ...
    ]
  }
}

```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| timestamp   | Integer| Timestamp in nanoseconds                   |              |
| side        | String | Trade side string                          | Buy, Sell    |
| priceEp     | Integer| Scaled trade price                         |              |
| size        | Integer| Scaled trade size                          |              |
| sequence    | Integer| Current message sequence                   |              |
| symbol      | String | Spot symbol name                           |              |

> Request sample

```
GET /md/trade?symbol=sBTCUSDT
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "sequence": 15934323,
    "symbol": "sBTCUSDT",
    "trades": [
      [
        1579164056368538508,
        "Sell",
        869600000000,
        1210000
      ],
      [
        1579164055036820552,
        "Sell",
        869600000000,
        580000
      ]
    ],
    "type": "snapshot"
  }
}

```

## Query 24 hours ticker for all symbols

> Request format

```
GET /md/spot/ticker/24hr/all
```

> Response format

```json
{
  "error": null,
  "id": 0,
  "result": [
    {
      "askEp": <ask priceEp>,
      "bidEp": <bid priceEp>,
      "highEp": <high priceEp>,
      "indexEp": <index priceEp>,
      "lastEp": <last priceEp>,
      "lowEp": <low priceEp>,
      "openEp": <open priceEp>,
      "symbol": <symbol>,
      "timestamp": <timestamp>,
      "turnoverEv": <turnoverEv>,
      "volumeEv": <volumeEv>
    },
    ...
    // other more symbols
    ...
  ]
}
```

| Field         | Type    | Description                                | Possible Value                  |
|---------------|---------|--------------------------------------------|---------------------------------|
| open priceEp  | Integer | The scaled open price in last 24 hours     |                                 |
| high priceEp  | Integer | The scaled highest price in last 24 hours  |                                 |
| low priceEp   | Integer | The scaled lowest price in last 24 hours   |                                 |
| index priceEp | Integer | The scaled index price in last 24 hours    |                                 |
| last priceEp  | Integer | The scaled last price                      |                                 |
| bid priceEp   | Integer | Scaled bid price                           |                                 |
| ask priceEp   | Integer | Scaled ask price                           |                                 |
| timestamp     | Integer | Timestamp in nanoseconds                   |                                 |
| symbol        | String  | symbol name                                | [Spot Symbols](#spot_symbols)   |
| turnoverEv    | Integer | The scaled turnover value in last 24 hours |                                 |
| volumeEv      | Integer | The scaled trade volume in last 24 hours   |                                 |


## Query 24 hours ticker

> Request format

```
GET /md/spot/ticker/24hr?symbol=<symbol>
```

> Response format

```json
{
  "error": null,
  "id": 0,
  "result": {
    "openEp": <open priceEp>,
    "highEp": <high priceEp>,
    "lowEp": <low priceEp>,
    "lastEp": <last priceEp>,
    "bidEp": <bid priceEp>,
    "askEp": <ask priceEp>,
    "symbol": <symbol>,
    "turnoverEv": <turnoverEv>,
    "volumeEv": <volumeEv>,
    "timestamp": <timestamp>
  }
}
```

| Field         | Type   | Description                                | Possible values                 |
|---------------|--------|--------------------------------------------|---------------------------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |                                 |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |                                 |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |                                 |
| last priceEp  | Integer| The scaled last price                      |                                 |
| bid priceEp   | Integer| Scaled bid price                           |                                 | 
| ask priceEp   | Integer| Scaled ask price                           |                                 |
| timestamp     | Integer| Timestamp in nanoseconds                   |                                 |
| symbol        | String | symbol name                                | [Trading symbols](#productinfo) |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |                                 |
| volumeEv      | Integer| The scaled trade volume in last 24 hours   |                                 |

> Request sample

```
GET /md/spot/ticker/24hr?symbol=sBTCUSDT
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "askEp": 892100000000,
    "bidEp": 891835000000,
    "highEp": 898264000000,
    "lastEp": 892486000000,
    "lowEp": 870656000000,
    "openEp": 896261000000,
    "symbol": "sBTCUSDT",
    "timestamp": 1590571240030003249,
    "turnoverEv": 104718804814499,
    "volumeEv": 11841148100
  }
}
```

# Spot Websocket API

## Heartbeat

> Request

```json
{
  "id": 0,
  "method": "server.ping",
  "params": []
}
```

> Response

```json
{
  "error": null,
  "id": 0,
  "result": "pong"
}
```

* Each client is required to actively send heartbeat (ping) message to Phemex data gateway ('DataGW' in short) with interval less than 30 seconds, otherwise DataGW will drop the connection. If a client sends a ping message, DataGW will reply with a pong message.
* Clients can use WS built-in ping message or the application level ping message to DataGW as heartbeat. The heartbeat interval is recommended to be set as *5 seconds*, and actively reconnect to DataGW if don't receive messages in *3 heartbeat intervals*.

## User authentication

> Request format

```javascript
{
  "method": "user.auth",
  "params": [
    "API",
    "<token>",
    "<signature>",
    <expiry>
  ],
  "id": 0
}
```

> Request sample

```json
{
  "method": "user.auth",
  "params": [
    "API",
    "806066b0-f02b-4d3e-b444-76ec718e1023",
    "8c939f7a6e6716ab7c4240384e07c81840dacd371cdcf5051bb6b7084897470e",
    1570091232
  ],
  "id": 0
}

```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "status": "success"
  }
}
```

Public channels like trade/orderbook/kline are published publicly without user authentication.
While for private channels like account/position/order data, the client should send user.auth message to Data Gateway to authenticate the session.

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| type        | String | Token type       | API             |
| token       | String | API Key     |                 |
| signature   | String | Signature generated by a funtion as HMacSha256(API Key + expiry) with ***API Secret*** ||
| expiry      | Integer| A future time after which request will be rejected, in epoch ***second***. Maximum expiry is request time plus 2 minutes ||

## Subscribe orderBook

> Request format

```javascript
{
  "id": <id>,
  "method": "orderbook.subscribe",
  "params": [
    "<symbol>"
  ]
}
```

Subscribe orderbook update messages with **depth = 30 and interval = 20ms**.

On each successful subscription, DataGW will immediately send the current Order Book (with default depth=30) snapshot to client and all later order book updates will be published.

> Request sample：

```json
{
  "id": 0,
  "method": "orderbook.subscribe",
  "params": [
    "sBTCUSDT"
  ]
}
```

## Subscribe full orderBook

> Request format

```javascript
{
  "id": <id>,
  "method": "orderbook.subscribe",
  "params": [
    "<symbol>",
    true
  ]
}
```

Subscribe orderbook update messages with **full depth and interval = 100ms**.

On each successful subscription, DataGW will immediately send the current full Order Book snapshot to client and all later order book updates will be published.

> Request sample：

```json
{
  "id": 0,
  "method": "orderbook.subscribe",
  "params": [
    "sBTCUSDT",
    true
  ]
}
```

## OrderBook message

> Message format：

```javascript
{
  "book": {
    "asks": [
      [
        <priceEp>,
        <qty>
      ],
      .
      .
      .
    ],
    "bids": [
      [
        <priceEp>,
        <qty>
      ],
      .
      .
      .
    ]
  },
  "depth": <depth>,
  "sequence": <sequence>,
  "timestamp": <timestamp>,
  "symbol": "<symbol>",

```

DataGW publishes order book message with types: incremental, snapshot. Snapshot messages are published with 60-second interval for client self-verification.

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| side        | String | Price level side | bid, ask        |
| priceEp     | Integer| Scaled price     |                 |
| qty         | Integer| Price level size. Non-zero qty indicates price level insertion or updation, and qty 0 indicates price level deletion. |                 |
| sequence    | Integer| Latest message sequence |          |
| depth       | Integer| Market depth     | 30 by default, 0 denotes fullbook |
| type        | String | Message type     | snapshot, incremental |


> Message sample: snapshot

```json
{
  "book": {
    "asks": [
      [
        892697000000,
        1781800
      ],
      [
        892708000000,
        7543500
      ]
    ],
    "bids": [
      [
        892376000000,
        6866500
      ],
      [
        892354000000,
        14209000
      ]
    ]
  },
  "depth": 30,
  "sequence": 677996311,
  "symbol": "sBTCUSDT",
  "timestamp": 1590570810187571000,
  "type": "snapshot"
}
```

> Message sample: incremental update

```json
{
  "book": {
    "asks": [],
    "bids": [
      [
        892387000000,
        4792900
      ],
      [
        892226000000,
        0
      ]
    ]
  },
  "depth": 30,
  "sequence": 677996941,
  "symbol": "sBTCUSDT",
  "timestamp": 1590570811244189000,
  "type": "incremental"
}
```

## Unsubscribe orderBook

> Request sample

```json
{
  "id": 0,
  "method": "orderbook.unsubscribe",
  "params": []
}
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "status": "success"
  }
}
```

It unsubscribes all orderbook related subscriptions.

## Subscribe trade

> Request format

```javascript
{
  "id": <id>,
  "method": "trade.subscribe",
  "params": [
    "<symbol>"
  ]
}
```

On each successful subscription, DataGW will send the 200 history trades immediately for the subscribed symbol and all the later trades will be published.

> Request sample

```json
{
  "id": 0,
  "method": "trade.subscribe",
  "params": [
    "sBTCUSDT"
  ]
}
```

## Trade message

> Message format

```javascript
{
  "trades": [
    [
      <timestamp>,
      "<side>",
      <priceEp>,
      <qty>
    ],
    .
    .
    .
  ],
  "sequence": <sequence>,
  "symbol": "<symbol>",
  "type": "<type>"
}
```

DataGW publishes trade message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.


| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Timestamp in nanoseconds for each trade ||
| side        | String | Execution taker side| bid, ask        |
| priceEp     | Integer| Scaled execution price  |                 |
| qty         | Integer| Execution size   |                 |
| sequence    | Integer| Latest message sequence ||
| symbol      | String | Spot symbol name     ||
| type        | String | Message type     |snapshot, incremental |

> Message sample: snapshot

```json
{
  "sequence": 1167852,
  "symbol": "sBTCUSDT",
  "trades": [
    [
      1573716998128563500,
      "Buy",
      867350000000,
      560000
    ],
    [
      1573716995033683000,
      "Buy",
      867350000000,
      520000
    ],
    [
      1573716991485286000,
      "Buy",
      867350000000,
      510000
    ],
    [
      1573716988636291300,
      "Buy",
      867350000000,
      120000
    ]
  ],
  "type": "snapshot"
}
```

> Message sample: snapshot

```json
{
  "sequence": 1188273,
  "symbol": "sBTCUSDT",
  "trades": [
    [
      1573717116484024300,
      "Buy",
      86730000000,
      210000
    ]
  ],
  "type": "incremental"
}
```

## Unsubscribe trade

> Request format: unsubscribe all trade subsciptions

```javascript
{
  "id": <id>,
  "method": "trade.unsubscribe",
  "params": [
  ]
}
```

> Request format: unsubscribe all trade subsciptions for a symbol

```javascript
{
  "id": <id>,
  "method": "trade.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

It unsubscribes all trade subscriptions or for a single symbol.

## Subscribe kline

> Request format

```javascript
{
  "id": <id>,
  "method": "kline.subscribe",
  "params": [
    "<symbol>",
    "<interval>"
  ]
}
```

On each successful subscription, DataGW will send the 1000 history klines immediately for the subscribed symbol and all the later kline update will be published in real-time.

> Request sample: subscribe 1-day kline

```json
{
  "id": 0,
  "method": "kline.subscribe",
  "params": [
    "sBTCUSDT",
    86400
  ]
}
```

## Kline message

> Message format

```javascript
{
  "kline": [
    [
      <timestamp>,
      "<interval>",
      <lastCloseEp>,
      <openEp>,
      <highEp>,
      <lowEp>,
      <closeEp>,
      <volumeEv>,
      <turnoverEv>,
    ],
    .
    .
    .
  ],
  "sequence": <sequence>,
  "symbol": "<symbol>",
  "type": "<type>"
}
```

> Message sample: snapshot

```json
{
  "kline": [
    [
      1590019200,
      86400,
      952057000000,
      952000000000,
      955587000000,
      947835000000,
      954446000000,
      1162621600,
      11095452729869
    ],
    [
      1589932800,
      86400,
      977566000000,
      978261000000,
      984257000000,
      935452000000,
      952057000000,
      11785486656,
      113659374080189
    ],
    [
      1589846400,
      86400,
      972343000000,
      972351000000,
      989607000000,
      949106000000,
      977566000000,
      11337554900,
      109928494593609
    ]
  ],
  "sequence": 380876982,
  "symbol": "sBTCUSDT",
  "type": "snapshot"
}
```

> Message sample: snapshot

```json
{
  "kline": [
    [
      1590019200,
      86400,
      952057000000,
      952000000000,
      955587000000,
      928440000000,
      941597000000,
      4231329700,
      40057408967508
    ]
  ],
  "sequence": 396865028,
  "symbol": "sBTCUSDT",
  "type": "incremental"
}
```

DataGW publishes kline message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Timestamp in nanoseconds for each trade ||
| interval    | Integer| Kline interval type      | 60, 300, 900, 1800, 3600, 14400, 86400, 604800, 2592000, 7776000, 31104000 |
| lastCloseEp | Integer| Scaled last close price  |                 |
| openEp      | Integer| Scaled open price        |                 |
| highEp      | Integer| Scaled high price        |                 |
| lowEp       | Integer| Scaled low price         |                 |
| closeEp     | Integer| Scaled close price       |                 |
| volumeEv    | Integer| Scaled trade voulme during the current kline interval ||
| turnoverEv  | Integer| Scaled turnover value    |                 |
| sequence    | Integer| Latest message sequence  |                 |
| symbol      | String | Spot symbol name         |                 |
| type        | String | Message type             |snapshot, incremental    |

## Unsubscribe kline

> Request format: unsubscribe all kline subscriptions

```javascript
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": []
}
```

> Request format: unsubscribe all kline subscriptions of a symbol

```javascript
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

It unsubscribes all kline subscriptions or for a single symbol.

## Subscribe wallet-order

> Request

```json
{
  "id": 0,
  "method": "wo.subscribe",
  "params": []
}
```

WO subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends WO messages back to client accordingly. 0 or more latest WO snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a users' wallets and open / max 100 closed / max 100 filled order event message history.

## Wallet-Order message

> Message format

```javascript
{
  "wallets": [{"userID":60463,...}, ...],
  "orders": [{"userID":60463, ...}],
  "sequence": <sequence>,
  "timestamp": <timestamp>,
  "type": "<type>"
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| type        | String | Message type     | snapshot, incremental |

> Message sample: snapshot

```json
{
  "orders": {
    "closed": [
      {
        "action": "New",
        "avgPriceEp": 0,
        "baseCurrency": "BTC",
        "baseQtyEv": 10000,
        "bizError": 0,
        "clOrdID": "123456",
        "createTimeNs": 1587463924959744800,
        "cumBaseQtyEv": 10000,
        "cumFeeEv": 0,
        "cumQuoteQtyEv": 66900000,
        "curBaseWalletQtyEv": 899990000,
        "curQuoteWalletQtyEv": 66900000,
        "cxlRejReason": 0,
        "feeCurrency": "BTC",
        "leavesBaseQtyEv": 0,
        "leavesQuoteQtyEv": 0,
        "ordStatus": "Filled",
        "ordType": "Limit",
        "orderID": "35217ade-3c6b-48c7-a280-8a1edb88013e",
        "pegOffsetValueEp": 0,
        "priceEp": 68000000,
        "qtyType": "ByBase",
        "quoteCurrency": "USDT",
        "quoteQtyEv": 66900000,
        "side": "Sell",
        "stopPxEp": 0,
        "symbol": "sBTCUSDT",
        "timeInForce": "GoodTillCancel",
        "transactTimeNs": 1587463924964876800,
        "triggerTimeNs": 0,
        "userID": 200076
      }
    ],
    "fills": [
      {
        "avgPriceEp": 0,
        "baseCurrency": "BTC",
        "baseQtyEv": 10000,
        "clOrdID": "123456",
        "execBaseQtyEv": 10000,
        "execFeeEv": 0,
        "execID": "8135ebe3-f767-577b-b70d-1a839d5178e0",
        "execPriceEp": 669000000000,
        "execQuoteQtyEv": 66900000,
        "feeCurrency": "BTC",
        "lastLiquidityInd": "RemovedLiquidity",
        "ordType": "Limit",
        "orderID": "35217ade-3c6b-48c7-a280-8a1edb88013e",
        "priceEp": 68000000,
        "qtyType": "ByBase",
        "quoteCurrency": "USDT",
        "quoteQtyEv": 66900000,
        "side": "Sell",
        "symbol": "sBTCUSDT",
        "transactTimeNs": 1587463924964876800,
        "userID": 200076
      }
    ],
    "open": [
      {
        "action": "New",
        "avgPriceEp": 0,
        "baseCurrency": "BTC",
        "baseQtyEv": 100000000,
        "bizError": 0,
        "clOrdID": "31f793f4-163d-aa3f-5994-0e1164719ba2",
        "createTimeNs": 1587547657438536000,
        "cumBaseQtyEv": 0,
        "cumFeeEv": 0,
        "cumQuoteQtyEv": 0,
        "curBaseWalletQtyEv": 630000005401500000,
        "curQuoteWalletQtyEv": 351802500000,
        "cxlRejReason": 0,
        "feeCurrency": "BTC",
        "leavesBaseQtyEv": 100000000,
        "leavesQuoteQtyEv": 0,
        "ordStatus": "New",
        "ordType": "Limit",
        "orderID": "b98b25c5-6aa4-4158-b9e5-477e37bd46d8",
        "pegOffsetValueEp": 0,
        "priceEp": 666500000000,
        "qtyType": "ByBase",
        "quoteCurrency": "USDT",
        "quoteQtyEv": 0,
        "side": "Sell",
        "stopPxEp": 0,
        "symbol": "sBTCUSDT",
        "timeInForce": "GoodTillCancel",
        "transactTimeNs": 1587547657442753000,
        "triggerTimeNs": 0,
        "userID": 200076
      }
    ]
  },
  "sequence": 349,
  "timestamp": 1587549121318737700,
  "type": "snapshot",
  "wallets": [
    {
      "balanceEv": 0,
      "currency": "LTC",
      "lastUpdateTimeNs": 1587481897840503600,
      "lockedTradingBalanceEv": 0,
      "lockedWithdrawEv": 0,
      "userID": 200076
    },
    {
      "balanceEv": 351802500000,
      "currency": "USDT",
      "lastUpdateTimeNs": 1587543489127498200,
      "lockedTradingBalanceEv": 0,
      "lockedWithdrawEv": 0,
      "userID": 200076
    },
    {
      "balanceEv": 630000005401500000,
      "currency": "BTC",
      "lastUpdateTimeNs": 1587547210089640400,
      "lockedTradingBalanceEv": 100000000,
      "lockedWithdrawEv": 0,
      "userID": 200076
    }
  ]
}
```

> Message sample: incremental

```json
{
  "orders": {
    "closed": [],
    "fills": [],
    "open": [
      {
        "action": "New",
        "avgPriceEp": 0,
        "baseCurrency": "BTC",
        "baseQtyEv": 100000000,
        "bizError": 0,
        "clOrdID": "0c1099e5-b900-5351-cf60-edb15ea2539c",
        "createTimeNs": 1587549529513521700,
        "cumBaseQtyEv": 0,
        "cumFeeEv": 0,
        "cumQuoteQtyEv": 0,
        "curBaseWalletQtyEv": 630000005401500000,
        "curQuoteWalletQtyEv": 351802500000,
        "cxlRejReason": 0,
        "feeCurrency": "BTC",
        "leavesBaseQtyEv": 100000000,
        "leavesQuoteQtyEv": 0,
        "ordStatus": "New",
        "ordType": "Limit",
        "orderID": "494a6cbb-32b3-4d6a-b9b7-196ea2506fb5",
        "pegOffsetValueEp": 0,
        "priceEp": 666500000000,
        "qtyType": "ByBase",
        "quoteCurrency": "USDT",
        "quoteQtyEv": 0,
        "side": "Sell",
        "stopPxEp": 0,
        "symbol": "sBTCUSDT",
        "timeInForce": "GoodTillCancel",
        "transactTimeNs": 1587549529518394000,
        "triggerTimeNs": 0,
        "userID": 200076
      }
    ]
  },
  "sequence": 350,
  "timestamp": 1587549529519959300,
  "type": "incremental",
  "wallets": [
    {
      "balanceEv": 630000005401500000,
      "currency": "BTC",
      "lastUpdateTimeNs": 1587547210089640400,
      "lockedTradingBalanceEv": 200000000,
      "lockedWithdrawEv": 0,
      "userID": 200076
    },
    {
      "balanceEv": 351802500000,
      "currency": "USDT",
      "lastUpdateTimeNs": 1587543489127498200,
      "lockedTradingBalanceEv": 0,
      "lockedWithdrawEv": 0,
      "userID": 200076
    }
  ]
}
```

## Unsubscribe wallet-order

> Request

```json
{
  "id": 0,
  "method": "wo.unsubscribe",
  "params": []
}
```

## Subscribe spot 24-hours ticker
> Reuqest sample

```json
{
  "method": "spot_market24h.subscribe",
  "params": [],
  "id": 0
}
```

## Spot 24-hours ticker message

> Message format

```javascript
{
  "spot_market24h": {
    "openEp": <open priceEp>,
    "highEp": <high priceEp>,
    "lowEp": <low priceEp>,
    "lastEp": <last priceEp>,
    "bidEp": <bid priceEp>,
    "askEp": <ask priceEp>,
    "symbol": "<symbol>",
    "turnoverEv": <turnoverEv>,
    "volumeEv": <volumeEv>
  },
  "timestamp": <timestamp>
}
```

> Message sample

```json
{
  "spot_market24h": {
    "askEp": 892100000000,
    "bidEp": 891835000000,
    "highEp": 898264000000,
    "lastEp": 892486000000,
    "lowEp": 870656000000,
    "openEp": 896261000000,
    "symbol": "sBTCUSDT",
    "timestamp": 1590571240030003249,
    "turnoverEv": 104718804814499,
    "volumeEv": 11841148100
  },
  "timestamp": 1576490244024818000
}
```

On each successful subscription, DataGW will publish 24-hour ticker metrics for all symbols every 1 second.

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| last priceEp  | Integer| The scaled last price                      |              |
| bid priceEp   | Integer| Scaled bid price                           |              |
| ask priceEp   | Integer| Scaled ask price                           |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Spot Symbol name                           |              |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volumeEv      | Integer| The scaled trade volume in last 24 hours   |              |

## Subscribe investment account

> Request sample

```json
{
  "id": 0,
  "method": "wm.subscribe",
  "params": []
}
```

On subscription to investment account then you will get your investment information of each currency type.

## Investment account message

> Message format

```javascript
{
  "investments":[
    {
      "currency": <currency>,
      "balanceEv": <balanceEv>,
      "userId": <userId>,
      "demandPendingInterestBalanceEv": <demandPendingInterestBalanceEv>,
      "demandInterestedBalanceEv": <demandInterestedBalanceEv>,
      "timedDepositBalanceEv": <timedDepositBalanceEv>,
      "currentTimeMillis": <currentTimeMillis>
  ]
}
```

> Message sample

```json
{
  "investments":[
    {
      "currency":"USDT",
      "balanceEv":21797700000,
      "userId":1234,
      "demandPendingInterestBalanceEv":0,
      "demandInterestedBalanceEv":0,
      "timedDepositBalanceEv":20000000000,
      "currentTimeMillis":1653972360161
    },
    {
      "currency":"BTC",
      "balanceEv":0,
      "userId":1234,
      "demandPendingInterestBalanceEv":0,
      "demandInterestedBalanceEv":0,
      "timedDepositBalanceEv":0,
      "currentTimeMillis":1653972360166
    }
  ]
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| currency    | String |Invested currency |      BTC,ETH    |
| balanceEv   | Integer|Invested amount   |        0        |
| userId      | Integer| User id     |                 |
| demandPendingInterestBalanceEv | Integer| Pending interest for flexible product  | 0 |
| demandInterestedBalanceEv      | Integer| Paid interest for flexible product     | 0 |
| timedDepositBalanceEv          | Integer| Amount for fixed product                 | 20000000000 |
| currentTimeMillis              | Integer| Time in milliseconds | 165397230166 |


# Margin Trading API

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case sensitive**.

<a id="spot_symbols"></a>

## Query product information

> Request

```
GET /public/products
```

* Spot symbols are defined in `.products[]` with **type=Spot**.
* Spot currencies are defined in `.currencies[]`.

## Price/Ratio/Value scales

Fields with post-fix "Rp", "Rr", "Rq" or "Rv" are real value.

## Common order fields

* Order type

| Order type | Description |
|-----------|-------------|
| Limit | -- |
| Market | -- |
| Stop | -- |
| StopLimit | -- |
| MarketIfTouched | -- |
| LimitIfTouched | -- |
| MarketAsLimit | -- |
| StopAsLimit | -- |
| MarketIfTouchedAsLimit | -- |

* Order status

| Order status | Description | 
|------------|-------------|
| Untriggered | Conditional order waiting to be triggered |
| Triggered | Conditional order being triggered|
| Rejected | Order rejected |
| New | Order placed in cross engine |
| PartiallyFilled | Order partially filled |
| Filled | Order fully filled |
| Canceled | Order canceled |

* TimeInForce

| TimeInForce | Description |
|------------|-------------|
| GoodTillCancel | -- |
| PostOnly | -- |
| ImmediateOrCancel | -- |
| FillOrKill | -- |

* Trigger source

| Trigger | Description |
|------------|-------------|
| ByLastPrice | Trigger by last price |

## Place order (HTTP PUT, *prefered*)

> Request format

```
PUT /margin-trade/orders/create?symbol=<symbol>&trigger=<trigger>&clOrdID=<clOrdID>&priceRp=<priceRp>&baseQtyRq=<baseQtyRq>&quoteQtyRq=<quoteQtyRq>&stopPxRp=<stopPxRp>&text=<text>&side=<side>&qtyType=<qtyType>&ordType=<ordType>&timeInForce=<timeInForce>&execInst=<execInst>&autoBorrow=<autoBorrow>&borrowCurrency=<borrowCurrency>&borrowQtyRq=<borrowQtyRq>&autoPayback=<autoPayback>&paybackCurrency=<paybackCurrency>&paybackQtyRq=<paybackQtyRq>
```

> Response format

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "orderID": "d1d09454-cabc-4a23-89a7-59d43363f16d",
    "clOrdID": "309bcd5c-9f6e-4a68-b775-4494542eb5cb",
    "priceRp": "0",
    "action": "New",
    "trigger": "UNSPECIFIED",
    "pegPriceType": "UNSPECIFIED",
    "stopDirection": "UNSPECIFIED",
    "bizError": 0,
    "symbol": "sBTCUSDT",
    "side": "Buy",
    "baseQtyRq": "0",
    "ordType": "Limit",
    "timeInForce": "GoodTillCancel",
    "ordStatus": "Created",
    "cumFeeRv": "0",
    "cumBaseQtyRq": "0",
    "cumQuoteQtyRq": "0",
    "leavesBaseQtyRq": "0",
    "leavesQuoteQtyRq": "0",
    "avgPriceRp": "0",
    "cumBaseAmountRv": "0",
    "cumQuoteAmountRv": "0",
    "quoteQtyRq": "0",
    "qtyType": "ByBase",
    "stopPxRp": "0",
    "pegOffsetValueRp": "0",
    "autoBorrow": false,
    "borrowCurrency": 1,
    "borrowQtyRq": "0",
    "autoPayback": false,
    "paybackCurrency": 1,
    "paybackPrincipalQtyRq": "0",
    "paybackInterestQtyRq": "0",
    "hourlyInterestRateRr": "0",
    "riskLevelRr": "0",
    "liqFeeRv": "0",
    "liqFeeRateRr": "0"
  }
}
```

| Field       | Type   | Required | Description               | Possible values |
|----------   |--------|----------|---------------------------|-----------------|
| symbol      | String | Yes      |                           | [Spot Symbols](#spot_symbols) |
| side        | Enum   | Yes      |                           |  Sell, Buy     | 
| qtyType     | Enum   | Yes      | Set order quantity by base or quote currency | ByBase, ByQuote|
| quoteQtyRq  | String| --       | Required if qtyType = ByQuote|  |
| baseQtyRq   | String| --       |                           | Required if qtyType = ByBase   |
| priceRp     | String|          |                           | real price            |
| stopPxRp    | String| --       | used in conditionalorder  |   |
| trigger     | Enum   | --       | Required in conditional order | ByLastPrice |
| timeInForce | Enum   | No       | Default GoodTillCancel    | GoodTillCancel, PostOnly,ImmediateOrCancel,FillOrKill |
| ordType     | Enum   | No       | Default to Limit          | Market, Limit, Stop, StopLimit, MarketIfTouched, LimitIfTouched|
| autoBorrow  | Boolean | No      |                           | false                |
| borrowCurrency  | String | No      |                           | BTC,USDT           |
| borrowQtyRq     | String | No      |                           |                 |
| autoPayback     | Boolean | No      |                           | false               |
| paybackCurrency   | String | No      |                           | BTC,USDT           |
| paybackQtyRq    | String | No      |                           |                 |


## Cancel order

> Request format

```
DELETE /margin-trade/orders?symbol=<symbol>&orderID=<orderID>
DELETE /margin-trade/orders?symbol=<symbol>&clOrdID=<clOrdID>
```

## Cancel all order by symbol

> Request format

```
DELETE /margin-trade/orders/all?symbol=<symbol>&untriggered=<untriggered>
```

| Field | Type | Required | Description |
|---------|--------|-------|-------------|
| symbol  | Enum   | Yes   | The symbol to cancel |
| untriggered | Boolean | No | set false to cancel non-conditiaonal order, true to conditional order |

## Query open order by order ID or client order ID

> Request format

```
GET /margin-trade/orders/active?symbol=<symbol>&orderID=<orderID>
GET /margin-trade/orders/active?symbol=<symbol>&clOrDID=<clOrdID>
```

## Query all open orders by symbol

> Request format

```
GET /margin-trade/orders?symbol=<symbol>
```

## Query margin orders details

> Request format

```
GET /margin/orders?symbol=<symbol>&ordStatus=<ordStatus>&ordType=<ordType>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter | Type    | Required | Description                  | Case     |
|-----------|---------|----------|------------------------------|----------|
| symbol    | String  | NO       | spot symbol                  | sBTCUSDT |
| ordStatus | String  | NO       |                              | Filled   |
| ordType   | String  | NO       |                              | Market   |
| start     | Long    | NO       | start order index, default 0 |          |
| end       | Long    | NO       | end order index, default 0   |          |
| pageNum   | Long    | NO       | page number, default 0       |          |
| pageSize  | Integer | NO       | pageable size, default 20    |          |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": 1,
    "rows": [
      {
        "orderId": "f6256e4f-0f0f-40bd-b26e-2c3e5afd02f5",
        "clOrdId": "2337fc19-7db0-f0c7-6ff9-404234a78c3c",
        "stopPxRp": "0",
        "avgPriceRp": "24747.95059307",
        "qtyType": "ByBase",
        "leavesBaseQtyRq": "0",
        "leavesQuoteQtyRq": "0",
        "baseQtyRq": "0.349332",
        "feeCurrency": "USDT",
        "stopDirection": "UNSPECIFIED",
        "symbol": "sBTCUSDT",
        "side": "Sell",
        "quoteQtyRq": "0",
        "priceRp": "22211.84",
        "ordType": "Market",
        "timeInForce": "ImmediateOrCancel",
        "ordStatus": "Filled",
        "execStatus": "TakerFill",
        "createTimeNs": 1676893028372924849,
        "cumFeeRv": "8.64525109",
        "cumBaseValueRv": "0.349332",
        "cumQuoteValueRv": "8645.25107658",
        "detailVos": null
      }
    ]
  }
}
```

## Query margin order trades details

> Request format

```
GET /margin/orders/trades?symbol=<symbol>&execType=<execType>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter | Type    | Required | Description                  | Case     |
|-----------|---------|----------|------------------------------|----------|
| symbol    | String  | NO       | spot symbol                  | sBTCUSDT |
| execType  | String  | NO       |                              | Trade    |
| start     | Long    | NO       | start order index, default 0 |          |
| end       | Long    | NO       | end order index, default 0   |          |
| pageNum   | Long    | NO       | page number, default 0       |          |
| pageSize  | Integer | NO       | pageable size, default 20    |          |


> Response format

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 1,
        "rows": [
            {
                "transactTimeNs": 1675181154118252379,
                "execType": "Amend",
                "qtyType": "ByQuote",
                "clOrdId": "6bbbf1d2-42a1-70ef-e272-168b524353b4",
                "orderId": "c111c403-ace7-43ec-a7a6-6c6ef012fdfe",
                "symbol": "sBTCUSDT",
                "side": "Buy",
                "priceRp": "25437.04",
                "baseQtyRq": "0",
                "quoteQtyRq": "2443.28",
                "action": "New",
                "execStatus": "TakerFill",
                "ordStatus": "PartiallyFilled",
                "ordType": "Market",
                "execInst": "None",
                "timeInForce": "ImmediateOrCancel",
                "stopDirection": "UNSPECIFIED",
                "stopPxRp": "0",
                "execId": "e7717c8a-5fd2-5360-a7e3-de5c29f24639",
                "execPriceRp": "23124.59",
                "execBaseQtyRq": "0.01126",
                "execQuoteQtyRq": "260.3828834",
                "leavesBaseQtyRq": "0",
                "leavesQuoteQtyRq": "2182.8971166",
                "execFeeRv": "0.00001126",
                "feeRateRr": "0.001",
                "baseCurrency": "BTC",
                "quoteCurrency": "USDT",
                "feeCurrency": "BTC"
            }
        ]
    }
}
```

## Query margin borrow interest history

> Request format

```
GET /margin/borrow/interests?currency=<currencyList>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter | Type    | Required | Description    | Case      |
|-----------|---------|----------|----------------|-----------|
| currency  | List    | NO       | currency list  | USDT,BTC  |
| start     | Long    | NO       | default 0      |           |
| end       | Long    | NO       | default 0      |           |
| pageNum   | Long    | NO       | default 0      |           |
| pageSize  | Integer | NO       | default 20     |           |


> Response format

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 1,
        "rows": [
            {
                "borrowCurrency": "USDT",
                "interestCalcTime": 1678975566487,
                "interestCurrency": "USDT",
                "hourlyInterestRv": "0.00013254",
                "hourlyRateRr": "0.00000551",
                "annualRateRr": "0.0482676"
            }
        ]
    }
}
```

## Query margin borrow history records

> Request format

```
GET /margin/borrow?currency=<currency>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter | Type    | Required | Description | Case |
|-----------|---------|----------|-------------|------|
| Currency  | String  | NO       |             | USDT |
| start     | Long    | NO       | default 0   |      |
| end       | Long    | NO       | default 0   |      |
| pageNum   | Long    | NO       | default 0   |      |
| pageSize  | Integer | NO       | default 20  |      |


> Response format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": 1,
    "rows": [
      {
        "currency": "USDT",
        "borrowTime": 1677211672675,
        "amountRv": "94.05",
        "type": 1,
        "status": "Success"
      }
    ]
  }
}
```

## Query margin payback history

> Request format

```
GET /margin/payback?currency=<currency>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter | Type    | Required | Description | Case |
|-----------|---------|----------|-------------|------|
| currency  | String  | NO       |             | USDT |
| start     | Long    | NO       | default 0   |      |
| end       | Long    | NO       | default 0   |      |
| pageNum   | Long    | NO       | default 0   |      |
| pageSize  | Integer | NO       | default 20  |      |


> Response format

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 1,
        "rows": [
            {
                "currency": "USDT",
                "repayTime": 1678976978535,
                "principalAmountRv": "24.05469895",
                "interestAmountRv": "0.019881",
                "liqFeeRv": "0",
                "type": 1,
                "status": "Success"
            }
        ]
    }
}
```

## Post margin borrow request

> Request format

```
POST /margin/borrow?currency=<currency>&amountRv=<amountRv>
```

* Request parameters

| Parameter | Type   | Required | Description              | Case    |
|-----------|--------|----------|--------------------------|---------|
| currency  | String | YES      | borrow currency          | USDT    |
| amountRv  | String | YES      | borrow amount real value | 1000.12 |


> Response format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "maxBorrowAmountRq": "80000",
    "borrowedAmountRq": "40",
    "currency": "USDT"
  }
}
```

## Post margin payback history

> Request format

```
POST /margin/payback?currency=<currency>&amountRv=<amountRv>
```

* Request parameters


| Parameter | Type   | Required | Description               | Case    |
|-----------|--------|----------|---------------------------|---------|
| currency  | String | YES      | payback currency          | USDT    |
| amountRv  | String | YES      | payback amount real value | 1000.12 |

> Response format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "maxBorrowAmountRq": "80000",
    "borrowedAmountRq": "40",
    "currency": "USDT"
  }
}
```


## Query wallets

* Request parameters


| Parameter | Type   | Required | Description               | Case         |
|-----------|--------|----------|---------------------------|--------------|
| currency  | String | No       | the currency to query     | BTC, USDT... |

NOTE: GET /margin-trade/wallets queries the wallets of all currencies

> Request format

```
GET /margin-trade/wallets?currency=<currency>
```

> Response format

```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "currency": "BTC",
      "balanceRq": "0",
      "lockedTradingBalanceRq": "0",
      "lockedWithdrawRq": "0",
      "borrowedRq": "0",
      "cumInterestRq": "0",
      "lastUpdateTimeNs": 0
    }
  ]
}
```


[Query order book](#query-order-book-3)
the same as the spot api

[Query full order book](#query-full-order-book-2)
the same as the spot api

[Query recent trades](#query-recent-trades-2)
the same as the spot api

[Query 24 hours ticker](#query-24-hours-ticker-2)
the same as the spot api


# Margin Trading Websocket API

the same as above Spot Websocket API

## Subscribe margin account and order

> Request

```json
{
  "id": 0,
  "method": "mao.subscribe",
  "params": []
}
```
> Message sample: incremental
```json
{
	"wallets_mao": [{
		"balanceRv": "6300000054.015",
		"currency": "BTC",
		"lastUpdateTimeNs": 1587547210089640382,
		"lockedTradingBalanceRv": "2",
		"lockedWithdrawRv": "0",
		"borrowedRv": "12",
		"cumInterestRv": "0.5",
		"hourlyInterestRateRr": "0.0012",
		"maxAmountToBorrowRv": "100",
		"userID": 200076
	}, {
		"balanceRv": "3518.025",
		"currency": "USDT",
		"lastUpdateTimeNs": 1587543489127498121,
		"lockedTradingBalanceRv": "0",
		"lockedWithdrawRv": "0",
		"borrowedRv": "12",
		"cumInterestRv": "0.5",
		"hourlyInterestRateRr": "0.0012",
		"maxAmountToBorrowRv": "100",
		"userID": 200076
	}],
	"orders_mao": {
		"closed": [],
		"fills": [],
		"open": [{
			"action": "New",
			"avgPriceRp": "0",
			"baseCurrency": "BTC",
			"baseQtyRq": "10000",
			"bizError": 0,
			"clOrdID": "0c1099e5-b900-5351-cf60-edb15ea2539c",
			"createTimeNs": 1587549529513521745,
			"cumBaseQtyRq": "0",
			"cumFeeRv": "0",
			"cumQuoteQtyRq": "0",
			"curBaseWalletQtyRq": "6300000.0540150",
			"curQuoteWalletQtyRq": "3518.0250",
			"cxlRejReason": 0,
			"feeCurrency": "BTC",
			"leavesBaseQtyRq": "1",
			"leavesQuoteQtyRq": "0",
			"ordStatus": "New",
			"ordType": "Limit",
			"orderID": "494a6cbb-32b3-4d6a-b9b7-196ea2506fb5",
			"pegOffsetValueRv": "0",
			"priceRp": "6665",
			"qtyType": "ByBase",
			"quoteCurrency": "USDT",
			"quoteQtyRq": "0",
			"side": "Sell",
			"stopPxRp": "0",
			"symbol": "sBTCUSDT",
			"timeInForce": "GoodTillCancel",
			"transactTimeNs": 1587549529518394235,
			"triggerTimeNs": 0,
			"userID": 200076
		}]
	},
	"sequence": 350,
	"timestamp": 1587549529519959388,
	"type": "incremental"
}
```


