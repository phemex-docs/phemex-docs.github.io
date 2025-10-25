# Unified Trading Account REST API

## Query collateral info

> Request Format

```
GET public/products-plus
```

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "collaterals": [
      {
        "currency": "BTC",
        "coefficient": "0.98",
        "defaultHourlyInterestRate": "0.00000210",
        "liquidityInDescendingOrder": 2,
        "perpCollateral": true,
        "perpLoan": false,
        "convertRate": "0.999",
        "debtLimit": "1"
      }
    ]
  }
}
```

* Response Fields

| Field                      | Type    | Description                                                                                                                                                                     |
|----------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| currency                   | String  | The currency name of the collateral configuration                                                                                                                               |
| coefficient                | String  | The collateral coefficient used to calculate the collateral’s value in USDT. For example, if the BTC collateral coefficient is 0.9, then its value in USDT will be 0.9 × btcQty |
| defaultHourlyInterestRate  | String  | The default hourly interest rate applied when incurring debt in this currency                                                                                                   |
| liquidityInDescendingOrder | Integer | The liquidity ranking of the currency when selecting assets to convert for repayment. For example, if both BTC and BNB are collaterals, BTC will be converted before BNB        |
| perpCollateral             | Integer | Indicates whether this currency can be used as collateral for perpetual trading                                                                                                 |
| perpLoan                   | Boolean | Indicates whether this currency can be borrowed when a trade is settled. Currently, only USDT is supported                                                                      |
| convertRate                | String  | The exchange rate applied when converting this collateral to another currency                                                                                                   |
| debtLimit                  | String  | The maximum debt limit for this currency. When the debt reaches this limit, automatic conversion for repayment will be triggered                                                |


## Query risk mode

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
    "riskMode": "MultiAsset",
    "lastUpdateTimeNs": 1701414861472716784
  }
}
```

* Response Fields

| Field            | Type       | Description                                                                             |
|------------------|------------|-----------------------------------------------------------------------------------------|
| riskMode         | String     | current risk mode: Classic (i.e., the traditional SingleAsset trading mode), MultiAsset |
| lastUpdateTimeNs | Long       | uta account update time in nanosecond                                                   |

## Switch risk mode

> Request Format

```
POST /uta-account/switch-mode?riskMode=<riskMode>
```
* Request parameters

| Parameter  | Type   | Required | Description      | Possible values            |
|------------|--------|----------|------------------|----------------------------|
| riskMode   | String | YES      | target risk mode | SINGLE_ASSET, MULTI_ASSETS |


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

| Field                 | Type    | Description                                                                                                                                        |
|-----------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| userId                | Long    | user ID                                                                                                                                            |
| switchModeSuccess     | Boolean | Whether the switch action is successful                                                                                                            |
| switchModeCheckPassed | Boolean | Whether the conditions are met for switching riskMode                                                                                              |
| switchModeCheckDetail | Object  | Detailed information on switch mode check: contractOrderPositionCheckPassed(check no active orders), contractDebtCheckPassed(check no unpaid debt) |

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
    "borrowedAmountRq": "60",
    "currency": "USDT"
  }
}
```
> **Example Explanation:**
> - If the user originally owed **100 USDT** and pays back **40 USDT**, 
>   the remaining debt (`borrowedAmountRq`) will be **60 USDT** in the response.

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

| Field                  | Type      | Description                                                                                                                                                              | Possible Values                                                                          |
|------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| userId                 | Long      | User ID                                                                                                                                                                  |                                                                                          |
| riskType               | String    | Type of the risk unit                                                                                                                                                    | `MultiAssetContractCross`, `SingleAssetContractCross`, `Isolated`                        |
| riskMode               | String    | Current risk mode                                                                                                                                                        | `Classic`(i.e., the traditional SingleAsset trading mode), `MultiAsset`, `Isolated`      |
| currency               | String    | Valuation currency of the risk unit                                                                                                                                      |                                                                                          |
| symbol                 | String    | Position symbol if the risk unit is `Isolated`; Major symbol of this currency contract if Cross                                                                          |                                                                                          |
| posSide                | String    | Position side                                                                                                                                                            | `Long`/`Short`/`Merge`, only used when risk mode is `Isolated`                           |
| userStatus             | String    | User current status                                                                                                                                                      | `Banned`, `Normal`                                                                       |
| marginRatioRr          | String    | To evaluate the risk of this user. If >= 1, the  the positions under this risk unit scope will be liquidated. <br /> margin ratio = totalMaintenanceMargin / totalEquity |                                                                                          |
| totalBalanceRv         | String    | Total balance under this risk unit scope                                                                                                                                 |                                                                                          |
| totalEquityRv          | String    | Total equity under this risk unit scope                                                                                                                                  |                                                                                          |
| totalPosUnpnlRv        | String    | Total unreleased pnl under this risk unit unit scope                                                                                                                     |                                                                                          |
| totalDebtRv            | String    | Total borrowed under this risk unit unit scope + total interest                                                                                                          |                                                                                          |
| totalPosCostRv         | String    | Total position cost under this risk unit unit scope                                                                                                                      |                                                                                          |
| totalPosMMRv           | String    | Total maintenance margin under this risk unit unit scope                                                                                                                 |                                                                                          |
| totalOrdUsedBalanceRv  | String    | Total order used under this risk unit unit scope                                                                                                                         |                                                                                          |
| fixUsedRv              | String    | Total fixed used under this risk unit unit scope                                                                                                                         |                                                                                          |

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

| Parameter | Type    | Required | Description                             | Possible values  |
|-----------|---------|----------|-----------------------------------------|------------------|
| currency  | String  | NO       | The currency used for the repayment     | USDT,BTC         |
| start     | Long    | NO       | Epoch milliseconds timestamp, default 0 |                  |
| end       | Long    | NO       | Epoch milliseconds timestamp, default 0 |                  |
| pageNum   | Long    | NO       | default 0                               |                  |
| pageSize  | Integer | NO       | default 20                              |                  |


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
* Response Fields

| Field             | Type     | Description                                                                                                    |
|-------------------|----------|----------------------------------------------------------------------------------------------------------------|
| currency          | String   | The currency used for the repayment                                                                            |
| repayTime         | String   | The timestamp (in milliseconds) when the repayment was made                                                    |
| principalAmountRv | String   | The principal amount repaid in this transaction                                                                |
| interestAmountRv  | String   | The interest amount repaid in this transaction                                                                 |
| liqFeeRv          | String   | The liquidation fee paid during the repayment (applicable only when auto-repayment occurs during liquidation)  |


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
* Response Fields

| Field            | Type   | Description                                                       |
|------------------|--------|-------------------------------------------------------------------|
| borrowCurrency   | String | The currency in which the funds were borrowed                     |
| interestCalcTime | Long   | The timestamp (in milliseconds) when the interest was calculated  |
| interestCurrency | String | The currency in which the interest amount is denominated          |
| hourlyInterestRv | String | The amount of interest accrued during the specified hour          |
| hourlyRateRr     | String | The hourly interest rate applied during the calculation period    |
| annualRateRr     | String | The equivalent annualized interest rate based on the hourly rate  |

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
    "total": 2,
    "rows": [
      {
        "linkKey": "c5c1c6be-1cfc-41be-aa8d-f4e2429356b9",
        "createTime": 1760603394000,
        "fromCurrency": "USDT",
        "toCurrency": "BTC",
        "fromAmountEv": 1000000000000,
        "toAmountEv": 9009468,
        "conversionRate": 900
      },
      {
        "linkKey": "789def56-305e-4f78-a18b-2c0b0108484d",
        "createTime": 1760448181000,
        "fromCurrency": "USDT",
        "toCurrency": "BTC",
        "fromAmountEv": 1000000000,
        "toAmountEv": 8970,
        "conversionRate": 897
      }
    ]
  }
}
```
* Response Fields

| Field          | Type   | Description                                                                   |
|----------------|--------|-------------------------------------------------------------------------------|
| linkKey        | String | The unique identifier for the conversion record                               |
| createTime     | Long   | The timestamp (in milliseconds) of the conversion                             |
| fromCurrency   | String | The source currency used in the conversion                                    |
| toCurrency     | String | The target currency received from the conversion                              |
| fromAmountEv   | String | The converted amount of the source currency, represented as an enlarged value |
| toAmountEv     | String | The converted amount of the target currency, represented as an enlarged value |
| conversionRate | String | The exchange rate applied during the conversion                               |


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
