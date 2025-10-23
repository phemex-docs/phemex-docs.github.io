# Unified Trading Account REST API

## Query asset mode

> Request Format

```
GET /uta-api/risk/risk-mode
```

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "assetMode": "MultiAsset",
    "lastUpdateTimeNs": 1701414861472716784
  }
}
```

* Response Fields

| Field            | Type       | Description                               |
|------------------|------------|-------------------------------------------|
| assetMode        | String     | current asset mode(Classic, MultiAsset)   |
| lastUpdateTimeNs | Long       | uta account update time in nanosecond     |

## Switch asset mode

> Request Format

```
POST /uta-account/switch-mode?riskMode=<riskMode>
```
* Request parameters

| Parameter  | Type   | Required | Description       | Possible values            |
|------------|--------|----------|-------------------|----------------------------|
| riskMode   | String | YES      | target asset mode | SINGLE_ASSET, MULTI_ASSETS |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "userId": 10002222,
    "switchModeSuccess": true,
    "switchModeCheckPassed": true,
    "switchModeCheckDetail": {
      "contractOrderPositionCheckPassed": true
    }
  }
}
```

* Response Fields

| Field                 | Type    | Description                                           |
|-----------------------|---------|-------------------------------------------------------|
| userId                | Long    | user ID                                               |
| switchModeSuccess     | Boolean | Whether the switch action is successful               |
| switchModeCheckPassed | Boolean | Whether the conditions are met for switching riskMode |
| switchModeCheckDetail | Object  | Detailed information on switch mode check             |

## Query asset

> Request Format

```
GET /uta-biz/assets?currency=<currency>
```
* Request parameters

| Parameter | Type   | Required | Description    | Possible values    |
|-----------|--------|----------|----------------|--------------------|
| currency  | String | YES      | Asset to query | BTC, USDT, ETH ... |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "userId": 10002222,
    "currency": "USDT",
    "balanceRv": "988.6059477872",
    "totalBonusRv": "0",
    "borrowedRv": "0",
    "interestRv": "0",
    "frozenBalanceAsCollateralRv": "0"
  }
}
```

* Response Fields

| Field                        | Type     | Description                                       |
|------------------------------|----------|---------------------------------------------------|
| userId                       | Long     | User ID                                           |
| currency                     | String   | Asset currency name                               |
| balanceRv                    | String   | Total balance of this currency                    |
| totalBonusRv                 | String   | Total bonus of this currency                      |
| borrowedRv                   | String   | Total borrowed balance of this currency wallet    |
| interestRv                   | String   | Total unpaid interest of this currency wallet     |
| frozenBalanceAsCollateralRv  | String   | The frozen balance used as collateral for trading |


## Payback debts

> Request Format

```
POST /uta-funds/contract/payback
```

> Request Format
```json
{
    "currency": "USDT",
    "amountRv": "40"
}
```
* Request parameters

| Parameter | Type   | Required | Description               | Possible values |
|-----------|--------|----------|---------------------------|-----------------|
| currency  | String | YES      | Currency to payback       | USDT            |
| amountRv  | String | YES      | Payback amount real value | 1000.12         |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "borrowedAmountRq": "40",
    "currency": "USDT"
  }
}
```

* Response Fields

| Field                       | Type     | Description                               |
|-----------------------------|----------|-------------------------------------------|
| currency                    | String   | currency to payback                       |
| borrowedAmountRq            | String   | Current borrowed amount after the payback |

## Query risk unit

> Request Format

```
GET /uta-api/risk/risk-units?currency=<currency>&riskType=<riskType>
```
* Request parameters

| Parameter | Type   | Required | Description                                  | Possible values                                             |
|-----------|--------|----------|----------------------------------------------|-------------------------------------------------------------|
| currency  | String | No       | Valuation currency of the risk unit to query | BTC, USDT, ETH ...                                          |
| riskType  | String | No       | Type of the risk unit                        | MultiAssetContractCross, SingleAssetContractCross, Isolated |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "userId": 10002222,
      "riskMode": "Classic",
      "currency": "BTC",
      "symbol": "BTCUSD",
      "posSide": "",
      "userStatus": "Normal",
      "marginRatioRr": "0.001",
      "totalBalanceRv": "0.00108802",
      "totalEquityRv": "0.00108802",
      "totalPosUnpnlRv": "0",
      "totalDebtRv": "0",
      "totalPosCostRv": "0",
      "totalPosMMRv": "0",
      "totalOrdUsedBalanceRv": "0",
      "fixedUsedRv": "0",
      "riskType": "SingleAssetContractCross"
    },
    {
      "userId": 10002222,
      "riskMode": "Classic",
      "currency": "USDT",
      "symbol": "BTCUSDT",
      "posSide": "",
      "userStatus": "Normal",
      "marginRatioRr": "0.001",
      "totalBalanceRv": "5523.514992992124",
      "totalEquityRv": "5538.429112992124",
      "totalPosUnpnlRv": "14.91412",
      "totalDebtRv": "0",
      "totalPosCostRv": "14.975552230788",
      "totalPosMMRv": "5.63402034904",
      "totalOrdUsedBalanceRv": "0",
      "fixedUsedRv": "0",
      "riskType": "SingleAssetContractCross"
    }
  ]
}
```

* Response Fields

| Field                  | Type      | Description                                                                                                                                                                    |
|------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| userId                 | Long      | User ID                                                                                                                                                                        |
| riskType               | String    | Type of the risk unit                                                                                                                                                          |
| riskMode               | String    | Current risk mode (Classic, MultiAsset, Isolated)                                                                                                                              |
| currency               | String    | Valuation currency of the risk unit                                                                                                                                            |
| symbol                 | String    | Position symbol if the risk unit is `Isolated`; Major symbol of this currency contract if Cross                                                                                |
| posSide                | String    | Position side (`Long`/`Short`/`Merge`), only used when risk mode is `Isolated`                                                                                                 |
| userStatus             | String    | User current status (`Banned`, `Normal`)                                                                                                                                       |
| marginRatioRr          | String    | To evaluate the risk of this user. If >= 1, the  the positions under this risk unit scope will be liquidated. <br /> margin ratio = totalPositionMaintenanceMargin/totalEquity |
| totalBalanceRv         | String    | Total balance under this risk unit scope                                                                                                                                       |
| totalEquityRv          | String    | Total equity under this risk unit scope                                                                                                                                        |
| totalPosUnpnlRv        | String    | Total unreleased pnl under this risk unit unit scope                                                                                                                           |
| totalDebtRv            | String    | Total borrowed under this risk unit unit scope + total interest                                                                                                                |
| totalPosCostRv         | String    | Total position cost under this risk unit unit scope                                                                                                                            |
| totalPosMMRv           | String    | Total maintenance margin under this risk unit unit scope                                                                                                                       |
| totalOrdUsedBalanceRv  | String    | Total order used under this risk unit unit scope                                                                                                                               |
| fixUsedRv              | String    | Total fixed used under this risk unit unit scope                                                                                                                               |

## Query borrow history

> Request Format

```
GET /uta-funds/contract/borrow?currency=<currency>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```
* Request parameters

| Parameter | Type    | Required | Description                              | Possible values  |
|-----------|---------|----------|------------------------------------------|------------------|
| currency  | String  | NO       | currency                                 | USDT,BTC         |
| start     | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| end       | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| pageNum   | Long    | NO       | default 0                                |                  |
| pageSize  | Integer | NO       | default 20                               |                  |


> Response Format

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
        "amountRv": "94.05"
      }
    ]
  }
}
```

* Response Fields

| Field                       | Type     | Description          |
|-----------------------------|----------|----------------------|
| currency                    | String   | Borrow currency name |
| borrowTime                  | String   | Borrow timestamp     |
| amountRv                    | String   | Amount borrowed      |

## Query payback history

> Request Format

```
GET /uta-funds/contract/payback?currency=<currency>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```
* Request parameters

| Parameter | Type    | Required | Description                              | Possible values  |
|-----------|---------|----------|------------------------------------------|------------------|
| currency  | String  | NO       | currency                                 | USDT,BTC         |
| start     | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| end       | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| pageNum   | Long    | NO       | default 0                                |                  |
| pageSize  | Integer | NO       | default 20                               |                  |


> Response Format

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
        "liqFeeRv": "0"
      }
    ]
  }
}
```

## Query interest history

> Request Format

```
GET /uta-funds/contract/borrow/interests?currency=<currencyList>&start=<start>&end=<end>&pageNum=<pageNum>&pageSize=<pageSize>
```
* Request parameters

| Parameter | Type    | Required | Description                              | Possible values  |
|-----------|---------|----------|------------------------------------------|------------------|
| currency  | String  | NO       | currency                                 | USDT,BTC         |
| start     | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| end       | Long    | NO       | Epoch milliseconds timestamp, default 0  |                  |
| pageNum   | Long    | NO       | default 0                                |                  |
| pageSize  | Integer | NO       | default 20                               |                  |


> Response Format

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

## Query convert history

> Request Format

```
GET /uta-exchanger/assets/convert?fromCurrency=<currency>&toCurrency=<toCurrency>
```
* Request parameters

| Parameter    | Type     | Required | Description                             | Possible values  |
|--------------|----------|----------|-----------------------------------------|------------------|
| fromCurrency | String   | NO       | From currency in the conversion         | USDT,BTC         |
| toCurrency   | String   | NO       | Target currency in the conversion       | USDT,BTC         |
| start        | Long     | NO       | Epoch milliseconds timestamp, default 0 |                  |
| end          | Long     | NO       | Epoch milliseconds timestamp, default 0 |                  |
| offset       | Integer  | NO       | Start from 0, default 0                 |                  |
| limit        | Integer  | NO       | Page size, default to 20, max 200       |                  |


> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "total": 1,
    "rows": [
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
  }
}
```
## Other API
Other APIs listed below are not changed and should be same as before
* Transfer API
* CreateOrder API
* ReplaceOrder API
* AssignIsolatedBalance API
* WebSocket Related API

The data and interface of websocket are same as before.
But in SingleAsset mode, users may subscribe to AOPS (see https://phemex-docs.github.io/#subscribe-account-order-position-aop-2) to get the latest balance and positions.
In MultiAsset mode, users need to subscribe to RAS (see https://phemex-docs.github.io/#subscribe-account-margin-rasO) to get the latest asset info in different currencies, while positions still need to get from AOPS
