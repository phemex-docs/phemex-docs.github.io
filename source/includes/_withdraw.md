# Deposit And Withdraw REST API

## Endpoint Security Type

* Please raise API Key via web and choose `Read & Trade & Withdraw permission`, btw IP Address is mandatory.
* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case-sensitive**.

## Price/Ratio/Value 

* Fields with post-fix "Rv" means real value without being scaled

## Query deposit address information

> Request Format

```
GET /phemex-deposit/wallets/api/depositAddress?currency=<currency>&chainName=<chainName>
```

* Request parameters

| Parameter | Type   | Required | Description | Case |
|-----------|--------|----------|-------------|------|
| currency  | String | YES      | coin name   | USDT |
| chainName | String | YES      | chain name  | ETH  |

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "address": "44exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "tag": ""
  }
}
```

* Response Fields 

| Field   | Type   | Description       |
|---------|--------|-------------------|
| address | String | deposit address   |
| tag     | String | chain address tag |



## Query deposit history records

> Request format

```
GET /phemex-deposit/wallets/api/depositHist?currency=<currency>&offset=<offset>&limit=<limit>&withCount=<withCount>
```

* Request parameters

| Parameter | Type    | Required | Description                   | Case    |
|-----------|---------|----------|-------------------------------|---------|
| currency  | List    | NO       | batch coin name list          | ETH,BTC |
| offset    | String  | NO       | query rows offset ,default 0  |         |
| limit     | Integer | NO       | limit rows return, default 20 |         |
| withCount | Boolean | NO       | return total rows or not      | True    |

> Response Format

```json
{
    "code": 0,
    "msg": "OK",
    "data": [
        {
            "accountType": 1,
            "address": "44exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "amountEv": 50000000,
            "chainCode": 1,
            "chainName": "BTC",
            "confirmations": 1,
            "confirmedHeight": null,
            "createdAt": 1670000000000,
            "currency": "BTC",
            "currencyCode": 1,
            "email": "abcdefg@gmail.com",
            "errorCode": null,
            "errMsg": null,
            "id": 1,
            "needConfirmHeight": null,
            "status": "Success",
            "transferStatus": null,
            "txHash": "567exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "type": "Deposit",
            "userId": 10000001
        }
    ]
}
```

* Response fields 

| Field             | Type    | Required | Description                  | Possible Values                                   |
|-------------------|---------|----------|------------------------------|---------------------------------------------------|
| depositTo         | String  | YES      | deposit account type         | SPOT, Contract                                    |
| address           | String  | Yes      | deposit Address              |                                                   |
| amount            | Long    | Yes      | deposit Amount               |                                                   |
| chainCode         | Integer | Yes      |                              |                                                   |
| chainName         | String  | Yes      |                              |                                                   |
| confirmations     | Long    |          | chain confirmed block        |                                                   |
| confirmedHeight   | Long    |          | chain block height           |                                                   |
| createdAt         | Long    | YES      | created time                 |                                                   |
| currency          | String  | YES      |                              |                                                   |
| currencyCode      | Integer | YES      |                              |                                                   |
| email             | String  |          | email Address                |                                                   |
| errorCode         | Integer |          |                              |                                                   |
| errMsg            | String  |          |                              |                                                   |
| id                | String  | YES      | record unique id             |                                                   |
| needConfirmHeight | Long    |          | chain confirmed total height |                                                   |
| status            | String  | YES      |                              | please refer to [deposit status](#deposit-status) |
| txHash            | String  |          |                              |                                                   |
| type              | Enum    | Yes      | Deposit Type                 | Deposit & CompensateDeposit                       |
| userId            | Long    | Yes      |                              |                                                   |

## Deposit status

| Status              | Description                                                 |
|---------------------|-------------------------------------------------------------|
| Success             | Succeed deposit to Phemex account                           |
| Rejected            | Possible deposit address not supported                      |
| SecurityChecking    | Risk control center is processing incoming withdraw request |
| SecurityCheckFailed | Failure to pass Risk control center security rules          |
| AmlCsApporve        | AML rule broken, Phemex support team manually approve       |
| Confirmed           | Deposit confirmed                                           |
| New                 | Newly submitted deposit                                     |

## Query deposit chain settings

> Request format

```
GET /phemex-deposit/wallets/api/chainCfg?currency=<currency>
```

* Request parameters

| Parameter | Type   | Required | Description | Case |
|-----------|--------|----------|-------------|------|
| currency  | String | YES      | coin name   | ETH  |

> Response Format

```json
{
    "code": 0,
    "msg": "OK",
    "data": [
        {
            "currency": "TRX",
            "currencyCode": 11,
            "minAmountRv": "0",
            "confirmations": 1,
            "chainCode": 11,
            "chainName": "TRX",
            "status": "Active"
        }
    ]
}
```

* Response fields

| Field         | Type    | Required | Description                     | Case |
|---------------|---------|----------|---------------------------------|------|
| currency      | String  | YES      | coin name                       | TRX  |
| currencyCode  | Integer | YES      | coin code                       | 11   |
| minAmountRv   | String  |          | minimal deposit amount          |      |
| confirmations | Long    |          | chain confirmation height       |      |
| chainCode     | Integer | YES      | chain code                      |      |
| chainName     | String  | YES      | chain name                      |      |
| status        | String  | YES      | chain status, Active Or Suspend |      |


## Query withdraw history records

> Request format

```
GET /phemex-withdraw/wallets/api/withdrawHist?currency=<currency>&chainName=<chainNameList>&offset=<offset>&limit=<limit>&withCount=<withCount>
```

* Request Parameter

| Parameter | Type    | Required | Description                                 | Case      |
|-----------|---------|----------|---------------------------------------------|-----------|
| currency  | List    | NO       | currency name list                          | USDT, ETH |
| chainName | List    | NO       | chain name list                             | ETH,TRX   |
| offset    | Long    | NO       | default 0                                   |           |
| limit     | Integer | NO       | default 20                                  |           |
| withCount | Boolean | NO       | Total conditional rows found, default False | True      |


> Response Format

```json
{
    "code": 0,
    "msg": "OK",
    "data": [
        {
            "id": 100,
            "address": "44exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "amountEv": 220000000,
            "chainCode": 11,
            "chainName": "TRX",
            "currency": "TRX",
            "currencyCode": 11,
            "email": "1234qwer@gmail.com",
            "expiredAt": 1657000000000,
            "feeEv": 100000000,
            "nickName": null,
            "phone": null,
            "rejectReason": "",
            "submitedAt": 1657000000000,
            "txHash": "44exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "userId": 100000001,
            "status": "Succeed"
        }
    ]
}
```

* Response Fields

| Field        | Type    | Required | Description             | Possible Values                                     |
|--------------|---------|----------|-------------------------|-----------------------------------------------------|
| id           | Long    | YES      | unique identifier       |                                                     |
| address      | String  | YES      | chain address           |                                                     |
| amountEv     | Long    | YES      | received amount         |                                                     |
| chainCode    | Integer | YES      | chain code              |                                                     |
| chainName    | String  | YES      | chain name              |                                                     |
| email        | String  |          |                         |                                                     |
| expiredAt    | Long    | YES      | expired time            |                                                     |
| feeEv        | Long    | YES      | withdraw real fee       |                                                     |
| nickName     | String  |          |                         |                                                     |
| phone        | String  |          |                         |                                                     |
| rejectReason | String  |          |                         |                                                     |
| submitedAt   | Long    | YES      |                         |                                                     |
| txHash       | String  |          | chain transfer hash key |                                                     |
| userId       | Long    | YES      |                         |                                                     |
| status       | String  | YES      | withdraw status         | please refer to [withdraw status](#withdraw-status) |

## Withdraw status

| Status                | Description                                                                  |
|-----------------------|------------------------------------------------------------------------------|
| Success               | Done                                                                         |
| Rejected              | Possible withdraw address mismatch                                           |
| Security Checking     | Risk control center is processing incoming withdraw request                  |
| Security check failed | Failure to pass Risk control center security rules                           |
| Pending Review        | Risk rule exposure                                                           |
| Address Risk          | Dangerous address detected                                                   |
| Expired               | Withdraw request timeout,normally 30 minutes                                 |
| Cancelled             | Manually cancelled by client                                                 |
| Pending Transfer      | Security checking rules passed, prepare to transfer assets to target address |

## Create withdraw request

> Request Format

```
POST /phemex-withdraw/wallets/api/createWithdraw?currency=<currency>&address=<address>&amount=<amount>&addressTag=<addressTag>&chainName=<chainName>
```

* Request parameters

| Parameter  | Type   | Required | Description      | Case                      |
|------------|--------|----------|------------------|---------------------------|
| currency   | String | YES      | currency name    | "ETH"                     |
| address    | String | YES      | withdraw address | "11xxxxxxxxxxxxxxxxxxxxx" |
| amount     | String | YES      | withdraw amount  | "1.234"                   |
| addressTag | String | NO       | Address tag      |                           |
| chainName  | String | YES      | chain Name       | "TRX"                     |

> Response Format

```json
{
    "code": 0,
    "msg": "OK",
    "data": {
        "id": 10000001,
        "address": "44exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "amountRv": "100",
        "chainCode": 11,
        "chainName": "TRX",
        "currency": "USDT",
        "currencyCode": 3,
        "email": "abc@gmail.com",
        "expiredAt": 1670000000000,
        "feeRv": "1",
        "nickName": null,
        "phone": null,
        "rejectReason": "",
        "submitedAt": 1670000000000,
        "txHash": null,
        "userId": 10000001,
        "status": "Success"
    }
}
```

* Response Fields

| Field        | Type      | Required | Description                        | Possible Values                                     |
|--------------|-----------|----------|------------------------------------|-----------------------------------------------------|
| id           | Long      | YES      | withdraw request unique identifier |                                                     |
| address      | String    | YES      | chain address                      |                                                     |
| amountRv     | String    | YES      | receive amount                     |                                                     |
| chainCode    | Integer   | YES      | chain code                         | 4                                                   |
| chainName    | String    | YES      | chain name                         | ETH                                                 |
| currency     | String    | YES      | coin name                          |                                                     |
| currencyCode | Integer   | YES      | coin code                          |                                                     |
| email        | String    |          | user email address                 |                                                     |
| expiredAt    | Timestamp | YES      | withdraw request expired time      |                                                     |
| feeRv        | String    | YES      | withdraw fee                       |                                                     |
| nickName     | String    |          |                                    |                                                     |
| phone        | String    |          |                                    |                                                     |
| rejectReason | String    |          |                                    |                                                     |
| submitedAt   | Timestamp | YES      |                                    |                                                     |
| txHash       | String    |          | chain transfer hash                |                                                     |
| userId       | Long      | YES      |                                    |                                                     |
| status       | String    | YES      |                                    | please refer to [withdraw status](#withdraw-status) |


## Cancel withdraw request

> Request format

```
POST /phemex-withdraw/wallets/api/cancelWithdraw?id=<id>
```

* Request parameters

| Parameter | Type | Required | Description                        | Case  |
|-----------|------|----------|------------------------------------|-------|
| id        | Long | YES      | withdraw request unique identifier | 10001 |

> Response Format

```json
{
    "code": 0,
    "msg": "OK",
    "data": "OK"
}
```

## Query withdraw chain settings

> Request format

```
GET /phemex-withdraw/wallets/api/asset/info?currency=<currency>&amount=<amount>
```

* Request parameters

| Parameter | Type   | Required | Description              | Case     |
|-----------|--------|----------|--------------------------|----------|
| currency  | String | NO       | coin name                | "ETH"    |
| amount    | String | NO       | withdraw amount with fee | "0.1234" |

> Response format

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "currency": "USDT",
    "currencyCode": 3,
    "balanceRv": "100.000001",
    "confirmAmountRv": "0",
    "allAvailableBalanceRv": "1.000001",
    "chainInfos": [
      {
        "chainCode": 4,
        "chainName": "ETH",
        "minWithdrawAmountRv": "2",
        "minWithdrawAmountWithFeeRv": "6",
        "withdrawFeeRv": "4",
        "receiveAmountRv": "0"
      },
      {
        "chainCode": 11,
        "chainName": "TRX",
        "minWithdrawAmountRv": "2",
        "minWithdrawAmountWithFeeRv": "3",
        "withdrawFeeRv": "1",
        "receiveAmountRv": "0"
      }
    ]
  }
}
```

* Response fields

| Field                      | Type    | Required | Description                                                                                 | Possible Value |
|----------------------------|---------|----------|---------------------------------------------------------------------------------------------|----------------|
| currency                   | String  | YES      | coin name                                                                                   | ETH            |
| currencyCode               | Integer | YES      | coin code                                                                                   | 4              |
| balanceRv                  | String  | YES      | total balance                                                                               |                |
| confirmAmountRv            | String  |          | apply for non kyc user only, lifetime remaining amount left                                 |                |
| allAvailableBalanceRv      | String  | YES      | total balance minus locked trading balance, locked withdraw balance and frozen balance |                |
| chainCode                  | Integer | YES      | chain code                                                                                  |                |
| chainName                  | String  | YES      | chain name                                                                                  |                |
| minWithdrawAmountRv        | String  |          | minimal withdraw amount                                                                     |                |
| minWithdrawAmountWithFeeRv | String  |          | minimal withdraw amount with fee                                                            |                |
| withdrawFeeRv              | String  |          | withdraw fee required for user input withdraw amount                                        |                |
| receiveAmountRv            | String  |          | received withdraw amount for user input withdraw amount                                     |                |



