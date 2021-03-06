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

with an HTTP status code of 200.

```$token``` is a string that must be submitted on
subsequent API requests in ```X-Auth-Token``` header. An authentication token expires 30m
passed last use, and expiration renews to last use time + 30m.

To avoid having to re-authenticate, client server should maintain working set of auth tokens.

If authentication fails, it returns corresponding error HTTP status code and a JSON
object that contains reasons of failure.

##### Error Responses

| Response Code | Error Code | Description |
|------:|:----------|:----------|
| 401 | ERROR_UNAUTHORIZED | Invalid/Missing Credential |
| 405 | ERROR_BAD_METHOD | Bad Method |


## 3. Supported APIs

### 3.1 Pushing Data

This API is used to push proprietary data as specified in [Data Format] (https://docs.google.com/spreadsheets/d/1JgDXeZpBNOmyEuZzC3nCRfMnsVITyDRmkcFnXSFrQd8/edit#gid=1661905527) to Perfect Price.

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

```json
{
    "date": $date,
    "data": $data
}
```

where an optional ```$date``` (default = submission date in UTC) is the date for
which the data belongs to. If ```$date``` is present, then the provided data
will be associated to the given date. It must be in ```YYYY/mm/dd``` format.

```$data``` is CSV data that conforms to [Data Format] (https://docs.google.com/spreadsheets/d/1JgDXeZpBNOmyEuZzC3nCRfMnsVITyDRmkcFnXSFrQd8/edit#gid=1661905527).

an example using python client:
```
import requests
response = self.client.post('https://api.pfpr.co/v1.0/data/rental',
                            data={'data': $rental_csv, 'date': '2018/05/18'},
                            headers={'X-Auth-Token': '1d20019491fb534ed276712bccda3282'})
```


Data pushed at any time of day should contain data for yesterday UTC. For instance, data pushed at
```2018/10/05 1AM UTC``` should contain data collected during the period of :

```
[2018/10/04 00:00:00 UTC, 2018/10/05 00:00:00 UTC)
```

For historical data, ALL historical data could be pushed into the 1st day of push, i.e. first day's
data pushed at ```2018/10/05 1AM UTC``` should contain data collected during the period of :

```
[first_day_of_data, 2018/10/05 00:00:00 UTC)
```

or if preferred, they could be broken into each day in history and can be pushed separately.
The latter approach will make data size smaller than 256KB. If it is necessary to push data > 256KB,
then please contact us for further discussion.

##### Returns

At a successful push  :

```
{
    "type": $type,
    "status": "success"
}
```

and at failure :

```
{
    "type": $type,
    "status": "failure",
    "error":
    {
        "code": $error_code,
        "message": $error_message
    }
}
```

If all records are pushed successfully, then the HTTP response code is 200, otherwise 4xx or 5xx will be returned accordingly. Also, for a general guideline, each pushed payload should not exceed 256KB in size.

##### Error Responses

| Response Code | Error Code | Description |
|------:|:----------|:----------|
| 401 | ERROR_INVALID_TOKEN | Invalid/Missing Auth Token |
| 405 | ERROR_BAD_METHOD | Bad Method |
| 409 | ERROR_INVALID_PARAMETERS | Invalid Parameters, e.g. gibberish $type |
| 413 | ERROR_OVER_LIMIT | Data Size > 256KB |
| 422 | ERROR_BAD_DATA | Corrupt/Invalid/Empty Data |

*Note that Error Code will be an integer, not the string as shown above. We put a symbolic name here for reference only.

Example :

```
# HTTP 409
{
    "status": "failure",
    "error":
    {
        "code": 2001,
        "message": "Invalid type : sldkfjsld"
    }
}
```


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
| ```property_id``` | Comma-separated property IDs, e.g. "12255" or "2626,333". If not supplied, the API returns rates for all active properties. | No |
| ```location``` | Neighborhood or City where the property is located, e.g. "Mayfair" or "Wembley,South_Kensington". | No |
| ```start_ts``` | Start time (UTC) of period for which rates are predicted for in ISO 8601 format, e.g. "20190901T000000Z". If not supplied, the default is the current UTC date. | No |
| ```end_ts``` | End time (UTC) of period for which rates are predicted for in ISO 601 format, e.g. "20201101T000000Z". If not supplied, the default is 365 days from the current UTC date. | No |
| ```currency``` | 3-letter ISO 4217 currency code. Default "USD". | No |


##### Returns

If prediction is available for this property, base price API will return the following JSON object :

```
{
    "currency": $currency,
    "period": {
        "end_ts": $end_ts,
        "start_ts": $start_ts
    },
    "predictions": [
        {
            "property_id": $property_id,
            "date": $check_in_date,
            "rates": [
                $rate_dicts
            ]
        }
    ]
}
```

where ```$start_ts``` and ```$end_ts``` are the same value as provided in the parameters,
while ```$rate_dicts``` is a list of dictionaries in the following format :

```
{
    "date": $check_in_date,
    "rate":
    [
        {
            "lor": $length_of_stay,
            "amount": $rate_amount
        },
        ...
    ]
}
```

where ```$check_in_date``` is the date during which rates in "rate" dictionary apply, and
```$length_of_stay``` is a string that specify length of stay as ```$min-$max``` which
means a rental of length ```$min``` days to ```$max``` days, or ```$min+``` which means a
rental of length ```$min``` or more days. ```$rate_amount``` is the rate for given length
of rental.

Code example:
```
import requests
request = requests.get(
    'https://api.pfpr.co/v1.0/rates?property_id=15046&currency=GBP&start_ts=20191001T000000Z&end_ts=20191002T000000Z',
    headers={'X-Auth-Token': '1d20019491fb534ed276712bccda3282'}
)
```

Example

```
{
    "currency": "GBP", 
    "period": {
        "end_ts": "20191002T000000Z", 
        "start_ts": "20191001T000000Z"
    }, 
    "predictions": [
        {
            "date": "20191001T000000Z", 
            "rates": [
                {
                    "lor": "7-7", 
                    "amount": 2331
                }, 
                {
                    "lor": "1-1", 
                    "amount": 262
                }, 
                {
                    "lor": "28-28", 
                    "amount": 7889
                }
            ], 
            "property_id": "15046"
        }, 
        {
            "date": "20191002T000000Z", 
            "rates": [
                {
                    "lor": "7-7", 
                    "amount": 2321
                }, 
                {
                    "lor": "1-1", 
                    "amount": 308
                }, 
                {
                    "lor": "28-28", 
                    "amount": 7893
                }
            ], 
            "property_id": "15046"
        }
    ]
}
```

##### Error Responses

| Response Code | Error Code | Description |
|------:|:----------|:----------|
| 401 | ERROR_INVALID_TOKEN | Invalid/Missing Auth Token |
| 403 | ERROR_FORBIDDEN | Forbidden Access |
| 405 | ERROR_BAD_METHOD | Bad Method |
| 409 | ERROR_INVALID_PARAMETERS | Invalid Parameters, e.g. non-existing $location or $start_ts > $end_ts |

Example :

```
# HTTP 409
{
    "status": "falure",
    "error":
    {
        "code": "ERROR_INVALID_PARAMETERS",
        "message": "start_ts (20180712T000000Z) is before end_ts (20180711T000000Z)."
    }
}
```

