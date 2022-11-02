# Contract REST API

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* `apiSecret` = `Base64::urlDecode(API Secret)`
* The `signature` is **case sensitive**.

### Signature Example 1: HTTP GET Request

* API REST Request URL: https://api.phemex.com/accounts/accountPositions?currency=BTC
   * Request Path: /accounts/accountPositions
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /accounts/accountPositions + currency=BTC + 1575735514 )

### Singature Example 2: HTTP GET Request with multiple query string

* API REST Request URL: https://api.phemex.com/orders/activeList?ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD 
    * Request Path: /orders/activeList
    * Request Query: ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD
    * Request Body: <null>
    * Request Expire: 1575735951
    * Signature: HMacSha256(/orders/activeList + ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD + 1575735951)
    * signed string is `/orders/activeListordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD1575735951`

### Signature Example 3: HTTP POST Request

* API REST Request URL: https://api.phemex.com/orders
   * Request Path: /orders
   * Request Query: <null>
   * Request Body: {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /orders + 1575735514 + {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0})
   * signed string is `/orders1575735514{"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}`


## Request/Response field explaination

### Leverage
   * The absolute value of `leverageEr` determines initial-margin-rate, i.e. `initialMarginRate = 1/abs(leverage)`
   * The sign of `leverageEr` indicates margin mode, i.e. `leverage <= 0` means `cross-margin-mode`, `leverage > 0` means `isolated-margin-mode`.
   * The result of setting `leverageEr` to `0` is leverage to maximum leverage supported by user selected risklimit, and margin-mode is `cross-margin-mode`.

### Cross Margin Mode 
   * `Position margin` includes two parts, one part is balance assigned to position, another part is account available balance. 
   * Position in cross-margin-mode may be affected by other position, because account available balance is shared among all positions in cross mode.

### Isolated Margin Mode
   * `Position margin` only includes balance assgined to position, by default it is initial-margin. 
   * Position in isolatd-margin-mode is independent of other positions.

## Price/Ratio/Value Scales

Fields with post-fix "Ep", "Er" or "Ev" have been scaled based on symbol setting.
* Fields with post-fix "Ep" are scaled prices, `priceScale` in [products](#query-product-information)
* Fields with post-fix "Er" are scaled ratios, `ratioScale` in [products](#query-product-information)
* Fields with post-fix "Ev" are scaled values, `valueScale` of `settleCurrency` in [products](#query-product-information) 

## Common Constants

* Order Type

| Order Type | Description |
|-----------|-------------|
| Limit | -- |
| Market | -- |
| Stop | -- |
| StopLimit | -- |
| MarketIfTouched | -- |
| LimitIfTouched | -- |


* Order Status

| Order Status | Description | 
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


* Execution instruction

| Execution Instruction | Description |
|------------|-------------|
| ReduceOnly | reduce position size, never increase position size |
| CloseOnTrigger | close the position  |


* Trigger Source

| Trigger | Description |
|------------|-------------|
| ByMarkPrice | trigger by mark price|
| ByLastPrice | trigger by last price |

## Query Product Information

> Request

```
GET /public/products 
```

* Contract symbols are defined in `.products[]` with **type=Perpetual**.
* Contract risklimit information are defined in `.risklimits[]`.

## Place Order (PUT request with query string)

> Request format

```
PUT /orders/create?clOrdID=<clOrdID>&symbol=<symbol>&reduceOnly=<reduceOnly>&closeOnTrigger=<closeOnTrigger>&orderQty=<orderQty>&displayQty=<displayQty>&ordType=<ordType>&priceEp=<priceEp>&side=<side>&text=<text>&timeInForce=<timeInForce>&stopPxEp=<stopPxEp>&takeProfitEp=<takeProfitEp>&stopLossEp=<stopLossEp>&pegOffsetValueEp=<pegOffsetValueEp>&pegPriceType=<pegPriceType>&trailingStopEp=<trailingStopEp>&triggerType=<triggerType>&tpTrigger=<tpTrigger>&tpSlTs=<tpSlTs>&slTrigger=<slTrigger>
```

> Response sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "bizError": 0,
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "symbol": "BTCUSD",
    "side": "Sell",
    "actionTimeNs": 1580547265848034600,
    "transactTimeNs": 0,
    "orderType": null,
    "priceEp": 98970000,
    "price": 9897,
    "orderQty": 1,
    "displayQty": 1,
    "timeInForce": null,
    "reduceOnly": false,
    "stopPxEp": 0,
    "closedPnlEv": 0,
    "closedPnl": 0,
    "closedSize": 0,
    "cumQty": 0,
    "cumValueEv": 0,
    "cumValue": 0,
    "leavesQty": 1,
    "leavesValueEv": 10104,
    "leavesValue": 0.00010104,
    "stopPx": 0,
    "stopDirection": "UNSPECIFIED",
    "ordStatus": "Created"
  }
}
```

| Field | Type | Required | Description | Possible values |
|-------|-------|--------|--------------|-----------------|
| symbol | String | Yes | Which symbol to place order |  | 
| clOrdID | String | Yes | client order id, max length is 40| |
| side |  Enum | Yes | Order direction, Buy or Sell | Buy, Sell | 
| orderQty | Integer | Yes | Order quantity | |
| priceEp | Integer | - | Scaled price, required for limit order | | 
| ordType | Enum | - | default to Limit | Market, Limit, Stop, StopLimit, MarketIfTouched, LimitIfTouched| 
| stopPxEp | Integer | - | Trigger price for stop orders | |
| timeInForce | Enum | - | Time in force. default to GoodTillCancel | GoodTillCancel, ImmediateOrCancel, FillOrKill, PostOnly| 
| reduceOnly | Boolean | - | whether reduce position side only. Enable this flag, i.e. reduceOnly=true, position side won't change | true, false |
| closeOnTrigger | Boolean | - | implicitly reduceOnly, plus cancel other orders in the same direction(side) when necessary | true, false|
| triggerType | Enum | - | Trigger source, whether trigger by mark price, index price or last price | ByMarkPrice, ByLastPrice |
| takeProfitEp | Integer | - | Scaled take profit price | |
| stopLossEp | Integer | - | Scaled stop loss price | | 
| slTrigger | Enum | - |Trigger source, by mark-price or last-price | ByMarkPrice, ByLastPrice |
| tpTrigger | Enum | - | Trigger source, by mark-price or last-price | ByMarkPrice, ByLastPrice |
| pegOffsetValueEp | Integer | - | Trailing offset from current price. Negative value when position is long, positive when position is short | |
| pegPriceType | Enum | - | Trailing order price type |TrailingStopPeg, TrailingTakeProfitPeg |


## Query Order Book

> Request format

```
GET /md/orderbook?symbol=<symbol>
```

> Response format

```
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
        .
        .
        .
      ],
      "bids": [
        [
          <priceEp>,
          <size>
        ],
        .
        .
        .
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
| symbol      | String | Contract symbol name                       |              |

> Request sample

```
GET /md/orderbook?symbol=BTCUSD
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
          87705000,
          1000000
        ],
        [
          87710000,
          200000
        ]
      ],
      "bids": [
        [
          87700000,
          2000000
        ],
        [
          87695000,
          200000
        ]
      ]
    },
    "depth": 30,
    "sequence": 455476965,
    "timestamp": 1583555482434235628,
    "symbol": "BTCUSD",
    "type": "snapshot"
  }
}
```

## Query Kline

> Request format

```
GET /exchange/public/md/v2/kline?symbol=<symbol>&resolution=<resolution>&limit=<limit>

```

> Response format

```
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": -1,
    "rows": [[<timestamp>, <interval>, <last_close>, <open>, <high>, <low>, <close>, <volume>, <turnover>], [...]]
  }
}
```

* Please be noted that kline interfaces have [rate limits](#rate-limits) rule,  please check the Other group under [api groups](#api-groups)

| Field      | Type    | Required | Description     | Possible Values                         |
|------------|---------|----------|-----------------|-----------------------------------------|
| symbol     | String  | Yes      | symbol name     | BTCUSD,ETHUSD,uBTCUSD,cETHUSD,XRPUSD... | 
| resolution | Integer | Yes      | kline interval  | described as below                      |
| limit      | Integer | No       | limit of result | described as below                      | 


* Value of `resolution`s

|Resolution|Description |
|----------|------------|
|60|MINUTE_1|
|300|MINUTE_5|
|900|MINUTE_15|
|1800|MINUTE_30|
|3600|HOUR_1|
|14400|HOUR_4|
|86400|DAY_1|
|604800|WEEK_1|
|2592000|MONTH_1|
|7776000|SEASON_1|
|31104000|YEAR_1|

* Value of `limit`s

| Limit    | Description |
|----------|-------------|
| 5        | limit 5     |
| 10       | limit 10    |
| 50      | limit 50    |
| 100     | limit 100   |
| 500     | limit 500   |
| 1000    | limit 1000  |


**NOTE** for backward compatibility reason, phemex also provides kline query with from/to, however, this interface is **NOT** recommended.

> Request format

```
GET /exchange/public/md/kline?symbol=<symbol>&to=<to>&from=<from>&resolution=<resolution>

```

| Field       | Type    | Required    | Description            | Possible Values                                                                                                |
|-------------|---------|-------------|------------------------|----------------------------------------------------------------------------------------------------------------|
|symbol       | String  | Yes         | symbol name            | BTCUSD,ETHUSD,uBTCUSD,cETHUSD,XRPUSD...                                                                        | 
| from        | Integer | Yes         | start time in seconds  | value aligned in resolution boundary                                                                           |
| to          | Integer | Yes         | end time in seconds    | value aligned in resolution boundary; Number of k-lines return between [`from`, `to`) should be less than 1000 | 
| resolution  | Integer | Yes         | kline interval         | the same as described above                                                                                    |


## Query Recent Trades

> Request format

```
GET /md/trade?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       |              |

> Response format

```
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
      .
      .
      .
    ]
  }
}
```

> Message sample

```
GET /md/trade?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "sequence": 15934323,
    "symbol": "BTCUSD",
    "trades": [
      [
        1579164056368538508,
        "Sell",
        86960000,
        121
      ],
      [
        1579164055036820552,
        "Sell",
        86960000,
        58
      ]
    ],
    "type": "snapshot"
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
| symbol      | String | Contract symbol name                       |              |

## Query 24 Hours Ticker

> Request format

```
GET v1/md/ticker/24hr?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       |              |

> Response format

```
{
  "error": null,
  "id": 0,
  "result": {
      "askEp": <best ask priceEp>,
      "bidEp": <best bid priceEp>,
      "fundingRateEr": <funding rateEr>,
      "highEp": <high priceEp>,
      "indexEp": <index priceEp>,
      "lastEp": <last priceEp>,
      "lowEp": <low priceEp>,
      "markEp": <mark priceEp>,
      "openEp": <open priceEp>,
      "openInterest": <open interest>,
      "predFundingRateEr": <predicated funding rateEr>,
      "symbol": <symbol>,
      "timestamp": <timestamp>,
      "turnoverEv": <turnoverEv>,
      "volume": <volume>
  }
}
```

> Request sample

```
GET v1/md/ticker/24hr?symbol=BTCUSD
```

> Response sample

```json
{
  "error": null,
  "id": 0,
  "result": {
    "close": 87425000,
    "fundingRate": 10000,
    "high": 92080000,
    "indexPrice": 87450676,
    "low": 87130000,
    "markPrice": 87453092,
    "open": 90710000,
    "openInterest": 7821141,
    "predFundingRate": 7609,
    "symbol": "BTCUSD",
    "timestamp": 1583646442444219017,
    "turnover": 1399362834123,
    "volume": 125287131
  }
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| close priceEp | Integer| The scaled close price in last 24 hours    |              |
| index priceEp | Integer| Scaled index price                         |              |
| mark priceEp  | Integer| Scaled mark price                          |              |
| open interest | Integer| current open interest                      |              |
| funding rateEr| Integer| Scaled funding rate                        |              |
| predicated funding rateEr| Integer| Scaled predicated funding rate  |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Contract symbol name                       |              |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volume        | Integer| Symbol trade volume in last 24 hours       |              |

## Query History Trades By symbol

> Request format

```
GET /exchange/public/nomics/trades?market=<symbol>&since=<since>
```

| Field  | Type     | Description                                                       | Possible values                   |
|--------|----------|-------------------------------------------------------------------|-----------------------------------|
| market | String   | the market of symbol                                              |                                   |
| since  | String   | Last id of response field, 0-0-0 is from the very initial trade   | default 0-0-0                     |
| start  | Integer  | Epoch time in milli-seconds of range start                        |                                   |
| end    | Integer  | Epoch time in milli-seconds of range end                          |                                   |

> Response format

```
{
  "code": 0,
  "data": [
    {
      "id": "<id>",
      "amount_quote": "<amount>",
      "price": "<price>",
      "side": "<side>",
      "timestamp": "<timestamp>",
      "type": "<type>"
    }
  ],
  "msg": "<msg>"
}
```

* Query History trades by symbol
* RateLimit of this api is 5 per second

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
  {
    "id": "1183-3-2",
    "timestamp": "2019-11-24T08:32:17.046Z",
    "price": "7211.00000000",
    "amount_quote": "1",
    "side": "sell",
    "type": "limit"
  },
  {
    "id": "1184-2-1",
    "timestamp": "2019-11-24T08:32:17.047Z",
    "price": "7211.00000000",
    "amount_quote": "1",
    "side": "buy",
    "type": "limit"
  }]
}

```


# Contract Websocket API

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

## User Authentication

> Request format

```
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

Public channels like trade/orderbook/kline are published publicly without user authentication.
While for private channels like account/position/order data, the client should send user.auth message to Data Gateway to authenticate the session.

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| type        | String | Token type       | API             |
| token       | String | API Key     |                 |
| signature   | String | Signature generated by a funtion as HMacSha256(API Key + expiry) with ***API Secret*** ||
| expiry      | Integer| A future time after which request will be rejected, in epoch ***second***. Maximum expiry is request time plus 2 minutes ||

## Subscribe OrderBook 

> Request format

```
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
    "BTCUSD"
  ]
}
```

## Subscribe Full OrderBook 

> Request format

```
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
    "BTCUSD",
    true
  ]
}
```

## OrderBook Message

> Message format：
 
```
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
        86765000,
        19609
      ],
      [
        86770000,
        7402
      ]
    ],
    "bids": [
      [
        86760000,
        18995
      ],
      [
        86755000,
        6451
      ]
    ]
  },
  "depth": 30,
  "sequence": 1191904,
  "symbol": "BTCUSD",
  "type": "snapshot"
}
```

> Message sample: incremental update

```json
{
  "book": {
    "asks": [
      [
        86775000,
        4621
      ]
    ],
    "bids": []
  },
  "depth": 30,
  "sequence": 1191905,
  "symbol": "BTCUSD",
  "type": "incremental"
}
```

## Unsubscribe OrderBook

> Request sample

```json
{
  "id": 0,
  "method": "orderbook.unsubscribe",
  "params": []
}
```

It unsubscribes all orderbook related subscriptions.

## Subscribe Trade

> Request format

```
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
    "BTCUSD"
  ]
}
```

## Trade Message

> Message format

```
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
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

> Message sample: snapshot

```json
{
  "sequence": 1167852,
  "symbol": "BTCUSD",
  "trades": [
    [
      1573716998128563500,
      "Buy",
      86735000,
      56
    ],
    [
      1573716995033683000,
      "Buy",
      86735000,
      52
    ],
    [
      1573716991485286000,
      "Buy",
      86735000,
      51
    ],
    [
      1573716988636291300,
      "Buy",
      86735000,
      12
    ]
  ],
  "type": "snapshot"
}
```

> Message sample: snapshot

```json
{
  "sequence": 1188273,
  "symbol": "BTCUSD",
  "trades": [
    [
      1573717116484024300,
      "Buy",
      86730000,
      21
    ]
  ],
  "type": "incremental"
}
```

## Unsubscribe Trade

> Request format: unsubscribe all trade subsciptions

```
{
  "id": <id>,
  "method": "trade.unsubscribe",
  "params": [
  ]
}
```

> Request format: unsubscribe all trade subsciptions for a symbol

```
{
  "id": <id>,
  "method": "trade.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

It unsubscribes all trade subscriptions or for a single symbol.

## Subscribe Kline

> Request format

```
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
    "BTCUSD",
    86400
  ]
}
```

## Kline Message

> Message format

```
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
      <volume>,
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
      95165000,
      95160000,
      95160000,
      95160000,
      95160000,
      164,
      1723413
    ],
    [
      1589932800,
      86400,
      97840000,
      97840000,
      98480000,
      92990000,
      95165000,
      246294692,
      2562249857942
    ],
    [
      1589846400,
      86400,
      97335000,
      97335000,
      99090000,
      94490000,
      97840000,
      212484260,
      2194232158593
    ]
  ],
  "sequence": 1118993873,
  "symbol": "BTCUSD",
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
      95165000,
      95160000,
      95750000,
      92585000,
      93655000,
      84414679,
      892414738605
    ]
  ],
  "sequence": 1122006398,
  "symbol": "BTCUSD",
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
| volume      | Integer| Trade voulme during the current kline interval ||
| turnoverEv  | Integer| Scaled turnover value    |                 |
| sequence    | Integer| Latest message sequence  ||
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

## Unsubscribe Kline

> Request format: unsubscribe all kline subscriptions

```
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": []
}
```

> Request format: unsubscribe all kline subscriptions of a symbol

```
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

It unsubscribes all kline subscriptions or for a single symbol.


## Subscribe Account-Order-Position (AOP)

> Request format

```
{
  "id": <id>,
  "method": "aop.subscribe",
  "params": []
}
```

AOP subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends AOP messages back to client accordingly. 0 or more latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a trading account information, holding positions, and open / max 100 closed / max 100 filled order event message history.

## Account-Order-Position (AOP) Message

> Message format

```
{
  "accounts": [
    {
      "accountID": 604630001,
      "currency": "BTC",
      .
      .
      .
    }
  ],
  "orders": [
    {
      "accountID": 604630001,
      .
      .
      .
    }
  ],
  "positions": [
    {
      "accountID": 604630001,
      .
      .
      .
    }
  ],
  "sequence":1,
  "timestamp":<timestamp>,
  "type":"<type>"
}

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| symbol      | String | Contract symbol name    |          |
| type        | String | Message type     | snapshot, incremental |



> Message sample: snapshot

```json
{
  "accounts": [
    {
      "accountBalanceEv": 100000024,
      "accountID": 675340001,
      "bonusBalanceEv": 0,
      "currency": "BTC",
      "totalUsedBalanceEv": 1222,
      "userID": 67534
    }
  ],
  "orders": [
    {
      "accountID": 675340001,
      "action": "New",
      "actionBy": "ByUser",
      "actionTimeNs": 1573711481897337000,
      "addedSeq": 1110523,
      "bonusChangedAmountEv": 0,
      "clOrdID": "uuid-1573711480091",
      "closedPnlEv": 0,
      "closedSize": 0,
      "code": 0,
      "cumQty": 2,
      "cumValueEv": 23018,
      "curAccBalanceEv": 100000005,
      "curAssignedPosBalanceEv": 0,
      "curBonusBalanceEv": 0,
      "curLeverageEr": 0,
      "curPosSide": "Buy",
      "curPosSize": 2,
      "curPosTerm": 1,
      "curPosValueEv": 23018,
      "curRiskLimitEv": 10000000000,
      "currency": "BTC",
      "cxlRejReason": 0,
      "displayQty": 2,
      "execFeeEv": -5,
      "execID": "92301512-7a79-5138-b582-ac185223727d",
      "execPriceEp": 86885000,
      "execQty": 2,
      "execSeq": 1131034,
      "execStatus": "MakerFill",
      "execValueEv": 23018,
      "feeRateEr": -25000,
      "lastLiquidityInd": "AddedLiquidity",
      "leavesQty": 0,
      "leavesValueEv": 0,
      "message": "No error",
      "ordStatus": "Filled",
      "ordType": "Limit",
      "orderID": "e9a45803-0af8-41b7-9c63-9b7c417715d9",
      "orderQty": 2,
      "pegOffsetValueEp": 0,
      "priceEp": 86885000,
      "relatedPosTerm": 1,
      "relatedReqNum": 2,
      "side": "Buy",
      "stopLossEp": 0,
      "stopPxEp": 0,
      "symbol": "BTCUSD",
      "takeProfitEp": 0,
      "timeInForce": "GoodTillCancel",
      "tradeType": "Trade",
      "transactTimeNs": 1573712555309040400,
      "userID": 67534
    },
    {
      "accountID": 675340001,
      "action": "New",
      "actionBy": "ByUser",
      "actionTimeNs": 1573711490507067000,
      "addedSeq": 1110980,
      "bonusChangedAmountEv": 0,
      "clOrdID": "uuid-1573711488668",
      "closedPnlEv": 0,
      "closedSize": 0,
      "code": 0,
      "cumQty": 3,
      "cumValueEv": 34530,
      "curAccBalanceEv": 100000013,
      "curAssignedPosBalanceEv": 0,
      "curBonusBalanceEv": 0,
      "curLeverageEr": 0,
      "curPosSide": "Buy",
      "curPosSize": 5,
      "curPosTerm": 1,
      "curPosValueEv": 57548,
      "curRiskLimitEv": 10000000000,
      "currency": "BTC",
      "cxlRejReason": 0,
      "displayQty": 3,
      "execFeeEv": -8,
      "execID": "80899855-5b95-55aa-b84e-8d1052f19886",
      "execPriceEp": 86880000,
      "execQty": 3,
      "execSeq": 1131408,
      "execStatus": "MakerFill",
      "execValueEv": 34530,
      "feeRateEr": -25000,
      "lastLiquidityInd": "AddedLiquidity",
      "leavesQty": 0,
      "leavesValueEv": 0,
      "message": "No error",
      "ordStatus": "Filled",
      "ordType": "Limit",
      "orderID": "7e03cd6b-e45e-48d9-8937-8c6628e7a79d",
      "orderQty": 3,
      "pegOffsetValueEp": 0,
      "priceEp": 86880000,
      "relatedPosTerm": 1,
      "relatedReqNum": 3,
      "side": "Buy",
      "stopLossEp": 0,
      "stopPxEp": 0,
      "symbol": "BTCUSD",
      "takeProfitEp": 0,
      "timeInForce": "GoodTillCancel",
      "tradeType": "Trade",
      "transactTimeNs": 1573712559100655600,
      "userID": 67534
    },
    {
      "accountID": 675340001,
      "action": "New",
      "actionBy": "ByUser",
      "actionTimeNs": 1573711499282604000,
      "addedSeq": 1111025,
      "bonusChangedAmountEv": 0,
      "clOrdID": "uuid-1573711497265",
      "closedPnlEv": 0,
      "closedSize": 0,
      "code": 0,
      "cumQty": 4,
      "cumValueEv": 46048,
      "curAccBalanceEv": 100000024,
      "curAssignedPosBalanceEv": 0,
      "curBonusBalanceEv": 0,
      "curLeverageEr": 0,
      "curPosSide": "Buy",
      "curPosSize": 9,
      "curPosTerm": 1,
      "curPosValueEv": 103596,
      "curRiskLimitEv": 10000000000,
      "currency": "BTC",
      "cxlRejReason": 0,
      "displayQty": 4,
      "execFeeEv": -11,
      "execID": "0be06645-90b8-5abe-8eb0-dca8e852f82f",
      "execPriceEp": 86865000,
      "execQty": 4,
      "execSeq": 1132422,
      "execStatus": "MakerFill",
      "execValueEv": 46048,
      "feeRateEr": -25000,
      "lastLiquidityInd": "AddedLiquidity",
      "leavesQty": 0,
      "leavesValueEv": 0,
      "message": "No error",
      "ordStatus": "Filled",
      "ordType": "Limit",
      "orderID": "66753807-9204-443d-acf9-946d15d5bedb",
      "orderQty": 4,
      "pegOffsetValueEp": 0,
      "priceEp": 86865000,
      "relatedPosTerm": 1,
      "relatedReqNum": 4,
      "side": "Buy",
      "stopLossEp": 0,
      "stopPxEp": 0,
      "symbol": "BTCUSD",
      "takeProfitEp": 0,
      "timeInForce": "GoodTillCancel",
      "tradeType": "Trade",
      "transactTimeNs": 1573712618104628700,
      "userID": 67534
    }
  ],
  "positions": [
    {
      "accountID": 675340001,
      "assignedPosBalanceEv": 0,
      "avgEntryPriceEp": 86875941,
      "bankruptCommEv": 75022,
      "bankruptPriceEp": 90000,
      "buyLeavesQty": 0,
      "buyLeavesValueEv": 0,
      "buyValueToCostEr": 1150750,
      "createdAtNs": 0,
      "crossSharedBalanceEv": 99998802,
      "cumClosedPnlEv": 0,
      "cumFundingFeeEv": 0,
      "cumTransactFeeEv": -24,
      "currency": "BTC",
      "dataVer": 4,
      "deleveragePercentileEr": 0,
      "displayLeverageEr": 1000000,
      "estimatedOrdLossEv": 0,
      "execSeq": 1132422,
      "freeCostEv": 0,
      "freeQty": -9,
      "initMarginReqEr": 1000000,
      "lastFundingTime": 1573703858883133200,
      "lastTermEndTime": 0,
      "leverageEr": 0,
      "liquidationPriceEp": 90000,
      "maintMarginReqEr": 500000,
      "makerFeeRateEr": 0,
      "markPriceEp": 86786292,
      "orderCostEv": 0,
      "posCostEv": 1115,
      "positionMarginEv": 99925002,
      "positionStatus": "Normal",
      "riskLimitEv": 10000000000,
      "sellLeavesQty": 0,
      "sellLeavesValueEv": 0,
      "sellValueToCostEr": 1149250,
      "side": "Buy",
      "size": 9,
      "symbol": "BTCUSD",
      "takerFeeRateEr": 0,
      "term": 1,
      "transactTimeNs": 1573712618104628700,
      "unrealisedPnlEv": -107,
      "updatedAtNs": 0,
      "usedBalanceEv": 1222,
      "userID": 67534,
      "valueEv": 103596
    }
  ],
  "sequence": 1310812,
  "timestamp": 1573716998131004000,
  "type": "snapshot"
}
```

> Message sample: incremental

```json
{
  "accounts": [
    {
      "accountBalanceEv": 99999989,
      "accountID": 675340001,
      "bonusBalanceEv": 0,
      "currency": "BTC",
      "totalUsedBalanceEv": 1803,
      "userID": 67534
    }
  ],
  "orders": [
    {
      "accountID": 675340001,
      "action": "New",
      "actionBy": "ByUser",
      "actionTimeNs": 1573717286765750000,
      "addedSeq": 1192303,
      "bonusChangedAmountEv": 0,
      "clOrdID": "uuid-1573717284329",
      "closedPnlEv": 0,
      "closedSize": 0,
      "code": 0,
      "cumQty": 0,
      "cumValueEv": 0,
      "curAccBalanceEv": 100000024,
      "curAssignedPosBalanceEv": 0,
      "curBonusBalanceEv": 0,
      "curLeverageEr": 0,
      "curPosSide": "Buy",
      "curPosSize": 9,
      "curPosTerm": 1,
      "curPosValueEv": 103596,
      "curRiskLimitEv": 10000000000,
      "currency": "BTC",
      "cxlRejReason": 0,
      "displayQty": 4,
      "execFeeEv": 0,
      "execID": "00000000-0000-0000-0000-000000000000",
      "execPriceEp": 0,
      "execQty": 0,
      "execSeq": 1192303,
      "execStatus": "New",
      "execValueEv": 0,
      "feeRateEr": 0,
      "leavesQty": 4,
      "leavesValueEv": 46098,
      "message": "No error",
      "ordStatus": "New",
      "ordType": "Limit",
      "orderID": "e329ae87-ce80-439d-b0cf-ad65272ed44c",
      "orderQty": 4,
      "pegOffsetValueEp": 0,
      "priceEp": 86770000,
      "relatedPosTerm": 1,
      "relatedReqNum": 5,
      "side": "Buy",
      "stopLossEp": 0,
      "stopPxEp": 0,
      "symbol": "BTCUSD",
      "takeProfitEp": 0,
      "timeInForce": "GoodTillCancel",
      "transactTimeNs": 1573717286765896400,
      "userID": 67534
    },
    {
      "accountID": 675340001,
      "action": "New",
      "actionBy": "ByUser",
      "actionTimeNs": 1573717286765750000,
      "addedSeq": 1192303,
      "bonusChangedAmountEv": 0,
      "clOrdID": "uuid-1573717284329",
      "closedPnlEv": 0,
      "closedSize": 0,
      "code": 0,
      "cumQty": 4,
      "cumValueEv": 46098,
      "curAccBalanceEv": 99999989,
      "curAssignedPosBalanceEv": 0,
      "curBonusBalanceEv": 0,
      "curLeverageEr": 0,
      "curPosSide": "Buy",
      "curPosSize": 13,
      "curPosTerm": 1,
      "curPosValueEv": 149694,
      "curRiskLimitEv": 10000000000,
      "currency": "BTC",
      "cxlRejReason": 0,
      "displayQty": 4,
      "execFeeEv": 35,
      "execID": "8d1848a2-5faf-52dd-be71-9fecbc8926be",
      "execPriceEp": 86770000,
      "execQty": 4,
      "execSeq": 1192303,
      "execStatus": "TakerFill",
      "execValueEv": 46098,
      "feeRateEr": 75000,
      "lastLiquidityInd": "RemovedLiquidity",
      "leavesQty": 0,
      "leavesValueEv": 0,
      "message": "No error",
      "ordStatus": "Filled",
      "ordType": "Limit",
      "orderID": "e329ae87-ce80-439d-b0cf-ad65272ed44c",
      "orderQty": 4,
      "pegOffsetValueEp": 0,
      "priceEp": 86770000,
      "relatedPosTerm": 1,
      "relatedReqNum": 5,
      "side": "Buy",
      "stopLossEp": 0,
      "stopPxEp": 0,
      "symbol": "BTCUSD",
      "takeProfitEp": 0,
      "timeInForce": "GoodTillCancel",
      "tradeType": "Trade",
      "transactTimeNs": 1573717286765896400,
      "userID": 67534
    }
  ],
  "positions": [
    {
      "accountID": 675340001,
      "assignedPosBalanceEv": 0,
      "avgEntryPriceEp": 86843828,
      "bankruptCommEv": 75056,
      "bankruptPriceEp": 130000,
      "buyLeavesQty": 0,
      "buyLeavesValueEv": 0,
      "buyValueToCostEr": 1150750,
      "createdAtNs": 0,
      "crossSharedBalanceEv": 99998186,
      "cumClosedPnlEv": 0,
      "cumFundingFeeEv": 0,
      "cumTransactFeeEv": 11,
      "currency": "BTC",
      "dataVer": 5,
      "deleveragePercentileEr": 0,
      "displayLeverageEr": 1000000,
      "estimatedOrdLossEv": 0,
      "execSeq": 1192303,
      "freeCostEv": 0,
      "freeQty": -13,
      "initMarginReqEr": 1000000,
      "lastFundingTime": 1573703858883133200,
      "lastTermEndTime": 0,
      "leverageEr": 0,
      "liquidationPriceEp": 130000,
      "maintMarginReqEr": 500000,
      "makerFeeRateEr": 0,
      "markPriceEp": 86732335,
      "orderCostEv": 0,
      "posCostEv": 1611,
      "positionMarginEv": 99924933,
      "positionStatus": "Normal",
      "riskLimitEv": 10000000000,
      "sellLeavesQty": 0,
      "sellLeavesValueEv": 0,
      "sellValueToCostEr": 1149250,
      "side": "Buy",
      "size": 13,
      "symbol": "BTCUSD",
      "takerFeeRateEr": 0,
      "term": 1,
      "transactTimeNs": 1573717286765896400,
      "unrealisedPnlEv": -192,
      "updatedAtNs": 0,
      "usedBalanceEv": 1803,
      "userID": 67534,
      "valueEv": 149694
    }
  ],
  "sequence": 1315725,
  "timestamp": 1573717286767188200,
  "type": "incremental"
}
```


## Unsubscribe Account-Order-Position (AOP)

> Request format

```
{
  "id": <id>,
  "method": "aop.unsubscribe",
  "params": []
}
```

It unsubscribes all account-order-positions.

## Subscribe 24 Hours Ticker
> Reuqest sample

```json
{
  "method": "market24h.subscribe",
  "params": [],
  "id": 0
}
```

## 24-Hours Ticker Message

> Message format

```
{
  "market24h": {
    "open": <open priceEp>,
    "high": <high priceEp>,
    "low": <low priceEp>,
    "close": <close priceEp>,
    "indexPrice": <index priceEp>,
    "markPrice": <mark priceEp>,
    "openInterest": <open interest>,
    "fundingRate": <funding rateEr>,
    "predFundingRate": <predicated funding rateEr>,
    "symbol": "<symbol>",
    "turnover": <turnoverEv>,
    "volume": <volume>
  },
  "timestamp": <timestamp>
}
```

> Message sample

```json
{
  "market24h": {
    "close": 87425000,
    "fundingRate": 10000,
    "high": 92080000,
    "indexPrice": 87450676,
    "low": 87130000,
    "markPrice": 87453092,
    "open": 90710000,
    "openInterest": 7821141,
    "predFundingRate": 7609,
    "symbol": "BTCUSD",
    "timestamp": 1583646442444219017,
    "turnover": 1399362834123,
    "volume": 125287131
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
| close priceEp | Integer| The scaled close price in last 24 hours    |              |
| index priceEp | Integer| Scaled index price                         |              |
| mark priceEp  | Integer| Scaled mark price                          |              |
| open interest | Integer| current open interest                      |              |
| funding rateEr| Integer| Scaled funding rate                        |              |
| predicated funding rateEr| Integer| Scaled predicated funding rate  |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Contract symbol name                       |              |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volume        | Integer| Symbol trade volume in last 24 hours       |              |
  
## Subscribe Price Tick

> Request sample: subscribe a single symbol. 

```json
{
  "method": "tick.subscribe",
  "params": [
    ".BTC"
  ],
  "id": 1580631267153
}
```

* Every trading symbol has a suite of other symbols, each symbol follows same patterns,
i.e. `index` symbol follows a pattern `.<BASECURRENCY>`,
     `mark` symbol follows a pattern `.M<BASECURRENCY>`,
     predicated funding rate's symbol follows a pattern `.<BASECURRENCY>FR`,
     while funding rate symbol follows a pattern `.<BASECURRENCY>FR8H`
* Price is retrieved by subscribing symbol tick.
* all available symbols in [products](#query-product-information)

## Tick Message

> Message format

```
{
  "tick": {
    "last": <priceEp>,
    "scale": <scale>,
    "symbol": <symbol>
    "timestamp": <timestamp_in_nano>
  }
}
```

> Message sample

```json
{
  "tick": {
    "last": 93385362,
    "scale": 4,
    "symbol": ".BTC",
    "timestamp": 1580635719408000000
  }
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| priceEp  | Integer| The scaled open price in last 24 hours          |              |
| scale    | Integer| The price scale factor, e.g. 4 denotes price scaled with factor 1e4. | 4,6,8  |
| symbol   | String | Symbol                                          |              |

