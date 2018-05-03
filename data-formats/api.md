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
| ```hash``` | a unique ID string assigned to GetAround, ```0c55451823bad3e61412d89839fc368e``` |
| ```client_key``` | API key, ```de808d252850cf982ac3992e583e4398``` |
| ```client_secret``` | API secret, ```842277b6356058b648639601fbade6a8``` |

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
subsequent API requests in ```X-Auth-Token``` header.

If authentication fails, it returns corresponding error HTTP status code and a JSON
object that contains reasons of failure. More info to be added here.

## 3. Base price

This API returns the __hourly__ base price per car. Supposedly, access to this API will be made at a slow pace,
e.g. once every six hours, based on the frequency at which related data on GetAround is freshed.

##### URI
```
    https://api.pfpr.co/$version/base
```
##### Method

```
    GET
```

##### Parameters

| Field | Description | Required |
|------:|:----------|:----------|
| ```car_id``` | A unique ID assigned to each car, ```car.id``` in Postgres DB | Yes |
| ```start_ts``` | Start time of period for which base price is predicted for. The format is TBD, e.g. ISO 8601 | Yes |
| ```end_ts``` | End time of period for which base price is predicted for. The format is TBD, e.g. ISO 8601 | Yes |

The period between ```start_ts``` and ```end_ts``` will be from future no further than 90 days from now. __\*\*\*\*\*__

##### Returns

If prediction is available for this car, base price API will return the following JSON object :

```json
{
    "car_id": $car_id,
    "period": {
        "start_ts": $start_ts,
        "end_ts": $end_ts
    },
    "prices": {
        "hourly": $hourly_price,
        "daily": $daily_price,
        "weekly": $weekly_price
    }
}
```

code example:
```
import requests
request = requests.get(
    'https://api.pfpr.co/v1.0/base?car_id=100004781371186&start_ts=20160601T014639Z&end_ts=20160602T014639Z',
    headers={'X-Auth-Token': '1d20019491fb534ed276712bccda3282'}
)
```

with an HTTP status code of 200, where ```$start_ts``` and ```$end_ts``` are the start and end time in 15 minute bucket,
which is nearest after the ```$start_ts``` and ```$end_ts``` in the input parameters. __\*__ ```$prices``` is a dictionary containing
hourly, daily, and weekly base prices for this car. Each of those prices is a __string__ with no currency symbol with digit
separators, e.g. ',' or '.', etc. __\*\*__

In practice, prices that are shown to users currently by GetAround are increments of $.25. __\*\*\*__

If prediction fails, it returns corresponding error HTTP status code and a JSON object that contains reasons of failure.
More detailed error codes will be documented here soon.

## 4. Multipliers

##### URI

This API returns the 15 minute multipliers per zone. Access to this API will be made at a much higher frequency
than base price API. Ideally, this will be at every 5 minutes once all data pipeline is set up to have real time
pricing in place.

```
    https://api.pfpr.co/$version/multipliers
```

##### Method

```
    GET
```

##### Parameters

| Field | Description |
|------:|:----------|
| ```car_id``` | A unique ID assigned to each car, ```car.id``` in Postgres DB | Yes |
| ```start_ts``` | Start time of period for which base price is predicted for. The format is TBD, e.g. ISO 8601 | Yes |
| ```end_ts``` | End time of period for which base price is predicted for. The format is TBD, e.g. ISO 8601 | Yes |

The period between ```start_ts``` and ```end_ts``` will be from future no further than 90 days from now. __\*\*\*\*\*__

##### Returns

If prediction is available for this zone, multipliers API will return the following JSON object :

```json
{
    "car_id": $car_id,
    "multiplier": $multiplier,
    "period": {
        "start_ts": $start_ts,
        "end_ts": $end_ts
    },
}
```

code example:
```
import urllib2
request = urllib2.Request(
    'https://api.pfpr.co/v1.0/multipliers?car_id=231333&start_ts=20160601T014639Z&end_ts=20160602T014639Z',
    headers={'X-Auth-Token': '1d20019491fb534ed276712bccda3282'}
)
```

with an HTTP status code of 200, where ```$multipliers``` is an array of real numbers to be multipled to base prices to compute
the real price to display for ```$period```. ```$multiplier``` is computed as the median __\*\*\*__ of all multipliers for the
15 minute buckets included in ```$period```. __\*__ __\*\*\*\*__ ```$start_ts``` and ```$end_ts``` are the start and end time,
which is nearest after the ```$start_ts``` and ```$end_ts``` in the input parameters.

If prediction fails, it returns corresponding error HTTP status code and a JSON object that contains reasons of failure.
More detailed error codes will be documented here soon.

## 5. Notes

##### May 25th 2016

__\*__ Time returned by the APIs are not the exact same start and end time from the input parameters
because our predictions are based on data from periods in 15m intervals. To denote the exact period
that predictions are made, actual start and end time based on 15m intervals are returned.

__\*\*__ Daily and weekly rates returned from base price API will be computed in the same way as what
currently GetAround does, i.e. scaling up hourly rates to obtain daily and weekly rates. If ready
to build more sophisticated models to predict base prices at daily or weekly basis, then we will
update the API accordingly after discussion and mutual agreement.

__\*\*\*__ Perfect Price will use median as the representative multiplier as requested by GetAround.
In the future, if we come up with a more sophisticated algorithm to compute multipliers dynamically
for any given ```$period```, we will deploy that after discussion and mutual agreement.

__\*\*\*\*__ Prices are snapped to nearest $.25 before base price API returns to caller, so that they
look practically reasonable to display to users.

__\*\*\*\*\*__ The 90 day period may change in the future, depending on how frequent API requests will
be made.
