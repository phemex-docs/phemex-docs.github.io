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

