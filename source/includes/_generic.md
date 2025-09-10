# Overview
Welcome to the [Phemex](https://phemex.com) API documentation. We offer REST and Websocket APIs to interact with our systems.

## General API information
* vip user endpoints (for whitelisted client IPs only):
  * Rest API: `https://vapi.phemex.com`
  * Websocket: `wss://vapi.phemex.com/ws`
* public user endpoints:
  * Rest API: `https://api.phemex.com`
  * Websocket (further URI or querystring not allowed): `wss://ws.phemex.com`
* testnet user endpoints:
  * Rest API: `https://testnet-api.phemex.com`
  * Websocket: `wss://testnet-api.phemex.com/ws`
* Phemex provides HTTP Rest API for client to operate orders, all endpoints return a JSON object.

## REST API Standards
### HTTP return codes

* HTTP `401` return code is used when unauthenticated
* HTTP `403` return code is used when lack of priviledge.
* HTTP `429` return code is used when breaking a request ratelimit.
* HTTP `5XX` return codes are used for Phemex internal errors. Note: This doesn't means the operation failed, the execution status is **UNKNOWN** and could be Succeed.

### REST request header

Every HTTP Rest Request must have the following Headers:

* **x-phemex-access-token** : This is *API-KEY* (id field) from Phemex site.
* **x-phemex-request-expiry** : This describes the Unix *EPoch SECONDS* to expire the request, normally it should be (Now() + 1 minute)
* **x-phemex-request-signature** : This is HMAC SHA256 signature of the http request. Secret is *API Secret*, its formula is : HMacSha256(URL Path + QueryString + Expiry + body)
* (Optional) **x-phemex-request-tracing**: a unique string to trace http-request, less than 40 bytes. This header is a must in resolving latency issues.

### Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case sensitive**.

#### Signature example 1: HTTP GET request

* API REST Request URL: `https://api.phemex.com/accounts/accountPositions?currency=BTC`
   
   
| Field | Value |
|------|-----|
| Request Path | `/accounts/accountPositions` |
| Request Query | `currency=BTC` |
| Request Body | `<null>` |
| Request Expiry | `1575735514` |
| Signature | `HMacSha256(/accounts/accountPositions + currency=BTC + 1575735514)` |

#### Singature example 2: HTTP GET request with multiple query string

* API REST Request URL: `https://api.phemex.com/orders/activeList?ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD`
    
| Field | Value |
|------|-----|
| Request Path | `/orders/activeList` |
| Request Query | `ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD` |
| Request Body | `<null>` |
| Request Expire | `1575735951` |
| Signature | `HMacSha256(/orders/activeList + ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD + 1575735951)` |
| Signed string | `/orders/activeListordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD1575735951` |

#### Signature example 3: HTTP POST request

* API REST Request URL: `https://api.phemex.com/orders`

| Field | Value |
|------|-----|
| Request Path | `/orders` |
| Request Query | `<null>` |
| Request Body | `{"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}` |
| Request Expiry | `1575735514` |
| Signature | `HMacSha256( /orders + 1575735514 + {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0})` |
| Signed string | `/orders1575735514{"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}` |

### REST response format

> Response general format

```javascript
{
  "code": <code>,
  "msg": <msg>,
  "data": <data>
}
```

* All restful API except ***starting*** with `/md` shares same response format.

| Field | Description |
|-------|------|
| code | 0 means `success`, non-zero means `error`|
| msg  | when code is non-zero, it gives short error description |
| data | operation dependant |

## Rate limits
* In order to protect the exchange, Phemex apply `API RateLimit` and `IP Ratelimit` on all requests.
* Rest API has a request capacity in one minute window on user basis.
* IP has request capacity in 5 minutes window.
* Ratelimit of API is independant of that in WEB/APP, so if one get ratelimited in API, one can place/cancel orders via WEB or APP.
* all requests to the domain `testnet-api.phemex.com` shared across the rate limit 500/5m in total.


### IP ratelimits
* Currently Phemex restricts every IP 5,000 requests in 5 minutes window. If exceeded this IP capacity, the user would be blocked in the following 5 minutes.
* `wss://ws.phemex.com` 200/5m per IP address, not mandatory to bind IP or get whitelisted.

### REST API ratelimit rules
* [Order spamming limitations](https://phemex.com/user-guides/order-spamming-limitations)
* All Phemex APIs are divided into 3 groups, `contract`, `spotOrder` and `others`.
* All APIs in the same group share a request capacity.
* Every API consume request shares at its own weight.
* If exceeded the ratelimit, http status 429 will returned, together with a reset seconds header `x-ratelimit-retry-after-<groupName>`
* Ratelimit headers
  Below headers are returned with API response. Postfix `-<groupName>` is empty if group is others

| Header name  | Description |
|--------------|-------------|
| x-ratelimit-remaining-*groupName*   | Remaining request permits in this minute |
| x-ratelimit-capacity-*groupName*    | Request ratelimit capacity |
| x-ratelimit-retry-after-*groupName* | Reset timeout in seconds for current ratelimited user |

* Group capacity

| Group Name | Capacity |
|------------|---------|
| Contract   |  500/minute |
| SpotOrder  |  500/minute |
| Others     |  100/minute |

### New Contract API RateLimit Rules (for VAPI/VIP only)
* Contract API currently employs new rate-limit rules based on symbols, according to Phemex internal configuration of uid individually.
* Under the new throttling rules, Contract API consumes both contract group capacity (5000/minute) and symbol group capacity (500/minute) at the same time, but the each capacity of different symbols are independent from one another.
* Contract API that may involve all symbols like 'cancel all' consumes request capacity under a special group named 'CONTACT_ALL_SYM'.

* Ratelimit headers added in the symbol dimension. 

| Header name                                     | Description                                                               |
|-------------------------------------------------|---------------------------------------------------------------------------|
| x-ratelimit-remaining-*groupName_symbol*        | Remaining request ratelimit in the symbol group in this minute              |
| x-ratelimit-capacity-*groupName_symbol*         | Total request capacity in the symbol group                            |
| x-ratelimit-retry-after-*groupName_symbol*      | Reset timeout in seconds for current ratelimited user in the symbol group |

* Special Contract Group Capacity

| Group Name      | Capacity    |
|-----------------|-------------|
| Contract        | 5000/minute |
| Contract_SYMBOL | 500/minute  |
| CONTACT_ALL_SYM | 500/minute  |

### API groups
* Contract group
  Contract group is for contract trading, it contains following API.

| Path | Method | Weight | Description |
|------|--------|--------|-------------|
| /orders | POST | 1 | Place new order  |
| /orders/replace | PUT | 1 | Amend order |
| /orders/cancel | DELETE | 1 | Cancel order |
| /orders/all | DELETE | 3 | Cancel all order by symbol |
| /orders | DELETE | 1 | cancel orders |
| /orders/activeList | GET | 1 | Query open orders by symbol |
| /orders/active | GET | 1 | Query open order by orderID  |
| /accounts/accountPositions | GET | 1 | Query account & position by currency |
| /accounts/positions | GET | 25 | Query positions with unrealized PNL |
| /g-orders | POST | 1 | Place new order |
| /g-orders/replace | PUT | 1 | Amend order |
| /g-orders/cancel | DELETE | 1 | Cancel order |
| /g-orders/all | DELETE | 3 | Cancel All Orders |
| /g-orders | DELETE | 1 | Bulk Cancel Orders |
| /g-orders/activeList | GET | 1 | Query open orders by symbol |
| /g-orders/active | GET | 1 | Query open order by orderID |
| /g-orders/create | PUT | 1 | Place order |
| /g-accounts/accountPositions | GET | 1 | Query Account Positions |
| /g-accounts/positions | GET | 25 | Query Account Positions with unrealized PNL |
| /g-positions/assign | POST | 1 | Assign Position Balance |
| /g-positions/leverage | PUT | 1 | Set Leverage |
| /g-positions/riskLimit | PUT | 1 | Set RiskLimit |
| /g-positions/switch-pos-mode-sync | PUT | 1 | /g-positions/switch-pos-mode-sync |

* SpotOrder group
  SpotOrder group is for spot trading, which contains following API.

| Path | Method | Weight | Description |
|------|--------|--------|-------------|
| /spot/orders | POST | 1 | Place spot order |
| /spot/orders | PUT | 1 | Amend spot order |
| /spot/orders | DELETE | 2 | Cancel spot order  |
| /spot/orders/all | DELETE | 2 | Cancel spot orders by symbol |
| /spot/orders/active | GET | 1 | Query open spot order |
| /spot/orders | GET | 1 | Query all open spot orders by symbol |

* Others
  APIs which is not in contract or spotOrder group is categorized into other group.
  Because the number of interfaces in Other group may grow over time, the list below is not exhausted, the items in the table highlight interfaces that has weight other than 1, or interfaces that Phemex deems important.

| Path | Method | Weight | Description |
|------|--------|--------|-------------|
| /exchange/public/md/kline | GET | 10      | kline query |

## Contact us
* [Phemex API Telegram Group](https://t.me/Phemex_API)
* [Phemex Customer Support](https://phemex.com/support)

## CCXT integration
* CCXT is our authorized SDK provider and you may access the Phemex API through CCXT. For more information, please visit: https://ccxt.trade.
* [CCXT GitHub](https://github.com/ccxt/ccxt)

## Code samples
[Java Sample Client](https://github.com/phemex/java-client)<br>   
[Python Sample Client](https://github.com/phemex/phemex-python-api)<br>    
[NodeJS Sample Client](https://github.com/phemex/phemex-node-example)<br>  
[C++ Market Data Sample Client](https://github.com/phemex/phemex-cpp-api)<br>    

## Error codes
### CxlRejReason field
| Code | Reason | Description |
|------|--------|-------------|
| 100  | CE_NO_ENOUGH_QTY | Qty is not enough |
| 101  | CE_WILLCROSS     | Passive order rejected due to price may cross |
| 116  | CE_NO_ENOUGH_BASE_QTY  | Base-qty is not enough in spot-trading  |
| 117  | CE_NO_ENOUGH_QUOTE_QTY | Quote-qty is not enough in spot-trading |

### BizError field
|  Code   |    Message  |   Description   |
|---------|-------------|-----------------|
|  19999  |  REQUEST_IS_DUPLICATED  |  Duplicated request ID  |
|  10001  |  OM_DUPLICATE_ORDERID  |  Duplicated order ID  |
|  10002  |  OM_ORDER_NOT_FOUND  |  Cannot find order ID  |
|  10003  |  OM_ORDER_PENDING_CANCEL  |  Cannot cancel while order is already in pending cancel status  |
|  10004  |  OM_ORDER_PENDING_REPLACE  |  Cannot cancel while order is already in pending cancel status  |
|  10005  |  OM_ORDER_PENDING  |  Cannot cancel while order is already in pending cancel status  |
|  11001  |  TE_NO_ENOUGH_AVAILABLE_BALANCE  |  Insufficient available balance  |
|  11002  |  TE_INVALID_RISK_LIMIT  |  Invalid risk limit value  |
|  11003  |  TE_NO_ENOUGH_BALANCE_FOR_NEW_RISK_LIMIT  |  Insufficient available balance  |
|  11004  |  TE_INVALID_LEVERAGE  |  Invalid input or new leverage is over maximum allowed leverage  |
|  11005  |  TE_NO_ENOUGH_BALANCE_FOR_NEW_LEVERAGE  |  Insufficient available balance  |
|  11006  |  TE_CANNOT_CHANGE_POSITION_MARGIN_WITHOUT_POSITION  |  Position size is zero. Cannot change margin  |
|  11007  |  TE_CANNOT_CHANGE_POSITION_MARGIN_FOR_CROSS_MARGIN  |  Cannot change margin under CrossMargin  |
|  11008  |  TE_CANNOT_REMOVE_POSITION_MARGIN_MORE_THAN_ADDED  |  Exceeds the maximum removable Margin  |
|  11009  |  TE_CANNOT_REMOVE_POSITION_MARGIN_DUE_TO_UNREALIZED_PNL  |  Exceeds the maximum removable Margin  |
|  11010  |  TE_CANNOT_ADD_POSITION_MARGIN_DUE_TO_NO_ENOUGH_AVAILABLE_BALANCE  |  Insufficient available balance  |
|  11011  |  TE_REDUCE_ONLY_ABORT  |  Cannot accept reduce only order  |
|  11012  |  TE_REPLACE_TO_INVALID_QTY  |  Order quantity Error  |
|  11013  |  TE_CONDITIONAL_NO_POSITION  |  Position size is zero. Cannot determine conditional order's quantity  |
|  11014  |  TE_CONDITIONAL_CLOSE_POSITION_WRONG_SIDE  |  Close position conditional order has the same side  |
|  11015  |  TE_CONDITIONAL_TRIGGERED_OR_CANCELED  |    |
|  11016  |  TE_ADL_NOT_TRADING_REQUESTED_ACCOUNT  |  Request is routed to the wrong trading engine  |
|  11017  |  TE_ADL_CANNOT_FIND_POSITION  |  Cannot find requested position on current account  |
|  11018  |  TE_NO_NEED_TO_SETTLE_FUNDING  |  The current account does not need to pay a funding fee  |
|  11019  |  TE_FUNDING_ALREADY_SETTLED  |  The current account already pays the funding fee  |
|  11020  |  TE_CANNOT_TRANSFER_OUT_DUE_TO_BONUS    |  Withdraw to wallet needs to remove all remaining bonus. However if bonus is used by position or order cost, withdraw fails.  |
|  11021  |  TE_INVALID_BONOUS_AMOUNT  |  Invalid bonus amount  |
|  11022  |  TE_REJECT_DUE_TO_BANNED |  Account is banned  |
|  11023  |  TE_REJECT_DUE_TO_IN_PROCESS_OF_LIQ  |  Account is in the process of liquidation  |
|  11024  |  TE_REJECT_DUE_TO_IN_PROCESS_OF_ADL  |  Account is in the process of auto-deleverage  |
|  11025  |  TE_ROUTE_ERROR = 11025  |  Request is routed to the wrong trading engine  |
|  11026  |  TE_UID_ACCOUNT_MISMATCH  |    |
|  11027  |  TE_SYMBOL_INVALID  |  Invalid number ID or name  |
|  11028  |  TE_CURRENCY_INVALID  |  Invalid currency ID or name  |
|  11029  |  TE_ACTION_INVALID  |  Unrecognized request type  |
|  11030  |  TE_ACTION_BY_INVALID  |    |
|  11031  |  TE_SO_NUM_EXCEEDS  |  Number of total conditional orders exceeds the max limit  |
|  11032  |  TE_AO_NUM_EXCEEDS    |  Number of total active orders exceeds the max limit  |
|  11033  |  TE_ORDER_ID_DUPLICATE    |  Details:Duplicated order ID  |
|  11034  |  TE_SIDE_INVALID    |  Details:Invalid side  |
|  11035  |  TE_ORD_TYPE_INVALID    |  Details:Invalid OrderType  |
|  11036  |  TE_TIME_IN_FORCE_INVALID    |  Details:Invalid TimeInForce  |
|  11037  |  TE_EXEC_INST_INVALID    |  Details:Invalid ExecType  |
|  11038  |  TE_TRIGGER_INVALID    |  Details:Invalid trigger type  |
|  11039  |  TE_STOP_DIRECTION_INVALID    |  Details:Invalid stop direction type  |
|  11040  |  TE_NO_MARK_PRICE    |  Cannot get valid mark price to create conditional order  |
|  11041  |  TE_NO_INDEX_PRICE    |  Cannot get valid index price to create conditional order  |
|  11042  |  TE_NO_LAST_PRICE    |  Cannot get valid last market price to create conditional order  |
|  11043  |  TE_RISING_TRIGGER_DIRECTLY    |  Conditional order would be triggered immediately  |
|  11044  |  TE_FALLING_TRIGGER_DIRECTLY    |  Conditional order would be triggered immediately  |
|  11045  |  TE_TRIGGER_PRICE_TOO_LARGE    |  Conditional order trigger price is too high  |
|  11046  |  TE_TRIGGER_PRICE_TOO_SMALL    |  Conditional order trigger price is too low  |
|  11047  |  TE_BUY_TP_SHOULD_GT_BASE    |  TakeProfit BUY conditional order trigger price needs to be greater than reference price  |
|  11048  |  TE_BUY_SL_SHOULD_LT_BASE    |  StopLoss BUY condition order price needs to be less than the reference price  |
|  11049  |  TE_BUY_SL_SHOULD_GT_LIQ    |  StopLoss BUY condition order price needs to be greater than liquidation price or it will not trigger  |
|  11050  |  TE_SELL_TP_SHOULD_LT_BASE    |  TakeProfit SELL conditional order trigger price needs to be less than reference price  |
|  11051  |  TE_SELL_SL_SHOULD_LT_LIQ    |  StopLoss SELL condition order price needs to be less than liquidation price or it will not trigger  |
|  11052  |  TE_SELL_SL_SHOULD_GT_BASE    |  StopLoss SELL condition order price needs to be greater than the reference price  |
|  11053  |  TE_PRICE_TOO_LARGE    |  Order price is too large  |
|  11054  |  TE_PRICE_WORSE_THAN_BANKRUPT    |  Order price cannot be more aggressive than bankrupt price if this order has instruction to close a position  |
|  11055  |  TE_PRICE_TOO_SMALL    |  Order price is too low  |
|  11056  |  TE_QTY_TOO_LARGE    |  Order quantity is too large  |
|  11057  |  TE_QTY_NOT_MATCH_REDUCE_ONLY    |  Does not allow ReduceOnly order without position  |
|  11058  |  TE_QTY_TOO_SMALL    |  Order quantity is too small  |
|  11059  |  TE_TP_SL_QTY_NOT_MATCH_POS    |  Position size is zero. Cannot accept any TakeProfit or StopLoss order  |
|  11060  |  TE_SIDE_NOT_CLOSE_POS    |  TakeProfit or StopLoss order has wrong side. Cannot close position  |
|  11061  |  TE_ORD_ALREADY_PENDING_CANCEL    |  Repeated cancel request  |
|  11062  |  TE_ORD_ALREADY_CANCELED    |  Order is already canceled  |
|  11063  |  TE_ORD_STATUS_CANNOT_CANCEL    |  Order is not able to be canceled under current status  |
|  11064  |  TE_ORD_ALREADY_PENDING_REPLACE    |  Replace request is rejected because order is already in pending replace status  |
|  11065  |  TE_ORD_REPLACE_NOT_MODIFIED    |  Replace request does not modify any parameters of the order  |
|  11066  |  TE_ORD_STATUS_CANNOT_REPLACE    |  Order is not able to be replaced under current status  |
|  11067  |  TE_CANNOT_REPLACE_PRICE    |  Market conditional order cannot change price  |
|  11068  |  TE_CANNOT_REPLACE_QTY    |  Condtional order for closing position cannot change order quantity, since the order quantity is determined by position size already  |
|  11069  |  TE_ACCOUNT_NOT_IN_RANGE    |  The account ID in the request is not valid or is not in the range of the current process  |
|  11070  |  TE_SYMBOL_NOT_IN_RANGE    |  The symbol is invalid  |
|  11071  |  TE_ORD_STATUS_CANNOT_TRIGGER    |    |
|  11072  |  TE_TKFR_NOT_IN_RANGE    |  The fee value is not valid  |
|  11073  |  TE_MKFR_NOT_IN_RANGE    |  The fee value is not valid  |
|  11074  |  TE_CANNOT_ATTACH_TP_SL    |  Order request cannot contain TP/SL parameters when the account already has positions  |
|  11075  |  TE_TP_TOO_LARGE    |  TakeProfit price is too large  |
|  11076  |  TE_TP_TOO_SMALL    |  TakeProfit price is too small  |
|  11077  |  TE_TP_TRIGGER_INVALID    |  Invalid trigger type  |
|  11078  |  TE_SL_TOO_LARGE    |  StopLoss price is too large  |
|  11079  |  TE_SL_TOO_SMALL    |  StopLoss price is too small  |
|  11080  |  TE_SL_TRIGGER_INVALID    |  Invalid trigger type  |
|  11081  |  TE_RISK_LIMIT_EXCEEDS    |  Total potential position breaches current risk limit  |
|  11082  |  TE_CANNOT_COVER_ESTIMATE_ORDER_LOSS    |  The remaining balance cannot cover the potential unrealized PnL for this new order  |
|  11083  |  TE_TAKE_PROFIT_ORDER_DUPLICATED    |  TakeProfit order already exists  |
|  11084  |  TE_STOP_LOSS_ORDER_DUPLICATED    |  StopLoss order already exists  |
|  11085  |  TE_CL_ORD_ID_DUPLICATE  |  ClOrdId is duplicated  |
|  11086  |  TE_PEG_PRICE_TYPE_INVALID  |  PegPriceType is invalid  |
|  11087  |  TE_BUY_TS_SHOULD_LT_BASE  |  The trailing order's StopPrice should be less than the current last price  |
|  11088  |  TE_BUY_TS_SHOULD_GT_LIQ  |  The traling order's StopPrice should be greater than the current liquidation price  |
|  11089  |  TE_SELL_TS_SHOULD_LT_LIQ  |  The traling order's StopPrice should be greater than the current last price  |
|  11090  |  TE_SELL_TS_SHOULD_GT_BASE  |  The traling order's StopPrice should be less than the current liquidation price  |
|  11091  |  TE_BUY_REVERT_VALUE_SHOULD_LT_ZERO  |  The PegOffset should be less than zero  |
|  11092  |  TE_SELL_REVERT_VALUE_SHOULD_GT_ZERO  |  The PegOffset should be greater than zero  |
|  11093  |  TE_BUY_TTP_SHOULD_ACTIVATE_ABOVE_BASE  |  The activation price should be greater than the current last price  |
|  11094  |  TE_SELL_TTP_SHOULD_ACTIVATE_BELOW_BASE  |  The activation price should be less than the current last price  |
|  11095  |  TE_TRAILING_ORDER_DUPLICATED  |  A trailing order exists already  |
|  11096  |  TE_CLOSE_ORDER_CANNOT_ATTACH_TP_SL  |  An order to close position cannot have trailing instruction  |
|  11097  |  TE_CANNOT_FIND_WALLET_OF_THIS_CURRENCY  |  This crypto is not supported  |
|  11098  |  TE_WALLET_INVALID_ACTION  |  Invalid action on wallet  |
|  11099  |  TE_WALLET_VID_UNMATCHED  |  Wallet operation request has a wrong wallet vid  |
|  11100  |  TE_WALLET_INSUFFICIENT_BALANCE  |  Wallet has insufficient balance  |
|  11101  |  TE_WALLET_INSUFFICIENT_LOCKED_BALANCE  |  Locked balance in wallet is not enough for unlock/withdraw request  |
|  11102  |  TE_WALLET_INVALID_DEPOSIT_AMOUNT  |  Deposit amount must be greater than zero  |
|  11103  |  TE_WALLET_INVALID_WITHDRAW_AMOUNT  |  Withdraw amount must be less than zero  |
|  11104  |  TE_WALLET_REACHED_MAX_AMOUNT  |  Deposit makes wallet exceed max amount allowed  |
|  11105  |  TE_PLACE_ORDER_INSUFFICIENT_BASE_BALANCE  |  Insufficient funds in base wallet  |
|  11106  |  TE_PLACE_ORDER_INSUFFICIENT_QUOTE_BALANCE  |  Insufficient funds in quote wallet  |
|  11107  |  TE_CANNOT_CONNECT_TO_REQUEST_SEQ  |  TradingEngine failed to connect with CrossEngine  |
|  11108  |  TE_CANNOT_REPLACE_OR_CANCEL_MARKET_ORDER  |  Cannot replace/amend market order  |
|  11109  |  TE_CANNOT_REPLACE_OR_CANCEL_IOC_ORDER  |  Cannot replace/amend ImmediateOrCancel order  |
|  11110  |  TE_CANNOT_REPLACE_OR_CANCEL_FOK_ORDER  |  Cannot replace/amend FillOrKill order  |
|  11111  |  TE_MISSING_ORDER_ID  |  OrderId is missing  |
|  11112  |  TE_QTY_TYPE_INVALID  |  QtyType is invalid  |
|  11113  |  TE_USER_ID_INVALID  |  UserId is invalid  |
|  11114  |  TE_ORDER_VALUE_TOO_LARGE  |  Order value is too large  |
|  11115  |  TE_ORDER_VALUE_TOO_SMALL  |  Order value is too small  |
|  11116  |  TE_BO_NUM_EXCEEDS| The total count of brakcet orders should equal or less than 5 |
|  11117  |  TE_BO_CANNOT_HAVE_BO_WITH_DIFF_SIDE| All bracket orders should have the same Side.|
|  11118  |  TE_BO_TP_PRICE_INVALID| Bracker order take profit price is invalid|
|  11119  |  TE_BO_SL_PRICE_INVALID| Bracker order stop loss price is invalid|
|  11120  |  TE_BO_SL_TRIGGER_PRICE_INVALID| Bracker order stop loss trigger price is invalid|
|  11121  |  TE_BO_CANNOT_REPLACE| Cannot replace bracket order.|
|  11122  |  TE_BO_BOTP_STATUS_INVALID| Bracket take profit order status is invalid |
|  11123  |  TE_BO_CANNOT_PLACE_BOTP_OR_BOSL_ORDER| Cannot place bracket take profit order |
|  11124  |  TE_BO_CANNOT_REPLACE_BOTP_OR_BOSL_ORDER| Cannot place bracket stop loss order|
|  11125  |  TE_BO_CANNOT_CANCEL_BOTP_OR_BOSL_ORDER| Cannot cancel bracket sl/tp order|
|  11126  |  TE_BO_DONOT_SUPPORT_API| Doesn't support bracket order via API|
|  11128  |  TE_BO_INVALID_EXECINST| ExecInst value is invalid|
|  11129  |  TE_BO_MUST_BE_SAME_SIDE_AS_POS| Bracket order should have the same side as position's side|
|  11130  |  TE_BO_WRONG_SL_TRIGGER_TYPE| Bracket stop loss order trigger type is invalid|
|  11131  |  TE_BO_WRONG_TP_TRIGGER_TYPE| Bracket take profit order trigger type is invalid|
|  11132  |  TE_BO_ABORT_BOSL_DUE_BOTP_CREATE_FAILED| Cancel bracket stop loss order due failed to create take profit order.|
|  11133  |  TE_BO_ABORT_BOSL_DUE_BOPO_CANCELED| Cancel bracket stop loss order due main order canceled.|
|  11134  |  TE_BO_ABORT_BOTP_DUE_BOPO_CANCELED| Cancel bracket take profit order due main order canceled.|

