## 1. Introduction

Access point and method, method of authentication, input and output format, as well as errors and exceptions will be explained.

## 2. Authentication

To access any API, first one must get an access token by authenticating using auth API :

##### URI

```
    https://auth.pfpr.co/$version/token
```

where ```$version``` is the version of authentication API. Use ```v1.0```.

##### Method

```
    GET
```

##### Parameters

| Field | Description |
|------:|:----------|
| ```hash``` | a unique ID string assigned, e.g. ```0c55451823bad3e61412d89839fc368e``` |
| ```client_key``` | API key, e.g. ```de808d252850cf982ac3992e583e4398``` |
| ```client_secret``` | API secret, e.g. ```842277b6356058b648639601fbade6a8``` |

##### Returns

If authentication is successful, auth API will return the following JSON object :

```
{
    'token' : $token
}
```
an example using python client:
```
import requests
requests.get('https://auth.pfpr.co/v1.0/token?client_key=de808d252850cf982ac3992e583e4398&client_secret=842277b6356058b648639601fbade6a8&hash=0c55451823bad3e61412d89839fc368e')
```

with an HTTP status code of 200, where ```$token``` is a string that must be submitted on
subsequent API requests in ```X-Auth-Token``` header. An authentication token expires 30m
passed last use, and expiration renews to last use time + 30m.

To avoid having to re-authenticate, client server should maintain working set of auth tokens.

If authentication fails, it returns corresponding error HTTP status code and a JSON
object that contains reasons of failure. More info to be added here.


## 3. Supported APIs

### 3.1 Pushing Data

This API is used to push proprietary data as specified in [Data Format] (https://docs.google.com/spreadsheets/d/1JgDXeZpBNOmyEuZzC3nCRfMnsVITyDRmkcFnXSFrQd8/edit#gid=1661905527) to Perfect Price. This API assumes that input data is in __JSON__ format.

##### URI

Access to this API will be made at a much higher frequency than base price API.
Ideally, this will be at every 5 minutes once all data pipeline is set up to
have real time pricing in place.

```
    https://api.pfpr.co/$version/data/$type
```

where ```$type``` could be one of :

| Field | Description |
|------:|:----------|
| ```inventory``` | Inventory |
| ```reservation``` | Reservation |
| ```rental``` | Rental |


##### Method

```
    PUT
```

##### Parameters

None

##### Body

A CSV file that conforms to [Data Format] (https://docs.google.com/spreadsheets/d/1JgDXeZpBNOmyEuZzC3nCRfMnsVITyDRmkcFnXSFrQd8/edit#gid=1661905527).

##### Returns

This API will return a JSON response as follows :

```json
{
    "type": $type,
    "status": [
        $status_for_1st_record,
        $status_for_2nd_record,
        ...
    ]
}
```

where a ```$status_for_i-th_record``` will look like examples below.

For a successfully processed record :

```json
{
    "status": "success",
}
```

and if failed :

```json
{
    "status": "failure",
    "error": {
        "code": $error_code,
        "message": $error_message
    }
}
```

If all records are pushed successfully, then the HTTP response code is 200, otherwise 4xx or 5xx will be returned accordingly. Also, for a general guideline, each pushed payload should not exceed 256KB in size.

### 3.2 Rates

This API returns latest rate predictions per segment over a given range.

##### URI
```
    https://api.pfpr.co/$version/rates
```

##### Method

```
    GET
```

##### Parameters

| Field | Description | Required |
|------:|:----------|:----------|
| ```location_code``` | Location code, e.g. "SFO" | Yes |
| ```car_class``` | Car class, often SIPP code, e.g. "IJBR" | Yes |
| ```start_ts``` | Start time (UTC) of period for which rates are predicted for in ISO 8601 format, e.g. "20180901T014639Z". Default is one day after this API is called. | No |
| ```end_ts``` | End time (UTC) of period for which rates are predicted for in ISO 601 format, e.g. "20181101T000000Z". Default is 371 days after this API is called. | No |
| ```currency``` | 3-letter ISO 4217 currency code. Default "USD". | No |


##### Returns

If prediction is available for this car, base price API will return the following JSON object :

```json
{
    "location_code": $location_code,
    "car_class": $car_class,
    "period": {
        "start_ts": $start_ts,
        "end_ts": $end_ts
    },
    "currency": $currency_code,
    "rates": [
        $rate_dicts
    ]
}
```

where ```$rate_dicts``` is a list of dictionaries keyed by date in ISO format, with value rate dictionary keyed by lor, e.g. 

```json
'rates' : [
    { 'date': '20180711T000000Z', 'rate': {'1-5': 628.0, '6+': 647.0} },
    { 'date': '20180712T000000Z', 'rate': {'1-5': 620.0, '6-7': 644.0, '8+': 721.0} }
   ]
 ```

code example:
```
import requests
request = requests.get(
    'https://api.pfpr.co/v1.0/base?location_code=SFO&car_class=ICAR&start_ts=20190719T000000Z&end_ts=20200719T000000Z',
    headers={'X-Auth-Token': '1d20019491fb534ed276712bccda3282'}
)
```

example output:
```json
{
    'currency': 'USD', 
    'location_code': 'SFO', 
    'rates': [
                 {'date': '20190719T000000Z', 'rate': '{"1-5": 590, "6-7": 3540}'}, 
                 {'date': '20190720T000000Z', 'rate': '{"1-5": 594, "6-7": 3564}'}, 
                 {'date': '20190721T000000Z', 'rate': '{"1-5": 588, "6-7": 3528}'}, 
                 {'date': '20190722T000000Z', 'rate': '{"1-5": 590, "6-7": 3540}'}
    ], 
    'period': { 
                'start_ts': '20190719T000000Z',
                'end_ts': '20200719T000000Z'
               }, 
    'car_class': 'ICAR'}
```
