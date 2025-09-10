# Copy Trade REST API

## Endpoint Security Type

* Please raise API Key via web using `Default API entry` type, choose `Read & Trade Permission`, and IP Address is
  `Don't Bind`.
* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your
  `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case-sensitive**.

## Query performance information of traders

> Request Format

```
GET /phemex-lb/public/api/trader/performance-info?strategyIds=<strategyIds>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter   | Type    | Required | Description                               | Case                     |
|-------------|---------|----------|-------------------------------------------|--------------------------|
| strategyIds | List    | YES      | one or more traders UID, separated by `,` | 113762135,600285,1106201 |
| pageNum     | Integer | NO       | page number, default 1                    | 1                        |
| pageSize    | Integer | NO       | page size, default 100                    | 100                      |

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "updateTime": "2025-08-11T22:27:06.000Z",
      "strategyId": 113762135,
      "totalRoi": "-1.0005",
      "totalPnL": "25.46",
      "latestAum": "43.32"
    },
    {
      "updateTime": "2025-08-11T00:09:25.000Z",
      "strategyId": 600285,
      "totalRoi": "0",
      "totalPnL": "0",
      "latestAum": "0"
    },
    {
      "updateTime": "2025-08-11T00:09:25.000Z",
      "strategyId": 1106201,
      "totalRoi": "0",
      "totalPnL": "0",
      "latestAum": "0"
    }
  ]
}
```

* Response Fields

| Field      | Type       | Description                                                         |
|------------|------------|---------------------------------------------------------------------|
| updateTime | String     | update timestamp of data                                            |
| strategyId | Long       | trader UID                                                          |
| totalRoi   | BigDecimal | total roi(summarized in USD) of trader, round to 4 decimal places   |
| totalPnL   | BigDecimal | total pnl(summarized in USD) of trader, round to 2 decimal places   |
| latestAum  | BigDecimal | lastest aum(summarized in USD) of trader, round to 2 decimal places |

## Query profit sharing information of traders

> Request format

```
GET /phemex-lb/public/api/trader/profit-sharing-info?strategyIds=<strategyIds>&dataFrom=<dataFrom>&dataEnd=<dataEnd>&pageNum=<pageNum>&pageSize=<pageSize>
```

Request parameters

| Parameter   | Type    | Required | Description                                                                     | Case                     |
|-------------|---------|----------|---------------------------------------------------------------------------------|--------------------------|
| strategyIds | List    | YES      | one or more traders UID, separated by `,`                                       | 113762135,600285,1106201 |
| dataFrom    | String  | NO       | starting date for querying data, the format is `yyyy-MM-dd`, default 2023-01-01 | 2025-07-01               |
| dataEnd     | String  | NO       | end date of querying data, the format is `yyyy-MM-dd`, default today            | 2025-08-11               |
| pageNum     | Integer | NO       | page number, default 1                                                          | 1                        |
| pageSize    | Integer | NO       | page size, default 100                                                          | 100                      |

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "strategyId": 113762135,
      "cycles": [
        {
          "updateTime": "2025-08-11T23:01:37.000Z",
          "cycle": "2025-08-03T16:00:00.001Z ~ 2025-08-10T16:00:00.000Z",
          "profitSharingRatio": "0.2",
          "profitShareAmount": "3.5045"
        },
        {
          "updateTime": "2025-08-11T23:01:37.000Z",
          "cycle": "2025-07-27T16:00:00.001Z ~ 2025-08-03T16:00:00.000Z",
          "profitSharingRatio": "0.12",
          "profitShareAmount": "0.0116"
        }
      ]
    },
    {
      "strategyId": 600285,
      "cycles": []
    },
    {
      "strategyId": 1106201,
      "cycles": []
    }
  ]
}
```

* Response fields

| Field              | Type       | Description                                                                        |
|--------------------|------------|------------------------------------------------------------------------------------|
| strategyId         | Long       | trader UID                                                                         |
| cycles             | List       | all cycles                                                                         |
| updateTime         | String     | update timestamp of data                                                           |
| cycle              | String     | time range of a single cycle                                                       |
| profitSharingRatio | BigDecimal | profit sharing ratio at the end of the cycle, round to 2 decimal places            |
| profitShareAmount  | BigDecimal | profit sharing amount (summarized in USD) for the cycle, round to 4 decimal places |

## Query commission information of traders

> Request format

```
GET /phemex-lb/public/api/trader/commission-info?strategyIds=<strategyIds>&dataFrom=<dataFrom>&dataEnd=<dataEnd>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request parameters

| Parameter   | Type    | Required | Description                                                                     | Case                     |
|-------------|---------|----------|---------------------------------------------------------------------------------|--------------------------|
| strategyIds | List    | YES      | one or more traders UID, separated by `,`                                       | 113762135,600285,1106201 |
| dataFrom    | String  | NO       | starting date for querying data, the format is `yyyy-MM-dd`, default 2025-05-15 | 2025-08-10               |
| dataEnd     | String  | NO       | end date of querying data, the format is `yyyy-MM-dd`, default today            | 2025-08-11               |
| pageNum     | Integer | NO       | page number, default 1                                                          | 1                        |
| pageSize    | Integer | NO       | page size, default 100                                                          | 100                      |

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "strategyId": 113762135,
      "cycles": [
        {
          "updateTime": "2025-08-11T00:59:14.000Z",
          "cycle": "2025-08-10T00:00:00.001Z ~ 2025-08-11T00:00:00.000Z",
          "totalCommissionFee": "4.5",
          "partnerCommisions": [
            {
              "partnerUid": 2001,
              "commissionRate": "0.05",
              "commissionFee": "3"
            },
            {
              "partnerUid": 3001,
              "commissionRate": "0.04",
              "commissionFee": "1.5"
            }
          ]
        }
      ]
    },
    {
      "strategyId": 600285,
      "cycles": [
        {
          "updateTime": "2025-08-11T00:59:14.000Z",
          "cycle": "2025-08-10T00:00:00.001Z ~ 2025-08-11T00:00:00.000Z",
          "totalCommissionFee": "3",
          "partnerCommisions": [
            {
              "partnerUid": 2001,
              "commissionRate": "0.05",
              "commissionFee": "3"
            }
          ]
        }
      ]
    },
    {
      "strategyId": 1106201,
      "cycles": []
    }
  ]
}
```

* Response fields

| Field              | Type       | Description                                                                           |
|--------------------|------------|---------------------------------------------------------------------------------------|
| strategyId         | Long       | trader UID                                                                            |
| cycles             | List       | all cycles                                                                            |
| updateTime         | String     | update timestamp of data                                                              |
| cycle              | String     | time range of a single cycle                                                          |
| totalCommissionFee | BigDecimal | total commission amount (summarized in USD) for the cycle, round to 4 decimal places  |
| partnerCommisions  | List       | commision information for KOLs bound to Trader                                        |
| partnerUid         | Long       | UID of KOL bound to trader                                                            |
| commissionRate     | BigDecimal | commission rate of KOL for the cycle, round to 2 decimal places                       |
| commissionFee      | BigDecimal | commission amount (summarized in USD) of KOL for the cycle, round to 4 decimal places |

## Query historical aum information of traders

> Request format

```
GET /phemex-lb/public/api/trader/historical-aum-info?strategyIds=<strategyIds>&dataFrom=<dataFrom>&dataEnd=<dataEnd>&pageNum=<pageNum>&pageSize=<pageSize>
```

* Request Parameter

| Parameter   | Type    | Required | Description                                                                     | Case                     |
|-------------|---------|----------|---------------------------------------------------------------------------------|--------------------------|
| strategyIds | List    | YES      | one or more traders UID, separated by `,`                                       | 113762135,600285,1106201 |
| dataFrom    | String  | NO       | starting date for querying data, the format is `yyyy-MM-dd`, default 2023-01-01 | 2025-07-01               |
| dataEnd     | String  | NO       | end date of querying data, the format is `yyyy-MM-dd`, default today            | 2025-08-11               |
| pageNum     | Integer | NO       | page number, default 1                                                          | 1                        |
| pageSize    | Integer | NO       | page size, default 100                                                          | 100                      |

> Response Format

```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "strategyId": 113762135,
      "cycles": [
        {
          "updateTime": "2025-08-11T22:27:06.000Z",
          "dateTimestamp": "2025-08-11",
          "numOfCopiers": 2,
          "periodAum": "43.32"
        },
        {
          "updateTime": "2025-08-10T00:09:19.000Z",
          "dateTimestamp": "2025-08-10",
          "numOfCopiers": 2,
          "periodAum": "0"
        }
      ]
    },
    {
      "strategyId": 600285,
      "cycles": [
        {
          "updateTime": "2025-08-11T00:09:25.000Z",
          "dateTimestamp": "2025-08-11",
          "numOfCopiers": 0,
          "periodAum": "0"
        },
        {
          "updateTime": "2025-08-09T00:25:47.000Z",
          "dateTimestamp": "2025-08-10",
          "numOfCopiers": 0,
          "periodAum": "0"
        }
      ]
    },
    {
      "strategyId": 1106201,
      "cycles": [
        {
          "updateTime": "2025-08-11T00:09:25.000Z",
          "dateTimestamp": "2025-08-11",
          "numOfCopiers": 0,
          "periodAum": "0"
        },
        {
          "updateTime": "2025-08-09T00:25:47.000Z",
          "dateTimestamp": "2025-08-10",
          "numOfCopiers": 0,
          "periodAum": "0"
        }
      ]
    }
  ]
}
```

* Response Fields

| Field         | Type       | Description                                                                                  |
|---------------|------------|----------------------------------------------------------------------------------------------|
| strategyId    | Long       | trader UID                                                                                   |
| cycles        | List       | all cycles                                                                                   |
| updateTime    | String     | update timestamp of data                                                                     |
| dateTimestamp | String     | a specific day                                                                               |
| numOfCopiers  | Integer    | the latest number of copiers before UTC 0 on a specific day                                  |
| periodAum     | BigDecimal | the latest aum (summarized in USD) before UTC 0 on a specific day, round to 2 decimal places |

