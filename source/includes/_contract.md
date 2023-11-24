# Contract REST API

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case sensitive**.

### Signature example 1: HTTP GET request

* API REST Request URL: https://api.phemex.com/accounts/accountPositions?currency=BTC
   * Request Path: /accounts/accountPositions
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /accounts/accountPositions + currency=BTC + 1575735514 )

### Singature example 2: HTTP GET request with multiple query string

* API REST Request URL: https://api.phemex.com/orders/activeList?ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD
    * Request Path: /orders/activeList
    * Request Query: ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD
    * Request Body: <null>
    * Request Expire: 1575735951
    * Signature: HMacSha256(/orders/activeList + ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD + 1575735951)
    * signed string is `/orders/activeListordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD1575735951`

### Signature example 3: HTTP POST request

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

### Cross margin mode
   * `Position margin` includes two parts, one part is balance assigned to position, another part is account available balance.
   * Position in cross-margin-mode may be affected by other position, because account available balance is shared among all positions in cross mode.

### Isolated margin mode
   * `Position margin` only includes balance assgined to position, by default it is initial-margin.
   * Position in isolatd-margin-mode is independent of other positions.

## Price/Ratio/Value scales

Fields with post-fix "Ep", "Er" or "Ev" have been scaled based on symbol setting.

* Fields with post-fix "Ep" are scaled prices, `priceScale` in [products](#query-product-information)
* Fields with post-fix "Er" are scaled ratios, `ratioScale` in [products](#query-product-information)
* Fields with post-fix "Ev" are scaled values, `valueScale` of `settleCurrency` in [products](#query-product-information)

## Query product information

> Request

```
GET /public/products
```

* USD-M perpetual contracts using USD as margin or COIN-M perpetual contracts using COIN as margin
* Contract symbols are defined in `.products[]` with **type=Perpetual**.
* Contract risklimit information are defined in `.risklimits[]`.
* Contract which delisted has status with 'Delisted' .


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


## Common order fields

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
| Created | order acked from order request, a transient state |
| Init | Same as `Created`, order acked from order request, a transient state |
| Untriggered | Conditional order waiting to be triggered |
| Triggered | Conditional order being triggered|
| Deactivated | untriggered conditonal order being removed |
| Rejected | Order rejected |
| New | Order placed into orderbook |
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
  
  
* Trade Type
  
| tradeType | Description |
|------------|-------------|
| Trade | -- |
| Funding | -- |
| LiqTrade | -- |
| AdlTrade | -- |

 
* Exec Status

| execStatus | Description |
|------------|-------------|
| Aborted | -- |
| MakerFill | -- |
| TakerFill | -- |
| Expired | -- |
| Canceled | -- |
| CreateRejected | -- |

    
* Pos Mode (for Hedged Mode only)

| Trigger | Description |
|------------|-------------|
| OneWay | can only hold one side position |
| Hedged | can hold both side position |
  

* Pos Side (for Hedged Mode only)

| Trigger | Description |
|------------|-------------|
| Long | Long position when pos mode is 'Hedged' |
| Short | Short position when pos mode is 'Hedged' |
| Merged | Merged position when pos mode is 'OneWay' |
    

* Funds Biz Type

| Description             | Code |
|-------------------------|------|
| REALIZED_PNL            | 1    |
| TRANSFER                | 2    |
| COMMISSION              | 3    |
| BONUS                   | 4    |
| SUB_TO_PARENT_TRANSFER  | 5    |
| PARENT_TO_SUB_TRANSFER  | 6    |
| COPY_TRADE_TRANSFER     | 90   |
| COPY_TRADE_SETTLEMENT   | 91   |
| COPY_TRADE_REFUNDS      | 92   |
| COPY_TRADE_PROFIT_SHARE | 93   |
| MARGIN_TRANSFER         | 213  |

## More order fields explained
| Field | Description |
|------|----------|
| bizError | 0 means processing normally, and non-zero values mean wrong state. `code` in response is equal to bizError if response contains only one order |
| cumQty | cumulative filled order quantity |
| cumValueEv | scaled cumulative filled order value |
| leavesQty | leaves order quantity |
| leavesValueEv | scaled leaves order value |

## Place order (HTTP PUT, *prefered*)

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

## Place order (HTTP POST)

> Request example

```
POST /orders
```

```json
{
  "actionBy": "FromOrderPlacement",
  "symbol": "BTCUSD",
  "clOrdID": "uuid-1573058952273",
  "side": "Sell",
  "priceEp": 93185000,
  "orderQty": 7,
  "ordType": "Limit",
  "reduceOnly": false,
  "triggerType": "UNSPECIFIED",
  "pegPriceType": "UNSPECIFIED",
  "timeInForce": "GoodTillCancel",
  "takeProfitEp": 0,
  "stopLossEp": 0,
  "pegOffsetValueEp": 0,
  "pegPriceType": "UNSPECIFIED"
}
```

## More order type examples

* Stop-loss orders are with ordType = Stop/StopLimit. And Take-profit orders are with ordType = MarketIfTouched/LimitIfTouched.
* Stop-loss order is triggered when price moves against order-side(buy/sell), while Take-profit order is triggered
  when price moves in profitable direction to order-side(buy/sell).

| Order Type                      | Side | Parameter requirements            | Trigger condition |
|---------------------------------|------|-----------------------------------|-------------------|
| Stop/StopLimit                  | Sell |  stopPxEp < last-price/mark-price | last/mark-price <= stopPxEp  |
| Stop/StopLimit                  | Buy  |  stopPxEp > last-price/mark-price | last/mark-price >= stopPxEp  |
| MarketIfTouched/LimitIfTouched  | Sell |  stopPxEp > last-price/mark-price | last/mark-price >= stopPxEp  |
| MarketIfTouched/LimitIfTouched  | Buy  |  stopPxEp < last-price/mark-price | last/mark-price <= stopPxEp  |

> Buy stop limit order: triggered order is placed as limit order (assume current last-price is 30k)

```json
{
  "clOrdID": "stop-loss-order-then-limit",
  "symbol": "BTCUSD",
  "side": "Sell",
  "ordType": "StopLimit",
  "triggerType": "ByMarkPrice",
  "stopPxEp": 299550000,
  "priceEp": 299650000,
  "orderQty": 10000
}
```

> Buy stop market order: triggered order is placed as market order (assume current last-price is 30k)

```json
{
  "clOrdID": "stop-loss-order-then-market",
  "symbol": "BTCUSD",
  "side": "Buy",
  "ordType": "Stop",
  "triggerType": "ByMarkPrice",
  "stopPxEp": 333550000,
  "priceEp": 0,
  "orderQty": 10000
}
```

* Fields description for Stop/Take-profit Orders:

| Field       | Type    | Description | Possible values |
|-------------|---------|-------------|-----------------|
| triggerType | String  | The trigger price type | ByMarkPrice, ByLastPrice |
| stopPxEp    | Integer | The order trigger price. It should be less than the last price if ordType= Stop/StopLimit/LimitIfTouched/MarketIfTouched and side = Sell, and greater than the last price if side = Buy. | |
| priceEp     | Integer | The order price after triggered. Required if ordType = StopLimit/LimitIfTouched. | |

> Take-profit sell limit order: triggered order is placed as limit order (Assume current last-price is 30k)

```json
{
  "clOrdID": "take-profit-order-then-limit",
  "symbol": "BTCUSD",
  "side": "Sell",
  "ordType": "LimitIfTouched",
  "triggerType": "ByMarkPrice",
  "stopPxEp": 333550000,
  "priceEp": 334550000,
  "orderQty": 10000
}
```

> Take-profit buy market order: triggered order is placed as market order (assume current last-price is 30k)

```json
{
  "clOrdID": "take-profit-order-then-market",
  "symbol": "BTCUSD",
  "side": "Buy",
  "ordType": "MarketIfTouched",
  "triggerType": "ByLastPrice",
  "stopPxEp": 299550000,
  "priceEp": 0,
  "orderQty": 10000
}
```

> Place order with stop-loss and take-profit

```json
{
  "clOrdID": "order-with-take-profit-stop-loss",
  "symbol": "BTCUSD",
  "side": "Buy",
  "priceEp": 300000000,
  "orderQty": 1000,
  "ordType": "Limit",
  "takeProfitEp": 3111100000,
  "tpTrigger": "ByLastPrice",
  "stopLossEp": 299990000,
  "slTrigger": "ByMarkPrice"
}
```

> Trailing stop order: (assume current position is long, current last-price is 32k)

```json
{
  "symbol": "BTCUSD",
  "side": "Sell",
  "ordType": "Stop",
  "orderQty": 0,
  "priceEp": 0,
  "triggerType": "ByLastPrice",
  "stopPxEp": 315000000,
  "timeInForce": "ImmediateOrCancel",
  "closeOnTrigger": true,
  "pegPriceType": "TrailingStopPeg",
  "pegOffsetValueEp": -10000000,
  "clOrdID": "cl-order-id"
}
```

> Trailing stop order with activiation price

```json
{
  "symbol": "BTCUSD",
  "side": "Sell",
  "ordType": "Stop",
  "orderQty": 0,
  "priceEp": 0,
  "triggerType": "ByLastPrice",
  "stopPxEp": 340000000,
  "timeInForce": "ImmediateOrCancel",
  "closeOnTrigger": true,
  "pegPriceType": "TrailingTakeProfitPeg",
  "pegOffsetValueEp": -10000000,
  "clOrdID": "cl-order-id"
}
```

* Fields description for trailing stop order:

| Field       | Type    | Description | Possible values |
|-------------|---------|-------------|-----------------|
| triggerType | String  | The trigger price type | ByMarkPrice, ByLastPrice |
| stopPxEp    | Integer | The order trigger price. It should be less than last-price if hold long position and vice versa. | |
| pegOffsetValueEp     | Integer | The offset price. It means to set offset price by an offset from the optimal price, and the sign is opposite to position side. e.g. Long Position => negative sign. Short Position => positive sign; | |

## Amend order by order ID

> Request format

```
PUT
/orders/replace?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&price=<price>&priceEp=<priceEp>&orderQty=<orderQty>&stopPx=<stopPx>&stopPxEp=<stopPxEp>&takeProfit=<takeProfit>&takeProfitEp=<takeProfitEp>&stopLoss=<stopLoss>&stopLossEp=<stopLossEp>&pegOffsetValueEp=<pegOffsetValueEp>&pegPriceType=<pegPriceType>
```

| Field  | Required | Description |
|--------|----------|-------------|
| symbol | Yes  | Order symbol, cannot be changed|
| orderID| No  | Order ID, cannot be changed |
| origClOrdID | No | Original clOrderID |
| price  | No | New order price |
| priceEp| No | New order price with scale |
| orderQty | No | New orderQty |
| stopPx | No | New stop price |
| stopPxEp | No | New stop price with scale |
| takeProfit | No | New stop profit price |
| takeProfitEp | No | New stop profit price with scale |
| stopLoss | No | New stop loss price |
| stopLossEp | No | New stop loss price with scale |
| pegOffsetValueEp | No | New trailing offset |
| pegPriceType | No | New peg price type |

orderID and origClOrdID can not be both empty

## Cancel order by order ID or client order ID

> Request format

```
DELETE /orders/cancel?symbol=<symbol>&orderID=<orderID>
DELETE /orders/cancel?symbol=<symbol>&clOrdID=<clOrdID>
```

> Request sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "bizError": 0,
    "orderID": "2585817b-85df-4dea-8507-5db1920b9954",
    "clOrdID": "4b19fd1e-a1a7-2986-d02a-0288ad5137d4",
    "symbol": "BTCUSD",
    "side": "Buy",
    "actionTimeNs": 1580533179846642700,
    "transactTimeNs": 1580532966633276200,
    "orderType": null,
    "priceEp": 80040000,
    "price": 8004,
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
    "leavesValueEv": 12493,
    "leavesValue": 0.00012493,
    "stopPx": 0,
    "stopDirection": "UNSPECIFIED",
    "ordStatus": "New"
  }
}

```

## Bulk cancel orders

> Request format

```
DELETE /orders?symbol=<symbol>&orderID=<orderID1>,<orderID2>,<orderID3>
```

## Cancel all orders

> Request format

```
DELETE /orders/all?symbol=<symbol>&untriggered=<untriggered>&text=<text>
```

* In order to cancel all orders, include conditional order and active order, one must invoke this API twice with
  different arguments.
* `untriggered=false` to cancel active order including triggerred conditional order.
* `untriggered=true` to cancel conditional order, the order is not triggerred.

| Field       | Type   | Required  | Description                    | Possible values         |
|-------------|--------|-----------|--------------------------------|-------------------------|
| symbol      | String | Yes       | Which symbol to cancel         |                         |
| untriggered | Boolean| No        | Default to false, default cancel non-conditional order; if intending to cancel conditional order, set this to true| true,false|
| text        | Comments| No       | Comments of this operation, limited to 40 characters  |  |

## Query trading account and positions

> Request format

```
GET /accounts/accountPositions?currency=<currency>
```

> Response sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "account": {
      "accountId": 0,
      "currency": "BTC",
      "accountBalanceEv": 0,
      "totalUsedBalanceEv": 0
    },
    "positions": [
      {
        "accountID": 0,
        "symbol": "BTCUSD",
        "currency": "BTC",
        "side": "None",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageEr": 0,
        "leverage": 0,
        "initMarginReqEr": 0,
        "initMarginReq": 0.01,
        "maintMarginReqEr": 500000,
        "maintMarginReq": 0.005,
        "riskLimitEv": 10000000000,
        "riskLimit": 100,
        "size": 0,
        "value": 0,
        "valueEv": 0,
        "avgEntryPriceEp": 0,
        "avgEntryPrice": 0,
        "posCostEv": 0,
        "posCost": 0,
        "assignedPosBalanceEv": 0,
        "assignedPosBalance": 0,
        "bankruptCommEv": 0,
        "bankruptComm": 0,
        "bankruptPriceEp": 0,
        "bankruptPrice": 0,
        "positionMarginEv": 0,
        "positionMargin": 0,
        "liquidationPriceEp": 0,
        "liquidationPrice": 0,
        "deleveragePercentileEr": 0,
        "deleveragePercentile": 0,
        "buyValueToCostEr": 1150750,
        "buyValueToCost": 0.0115075,
        "sellValueToCostEr": 1149250,
        "sellValueToCost": 0.0114925,
        "markPriceEp": 93169002,
        "markPrice": 9316.9002,
        "markValueEv": 0,
        "markValue": null,
        "estimatedOrdLossEv": 0,
        "estimatedOrdLoss": 0,
        "usedBalanceEv": 0,
        "usedBalance": 0,
        "takeProfitEp": 0,
        "takeProfit": null,
        "stopLossEp": 0,
        "stopLoss": null,
        "realisedPnlEv": 0,
        "realisedPnl": null,
        "cumRealisedPnlEv": 0,
        "cumRealisedPnl": null
      }
    ]
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| currency    | String | trading account's settle currency. Use to identify trading account. | BTC, USD, ETH |

<aside class="notice">
<i>unRealizedPnlEv</i> is not up to date and needs to be calculated in client side with latest <i>markPrice</i>. The formula is as below.
</aside>

```
Inverse long contract: unRealizedPnl = (posSize * contractSize) / avgEntryPrice - (posSize * contractSize) / markPrice
Inverse short contract: unRealizedPnl =  (posSize *contractSize) / markPrice - (posSize * contractSize) / avgEntryPrice
Linear long contract:  unRealizedPnl = (posSize * contractSize) * markPrice - (posSize * contractSize) * avgEntryPrice
Linear short contract:  unRealizedPnl = (posSize * contractSize) * avgEntryPrice - (posSize * contractSize) * markPrice

posSize is a signed vaule. contractSize is a fixed value.
```

## Query trading account and positions with realtime unPnL

> Request format

```
GET /accounts/positions?currency=<currency>
```

> Response sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "account": {
      "accountId": 111100001,
      "currency": "BTC",
      "accountBalanceEv": 879599942377,
      "totalUsedBalanceEv": 285,
      "bonusBalanceEv": 0
    },
    "positions": [
      {
        "accountID": 111100001,
        "symbol": "BTCUSD",
        "currency": "BTC",
        "side": "Buy",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageEr": 0,
        "initMarginReqEr": 1000000,
        "maintMarginReqEr": 500000,
        "riskLimitEv": 10000000000,
        "size": 5,
        "valueEv": 26435,
        "avgEntryPriceEp": 189143181,
        "posCostEv": 285,
        "assignedPosBalanceEv": 285,
        "bankruptCommEv": 750000,
        "bankruptPriceEp": 5000,
        "positionMarginEv": 879599192377,
        "liquidationPriceEp": 5000,
        "deleveragePercentileEr": 0,
        "buyValueToCostEr": 1150750,
        "sellValueToCostEr": 1149250,
        "markPriceEp": 238287555,
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossEv": 0,
        "usedBalanceEv": 285,
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlEv": -8913353,
        "cumFundingFeeEv": 123996,
        "cumTransactFeeEv": 940245,
        "realisedPnlEv": 0,
        "unRealisedPnlEv": 5452,
        "cumRealisedPnlEv": 0
      }
    ]
  }
}

```

The API return latest position unrealized pnl with **considerable** cost, thus its ratelimit weight is very high.
<aside class="notice">
For better performance and latency, highly recommend calculating <i>unRealizedPnlEv</i> in client side with latest <i>markPrice</i> to avoid ratelimit
penalty.
</aside>

## Set leverage

> Request format

```
PUT /positions/leverage?symbol=<symbol>&leverage=<leverage>&leverageEr=<leverageEr>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|-----------------|
| symbol               | String | Which postion to change              |                 |
| leverage             | Integer| Unscaled leverage                          |                 |
| leverageEr           | Integer| Ratio scaled leverage, leverage wins when both leverage and leverageEr provided|  |

## Set position risklimit

> Request format

```
PUT /positions/riskLimit?symbol=<symbol>&riskLimit=<riskLimit>&riskLimitEv=<riskLimitEv>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|-----------------|
| symbol               | String | Which postion to change              |                 |
| riskLimit            | Integer| Unscaled value, reference BTC/USD value scale|               |
| riskLimitEv          | Integer| Value scaled risklimit, riskLimitEv wins when both riskLimit and riskLimitEv provided|  |

## Assign position balance in isolated marign mode

> Request format

```
POST /positions/assign?symbol=<symbol>&posBalance=<posBalance>&posBalanceEv=<posBalanceEv>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | String | Which symbol to change              |  |
| posBalance           | Integer| Unscaled raw value                       |  |
| posBalanceEv         | Integer| The scaled value for position balance, posBalanceEv wins when both posBalance and posBalanceEv provided|  |

## Query open orders by symbol

> Request format

```
GET /orders/activeList?symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "rows": [
      {
        "bizError": 0,
        "orderID": "9cb95282-7840-42d6-9768-ab8901385a67",
        "clOrdID": "7eaa9987-928c-652e-cc6a-82fc35641706",
        "symbol": "BTCUSD",
        "side": "Buy",
        "actionTimeNs": 1580533011677666800,
        "transactTimeNs": 1580533011677666800,
        "orderType": null,
        "priceEp": 84000000,
        "price": 8400,
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
        "leavesQty": 0,
        "leavesValueEv": 0,
        "leavesValue": 0,
        "stopPx": 0,
        "stopDirection": "Falling",
        "ordStatus": "Untriggered"
      },
      {
        "bizError": 0,
        "orderID": "93397a06-e76d-4e3b-babc-dff2696786aa",
        "clOrdID": "71c2ab5d-eb6f-0d5c-a7c4-50fd5d40cc50",
        "symbol": "BTCUSD",
        "side": "Sell",
        "actionTimeNs": 1580532983785506600,
        "transactTimeNs": 1580532983786370300,
        "orderType": null,
        "priceEp": 99040000,
        "price": 9904,
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
        "leavesValueEv": 10096,
        "leavesValue": 0.00010096,
        "stopPx": 0,
        "stopDirection": "UNSPECIFIED",
        "ordStatus": "New"
      }
    ]
  }
}
```

* Order status includes `New`, `PartiallyFilled`, `Filled`, `Canceled`, `Rejected`, `Triggered`, `Untriggered`;
* Open order status includes `New`, `PartiallyFilled`, `Untriggered`;

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol to query |   |

## Query closed orders by symbol

> Request format

```
GET /exchange/order/list?symbol=<symbol>&start=<start>&end=<end>&offset=<offset>&limit=<limit>&ordStatus=<ordStatus>&withCount=<withCount>
```

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": 39,
    "rows": [
      {
        "orderID": "7d5a39d6-ff14-4428-b9e1-1fcf1800d6ac",
        "clOrdID": "e422be37-074c-403d-aac8-ad94827f60c1",
        "symbol": "BTCUSD",
        "side": "Sell",
        "orderType": "Limit",
        "actionTimeNs": 1577523473419470300,
        "priceEp": 75720000,
        "price": null,
        "orderQty": 12,
        "displayQty": 0,
        "timeInForce": "GoodTillCancel",
        "reduceOnly": false,
        "takeProfitEp": 0,
        "takeProfit": null,
        "stopLossEp": 0,
        "closedPnlEv": 0,
        "closedPnl": null,
        "closedSize": 0,
        "cumQty": 0,
        "cumValueEv": 0,
        "cumValue": null,
        "leavesQty": 0,
        "leavesValueEv": 0,
        "leavesValue": null,
        "stopLoss": null,
        "stopDirection": "UNSPECIFIED",
        "ordStatus": "Canceled",
        "transactTimeNs": 1577523473425416400
      },
      {
        "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
        "clOrdID": "uuid-1577463487504",
        "symbol": "BTCUSD",
        "side": "Sell",
        "orderType": "Limit",
        "actionTimeNs": 1577963507348468200,
        "priceEp": 71500000,
        "price": null,
        "orderQty": 700,
        "displayQty": 700,
        "timeInForce": "GoodTillCancel",
        "reduceOnly": false,
        "takeProfitEp": 0,
        "takeProfit": null,
        "stopLossEp": 0,
        "closedPnlEv": 0,
        "closedPnl": null,
        "closedSize": 0,
        "cumQty": 700,
        "cumValueEv": 9790209,
        "cumValue": null,
        "leavesQty": 0,
        "leavesValueEv": 0,
        "leavesValue": null,
        "stopLoss": null,
        "stopDirection": "UNSPECIFIED",
        "ordStatus": "Filled",
        "transactTimeNs": 1578026629824704800
      }
    ]
  }
}
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol to query | |
| start  | Integer | start time range, Epoch millis，available only from the last 2 month | |
| end  | Integer | end time range, Epoch millis | |
| offset | Integer | offset to resultset | |
| limit | Integer | limit of resultset, max 200 | |
| ordStatus | String | order status list filter | New, PartiallyFilled, Untriggered, Filled, Canceled |

## Query user order by order ID or client order ID

> Request format

Query user order history from database. Data is limited for the last 2 months.

```
GET /exchange/order?symbol=<symbol>&orderID=<orderID1,orderID2>
GET /exchange/order?symbol=<symbol>&clOrdID=<clOrdID1,clOrdID2>
```

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "orderID": "7d5a39d6-ff14-4428-b9e1-1fcf1800d6ac",
      "clOrdID": "e422be37-074c-403d-aac8-ad94827f60c1",
      "symbol": "BTCUSD",
      "side": "Sell",
      "orderType": "Limit",
      "actionTimeNs": 1577523473419470300,
      "priceEp": 75720000,
      "price": null,
      "orderQty": 12,
      "displayQty": 0,
      "timeInForce": "GoodTillCancel",
      "reduceOnly": false,
      "takeProfitEp": 0,
      "takeProfit": null,
      "stopLossEp": 0,
      "closedPnlEv": 0,
      "closedPnl": null,
      "closedSize": 0,
      "cumQty": 0,
      "cumValueEv": 0,
      "cumValue": null,
      "leavesQty": 0,
      "leavesValueEv": 0,
      "leavesValue": null,
      "stopLoss": null,
      "stopDirection": "UNSPECIFIED",
      "ordStatus": "Canceled",
      "transactTimeNs": 1577523473425416400
    },
    {
      "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
      "clOrdID": "uuid-1577463487504",
      "symbol": "BTCUSD",
      "side": "Sell",
      "orderType": "Limit",
      "actionTimeNs": 1577963507348468200,
      "priceEp": 71500000,
      "price": null,
      "orderQty": 700,
      "displayQty": 700,
      "timeInForce": "GoodTillCancel",
      "reduceOnly": false,
      "takeProfitEp": 0,
      "takeProfit": null,
      "stopLossEp": 0,
      "closedPnlEv": 0,
      "closedPnl": null,
      "closedSize": 0,
      "cumQty": 700,
      "cumValueEv": 9790209,
      "cumValue": null,
      "leavesQty": 0,
      "leavesValueEv": 0,
      "leavesValue": null,
      "stopLoss": null,
      "stopDirection": "UNSPECIFIED",
      "ordStatus": "Filled",
      "transactTimeNs": 1578026629824704800
    }
  ]
}
```

## Query user trade

> Request format

```
GET /exchange/order/trade?symbol=<symbol>&start=<start>&end=<end>&limit=<limit>&offset=<offset>&withCount=<withCount>
```

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": 79,
    "rows": [
      {
        "transactTimeNs": 1578026629824704800,
        "symbol": "BTCUSD",
        "currency": "BTC",
        "action": "Replace",
        "side": "Sell",
        "tradeType": "Trade",
        "execQty": 700,
        "execPriceEp": 71500000,
        "orderQty": 700,
        "priceEp": 71500000,
        "execValueEv": 9790209,
        "feeRateEr": -25000,
        "execFeeEv": -2447,
        "ordType": "Limit",
        "execID": "b01671a1-5ddc-5def-b80a-5311522fd4bf",
        "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
        "clOrdID": "uuid-1577463487504",
        "execStatus": "MakerFill"
      },
      {
        "transactTimeNs": 1578009600000000000,
        "symbol": "BTCUSD",
        "currency": "BTC",
        "action": "SettleFundingFee",
        "side": "Buy",
        "tradeType": "Funding",
        "execQty": 700,
        "execPriceEp": 69473435,
        "orderQty": 0,
        "priceEp": 0,
        "execValueEv": 10075793,
        "feeRateEr": 4747,
        "execFeeEv": 479,
        "ordType": "UNSPECIFIED",
        "execID": "381fbe21-a116-472d-a547-9e2368dcc194",
        "orderID": "00000000-0000-0000-0000-000000000000",
        "clOrdID": "SettlingFunding",
        "execStatus": "Init"
      }
    ]
  }
}

```

| Field     | Type     | Required | Description                                                     | Possible Values                 |
|-----------|----------|----------|-----------------------------------------------------------------|---------------------------------|
| symbol    | String   | Yes      | Trading symbol                                                  | BTCUSD, ETHUSD ...              |
| tradeType | String   | No       | Trade type of execution order                                   | Trade,Funding,AdlTrade,LiqTrade |
| start     | Integer  | No       | Epoch time in milli-seconds of range start. available only from the last 2 month                    | --                              |
| end       | Integer  | No       | Epoch time in milli-seconds of range end                        | --                              |
| limit     | Integer  | No       | The expected count of returned data-set. Default to 50. Max to 200| --                              |
| offset    | Integer  | No       | Offset of total dataset in a range                              | --                              |
| withCount | Boolean  | No       | A flag to tell if the count of total result set is required     | --                              |

* Response data would include normal trade, funding records, liquidation, ADL trades,etc.
* Field `tradeType` shall be used to identify message types.

|TradeTypes| Description |
|---------|--------------|
| Trade | Normal trades |
| Funding | Funding on positions |
| AdlTrade |  Auto-delevearage trades |
| LiqTrade | Liquidation trades |

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

## Query full order book

> Request format

```
GET /md/fullbook?symbol=<symbol>
```

> Request sample

```
GET /md/fullbook?symbol=BTCUSD
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
    "depth": 0,
    "sequence": 455476965,
    "timestamp": 1583555482434235628,
    "symbol": "BTCUSD",
    "type": "snapshot"
  }
}
```

<aside class="notice">
The depth value is 0 in full book response.
</aside>

## Query kline

> Request format

```
GET /exchange/public/md/v2/kline?symbol=<symbol>&resolution=<resolution>&limit=<limit>

```

> Response format

```javascript
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": -1,
    "rows": [[<timestamp>, <interval>, <last_close>, <open>, <high>, <low>, <close>, <volume>, <turnover>], [...]]
  }
}
```

<aside class="notice">
The API has <a href="#rate-limits">ratelimits</a> rule, and please check the <i>Other</i> group under <a href="#api-groups">API groups</a>. Kline under generation beyond the latest interval is not included in the response.</a>
</aside>

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


**NOTE**: for backward compatibility reason, phemex also provides kline query with from/to, however, this interface is **NOT** recommended.

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


## Query recent trades

> Request format

```
GET /md/trade?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       |              |

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
      .
      .
      .
    ]
  }
}
```

> Request sample

```
GET /md/trade?symbol=BTCUSD
```

> Response sample

```json
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

## Query 24 hours ticker

> Request format

```
GET /md/v1/ticker/24hr?symbol=<symbol>
```

the old url , v1/md/ticker/24hr?symbol=<symbol> , will be removed later, which have the same response format as above url.


| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       |              |

> Response format

```javascript
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
GET /md/v1/ticker/24hr?symbol=BTCUSD
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


## Query 24 hours ticker for all symbols

you can use path below to get data for all symbols, which has reponse data with array list

```
  GET /md/v1/ticker/24hr/all
```



## Query history trades by symbol

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

```javascript
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
* RateLimit of this API is 5 per second

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

## Query funding rate history

> Request format

```
GET /api-data/public/data/funding-rate-history?symbol=<symbol>&start=<start>&end=<end>&limit=<limit>
```

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "rows": [
      {
        "symbol": ".BTCFR8H",
        "fundingRate": "0.00007058",
        "fundingTime": 1680796800000,
        "intervalSeconds": 28800
      },
      {
        "symbol": ".BTCFR8H",
        "fundingRate": "0.00006672",
        "fundingTime": 1680825600000,
        "intervalSeconds": 28800
      }
    ]
  }
}
```

| Field  | Type    | Required | Description                                       | Possible Values |
|--------|---------|----------|---------------------------------------------------|-----------------|
| symbol | String  | True     | funding rate symbol                               | .BTCFR8H        |
| start  | Long    | False    | start timestamp in ms of funding time (INCLUSIVE) | 1679852520918   |
| end    | Long    | False    | end timestamp in ms of funding time (INCLUSIVE)   | 1679852520918   |
| limit  | Integer | False    | default 100, max 100                              | 100             |

* If `start` and `end` parameters are not specified, the API will return the most recent data within the specified `limit`.
* If the `start` parameter is provided while `end` is not, the API will return from `start` plus `limit` size of data.
* If the number of items between `start` and `end` exceeds the specified `limit`, the API will return from `start` plus `limit` size of data.
* The API returns data in ascending order based on the `fundingTime` attribute.

## Query funding fee history

> Request format

```
GET /api-data/futures/funding-fees?symbol=<symbol>
```

* Request parameters

| Parameter | Type    | Required | Description                   | Case                |
|-----------|---------|----------|-------------------------------|---------------------|
| symbol    | String  | True     | the currency to query         | uBTCUSDT...         |
| offset    | Integer | False    | page starts from 0            | default 0           |
| limit     | Integer | False    | page size                     | default 20, max 200 |

> Response sample

```json
[
  {
    "createTime": 0,
    "currency": "string",
    "execFeeEv": 0,
    "execPriceEp": 0,
    "execQty": 0,
    "execValueEv": 0,
    "feeRateEr": 0,
    "fundingRateEr": 0,
    "side": "string",
    "symbol": "string"
  }
]
```

## Query contract fee rate

> Request format

```
GET /api-data/futures/fee-rate?settleCurrency=<settleCurrency>
```

* Request parameters

| Parameter      | Type    | Required | Description                  | Case                  |
|----------------|---------|----------|------------------------------|-----------------------|
| settleCurrency | String  | True     | the settle currency to query | USDT,USD,BTC,ETH, ... |

**NOTE**: *RateEr (ratio scale) in response field are const 8

> Response sample

```json
{
  "symbolFeeRates": [
    {
      "takerFeeRateEr": 55000,
      "makerFeeRateEr": 6000,
      "symbol": "cETHUSD"
    }
  ]
}
```

## Query Funds Detail

> Request format

```
GET /api-data/futures/v2/tradeAccountDetail?currency=<currecny>&type=<type>&limit=<limit>&offset=<offset>&start=<start>&end=<end>&withCount=<withCount>
```

* Request parameters

| Parameter | Type    | Required | Description                    | Case                             |
|-----------|---------|----------|--------------------------------|----------------------------------|
| currency  | String  | False    | the currency to query          | USDT,USD,BTC,ETH, ...            |
| type      | Integer | False    | Funds Biz Type                 | REALIZED_PNL(1),TRANSFER(2), ... |
| start     | Integer | False    | start time range, Epoch millis | default 0                        |
| end       | Integer | False    | end time range, Epoch millis   | default 0                        |
| offset    | Integer | False    | offset to resultset, max 1000  | default 0                        |
| limit     | Integer | False    | limit of resultset             | default 20                       |
| withCount | Integer | False    | result with total count        | default false                    |

> Response sample

```json
[
  {
    "typeDesc": "REALIZED_PNL",
    "amountRv": "-0.01010945",
    "balanceRv": "1464.64081733",
    "createTime": 1682035200000,
    "currency": "USDT"
  }
]
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
    "BTCUSD"
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
    "BTCUSD",
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

## Unsubscribe orderBook

> Request sample

```json
{
  "id": 0,
  "method": "orderbook.unsubscribe",
  "params": []
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
    "BTCUSD"
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
    "BTCUSD",
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


## Subscribe account-order-position (AOP)

> Request format

```javascript
{
  "id": <id>,
  "method": "aop.subscribe",
  "params": []
}
```

AOP subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends AOP messages back to client accordingly. 0 or more latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a trading account information, holding positions, and open / max 100 closed / max 100 filled order event message history.

## Account-order-position (AOP) message

> Message format

```javascript
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
  "index_market24h": {
    "highEp": 10009,
    "lastEp": 10007,
    "lowEp": 10004, 
    "openEp": 10004,
    "symbol": ".USDT"
  },
  "timestamp": 1681496204104375396
}
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


## Unsubscribe account-order-position (AOP)

> Request format

```javascript
{
  "id": <id>,
  "method": "aop.unsubscribe",
  "params": []
}
```

It unsubscribes all account-order-positions.

## Subscribe 24 hours ticker
> Reuqest sample

```json
{
  "method": "market24h.subscribe",
  "params": [],
  "id": 0
}
```

## 24-Hours ticker message

> Message format

```javascript
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

## Subscribe price tick

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

## Tick message

> Message format

```javascript
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


# Hedged Contract Rest API


## Query product information

> Request

```
GET /public/products
```

* USDT-M perpetual contracts using USDT as margin.
* You can find products info with hedged mode under node 'perpProductsV2'.
* Contract risklimit information are defined in `.riskLimitsV2[]`.
* Contract which delisted has status with 'Delisted' .




## Place order (HTTP PUT, *prefered*)

> Request format

```
PUT /g-orders/create?clOrdID=<clOrdID>&symbol=<symbol>&reduceOnly=<reduceOnly>&closeOnTrigger=<closeOnTrigger>&orderQtyRq=<orderQtyRq>&ordType=<ordType>&priceRp=<priceRp>&side=<side>&posSide=<posSide>&text=<text>&timeInForce=<timeInForce>&stopPxRp=<stopPxRp>&takeProfitRp=<takeProfitRp>&stopLossRp=<stopLossRp>&pegOffsetValueRp=<pegOffsetValueRp>&pegPriceType=<pegPriceType>&triggerType=<triggerType>&tpTrigger=<tpTrigger>&tpSlTs=<tpSlTs>&slTrigger=<slTrigger>
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 1580547265848034600,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "98970000",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 0,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```

| Field            | Type    | Required | Description                                                  | Possible values                                              |
| ---------------- | ------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| clOrdID          | String  | -        | client order id, max length is 40                            |                                                              |
| symbol           | String  | Yes      | Which symbol to place order                                  | [Trading symbols](#symbpricesub)                             |
| reduceOnly       | Boolean | -        | whether reduce position side only. Enable this flag, i.e. reduceOnly=true, position side won't change | true, false                                                  |
| closeOnTrigger   | Boolean | -        | implicitly reduceOnly, plus cancel other orders in the same direction(side) when necessary | true, false                                                  |
| orderQtyRq       | String  | -        | Order real quantity                                          | "1"                                                          |
| ordType          | String  | -        | Order type, default to Limit                                 | Market,Limit,Stop,StopLimit,MarketIfTouched,LimitIfTouched,<br />ProtectedMarket,MarketAsLimit,StopAsLimit,<br />MarketIfTouchedAsLimit,Bracket,BoTpLimit,BoSlLimit,BoSlMarket |
| priceRp          | String  | -        | Real price, required for limit order                         | "1"                                                          |
| side             | String  | Yes      | Order direction, Buy or Sell                                 | Buy, Sell                                                    |
| posSide          | String  | Yes      | Position direction                                           | "Merged" for oneway mode , <br />"Long" / "Short" for hedge mode |
| text             | String  | -        | Order comments                                               |                                                              |
| timeInForce      | String  | -        | Time in force. default to GoodTillCancel                     | GoodTillCancel, ImmediateOrCancel, FillOrKill, PostOnly      |
| stopPxRp         | String  | -        | Trigger price for stop orders                                | "1"                                                          |
| takeProfitRp     | String  | -        | Real take profit price                                       | "1"                                                          |
| stopLossRp       | String  | -        | Real stop loss price                                         | "1"                                                          |
| pegOffsetValueRp | String  | -        | Trailing offset from current price. Negative value when position is long, positive when position is short | "1"                                                          |
| pegPriceType     | String  | -        | Trailing order price type                                    | LastPeg, MidPricePeg, MarketPeg, PrimaryPeg, TrailingStopPeg, TrailingTakeProfitPeg |
| triggerType      | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| tpTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| slTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |


## Place order (HTTP POST)

* HTTP Request:

  Request fields are the same as [above place-order](#placeorderwithput)
> Request format

```
POST /g-orders
```
body:
```json
{
  "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
  "closeOnTrigger": true,
  "displayQtyRq": "0.01",
  "ordType": "Limit",
  "orderQtyRq": "0.01",
  "pegOffsetValueRp": "1271.9",
  "pegPriceType": "LastPeg",
  "posSide": "Long",
  "priceRp": "1271.9",
  "reduceOnly": true,
  "side": "Buy",
  "slTrigger": "ByMarkPrice",
  "stopLossRp": "1271.9",
  "stopPxRp": "1271.9",
  "symbol": "BTCUSDT",
  "takeProfitRp": "1271.9",
  "text": "string",
  "timeInForce": "GoodTillCancel",
  "tpTrigger": "ByMarkPrice",
  "triggerType": "ByMarkPrice"
}
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 1580547265848034600,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "98970000",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 0,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```

| Field            | Type    | Required | Description                                                  | Possible values                                              |
| ---------------- | ------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| clOrdID          | String  | -        | client order id, max length is 40                            |                                                              |
| symbol           | String  | Yes      | Which symbol to place order                                  | [Trading symbols](#symbpricesub)                             |
| reduceOnly       | Boolean | -        | whether reduce position side only. Enable this flag, i.e. reduceOnly=true, position side won't change | true, false                                                  |
| closeOnTrigger   | Boolean | -        | implicitly reduceOnly, plus cancel other orders in the same direction(side) when necessary | true, false                                                  |
| orderQtyRq       | String  | -        | Order real quantity                                          | "1"                                                          |
| ordType          | String  | -        | Order type, default to Limit                                 | Market,Limit,Stop,StopLimit,MarketIfTouched,LimitIfTouched,<br />ProtectedMarket,MarketAsLimit,StopAsLimit,<br />MarketIfTouchedAsLimit,Bracket,BoTpLimit,BoSlLimit,BoSlMarket |
| priceRp          | String  | -        | Real price, required for limit order                         | "1"                                                          |
| side             | String  | Yes      | Order direction, Buy or Sell                                 | Buy, Sell                                                    |
| posSide          | String  | Yes      | Position direction                                           | "Merged" for oneway mode , <br />"Long" / "Short" for hedge mode |
| text             | String  | -        | Order comments                                               |                                                              |
| timeInForce      | String  | -        | Time in force. default to GoodTillCancel                     | GoodTillCancel, ImmediateOrCancel, FillOrKill, PostOnly      |
| stopPxRp         | String  | -        | Trigger price for stop orders                                | "1"                                                          |
| takeProfitRp     | String  | -        | Real take profit price                                       | "1"                                                          |
| stopLossRp       | String  | -        | Real stop loss price                                         | "1"                                                          |
| pegOffsetValueRp | String  | -        | Trailing offset from current price. Negative value when position is long, positive when position is short | "1"                                                          |
| pegPriceType     | String  | -        | Trailing order price type                                    | LastPeg, MidPricePeg, MarketPeg, PrimaryPeg, TrailingStopPeg, TrailingTakeProfitPeg |
| triggerType      | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| tpTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| slTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |


## Amend order by orderID

> Request format

```
PUT /g-orders/replace?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&price=<price>&priceRp=<priceRp>&orderQtyRq=<orderQtyRq>&stopPxRp=<stopPxRp>&takeProfitRp=<takeProfitRp>&stopLossRp=<stopLossRp>&pegOffsetValueRp=<pegOffsetValueRp>&pegPriceType=<pegPriceType>&triggerType=<triggerType>&posSide=<posSide>
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 0,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRp": "1271.9",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 0,
    "trigger": "ByMarkPrice",
    "takeProfitRp":"1271.9",
    "stopLossRp":"1271.9"
  },
  "msg": "string"
}
```

| Field            | Required | Description                           |
| ---------------- | -------- | ------------------------------------- |
| symbol           | Yes      | order symbol, cannot be changed       |
| orderID          | -        | order id, cannot be changed           |
| origClOrdID      | -        | original clOrderID, cannot be changed |
| priceRp          | -        | new order price, real value           |
| orderQtyRq       | -        | new orderQty, real value              |
| stopPxRp         | -        | new stop price, real value            |
| takeProfitRp     | -        | new stop profit price, real value     |
| stopLossRp       | -        | new stop loss price, real value       |
| pegOffsetValueRp | -        | new trailing offset, real value       |
| pegPriceType     | -        | new peg price type                    |
| triggerType      | -        | new triggerType                       |
| posSide          | Yes      | posSide to check, can not be changed  |

orderID and origClOrdID can not be both empty


## Cancel Single Order by orderID

> Request format

```
DELETE /g-orders/cancel?orderID=<orderID>&posSide=<posSide>&symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 450000000,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "0.01",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "0.01",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "0.01",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 450000000,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```

| Field   | Type   | Required | Description                  |
| ------- | ------ | -------- | ---------------------------- |
| orderID | String | No      | order id, cannot be changed, orderID and clOrdID can not be both empty  |
| clOrdID | String | No      | clOrdID id, cannot be changed  |
| symbol  | String | Yes      | which symbol to cancel order |
| posSide | String | Yes      | position direction           |


## Bulk Cancel Orders

> Request format

```
DELETE /g-orders?symbol=<symbol>&orderID=<orderID1>,<orderID2>,<orderID3>&posSide=<posSide>
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 450000000,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "0.01",
    "reduceOnly": true,
    "side": "string",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 450000000,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```

| Field   | Type   | Required | Description                                          |
| ------- | ------ | -------- | ---------------------------------------------------- |
| orderID | String | No      | list of order ids to be cancelled, cannot be changed, orderID and clOrdID can not be both empty|
| clOrdID | String | No      | list of clOrdIDs to be cancelled, cannot be changed  |
| symbol  | String | Yes      | which symbol to cancel order                         |
| posSide | String | Yes      | position direction                                   |


## Cancel All Orders

* Cancel all orders for hedge supported symbols.
* In order to cancel all orders, include conditional order and active order, one must invoke this API twice with different arguments.
  * `untriggered=false` to cancel active order including triggerred conditional order.
  * `untriggered=true` to cancel conditional order, the order is not triggerred.

> Request format

```
DELETE /g-orders/all?symbol=<symbol>&untriggered=<untriggered>&text=<text>
```

> Response sample

```json
{
  "code": 0,
  "data": 0,
  "msg": "string"
}
```

| Field       | Type    | Required | Description                          |
| ----------- | ------- | -------- | ------------------------------------ |
| symbol      | String  | -        | list of symbols to cancel all orders |
| untriggered | boolean | -        |                                      |
| text        | String  | -        |                                      |


## Query Account Positions

> Request format

```
GET /g-accounts/accountPositions?currency=<currency>&symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "data": {
    "account": {
      "accountBalanceRv": "1271.9",
      "accountId": 123450001,
      "bonusBalanceRv": "1271.9",
      "currency": "BTC",
      "totalUsedBalanceRv": "1271.9",
      "userID": 12345
    },
    "positions": [
      {
        "accountID": 123450001,
        "assignedPosBalanceRv": "1271.9",
        "avgEntryPrice": "1271.9",
        "avgEntryPriceRp": "1271.9",
        "bankruptCommRv": "1271.9",
        "bankruptPriceRp": "1271.9",
        "buyValueToCostRr": "0.1",
        "cumClosedPnlRv": "1271.9",
        "cumFundingFeeRv": "1271.9",
        "cumTransactFeeRv": "1271.9",
        "curTermRealisedPnlRv": "1271.9",
        "currency": "BTC",
        "deleveragePercentileRr": "0.1",
        "estimatedOrdLossRv": "1271.9",
        "execSeq": 0,
        "initMarginReqRr": "0.1",
        "lastFundingTimeNs": 450000000,
        "lastTermEndTimeNs": 450000000,
        "leverageRr": "0",
        "liquidationPriceRp": "1271.9",
        "maintMarginReqRr": "0.1",
        "makerFeeRateRr": "0.1",
        "markPriceRp": "1271.9",
        "posCostRv": "1271.9",
        "posMode": "Hedged",
        "posSide": "Long",
        "positionMarginRv": "1271.9",
        "positionStatus": "Normal",
        "riskLimitRv": "0.1",
        "sellValueToCostRr": "0.1",
        "side": "Sell",
        "size": "0",
        "symbol": "BTC",
        "takerFeeRateRr": "0.1",
        "term": 0,
        "transactTimeNs": 450000000,
        "usedBalanceRv": "1271.9",
        "userID": 12345,
        "valueRv": "1271.9"
      }
    ]
  },
  "msg": "string"
}
```
* Request parameters

| Field    | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| symbol   | String | -        |             |
| currency | String | Yes      |             |

* Response Fields 

| Field      | Type   | Description                                                 |
|------------|--------|-------------------------------------------------------------|
| leverageRr | Int    | when negative, cross margin; when positive, isolated margin |

## Query Account Positions with unrealized PNL

> Request format

```
GET /g-accounts/positions?currency=<currency>
```

> Response sample

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "account": {
      "userID": 4200013,
      "accountId": 42000130003,
      "currency": "USDT",
      "accountBalanceRv": "730.97309163",
      "totalUsedBalanceRv": "1.02037554",
      "bonusBalanceRv": "0"
    },
    "positions": [
      {
        "accountID": 42000130003,
        "symbol": "XEMUSDT",
        "currency": "USDT",
        "side": "Buy",
        "positionStatus": "Normal",
        "leverageRr": "-10",
        "initMarginReqRr": "0.1",
        "maintMarginReqRr": "0.01",
        "riskLimitRv": "200000",
        "sizeRq": "186",
        "valueRv": "9.951",
        "avgEntryPriceRp": "0.0535",
        "posCostRv": "1.00047354",
        "assignedPosBalanceRv": "1.086606978",
        "bankruptCommRv": "0.00001116",
        "bankruptPriceRp": "0.0001",
        "positionMarginRv": "730.97308047",
        "liquidationPriceRp": "0.0001",
        "deleveragePercentileRr": "0",
        "buyValueToCostRr": "0.10114",
        "sellValueToCostRr": "0.10126",
        "markPriceRp": "0.053036917",
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossRv": "0",
        "usedBalanceRv": "1.086606978",
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlRv": "0",
        "cumFundingFeeRv": "0",
        "cumTransactFeeRv": "0.0059706",
        "realisedPnlEv": 0,
        "unRealisedPnlRv": "-0.086133438",
        "cumRealisedPnlEv": 0,
        "term": 1,
        "lastTermEndTimeNs": 0,
        "lastFundingTimeNs": 0,
        "curTermRealisedPnlRv": "-0.0059706",
        "execSeq": 2260172450,
        "posSide": "Long",
        "posMode": "Hedged"
      },
      {
        "accountID": 42000130003,
        "symbol": "XEMUSDT",
        "currency": "USDT",
        "side": "None",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageRr": "-10",
        "initMarginReqRr": "0.1",
        "maintMarginReqRr": "0.01",
        "riskLimitRv": "200000",
        "sizeRq": "0",
        "valueRv": "0",
        "avgEntryPriceRp": "0",
        "posCostRv": "0",
        "assignedPosBalanceRv": "0",
        "bankruptCommRv": "0",
        "bankruptPriceRp": "0",
        "positionMarginRv": "0",
        "liquidationPriceRp": "0",
        "deleveragePercentileRr": "0",
        "buyValueToCostRr": "0.10114",
        "sellValueToCostRr": "0.10126",
        "markPriceRp": "0.053036917",
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossRv": "0",
        "usedBalanceRv": "0",
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlRv": "0",
        "cumFundingFeeRv": "0",
        "cumTransactFeeRv": "0",
        "realisedPnlEv": 0,
        "unRealisedPnlRv": "0",
        "cumRealisedPnlEv": 0,
        "term": 1,
        "lastTermEndTimeNs": 0,
        "lastFundingTimeNs": 0,
        "curTermRealisedPnlRv": "0",
        "execSeq": 0,
        "posSide": "Short",
        "posMode": "Hedged"
      }
    ]
  }
}
```
* Request parameters

| Field    | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| currency | String | Yes      |             |

* Response Fields 

| Field      | Type   | Description                                                 |
|------------|--------|-------------------------------------------------------------|
| leverageRr | Int    | when negative, cross margin; when positive, isolated margin |

<b>NOTE:</b> Highly recommend calculating `unRealisedPnlRv` in client side with latest `markPriceRp` to avoid ratelimit
penalty.

## Switch Position Mode Synchronously

> Request format

```
PUT /g-positions/switch-pos-mode-sync?symbol=<symbol>&targetPosMode=<targetPosMode>
```

> Response sample

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```

| Field         | Type   | Required | Description                    | Possible values |
| ------------- | ------ | -------- | ------------------------------ |-----------------|
| symbol        | String | Yes      | symbol to switch position mode |                 |
| targetPosMode | String | Yes      | the target position mode       | OneWay, Hedged  |


## Set Leverage

> Request format

```
PUT /g-positions/leverage?leverageRr=<leverage>&longLeverageRr=<longLeverageRr>&shortLeverageRr=<shortLeverageRr>&symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```

| Field           | Type   | Required | Description                                                  |
| --------------- | ------ | -------- | ------------------------------------------------------------ |
| symbol          | String | Yes      | symbol to set leverage                                       |
| leverageRr      | String | -        | new leverage value, if leverageRr exists, the position side is merged. <br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |
| longLeverageRr  | String | -        | new long leverage value, if  longLeverageRr exists, the position side is hedged.<br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |
| shortLeverageRr | String | -        | new short leverage value, if shortLeverageRr exists, the position side is hedged<br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |

## Set RiskLimit

> Request format

```
PUT /g-positions/riskLimit?posSide=<posSide>&riskLimitRv=<riskLimitRv>&symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```

| Field       | Type   | Required | Description                        |
| ----------- | ------ | -------- | ---------------------------------- |
| symbol      | String | Yes      | symbol to be set risk limt         |
| riskLimitRv | String | Yes      | real value of risk limit to be set |
| posSide     | String | Yes      | position side to set risk limit    |


## Assign Position Balance

> Request format

```
POST /g-positions/assign?posBalanceRv=<posBalanceRv>&posSide=<posSide>&symbol=<symbol>
```

> Response sample

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```

| Field        | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| symbol       | String | Yes      | symbol to assign position balance        |
| posSide      | String | Yes      | position side to assign position balance |
| posBalanceRv | String | Yes      | the position balance value               |


## Query open orders by symbol

* Order status includes `New`, `PartiallyFilled`, `Filled`, `Canceled`, `Rejected`, `Triggered`, `Untriggered`;
* Open order status includes `New`, `PartiallyFilled`, `Untriggered`;

> Request format

```
GET /g-orders/activeList?symbol=<symbol>
```

> Response sample
  * Full order
```
{
    "code": 0,
    "msg": "",
    "data": {
        "rows": [
            {
                "bizError": 0,
                "orderID": "c2621102-1cc0-4686-b520-9879311bcc26",
                "clOrdID": "",
                "symbol": "BTCUSDT",
                "side": "Sell",
                "actionTimeNs": 1678163665765381733,
                "transactTimeNs": 1678163665769528669,
                "orderType": "Limit",
                "priceRp": "22490.4",
                "orderQtyRq": "0.005",
                "displayQtyRq": "0.005",
                "timeInForce": "GoodTillCancel",
                "reduceOnly": false,
                "closedPnlRv": "0",
                "closedSizeRq": "0",
                "cumQtyRq": "0",
                "cumValueRv": "0",
                "leavesQtyRq": "0.005",
                "leavesValueRv": "112.452",
                "stopDirection": "UNSPECIFIED",
                "stopPxRp": "0",
                "trigger": "UNSPECIFIED",
                "pegOffsetValueRp": "0",
                "pegOffsetProportionRr": "0",
                "execStatus": "New",
                "pegPriceType": "UNSPECIFIED",
                "ordStatus": "New",
                "execInst": "CloseOnTrigger",
                "takeProfitRp": "0",
                "stopLossRp": "0"
            }
        ],
        "nextPageArg": ""
    }
}
```

| Field   | Type   | Description                                | Possible values |
|---------|--------|--------------------------------------------|--------------|
| symbol  | String | which symbol to query | [Trading symbols](#symbpricesub)  |


## Query closed orders by symbol

* This API is for ***closed*** orders. For open orders, please use [open order query](#queryopenorder)

> Request format

```
GET /exchange/order/v2/orderList?symbol=<symbol>&currency=<currency>&ordStatus=<ordStatus>&ordType=<ordType>&start=<start>&end=<end>&offset=<offset>&limit=<limit>&withCount=<withCount>
```

> Response sample

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 1,
        "rows": [
            {
                "createdAt": 1666179379726,
                "symbol": "ETHUSDT",
                "orderQtyRq": "0.78",
                "side": 1,
                "priceRp": "1271.9",
                "execQtyRq": "0.78",
                "leavesQtyRq": "0",
                "execPriceRp": "1271.9",
                "orderValueRv": "992.082",
                "leavesValueRv": "0",
                "cumValueRv": "992.082",
                "stopDirection": 0,
                "stopPxRp": "0",
                "trigger": 0,
                "actionBy": 1,
                "execFeeRv": "0.0012719",
                "ordType": 2,
                "ordStatus": 7,
                "clOrdId": "2739dc9",
                "orderId": "2739dc90-41c6-449f-8774-0a62c8d8e320",
                "execStatus": 6,
                "bizError": 0,
                "totalPnlRv": null,
                "avgTransactPriceRp": null,
                "orderDetailsVos": null,
                "tradeType": 1
            }
        ]
    }
}
```

| Field     | Type    | Required | Description                                                         | Possible values                                                                                                                                                                                                      |
|----|----|----|---------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol    | String  | No       | which symbol to query                                         | [Trading symbols](#symbpricesub)                                                                                                                                                                                     |
|currency|String|Yes| which currency to query                                       |                                                                                                                                                                                                                   |
| ordStatus | Integer | No       | order status code list filter                                       | New(5), PartiallyFilled(6), Untriggered(1), Filled(7), Canceled(8)                                                                                                                                                   |
| ordType   | Integer | No       | order type code list filter                                         | Market (1), Limit (2), Stop (3), StopLimit (4), MarketIfTouched (5), LimitIfTouched (6), ProtectedMarket (7), MarketAsLimit (8), StopAsLimit (9), MarketIfTouchedAsLimit (10), Bracket (11), BoTpLimit (12), BoSlLimit (13), BoSlMarket (14)    |
| start     | Integer | Yes      | start time range, Epoch millis，available only from the last 2 month ||
| end       | Integer | Yes      | end time range, Epoch millis                                        ||
| offset    | Integer | Yes      | offset to resultset                                                 ||
| limit     | Integer | Yes      | limit of resultset, max 200                                         ||
| withCount | boolean | No       | if true, result info will contains count info.                      | true,false                                                                                                                                                                                                           |

Response

| Field      | Type    | Description          | Possible values                                                                     |
|------------|---------|----------------------|-------------------------------------------------------------------------------------|
| execStatus | Integer | exec status code     | Aborted(2), MakerFill(6), TakerFill(7), Expired(8), Canceled(11), CreateRejected(19)|
| tradeType  | Integer | trade type code      | Trade(1), Funding(4), LiqTrade(6), AdlTrade(7)                                         |
| side       | Integer | side code            | Buy(1), Sell(2) |
| orderType  | Integer | order type code      | Market(1), Limit(2), Stop(3), StopLimit(4), MarketIfTouched(5), LimitIfTouched(6)     |
| ordStatus  | Integer | order status code    | Created(0), Untriggered(1), Deactivated(2), Triggered(3), Rejected(4), New(5), PartiallyFilled(6), Filled(7), Canceled(8)|
| actionBy   | Integer | action by code       | ByUser(1)                                                                           |
| action     | Integer | user code            | New(1), Cancel(2), Replace(3), CancelAll(4),  LiqRequest(11), SettleFundingFee(13)  |
| trigger    | Integer | trigger code         | UNSPECIFIED(0), ByMarkPrice(1), ByLastPrice(3)                                        |
        

## Query closed positions

> Request format

```
GET /api-data/g-futures/closedPosition?symbol=<symbol>&currency=<currency>
```

> Response sample

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 2,
        "rows": [
            {
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "term": 0,
                "closedSizeRq": 1,
                "side": 1,
                "cumEntryValueRv": None<!-- No special highlighting for None -->,
                "closedPnlRv": "-0.2",
                "exchangeFeeRv": "0.007113",
                "fundingFeeRv": "0.78",
                "finished": "0",
                "openedTimeNs": 1694542394835,
                "updatedTimeNs": 1694542398837,
                "openPrice": "5.93900000",
                "closePrice": 2,
                "roi": "-0.01008799",
                "leverage": 6
            },
            {
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "term": 0,
                "closedSizeRq": 20,
                "side": 1,
                "cumEntryValueRv": None<!-- No special highlighting for None -->,
                "closedPnlRv": "-55",
                "exchangeFeeRv": "0.7113",
                "fundingFeeRv": "3.52",
                "finished": "0",
                "openedTimeNs": 1693542394835,
                "updatedTimeNs": 1693542398837,
                "openPrice": "1998",
                "closePrice": "1888",
                "roi": "-0.28799",
                "leverage": 20
            }
        ]
    }
}
```

| Field     | Type    | Required | Description                                    | Possible values                                                    |
|-----------|---------|----------|------------------------------------------------|--------------------------------------------------------------------|
| symbol    | String  | No       | which symbol to query                          | [Trading symbols](#symbpricesub)                                   |
| currency  | String  | No       | which currency to query                        | USDT...                                                            |
| offset    | Integer | No       | offset to resultset                            |                                                                    |
| limit     | Integer | No       | limit of resultset, max 200                    |                                                                    |
| withCount | Boolean | No       | if true, result info will contains count info. | true, false                                                        |

**NOTE**:  
1\) symbol and currency cannot both be empty.<br> 
2\) user trade queries from database and its data is limited for the last 90 days.

## Query user trade

> Request format

```
GET /exchange/order/v2/tradingList?symbol=<symbol>&currency=<currency>&execType=<execType>&offset=<offset>&limit=<limit>&withCount=<withCount>
```

> Response sample

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 4,
        "rows": [
            {
                "createdAt": 1666226932259,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.01",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.78",
                "priceRp": "1271.9",
                "execValueRv": "12.719",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0012719",
                "ordType": 2,
                "execId": "8718cae",
                "execStatus": 6
            },
            {
                "createdAt": 1666226903754,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.07",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.07",
                "priceRp": "1271.9",
                "execValueRv": "89.033",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0089033",
                "ordType": 2,
                "execId": "8b8a8a0",
                "execStatus": 6
            }
        ]
    }
}
```

| Field     | Type    | Required | Description                                    | Possible values                                                    |
|-----------|---------|----------|------------------------------------------------|--------------------------------------------------------------------|
| symbol    | String  | No       | which symbol to query                          | [Trading symbols](#symbpricesub)                                   |
| currency  | String  | Yes      | which currency to query                        | USDT...                                                            |
| execType  | Integer | No       | trade type code list filter                    | Trade(1),LiqTrade(6),AdlTrade(7)                                   |
| offset    | Integer | Yes      | offset to resultset                            |                                                                    |
| limit     | Integer | Yes      | limit of resultset, max 200                    |                                                                    |
| withCount | Boolean | No       | if true, result info will contains count info. | true,false                                                         |

**NOTE**:  
1\) symbol and currency cannot both be empty.<br> 
2\) user trade queries from database and its data is limited for the last 90 days.

* Possible trade types

|TradeTypes| Description              |
|----------|--------------------------|
| Trade    | Normal trades            |
| Funding  | Funding on positions     |
| AdlTrade |  Auto-delevearage trades |
| LiqTrade | Liquidation trades       |

* Response

| Field      | Type    | Description            | Possible values                                                    |
|------------|---------|------------------------|--------------------------------------------------------------------|
| execStatus | Integer | exec status code | Aborted(2), MakerFill(6), TakerFill(7), Expired(8), Canceled(11), CreateRejected(19)|
| tradeType | Integer | trade type code | Trade(1),LiqTrade(6),AdlTrade(7) |
| side | Integer | side code | Buy(1),Sell(2) |
| orderType | Integer | order type code | Market (1),Limit (2),Stop(3),StopLimit (4),MarketIfTouched (5),LimitIfTouched (6)|
| ordStatus | Integer | order status code | Created(0),Untriggered(1),Deactivated(2),Triggered(3),Rejected(4),New(5),PartiallyFilled(6),Filled(7),Canceled(8)|
| actionBy | Integer | action by code | ByUser(1)|
| trigger | Integer | trigger code | UNSPECIFIED(0),ByMarkPrice(1),ByLastPrice(3)|
        

## Query Order Book

> Request format

```

  GET /md/v2/orderbook?symbol=<symbol>

```


> Response format

```javascript
  {
    "error": null,
    "id": 0,
    "result": {
    "orderbook_p": {
      "asks": [
        [
          <priceRp>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <priceRp>,
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

| Field   | Type    | Description           | Possible values                   |
|---------|---------|-----------------------|-----------------------------------|
| symbol  | String  | Contract symbol name  | [Trading symbols](#symbpricesub)  |


| Field      | Type     | Description                 | Possible values                  |
|------------|----------|-----------------------------|----------------------------------|
| timestamp  | Integer  | Timestamp in nanoseconds    |                                  |
| priceRp    | String   | Real book level price       |                                  |
| sizeRq     | String   | Real book level size        |                                  |
| sequence   | Integer  | current message sequence    ||
| symbol     | String   | Contract symbol name        | [Trading symbols](#symbpricesub) |

- Sample：

```
  GET /md/v2/orderbook?symbol=BTCUSDT
```

```json
  {
      "error": null,
      "id": 0,
      "result": {
          "depth": 30,
          "orderbook_p": {
              "asks": [
                  [
                      "20675.7",
                      "0.736"
                  ],
                  [
                      "20676.2",
                      "0.613"
                  ]
              ],
              "bids": [
                  [
                      "20672.4",
                      "0.818"
                  ],
                  [
                      "20672.2",
                      "0.614"
                  ]
              ]
          },
          "sequence": 77770771,
          "symbol": "BTCUSDT",
          "timestamp": 1666860123727907896,
          "type": "snapshot"
      }
  }

```

## Query kline

**NOTE**: kline interfaces have [rate limits](https://github.com/phemex/phemex-api-docs/blob/master/Generic-API-Info.en.md#rate-limits) rule,  please check the Other group under [api groups](https://github.com/phemex/phemex-api-docs/blob/master/Generic-API-Info.en.md#api-groups). Kline under generation beyond the latest interval is not included in the response.

> Request format

```
  GET /exchange/public/md/v2/kline/last?symbol=<symbol>&resolution=<resolution>&limit=<limit>
```

> Response format

```javascript
{
"code": 0,
"msg": "OK",
"data": {
"total": -1,
"rows": [[<timestamp>, <interval>, <last_close>, <open>, <high>, <low>, <close>, <volume>, <turnover>], [...]]
}
}
```

| Field      | Type    | Required | Description                 | Possible values                                                                                                                                                       |
|------------|---------|----------|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol     | String  | Yes      | which symbol to query | [Trading symbols](#symbpricesub)                                                                                                                                      |
| resolution | Integer | Yes      | Kline Interval              | MINUTE_1(60),MINUTE_5(300),MINUTE_15(900),MINUTE_30(1800),HOUR_1(3600),HOUR_4(14400),DAY_1(86400),WEEK_1(604800),MONTH_1(2592000),SEASON_1(7776000),YEAR_1(31104000)  |
| limit      | Integer | No       | records limit               | 5                                                                                                                                                                     |


* Value of `resolution`s

|resolution|Description |
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

| limit    | Description |
|----------|-------------|
| 5        | limit 5     |
| 10       | limit 10    |
| 50      | limit 50    |
| 100     | limit 100   |
| 500     | limit 500   |
| 1000    | limit 1000  |


**NOTE**: for backward compatibility reason, phemex also provides kline query with from/to, however, this interface is **NOT** recommended.

```
GET /exchange/public/md/v2/kline/list?symbol=<symbol>&to=<to>&from=<from>&resolution=<resolution>
```


| Field       | Type    | Required    | Description            | Possible Values                                                                                                |
|-------------|---------|-------------|------------------------|----------------------------------------------------------------------------------------------------------------|
|symbol       | String  | Yes         | symbol name            | BTCUSD,ETHUSD,uBTCUSD,cETHUSD,XRPUSD...                                                                        | 
| from        | Integer | Yes         | start time in seconds  | value aligned in resolution boundary                                                                           |
| to          | Integer | Yes         | end time in seconds    | value aligned in resolution boundary; Number of k-lines return between [`from`, `to`) should be less than 2000 | 
| resolution  | Integer | Yes         | kline interval         | the same as described above                                                                                    |

## Query Trade

> Request format
  ```
  GET /md/v2/trade?symbol=<symbol>
  ```

> Response format

```javascript
  {
    "error": null,
    "id": 0,
    "result": {
    "sequence": <sequence>,
    "symbol": <symbol>,
    "trades_p": [
                    [
                        <timestamp>,
                        <Side>,
                        <PriceRp>,
                        <SizeRq>
                    ],
                    [
                        <timestamp>,
                        <Side>,
                        <PriceRp>,
                        <SizeRq>
                    ],
                    ...
                    ...
                    ...
                ],
    "type": "snapshot"
    }
  }
  ```

| Field   | Type    | Description           | Possible values                   |
|---------|---------|-----------------------|-----------------------------------|
| symbol  | String  | Contract symbol name  | [Trading symbols](#symbpricesub)  |


| Field      | Type    |Description| Possible values                   |
|------------|---------|----|-----------------------------------|
| timestamp  | Integer |Timestamp in nanoseconds||
| Side       | String  |Trade Side, Buy or Sell||
| priceRp    | String  |Real trade price||
| sizeRq     | String  |Real trade size||
| sequence   | Integer |current message sequence||
| symbol     | String  |Contract symbol name| [Trading symbols](#symbpricesub)  |

- Sample:

```
  GET /md/v2/trade?symbol=BTCUSDT
  
  {
    "error": null,
    "id": 0,
    "result": {
        "sequence": 77766796,
        "symbol": "BTCUSDT",
        "trades_p": [
            [
                1666860389799499000,
                "Buy",
                "20702.1",
                "0.949"
            ],
            [
                1666860377636154600,
                "Sell",
                "20675.5",
                "0.785"
            ],
            ...
            ...
            ...
        ],
        "type": "snapshot"
    }
  }
```

## Query 24 ticker

there are two differnent response format for 24 ticker

> Request format 1 (v2 will be removed later, v3 is recommended)
```
  GET /md/v2/ticker/24hr?symbol=<symbol>
```
```
  GET /md/v2/ticker/24hr?symbol=<symbol>
```

> Response sample

```json
{
    "error": null,
    "id": 0,
    "result": {
        "closeRp": "1903.16",
        "fundingRateRr": "0.0001",
        "highRp": "1932.31",
        "indexPriceRp": "1903.62867093",
        "lowRp": "1854.52",
        "markPriceRp": "1903.16",
        "openInterestRv": "6880.97",
        "openRp": "1891.97",
        "predFundingRateRr": "0.0001",
        "symbol": "ETHUSDT",
        "timestamp": 1681349614932856300,
        "turnoverRv": "127962734.6031",
        "volumeRq": "67460.4"
    }
}
```

|Field|Type|Required|Description| Possible values                   |
|----|----|----|----|-----------------------------------|
|symbol|String|Yes|which symbol to query| [Trading symbols](#symbpricesub)  |

```
  GET /md/v2/ticker/24hr?symbol=BTCUSDT
```

> Request format 2 (recommended)
```
  GET /md/v3/ticker/24hr?symbol=<symbol>
```

> Response sample

```json
{
    "error": null,
    "id": 0,
    "result": {
        "askRp": "1906.03",
        "bidRp": "1906",
        "fundingRateRr": "0.0001",
        "highRp": "1932.31",
        "indexRp": "1906.275",
        "lastRp": "1905.63",
        "lowRp": "1854.52",
        "markRp": "1905.733555189",
        "openInterestRv": "7115",
        "openRp": "1885.18",
        "predFundingRateRr": "0.0001",
        "symbol": "ETHUSDT",
        "timestamp": 1681350167054888200,
        "turnoverRv": "128200614.5585",
        "volumeRq": "67580.6"
    }
}
```

```
  GET /md/v3/ticker/24hr?symbol=BTCUSDT
```

## Query 24 ticker for all symbols

also there are two differnent response format for 24 ticker with all ticker

you can use path below (v3 is recommended) to get data with array list

```
  GET /md/v2/ticker/24hr/all
```

```
  GET /md/v3/ticker/24hr/all  
```


## Query Orders History

> Request format

```
GET /api-data/g-futures/orders?symbol=<symbol>
```


> Response sample

```json
[
    {
        "actionTimeNs": 1667562110213260743,
        "bizError": 0,
        "clOrdId": "cfffa744-712d-867a-e397-9888eec3f6d1",
        "closedPnlRv": "0",
        "closedSizeRq": "0",
        "cumQtyRq": "0.001",
        "cumValueRv": "20.5795",
        "displayQtyRq": "0.001",
        "leavesQtyRq": "0",
        "leavesValueRv": "0",
        "orderId": "743fc923-cb01-4261-88d1-b35dba2cdac0",
        "orderQtyRq": "0.001",
        "ordStatus": "Filled",
        "ordType": "Market",
        "priceRp": "21206.7",
        "reduceOnly": false,
        "side": "Buy",
        "stopDirection": "UNSPECIFIED",
        "stopLossRp": "0",
        "symbol": "BTCUSDT",
        "takeProfitRp": "0",
        "timeInForce": "ImmediateOrCancel",
        "transactTimeNs": 1667562110221077395
    }
]
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | False    | the symbol to query       | "BTCUSDT" ...                   |
| symbols  | String         | False    | the symbols to query      | "BTCUSDT, LINKUSDT" ...         |
| currency | String         | False    | the currency to query     | "USDT" ...                      |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

**NOTE**:  
1\) symbol and currency cannot both be empty.<br> 
2\) When the symbol parameter is present, searching by symbol is prioritised.<br> 
3\) If only the currency is provided, it retrieves all symbols under that currency.<br>
4\) Searching for specific symbols under a currency needs both symbols and currency parameter.<br>
          

## Query Orders By Ids

> Request format

```
GET /api-data/g-futures/orders/by-order-id?symbol=<symbol>
```

> Response sample

```json
[
    {
        "orderId": "743fc923-cb01-4261-88d1-b35dba2cdac0",
        "clOrdId": "cfffa744-712d-867a-e397-9888eec3f6d1",
        "symbol": "BTCUSDT",
        "side": "Buy",
        "ordType": "Market",
        "actionTimeNs": 1667562110213260743,
        "priceRp": "21206.7",
        "orderQtyRq": "0.001",
        "displayQtyRq": "0.001",
        "timeInForce": "ImmediateOrCancel",
        "reduceOnly": false,
        "takeProfitRp": "0",
        "stopLossRp": "0",
        "closedPnlRv": "0",
        "closedSizeRq": "0",
        "cumQtyRq": "0.001",
        "cumValueRv": "20.5795",
        "leavesQtyRq": "0",
        "leavesValueRv": "0",
        "stopDirection": "UNSPECIFIED",
        "ordStatus": "Filled",
        "transactTimeNs": 1667562110221077395,
        "bizError": 0
    }
]
```

| Field    | Type     | Required | Description            | Possible Values                                                                                                                     |
|----------|----------|----------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| symbol   | String   | True     | the currency to query  | BTCUSDT ...                                                                                                                         |
| orderID  | String   | False    | order id               | orderID and clOrdID can not be both empty. If both IDs are given, it will return list of orders which match both orderID or clOrdID |
| clOrdID  | String   | False    | client order id        | refer to orderID                                                                                                                    |


## Query Trades History

> Request format

```
GET /api-data/g-futures/trades?symbol=<symbol>
```

> Response sample

```json
[
    {
        "transactTimeNs": 1669407633926215067,
        "symbol": "BTCUSDT",
        "currency": "USDT",
        "action": "New",
        "posSide": "Short",
        "side": "Sell",
        "tradeType": "Trade",
        "execQtyRq": "0.001",
        "execPriceRp": "16600",
        "orderQtyRq": "0.001",
        "priceRp": "16600",
        "execValueRv": "16.6",
        "feeRateRr": "0.0001",
        "execFeeRv": "0.00166",
        "closedSizeRq": "0",
        "closedPnlRv": "0",
        "ordType": "LimitIfTouched",
        "execID": "5c3d96e1-8874-53b6-b6e5-9dcc4d28b4ab",
        "orderID": "fcdfeafa-ed68-45d4-b2bd-7bc27f2b2b0b",
        "clOrdID": "",                                
        "execStatus": "MakerFill",                                                  
    }
]
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | False    | the symbol to query       | "BTCUSDT" ...                   |
| symbols  | String         | False    | the symbols to query      | "BTCUSDT, LINKUSDT" ...         |
| currency | String         | False    | the currency to query     | "USDT" ...                      |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

**NOTE**:  
1\) symbol and currency cannot both be empty.<br> 
2\) When the symbol parameter is present, searching by symbol is prioritised.<br> 
3\) If only the currency is provided, it retrieves all symbols under that currency.<br>
4\) Searching for specific symbols under a currency needs both symbols and currency parameter.<br>


## Query funding rate history

> Request format

```
GET /api-data/public/data/funding-rate-history?symbol=<symbol>&start=<start>&end=<end>&limit=<limit>
```

> Response sample

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "rows": [
      {
        "symbol": ".ETHUSDTFR8H",
        "fundingRate": "0.0001",
        "fundingTime": 1680768000000,
        "intervalSeconds": 28800
      },
      {
        "symbol": ".ETHUSDTFR8H",
        "fundingRate": "0.0001",
        "fundingTime": 1680796800000,
        "intervalSeconds": 28800
      }
    ]
  }
}
```

| Field  | Type    | Required | Description                                       | Possible Values |
|--------|---------|----------|---------------------------------------------------|-----------------|
| symbol | String  | True     | funding rate symbol                               | .ETHUSDTFR8H    |
| start  | Long    | False    | start timestamp in ms of funding time (INCLUSIVE) | 1679852520918   |
| end    | Long    | False    | end timestamp in ms of funding time (INCLUSIVE)   | 1679852520918   |
| limit  | Integer | False    | default 100, max 100                              | 100             |

* If `start` and `end` parameters are not specified, the API will return the most recent data within the specified `limit`.
* If the `start` parameter is provided while `end` is not, the API will return from `start` plus `limit` size of data.
* If the number of items between `start` and `end` exceeds the specified `limit`, the API will return from `start` plus `limit` size of data.
* The API returns data in ascending order based on the `fundingTime` attribute.

## Query funding fee history

> Request format

```
GET /api-data/g-futures/funding-fees?symbol=<symbol>
```

* Request parameters

| Parameter | Type    | Required | Description                   | Case                |
|-----------|---------|----------|-------------------------------|---------------------|
| symbol    | String  | True     | the currency to query         | BTCUSDT...          |
| offset    | Integer | False    | page starts from 0            | default 0           |
| limit     | Integer | False    | page size                     | default 20, max 200 |

> Response sample

```json
[
  {
    "symbol": "ETHUSDT",
    "currency": "USDT",
    "execQtyRq": "0.16",
    "side": "Buy",
    "execPriceRp": "1322.84500459",
    "execValueRv": "211.65520073",
    "fundingRateRr": "0.0001",
    "feeRateRr": "0.0001",
    "execFeeRv": "0.02116552",
    "createTime": 1671004800021
  }
]
```

# Hedged Contract Websocket API
* Each client is required to actively send heartbeat (ping) message to Phemex data gateway ('DataGW' in short) with interval less than 30 seconds, otherwise DataGW will drop the connection. If a client sends a ping message, DataGW will reply with a pong message.
* Clients can use WS built-in ping message or the application level ping message to DataGW as heartbeat. The heartbeat interval is recommended to be set as *5 seconds*, and actively reconnect to DataGW if don't receive messages in *3 heartbeat intervals*.
 

API Rate Limits

* Each Client has concurrent connection limit to *5* in maximum.
* Each connection has subscription limit to *20* in maximum.
* Each connection has throttle limit to *20* request/s.

## Heartbeat

> Request format
```json
{
  "id": 1234,
  "method": "server.ping",
  "params": []
}
```

> Response sample

```json
{
  "error": null,
  "id": 1234,
  "result": "pong"
}
```

## API User Authentication

Market trade/orderbook are published publicly without user authentication.
While for client private account/position/order data, the client should send user.auth message to Data Gateway to authenticate the session.

> Request format
```json
{
  "method": "user.auth",
  "params": [
    "API",
    "<token>",
    "<signature>",
    "<expiry>"
  ],
  "id": 1234
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
  "id": 1234
}
```

> Response sample
```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| type        | String | Token type       | API             |
| token       | String | API Key     |                 |
| signature   | String | Signature generated by a funtion as HMacSha256(API Key + expiry) with ***API Secret*** ||
| expiry      | Integer| A future time after which request will be rejected, in epoch ***second***. Maximum expiry is request time plus 2 minutes ||


## Subscribe OrderBook for new Model

On each successful subscription, DataGW will immediately send the current Order Book snapshot to client and all later order book updates will be published.


> Request sample
```json
{
  "id": 1234,
  "method": "orderbook_p.subscribe",
  "params": [
    "BTCUSDT"
  ]
}
```
> Response sample
```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```


## OrderBook Message:

DataGW publishes order book message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published with 60-second interval for client self-verification.

> Response format
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
}

```

* Sample：

> Response sample
```json
{"depth":30,"orderbook_p":{"asks":[["20702.9","0.718"],["20703.9","0.524"],["20704.9","0"],["20720.8","0"]],"bids":[["20703.1","0"],["20701.3","0"],["20701.2","0"],["20700.5","1.622"],["20473.7","1.074"],["20441.3","0.904"]]},"sequence":77668172,"symbol":"BTCUSDT","timestamp":1666854171201355264,"type":"incremental"}
```
```json
{"depth":30,"orderbook_p":{"asks":[],"bids":[["20700.5","0"],["20340.5","0.06"]]},"sequence":77668209,"symbol":"BTCUSDT","timestamp":1666854173705089711,"type":"incremental"}
```

| Field       | Type    | Description             | Possible values |
|-------------|---------|-------------------------|-----------------|
| side        | String  | Price level side        | bid, ask        |
| priceEp     | String  | Raw price               |                 |
| qty         | String  | Price level size        |                 |
| sequence    | Integer | Latest message sequence |          |
| depth       | Integer | Market depth            | 30              |
| type        | String  | Message type            | snapshot, incremental |


## Unsubscribe OrderBook

It unsubscribes all orderbook related subscriptions.

> Request format

```javascript
{
  "id": <id>,
  "method": "orderbook_p.unsubscribe",
  "params": []
}
```

> Response format
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


## Subscribe Trade

On each successful subscription, DataGW will send the 200 history trades immediately for the subscribed symbol and all the later trades will be published.


> Request sample

```json
{
  "id": 1234,
  "method": "trade_p.subscribe",
  "params": [
    "BTCUSDT"
  ]
}
```
> Response sample

```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

## Trade Message Format：

DataGW publishes trade message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.


> Response format
```javascript
{
  "trades": [
    [
      <timestamp>,
      "<side>",
      "<price>",
      "<qty>"
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


> Response sample
```json
{
    "sequence": 77702250,
    "symbol": "BTCUSDT",
    "trades_p": [
        [
            1666856076819029800,
            "Sell",
            "20700.3",
            "0.649"
        ]
    ],
    "type": "incremental"
}
```
```json
{
    "sequence": 77663551,
    "symbol": "BTCUSDT",
    "trades_p": [
        [
            1666856062351916300,
            "Sell",
            "20703.6",
            "0.669"
        ],
        [
            1666854025545354000,
            "Buy",
            "20699",
            "0.001"
        ]
    ],
    "type": "snapshot"
}
```

| Field     | Type   | Description                             | Possible values |
|-----------|--------|-----------------------------------------|-----------------|
| timestamp | Integer| Timestamp in nanoseconds for each trade ||
| side      | String | Execution taker side                    | bid, ask        |
| price     | String| Raw price                               |                 |
| qty       | String| Execution size                          |                 |
| sequence  | Integer| Latest message sequence                 ||
| symbol    | String | Contract symbol name                    ||
| type      | String | Message type                            |snapshot, incremental |



## Unsubscribe Trade

It unsubscribes all trade subscriptions or for a symbol.

> Request format
* unsubscribe all trade subsciptions
```javascript
{
  "id": <id>,
  "method": "trade_p.unsubscribe",
  "params": [
  ]
}
```
> Request format
* unsubscribe all trade subsciptions for a symbol
```javascript
{
  "id": <id>,
  "method": "trade_p.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

> Response sample
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

## Subscribe Kline

On each successful subscription, DataGW will send the 1000 history klines immediately for the subscribed symbol and all the later kline update will be published in real-time.

> Request format
```javascript
{
  "id": <id>,
  "method": "kline_p.subscribe",
  "params": [
    "<symbol>",
    "<interval>"
  ]
}
```

> Response format
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


> Request sample
* subscribe 1-day kline
```json
{
  "id": 1234,
  "method": "kline_p.subscribe",
  "params": [
    "BTCUSDT",
    86400
  ]
}
```

> Response sample
```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

## Kline Message Format：

DataGW publishes kline message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.

> Response format
```javascript
{
  "kline": [
    [
      <timestamp>,
      "<interval>",
      <lastClose>,
      <open>,
      <high>,
      <low>,
      <close>,
      <volume>,
      <turnover>,
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


> Response sample
```json
{
    "kline_p": [
        [
            1666856340,
            60,
            "20689.5",
            "20686.2",
            "20695.4",
            "20686.2",
            "20691.6",
            "2.742",
            "56731.5609"
        ],
        [
            1666856280,
            60,
            "20700.1",
            "20712.7",
            "20712.7",
            "20689.1",
            "20689.5",
            "4.407",
            "91208.3065"
        ]        
    ],
    "sequence": 77711279,
    "symbol": "BTCUSDT",
    "type": "snapshot"
}
```
```json
{
    "kline_p": [
        [
            1666856520,
            60,
            "20685",
            "20684.8",
            "20684.8",
            "20675.2",
            "20675.2",
            "3.547",
            "73353.8417"
        ]
    ],
    "priceScale": 0,
    "sequence": 77715046,
    "symbol": "BTCUSDT",
    "type": "incremental"
}
```

| Field      | Type    | Description                                    | Possible values |
|------------|---------|------------------------------------------------|-----------------|
| timestamp  | Integer | Timestamp in nanoseconds for each trade        ||
| interval   | Integer | Kline interval type                            | 60, 300, 900, 1800, 3600, 14400, 86400, 604800, 2592000, 7776000, 31104000 |
| lastClose  | String  | Unscaled last close price                      |                 |
| open       | String  | Unscaled open price                              |                 |
| high       | String  | Unscaled high price                              |                 |
| low        | String  | Unscaled low price                               |                 |
| close      | String  | Unscaled close price                             |                 |
| volume     | String  | Trade voulme during the current kline interval ||
| turnover   | String  | Unscaled turnover value                          |                 |
| sequence   | Integer | Latest message sequence                        ||
| symbol     | String  | Contract symbol name                           ||
| type       | String  | Message type                                   |snapshot, incremental |


## Unsubscribe Kline

It unsubscribes all kline subscriptions or for a symbol.

> Request format
* unsubscribe all Kline subscriptions
```javascript
{
  "id": <id>,
  "method": "kline_p.unsubscribe",
  "params": []
}
```
> Request format
* unsubscribe all Kline subscriptions of a symbol
```javascript
{
  "id": <id>,
  "method": "kline_p.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

> Response sample
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


## Subscribe Account-Order-Position (AOP)

AOP subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends AOP messages back to client accordingly. 0 or more latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a trading account information, holding positions, and open / max 100 closed / max 100 filled order event message history.

> Request format
```javascript
{
  "id": <id>,
  "method": "aop_p.subscribe",
  "params": []
}
```

> Response format
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

> Request sample
```json
{
  "id": 1234,
  "method": "aop_p.subscribe",
  "params": []
}
```
> Response sample
```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

## Account-Order-Position (AOP) Message Sample:

> Response sample
```json
{"index_market24h": {"highEp": 10009,"lastEp": 10007,"lowEp": 10004, "openEp": 10004,"symbol": ".USDT"},"timestamp": 1681496204104375396}
{"accounts_p":[{"accountBalanceRv":"1508.452588802237","accountID":9328670003,"bonusBalanceRv":"0","currency":"USDT","totalUsedBalanceRv":"343.132599666883","userID":932867}],"orders_p":[{"accountID":9328670003,"action":"New","actionBy":"ByUser","actionTimeNs":1666858780876924611,"addedSeq":77751555,"apRp":"0","bonusChangedAmountRv":"0","bpRp":"0","clOrdID":"c0327a7d-9064-62a9-28f6-2db9aaaa04e0","closedPnlRv":"0","closedSize":"0","code":0,"cumFeeRv":"0","cumQty":"0","cumValueRv":"0","curAccBalanceRv":"1508.489893982237","curAssignedPosBalanceRv":"24.62786650928","curBonusBalanceRv":"0","curLeverageRr":"-10","curPosSide":"Buy","curPosSize":"0.043","curPosTerm":1,"curPosValueRv":"894.0689","curRiskLimitRv":"1000000","currency":"USDT","cxlRejReason":0,"displayQty":"0.003","execFeeRv":"0","execID":"00000000-0000-0000-0000-000000000000","execPriceRp":"20723.7","execQty":"0","execSeq":77751555,"execStatus":"New","execValueRv":"0","feeRateRr":"0","leavesQty":"0.003","leavesValueRv":"63.4503","message":"No error","ordStatus":"New","ordType":"Market","orderID":"fa64c6f2-47a4-4929-aab4-b7fa9bbc4323","orderQty":"0.003","pegOffsetValueRp":"0","posSide":"Long","priceRp":"21150.1","relatedPosTerm":1,"relatedReqNum":11,"side":"Buy","slTrigger":"ByMarkPrice","stopLossRp":"0","stopPxRp":"0","symbol":"BTCUSDT","takeProfitRp":"0","timeInForce":"ImmediateOrCancel","tpTrigger":"ByLastPrice","tradeType":"Amend","transactTimeNs":1666858780881545305,"userID":932867},{"accountID":9328670003,"action":"New","actionBy":"ByUser","actionTimeNs":1666858780876924611,"addedSeq":77751555,"apRp":"0","bonusChangedAmountRv":"0","bpRp":"0","clOrdID":"c0327a7d-9064-62a9-28f6-2db9aaaa04e0","closedPnlRv":"0","closedSize":"0","code":0,"cumFeeRv":"0","cumQty":"0.003","cumValueRv":"62.1753","curAccBalanceRv":"1508.452588802237","curAssignedPosBalanceRv":"24.62786650928","curBonusBalanceRv":"0","curLeverageRr":"-10","curPosSide":"Buy","curPosSize":"0.046","curPosTerm":1,"curPosValueRv":"956.2442","curRiskLimitRv":"1000000","currency":"USDT","cxlRejReason":0,"displayQty":"0.003","execFeeRv":"0.03730518","execID":"b6c8d16b-c777-510c-8476-80f399b2d5ad","execPriceRp":"20725.1","execQty":"0.003","execSeq":77751555,"execStatus":"TakerFill","execValueRv":"62.1753","feeRateRr":"0.0006","lastLiquidityInd":"RemovedLiquidity","leavesQty":"0","leavesValueRv":"0","message":"No error","ordStatus":"Filled","ordType":"Market","orderID":"fa64c6f2-47a4-4929-aab4-b7fa9bbc4323","orderQty":"0.003","pegOffsetValueRp":"0","posSide":"Long","priceRp":"21150.1","relatedPosTerm":1,"relatedReqNum":11,"side":"Buy","slTrigger":"ByMarkPrice","stopLossRp":"0","stopPxRp":"0","symbol":"BTCUSDT","takeProfitRp":"0","timeInForce":"ImmediateOrCancel","tpTrigger":"ByLastPrice","tradeType":"Trade","transactTimeNs":1666858780881545305,"userID":932867}],"positions_p":[{"accountID":9328670003,"assignedPosBalanceRv":"30.861734862748","avgEntryPriceRp":"20787.917391304","bankruptCommRv":"0.0000006","bankruptPriceRp":"0.1","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.10114","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"0.089061821453","cumTransactFeeRv":"0.57374652","curTermRealisedPnlRv":"-0.662808341453","currency":"USDT","dataVer":11,"deleveragePercentileRr":"0","displayLeverageRr":"0.79941382","estimatedOrdLossRv":"0","execSeq":77751555,"freeCostRv":"0","freeQty":"-0.046","initMarginReqRr":"0.1","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"-10","liquidationPriceRp":"0.1","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"20735.47347096","minPosCostRv":"0","orderCostRv":"0","posCostRv":"30.284669572349","posMode":"Hedged","posSide":"Long","positionMarginRv":"1196.181723398102","positionStatus":"Normal","riskLimitRv":"1000000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.10126","side":"Buy","size":"0.046","symbol":"BTCUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780881545305,"unrealisedPnlRv":"-2.41242033584","updatedAtNs":0,"usedBalanceRv":"30.861734862748","userID":932867,"valueRv":"956.2442"},{"accountID":9328670003,"assignedPosBalanceRv":"9.473634984","avgEntryPriceRp":"20786.455555556","bankruptCommRv":"1.153171711445","bankruptPriceRp":"1000000","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.10114","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"-0.074563385402","cumTransactFeeRv":"0.44898744","curTermRealisedPnlRv":"-0.374424054598","currency":"USDT","dataVer":10,"deleveragePercentileRr":"0","displayLeverageRr":"0.63759936","estimatedOrdLossRv":"0","execSeq":77731059,"freeCostRv":"0","freeQty":"0.036","initMarginReqRr":"0.1","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"-10","liquidationPriceRp":"1000000","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"20735.47347096","minPosCostRv":"0","orderCostRv":"0","posCostRv":"9.473634984","posMode":"Hedged","posSide":"Short","positionMarginRv":"1173.640452407909","positionStatus":"Normal","riskLimitRv":"1000000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.10126","side":"Sell","size":"0.036","symbol":"BTCUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"1.83535504544","updatedAtNs":0,"usedBalanceRv":"9.473634984","userID":932867,"valueRv":"748.3124"},{"accountID":9328670003,"assignedPosBalanceRv":"156.56916092763","avgEntryPriceRp":"1563.815","bankruptCommRv":"0.187657798123","bankruptPriceRp":"1042.55","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.33433334","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"-89.82","cumFundingFeeRv":"0.104061681397","cumTransactFeeRv":"0.4835887","curTermRealisedPnlRv":"-0.328183513333","currency":"USDT","dataVer":14,"deleveragePercentileRr":"0","displayLeverageRr":"2.99999994","estimatedOrdLossRv":"0","execSeq":77731060,"freeCostRv":"0","freeQty":"-0.3","initMarginReqRr":"0.33333333","lastFundingTime":1666857600000000000,"lastTermEndTime":1666755074905395643,"leverageRr":"3","liquidationPriceRp":"1058.2","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"1558.51230846","minPosCostRv":"0","orderCostRv":"0","posCostRv":"156.56916092763","posMode":"Hedged","posSide":"Long","positionMarginRv":"156.381503129507","positionStatus":"Normal","riskLimitRv":"500000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.33473333","side":"Buy","size":"0.3","symbol":"ETHUSDT","takerFeeRateRr":"-1","term":2,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"-1.590807462","updatedAtNs":0,"usedBalanceRv":"156.56916092763","userID":932867,"valueRv":"469.1445"},{"accountID":9328670003,"assignedPosBalanceRv":"146.228068892505","avgEntryPriceRp":"1455.495","bankruptCommRv":"0.349516231597","bankruptPriceRp":"1941.75","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.33433334","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"-0.159460679685","cumTransactFeeRv":"0.2619891","curTermRealisedPnlRv":"-0.102528420315","currency":"USDT","dataVer":14,"deleveragePercentileRr":"0","displayLeverageRr":"2.99323302","estimatedOrdLossRv":"0","execSeq":77731060,"freeCostRv":"0","freeQty":"0.3","initMarginReqRr":"0.33333333","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"3","liquidationPriceRp":"1927.21","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"1558.51230846","minPosCostRv":"0","orderCostRv":"0","posCostRv":"145.898817344505","posMode":"Hedged","posSide":"Short","positionMarginRv":"145.878552660908","positionStatus":"Normal","riskLimitRv":"500000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.33473333","side":"Sell","size":"0.3","symbol":"ETHUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"-30.905192538","updatedAtNs":0,"usedBalanceRv":"146.228068892505","userID":932867,"valueRv":"436.6485"}],"sequence":68744,"timestamp":1666858780883525030,"type":"incremental","version":0}

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| symbol      | String | Contract symbol name    |          |
| type        | String | Message type     | snapshot, incremental |



## Unsubscribe Account-Order-Position (AOP)

> Request format
```javascript
{
  "id": <id>,
  "method": "aop_p.unsubscribe",
  "params": []
}
```

> Response format
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


## Subscribe 24 Hours Ticker
On each successful subscription, DataGW will publish 24-hour ticker metrics for all symbols every 1 second.

> Request format
```javascript
* Subscribe single symbol
{
  "id": <id>,
  "method": "market24h_p.subscribe",
  "params": []
}

* Subscribe all symbols
{
  "id": <id>,
  "method": "perp_market24h_pack_p.subscribe",
  "params": []
}
```

> Response format
```javascript
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

> Request sample
```json
{
  "method": "perp_market24h_pack_p.subscribe",
  "params": [],
  "id": 1234
}
```

> Response sample
```json
{
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

## Hours Ticker Message Format：

> Response format
```javascript
{
    "data": [
        [
            <symbol>,
            <openRp>,
            <highRp>,
            <lowRp>,
            <lastRp>,
            <volumeRq>,
            <turnoverRv>,
            <openInterestRv>,
            <indexRp>,
            <markRp>,
            <fundingRateRr>,
            <predFundingRateRr>
        ]
    ],
    "fields": [
        "symbol",
        "openRp",
        "highRp",
        "lowRp",
        "lastRp",
        "volumeRq",
        "turnoverRv",
        "openInterestRv",
        "indexRp",
        "markRp",
        "fundingRateRr",
        "predFundingRateRr"
    ],
    "method": "perp_market24h_pack_p.update",
    "timestamp": 1666862556850547000,
    "type": "snapshot"
}
```


> Response sample
```json
{
    "data": [
        [
            "ETHUSDT",
            "1533.72",
            "1594.17",
            "1510.05",
            "1547.52",
            "545942.34",
            "848127644.5712",
            "0",
            "1548.31694379",
            "1548.44513153",
            "0.0001",
            "0.0001"
        ],
        [
            "BTCUSDT",
            "20614.5",
            "21628.4",
            "19258.6",
            "20626.3",
            "8819.819",
            "182892627.4297",
            "0",
            "20641.8167574",
            "20643.52572781",
            "0.0001",
            "0.0001"
        ]
    ],
    "fields": [
        "symbol",
        "openRp",
        "highRp",
        "lowRp",
        "lastRp",
        "volumeRq",
        "turnoverRv",
        "openInterestRv",
        "indexRp",
        "markRp",
        "fundingRateRr",
        "predFundingRateRr"
    ],
    "method": "perp_market24h_pack_p.update",
    "timestamp": 1666862556850547000,
    "type": "snapshot"
}
```

| Field             | Type    | Description                                  | Possible values                  |
|-------------------|---------|----------------------------------------------|----------------------------------|
| symbol            | String  | Contract symbol name                         | [Trading symbols](#symbpricesub) |
| openRp            | String  | The unscaled open price in last 24 hours     |                                  |
| highRp            | String  | The unscaled highest price in last 24 hours  |                                  |
| lowRp             | String  | The unscaled lowest price in last 24 hours   |                                  |
| lastRp            | String  | The unscaled close price in last 24 hours    |                                  |
| volumeRq          | String  | Symbol trade volume in last 24 hours         |                                  |
| turnoverRv        | String  | The unscaled turnover value in last 24 hours |                                  |
| openInterestRv    | String  | current open interest                        |                                  |
| indexRp           | String  | Unscaled index price                         |                                  |
| markRp            | String  | Unscaled mark price                          |                                  |
| fundingRateRr     | String  | Unscaled funding rate                        |                                  |
| predFundingRateRr | String  | Unscaled predicated funding rate             |                                  |



<a id="symbpricesub"></a>

## Subscribe tick event for symbol price

* Every trading symbol has a suite of other symbols, each symbol follows same patterns,
  i.e. `index` symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>`,
  `mark` symbol follows a pattern `.M<BASECURRENCY><QUOTECURRENCY>`,
  predicated funding rate's symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>FR`,
  while funding rate symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>FR8H`
* Price is retrieved by subscribing symbol tick.
* all available symbols (pfr=predicated funding rate)

| symbol  | index symbol  | mark symbol       | pfr symbol   | funding rate symbol |
|---------|---------------|-------------------|---------------|--------------|
| BTCUSDT | .BTCUSDT      | .MBTCUSDT         | .BTCUSDTFR       | .BTCUSDTFR8H            |
| ETHUSDT | .ETHUSDT      | .METHUSDT         | .ETHUSDTFR       | .ETHUSDTFR8H            |
| XRPUSDT | .XRPUSDT      | .MXRPUSDT         | .XRPUSDTFR       | .XRPUSDTFR8H            |
| ADAUSDT | .ADAUSDT      | .MADAUSDT         | .ADAUSDTFR       | .ADAUSDTFR8H            |


> Request format
* The symbol in params can be replace by any symbol.
```javascript
{
    "method": "tick_p.subscribe",
        "params": [ <symbol> ],
        "id": <id>
}
```

> Response format
```javascript
{
    "error": null,
        "id": <id>,
        "result": {
            "status": "success"
        }
}
```

## push event

> Response format
```javascript
{
    "tick": {
        "last": <price>,
        "symbol": <symbol>
        "timestamp": <timestamp_nano>
    }
}
```


> Response sample
```json
{"tick_p":{"last":"20639.38692364","symbol":".BTCUSDT","timestamp":1666863393552000000}}
```
```json
{"tick_p":{"last":"20639.15408363","symbol":".BTCUSDT","timestamp":1666863394538132741}}
```

