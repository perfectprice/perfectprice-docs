# Car rental raw data

### 1 Introduction

This document covers data coming __into__ Perfect Price. This applies (1) when you are getting set up with Perfect Price and (2) on an ongoing basis, so the AI can constantly learn from experience and react to market changes. 

Data will power the AI models as well as dashboards, analysis and other big data visiualization tools for your team (and internal Perfect Price data scientists) to understand and manage price, utilization, profit and revenue. 

We are __sometimes__ able to customize various data structures or import your data in the format you can easily export it in, and transform it into the format we need. So please ask us if your data does not line up perfectly with our formats! We don't expect it to. 

### 2 Architecture Overview

Our system stores data in a hadoop cluster in the cloud, in [JSON](https://en.wikipedia.org/wiki/JSON). This is then stored and used for charting, modeling, or other purposes. Wherever possible we do all computations on the raw data, rather than importing aggregated data. 

Models are typically run daily, with burst or surge models run intraday (up to every 5 minutes). The more rapidly we receive data updates from your system, the more responsive and "human like" the AI can be. However, we have flexibility around frequency of updates, and not everything needs to be updated at high frequency. 

#### 2.1 Data categories

The broad categories of data we need are as follows:

| Type | Description | Can be manually updated? | Can be batch updated? |
|-------------:|:-------------|:-------------|:-------------|
| Vehicle Data | Data on each vehicle in your fleet used for availability, location and cost calculations | No | Yes |
| Location data | Data on each location or virtual location, used for cost calculations | Yes | Yes |
| Bookings | Raw bookings in bulk or as they come in | No | Historical |
| Rentals | Raw data on actual rentals (contracts) as they happen | No | Historical |
| Cost data | Any model you may already have on operating costs for optimizations | Yes | Yes |
| Market price data | "Shopped" data on competitor prices, historical or going forward | No | Historical |  

We will discuss each category and the required format below. Remember, we can help get the data into the right format even if you cannot.

#### 2.2 Note on PII

Please do *not* send personally identifiable information (PII) such as email, real name, phone numbers, etc. Customer specific data needs to be either hashed (using [SHA-1](https://en.wikipedia.org/wiki/SHA-1) for example) or not sent at all. 

### 3 Formats

For one time, historical exports, we can accept 3 formats:
* JSON (preferred)
* XML
* CSV (best for direct SQL exports)

Our system will map your fields to the fields that are expected. See below for required and optional fields. 

For ongoing updates, you can set up an API or webservice that we can query on a regular basis, or which pushes updates to our system when a change occurs. Changes include:
* new bookings (reservations)
* new rentals or pickups
* new returns
* changes in availability of a car (recall, repair, delivery, acquisition, disposal, etc.)

Batch updates or individual updates are both ok. 

#### 4 Raw Data Requirements

We need to estimate demand for each car, rate code or vehicle class to optimize. Therefore we need comprehensive information across several categories:

#### 4.1 Vehicles

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| vehicle\_unique\_id | Unique identifier for that vehicle | string | Yes | EXAMPLE |
| vehicle\_name | Human readable name of vehicle | string | Yes | Sprinter Bench |
| body\_style | Body style of vehicle | ??? | No | custom_van |
| car\_classes | Standardized 4- or 5-letter car classes vehicle is associated with can have multiple | string | No | XXAR,CCAR |
| make | Brand or make of car | String | Yes | Mercedes |
| model | Manufacturer name of car | String | Yes | Sprinter |
| year  | Model year of car | integer | Yes | 2016|
| acquisition\_cost | Full amount paid for vehicle in cents, including any taxes etc. For cost and depreciation calculations | integer | Yes for optimization | 1500000 |
| acquisition\_cost\_currency | Currency of acquisition cost | string | USD if not specified | CAD |
| disposal\_value | Net amount received for vehicle if sold | integer | No | 1358900 |
| disposal\_value\_currency | Currency of disposal value | string | USD if not specified | CAD |
| cost\_per\_mile | Cost of operation including depreciation per mile for this vehicle, if known | integer | No | 37 |
| current\_location | Location where vehicle is currently available; note this should match the key of one of your locations | string | Yes | SFO |
| start\_date\_current\_location | Date car became or will be associated with current location | UTC Date String | No | 2013-12-12T18:29:50.588Z |
| end\_date\_current\_location | Date car became or will become not associated with current location | UTC Date String | No | 2013-12-12T18:29:50.588Z |
| next\_location | Location car will become associated with next, after ending association with current location | UTC Date String | No | 2013-12-12T18:29:50.588Z |
| rate\_codes | Rate code or codes this vehicle is associated with | string | No | XXAR_3, XXAR_1, XXAR_7, MCAR_1, MCAR_3, MCAR_7 |
| is\_available | Whether the car is currently available for rent | boolean | No | TRUE |
| status | Whether the car on rent, on repair, etc. | string | No | on_rent |
| air\_conditioner | Whether car has air conditioner/refrigeration| string | No | TRUE |
| bluetooth | Whether car has bluetooth | | | |
| is\_autonomous| Whether car is autonomous capable | boolean | No, default is FALSE | TRUE |
| autonomous\_level | Whether car is autonomous capable and if so what level, 0-5 | integer | No, default is 0 | 5 |
| heated\_seats | Whether car has heated seats | string | No | Yes |
| cooled\_seats | Whether car has cooled seats | string | No | Yes |
| number\_of\_doors | How many doors on the car | string | No | 5 |
| large\_bag\_capacity | How many large bags will fit | integer | No | 3 |
| small\_bag\_capacity | How many small bags will fit | integer | No | 3 |
| drivetrain | Manual or automatic | string | No | manual |
| leather\_seats | Whether seats are leather | string | No | leather |
| marketing\_blurb | Text about vehicle used in marketing/website description| string | No | This fast Italian car is perfect for weekend getaways, with its huge engine and drop top. |
| navigation | Whether car has built-in navigation | boolean | No | TRUE |
| passenger\_capacity | Number of passengers who can legally use car | integer | Yes | 5 |
| sunroof | Whether the vehicle has a sunroof | boolean | Yes | TRUE |
| transmission | Whether the vehicle is manual (standard) or automatic transmission | char | a or m | a |
| channels | Display on Sabre, Worldspan, etc.  | string | No | Sabre,Worldspan |

#### 4.2 Rate codes

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| rate\_code\_id | id for this rate code, in the event the code changes   
| rate\_code | The rate code key in your system | string | Yes | EXAMPLE |
| lor | Length of rent | integer | Yes | 3 | 
| lor\_units | Units for LOR | char one of h, d, w, m | Yes | d |
| default_price | The default price for this rate code, or the current price, in cents | Integer | No | 2000 |
| default_price_currency | Currency for default price | string | No | USD |
| vehicles | Array of vehcile IDs associated with this rate code at the current time | string | No | EXAMPLE |

#### 4.3 Car Classes (Vehicle types)


| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| car\_class\_id | id for this car class, in the event the code changes | string | Yes | 3205ofaiu3029 |
| car\_class | The rate code key in your system | string | Yes | EXAMPLE |
| lor | Length of rents available for this car class | string | Yes | 3 | 
| lor\_units | Units for LOR | char one of h, d, w, m | Yes | d |
| default_price | The default price for this rate code, or the current price, in cents | Integer | No | 2000 |
| default_price_currency | Currency for default price | string | No | USD |
| vehicles | Array of vehcile IDs associated with this rate code at the current time | string | No | EXAMPLE |


#### 4.4 Locations

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| location\_id | id of location | string | Yes | 32lkjghew2 |
| location\_name | name of location | string | Yes | San Francisco Airport|
| location\_code | code of location if applicable | string | No | SFO | 
| active | flag if active, default assumes yes | boolean | Yes | TRUE | 
| street\_address | Street address of location | string | No | 123 Main Street, \# 321 |
| city | City of location | string | No | San Francisco | 
| state | State or province of location | string | No | CA |
| country | Country of location | string | No | USA | 
| postal code | Postal code of location | string | No | 94101 |
| latlong | String expression of lat, long of location | string in [ISO 6709](https://en.wikipedia.org/wiki/ISO_6709) format | No |  _+40.6894-074.0447_/ |
| airport | Airport code associated with location if an airport location | string | No | SFO |
| is\_deleted | Whether the location is deleted/no longer a location | boolean | No | FALSE |
| ota\_code | Location code used by online travel agencies or GDS systems | string | No | EXAMPLE |

#### 4.5 Reservations (or Bookings)
| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| reservation\_id | unique id of reservation, joinable with other tables | string | Yes | 32lkjghew2 |
| created\_time | Time and date reservation was created in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| last\_modified\_time | Time and date reservation was last modified in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_time | Time and date vehicle will be picked up in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_location | Location vehicle will be picked up at, should be linked to your Locations | string | Yes | SFO |
| dropoff\_time | Time and date vehicle will be dropped off in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| dropoff\_location | Location vehicle will be dropped off at, should be linked to your Locations | string | Yes | LAX |
| rate\_codes | Array of rate codes for reservation, must be an ID of one of your rate code(s) | string | No | CCAR_1_day, CCAR_1_week |
| car\_class | Car or vehicle class associated with reservatoin, must be an id of one of your car classes | string | No | BMW_Mini |
| total\_amount | Total amount of booking cost in cents | integer | No | 13400 |
| fees\_amount | Array of name and amount of tax or fee for each tax or fee charged | string | No | EXAMPLE |
| ancillary\_services | Array of ancillary services and charges for those services | string | No | EXAMPLE |
| ancillary\_amount | Cost of ancillary charges in cents | integer | No | 13400 |
| deposit\_amount | Amount of deposit in cents | integer | No | 10000 | 
| prepaid\_amount | Amount of nonrefundable prepaid payment | No | 13400 | 
| refundable | Whether or not the amount paid is refundable | integer | No | TRUE | 
| currency | Currency for all revenue and rate amounts | string | No | USD | 
| customer\_id | Unique identifier for customer, possibly a hashed email address, that can be joined with other data to inform segmentation, etc. | string | No | 2k31j4lkhago9h08h34 |

#### 4.6 Rentals (or Contracts)

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| reservation\_id | unique id of reservation, joinable with other tables | string | Yes | 32lkjghew2 |
| rental\_id | unique id of rental or contract, joinable with other tables | string | Yes | 32lkjghew2 |
| created\_time | Time and date rental was created in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| last\_modified\_time | Time and date rental was last modified in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_time | Time and date vehicle was picked up in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_location | Location vehicle was picked up at, should be linked to your Locations | string | Yes | SFO |
| dropoff\_time | Time and date vehicle was or will be dropped off in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| dropoff\_location | Location vehicle will be dropped off, should be linked to your Locations | string | Yes | LAX |
| rate\_codes | Array of rate codes for reservation, must be an ID of one of your rate code(s) | string | No | CCAR_1_day, CCAR_1_week |
| car\_class | Car or vehicle class associated with reservatoin, must be an id of one of your car classes | string | No | BMW_Mini |
| total\_amount | Total amount of cost in cents | integer | No | 13400 |
| fees\_amount | Array of name and amount of tax or fee for each tax or fee charged | string | No | EXAMPLE |
| ancillary\_services | Array of ancillary services and charges for those services | string | No | EXAMPLE |
| ancillary\_amount | Cost of ancillary charges in cents | integer | No | 13400 |
| deposit\_amount | Amount of deposit in cents | integer | No | 10000 | 
| prepaid\_amount | Amount of nonrefundable prepaid payment | No | 13400 | 
| refundable | Whether or not the amount paid is refundable | integer | No | TRUE | 
| currency | Currency for all revenue and rate amounts | string | No | USD | 
| customer\_id | Unique identifier for customer, possibly a hashed email address, that can be joined with other data to inform segmentation, etc. | string | No | 2k31j4lkhago9h08h34 |

#### 4.7. Looks or pageviews

When a user views a booking page, or you get a request from a GDS or online booking tool for a price quote, knowing that is important data in determining whether there is demand for a particular vehicle–that you're getting or not getting. 

These are event logs and we receive them in [JSON](https://en.wikipedia.org/wiki/JSON) format that contains common fields and event
type specific fields in the following format :

```json
{
    $common_fields,
    "properties": $event_specific_fields
}
```

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| id | unique id of event, joinable with other tables | string | Yes | 32lkjghew2 |
| datetime | date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. This is the beginning time of the aggregation period. | string | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| event\_type | type of view, that is request from GDS or webpage visit | string | Yes | 'sabre', 'worldspan', 'pageview', etc. |
| car\_classes | car classes viewed or in view if multiple for this event | string - array | Yes | EXAMPLE |

Standard values of __event\_type__ for different types of events :

| Type | Value |
|-------------:|:-------------|
| sabre | look from sabre gds |
| worldspan | look from worldspan gds |
| pageview | view of car on your website |

For any other type of events, either use custom values for __event\_type__ as needed or do not add collect them.
Some examples of custom __event\_type__ values includes :

| Type | Value |
|-------------:|:-------------|
| tapping a category | tap\_category |

An example of common fields :

```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "pageview",
    "properties": $event_specific_fields
}
```

#### 3.3 Event Specific Fields

##### Variants vs. Options

When a product is presented to users with properties that can take one or more than one among a variety of values,
e.g. size 'Small' from a set of choices [ 'Small', 'Medium', 'Large' ], or color 'Red' among [ 'Blue', 'Green',
'Purple', 'Red', 'White' ], etc., a particulr set of choices of those properties may result in changes in prices
depending on the choice. For instance, a tea product may be sold in a 1oz tin can at $10.00 USD, or as a 20 sachet
bags at $20.00 USD, or a T-shirt comes in different colors and sizes, while they all are sold at $15.00 USD.
When a variation incurs change in price, each variation is called a __variant__, while if not, it is called an
__option__. Distinguishing variants from options is important as our analysis is tightly coupled with prices.

Following sections describe formats of varying event\_specific\_fields for different types of events, where
definitions and examples are presented without common\_fields for simplicity of explanation. Certain non-required
fields may be omitted if not available or applicable.

---

##### Product Detail Page View

This event occurs when a user views a page that contains details about a single product, or a preview info of
a product via simple overlays like a sliding <DIV> or a pop-up modal dialog. For this event type,
event\_specific\_fields is defined as follows :

```json
{
    "availability": $availability,
    "categories": $categories,
    "catalog_discount": $catalog_discount,
    "currency": $currency,
    "image_url": $image_url,
    "market": $market,
    "options": $options,
    "price": $price,
    "properties": $properties,
    "sku": $sku,
    "title": $title,
    "uii": $uii,
    "url": $url,
    "variants": $variants
}
```

where

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| availability | a boolean value, True if this product is displayed as available, False otherwise | Yes, if there are no variants. |
| catalog_discount | discount publicly announced and applied automatically to this product. This may not be directly what's displayed to user, e.g. original price before discount shown along with discounted price, or sale %, etc. In such case, this value must be computed from those values. | No |
| categories | a list of categories name(s), e.g. "Home > Groceries > Vegetables > Organic" will be ["Home", "Groceries", "Vegetables", "Organic"] | No |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. | No |
| image_url | a URL of the thumbnail image of this product. | No |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. | No |
| options | an array of options for this product. See below for more detail. | No |
| price | price of this product, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes, if there are no variants. |
| properties | a dictionary of key value pairs that contain static information about the product, e.g. brand, width, etc. | No |
| sku | SKU of product. | No |
| title | name of a product e.g. Earl Grey Tea. In case variants exist, this is the represenatative product name for all variants. | Yes |
| uii | a string that uniquely identifies a product, e.g. stock number, SKU, UPC, etc. | Yes |
| url | URL address of product page user visited | Yes, if a web page view log. |
| variants | an array of variants for this product. See below for more detail. | No |

Field "options" is defined as follows :

```json
[
    {
        "type": $option_name,
        "values": $option_value_array,
        "unit": $option_unit
    },
    ...
]
```

where

| Field | Description | Required |
| -------------:|:-------------|:------------- |
| option_name | a string that indicates the name of this option, e.g. "size", "weight", "clarity", "color", etc. | Yes |
| option\_value\_array | an array of values one or more of which this option can take | Yes |
| option\_unit | a string for option\_value | No |

An example of options :

```json
[
    {
        "type": "Waist",
        "values": [
            "28",
            "30",
            "32",
            "34",
            "36",
            "38",
            "40"
        ],
        "unit": "inch"

##### Examples

---

