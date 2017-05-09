# __Competitive Rate Data Format__

### 1 Introduction

This document covers competitive rate data coming __into__ Perfect Price. This applies both when you are getting set up with Perfect Price and on an ongoing basis, so the AI can constantly learn from experience and react to market changes.

We are __sometimes__ able to customize various data structures or import your data in the format you can easily export it in, and transform it into the format we need. So please ask us if your data does not line up perfectly with our formats! We don't expect it to.

It is __usually__ worthwhile to send us an export of data in the format you have easy access to. Sometimes we may be able to [transform](https://en.wikipedia.org/wiki/Extract,_transform,_load) it into the format we need more easily or reliably than your systems can.

### 2 Architecture Overview

Our system stores data in a Hadoop cluster in the cloud, in [JSON](https://en.wikipedia.org/wiki/JSON). This is then stored and used for charting, modeling, or other purposes. Wherever possible we do all computations on the raw data, rather than importing aggregated data.

Models are typically run daily, with burst or surge models run intraday (up to every 5 minutes). The more rapidly we receive data updates from your system, the more responsive and "human like" the AI can be. However, we have flexibility around frequency of updates, and not everything needs to be updated at high frequency.

We duplicate all data on our side. If you have a small data set and a data warehouse that allows it, we can re-read all data in for several years each time we run the model. This enables us to pull in updates that you may retroactively make to your data and include that auotmatically in our models. However that can be an expensive and inefficient process and therefore we can also simply pull and write new data, which is how most customers prefer to run the system.

Data on our system will persist if you change your rental management or reservation system, as it is in a standard format.

#### 2.1 Data Categories

The broad categories of data we need are as follows:

| Type | Description |
|-------------:|:-------------|
| Market Prices | "Shopped" data on competitor prices, historical or going forward |
| Dimensions | Dimensional data used in market price data, e.g. vendor, rate, city codes, etc.  |

#### 2.2 Note on PII

Please do *not* send personally identifiable information (PII) such as email, real name, phone numbers, etc. Customer specific data needs to be either hashed (using [SHA-1](https://en.wikipedia.org/wiki/SHA-1) for example) or not sent at all.

### 3 Data Exports

The data will end up in the below formats to be consumed and used by our AI. Some customizations are possible. Remember, we can help get the data into the right format even if your system cannot.

#### 3.1 Historical Data

Historical exports serve to either backfill months or years of data and/or to give Perfect Price's team a sample of data so we can provide better guidance in architecting a scalable, stable method of sending data.

For one time, historical exports, we can accept 3 formats:
* JSON (preferred)
* XML
* CSV (best for direct SQL exports)

If a CSV, our system will map your fields to the fields that are expected. This can be a more brittle method of data transfer and we recommend against it for complex data structures or any data structure to be used on an ongoing basis that may change. Our most preferred data format is JSON for the compactness and the flexibility

#### 3.2 Incremental Data

For ongoing updates, we can accept the same 3 formats, with conditions:
* JSON (preferred) - ideally, in API/webservice
* XML - API/webservice preferred
* CSV - Direct export or FTP dump.

The ideal method is always an API or webservice to send us incremental data, or for us to query for data updated since the last query. This is less efficient than a 1-time data export, but a well architected API or Webservice could support large batch transfers.

Examples of what you would push to us, or what we would be querying for include:

* new market (shopped) rates
* updated dimensions, if any

The closer to real time data gets transferred, the faster can our AI react.

#### 4 Data Formats

In all cases we strongly prefer data be as __raw__ as possible. That is, the actual shopped rates. It is easier and preferable for us to make computations on our side rather than work from already computed numbers. For example, giving us the specific rate for a certain reservation window, car type, dates, location, etc. is much better than just giving an average rate.

For compactness and specificity, market rate data often contain dimensional fields such as vendor codes, SIPP, etc. Dimensional data could be provided for convenience and human readability, mostly once during setup, or rarely as they change.

#### 4.1 Market Prices

```json
{
    "organization":
    {
        "id": $organization_id,
        "name": $organization_name
    },
    "pickup":
    {
        "city": $pickup_city,
        "datetime":
        {
            "start": $pickup_start_datetime,
            "end": $pickup_end_datetime
        }
    },
    "dropoff":
    {
        "city": $dropoff_city,
        "datetime":
        {
            "start": $dropoff_start_datetime,
            "end": $dropoff_end_datetime
        }
    },
    "vendor":
    {
        "code": $vendor_code,
        "name": $vendor_name
    },
    "shopped": $shopped_datetime,
    "car_type": $car_type,
    "lor": $length_of_rental,
    "currency": $currency,
    "rate": $rate,
    "total_rate": $total_rate,
    "total": $total,
    "optional": $optional
}
```

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| organization\_id | Unique identifier for the organization this rate is shopped for | string | Yes | "1", "ABC" |
| organization\_name | Human readable name of the organization | string | No | "Hertz", "Avis", "Mcar" |
| pickup\_city | Code or name of city a car is picked up. | string | Yes | IATA airport codes "NYC" for New York City |
| pickup\_start\_datetime | Starting date time of pick up window in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. | string | Yes | "2016-05-11T00:57:07Z" |
| pickup\_end\_datetime | Ending date time of pick up window in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. | string | Yes | "2016-05-11T00:57:07Z" |
| dropoff\_city | Code or name of city a car is dropped off. | string | Yes | IATA airport codes "NYC" for New York City |
| dropoff\_start\_datetime | Starting date time of drop off window in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. | string | Yes | "2016-05-11T00:57:07Z" |
| dropoff\_end\_datetime | Ending date time of drop off window in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. | string | Yes | "2016-05-11T00:57:07Z" |
| vendor\_code | Code of competing vendor associated with shopped rate. | string | Yes | "23.99" |
| vendor\_name | Name of competing vendor associated with shopped rate. | string | Yes | "Avis", "Hertz" |
| shopped\_datetime | Date time rate is shopped in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. | string | Yes | "2016-05-11T00:57:07Z" |
| car\_type | Unique identifer of the car type or rate code associated with the rate. | string | Yes | SIPP code "CDMR", "Economy" |
| length\_of\_rental | Length of rental, which could take variable forms. | variable | Yes | See further detail below. |
| currency | Currency code, by default [ISO-4217](https://en.wikipedia.org/wiki/ISO_4217). | string | Yes | "USD", "KRW" |
| rate | Rental rate for the given unit of time, by default daily. | string | Yes | "23.99" |
| total\_rate | Total amount for rent, i.e. rate * lor, if applicable. | string | No | "239.90" |
| total | Total final amount charged including additional fees, if applicable. | string | No | "250.00" |
| optional | Any optional fields, e.g. shop request ID. | No | Yes | "SHOP-10234" |

__Length of Rental__

Length of rental (LOR) is the number of a unit time, usually a day, that the shopped rate applies individually. LOR could be given as a single number :

```json
{
    "lor": 3
}
```

or as a list of numbers :

```json
{
    "lor": [1, 2, 3]
}
```

or as a range :

```json
{
    "lor":
    {
        "start": 1,
        "end": 3  
    }
}
```

Unless specified, the time unit is a __day__. If the unit time is other than a day, then it could be optionally specified as :

```json
{
    "lor": $length_of_rental,
    "lor_unit": $lor_unit
}
```

where $lor_unit is a string that takes one of the values :

| Value | Description |
|:------:|:-----------:|
| minute | A minute. |
| hour | An hour. |
| day | A day. |
| week | A week. |
| month | 30 days. |
| year | A year. |


#### 4.2 Dimensions

A dimensional field is a field that takes one of a predefined list of possible values, e.g. day of week, month of year, SIPP, vendor code, currency, airport codes, etc. Using standardized coding system for dimensional fields results in compact and unambiguous data, while it is not human friendly.

Dimensional data is a mapping of each dimensional value to a human readable descriptive piece of information. For instance, day of week dimension field could be defined as follows :

```json
{
    "info":
    {
        "name": "Day of Week",
    },
    "values":
    [
        {
            "key": 0,
            "value": "Monday"
        },
        {
            "key": 1,
            "value": "Tuesday"
        },
        ...
        {
            "key": 6,
            "value": "Sunday"
        }
    ]
}
```

If ordering is necessary, each element of "values" could contain an integer "order" value, e.g. :

```json
{
    "info":
    {
        "name": "Day of Week",
    },
    "values":
    [
        {
            "key": 0,
            "value": "Monday",
            "order": 0
        },
        {
            "key": 1,
            "value": "Tuesday",
            "order": 10
        },
        ...
        {
            "key": 6,
            "value": "Sunday",
            "order": 60
        }
    ]
}
```

* Note that __all__ elements of "values" should have "order" if ordering is necessary, or __all__ must not have it. If "order" is specified to only partial elements, they are completely ignored, i.e. no == random ordering.
