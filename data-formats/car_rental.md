# Car rental raw data

### 1. Introduction

This document covers data coming __into__ Perfect Price. This applies (1) when you are getting set up with Perfect Price and (2) on an ongoing basis, so the AI can constantly learn from experience and react to market changes. 

Data will power the AI models as well as dashboards, analysis and other big data visiualization tools for your team (and internal Perfect Price data scientists) to understand and manage price, utilization, profit and revenue. 

### 2 Architecture Overview

Our system replicates data in a hadoop cluster in the cloud, in JSON. Much of this is then indexed into more accessible database formats (SQL, etc.) for charting, modeling, or other purposes. Wherever possible we do all computations on the raw data, rather than importing aggregated data. 

Models are typically run daily, with burst or surge models run intraday (up to every 5 minutes). The more rapidly we receive data updates from your system, the more responsive and "human like" the AI can be. 

Please do *not* send personally identifiable information (PII) such as email, real name, phone numbers, etc. Instead please send a hash (SHA-1) that is consistent so that we can, if necessary, track a customer over time without PII.

Data is broken into the following categories:

| Type | Description | Can be manually updated? | Can be batch updated? |
|-------------:|:-------------|:-------------|:-------------|
| Vehicle Data | Data on each vehicle in your fleet used for availability, location and cost calculations | No | Yes |
| Location data | Data on each location or virtual location, used for cost calculations | Yes | Yes |
| Bookings | Raw bookings in bulk or as they come in | No | Historical |
| Rentals | Raw data on actual rentals (contracts) as they happen | No | Historical |
| Cost data | Any model you may already have on operating costs for optimizations | Yes | Yes |
| Market price data | "Shopped" data on competitor prices, historical or going forward | No | Historical |  

We will discuss each category, the required format, and automation. 

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
|-------------:|:-------------|:-------------|:-------------|
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
| model\_lock ...  | MCAR FIELD  DO WE NEED? | | | |
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
|-------------:|:-------------|:-------------|:-------------|
| rate\_code\_id | id for this rate code, in the event the code changes   
| rate\_code | The rate code key in your system | string | Yes | EXAMPLE |
| lor | Length of rent | integer | Yes | 3 | 
| lor\_units | Units for LOR | char one of h, d, w, m | Yes | d |
| default_price | The default price for this rate code, or the current price, in cents | Integer | No | 2000 |
| default_price_currency | Currency for default price | string | No | USD |
| vehicles | Array of vehcile IDs associated with this rate code at the current time | string | No | EXAMPLE |

#### 4.3 Car Classes (Vehicle types)


| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| car\_class\_id | id for this car class, in the event the code changes | string | Yes | 3205ofaiu3029 |
| car\_class | The rate code key in your system | string | Yes | EXAMPLE |
| lor | Length of rents available for this car class | string | Yes | 3 | 
| lor\_units | Units for LOR | char one of h, d, w, m | Yes | d |
| default_price | The default price for this rate code, or the current price, in cents | Integer | No | 2000 |
| default_price_currency | Currency for default price | string | No | USD |
| vehicles | Array of vehcile IDs associated with this rate code at the current time | string | No | EXAMPLE |


#### 4.4 Locations

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| location\_id | id of location | string | Yes | 32lkjghew2 |
| location\_name | name of location | string | Yes | San Francisco Airport|
| location\_code | code of location if applicable | string | No | SFO | 
| active | flag if active, default assumes yes | boolean | Yes | TRUE | 
| street\_address | Street address of location | string | No | 123 Main Street, \# 321 |
| city | City of location | string | No | San Francisco | 
| state | State or province of location | string | No | CA |
| country | Country of location | string | No | USA | 
| postal code | Postal code of location | string | No | 94101 |
| latlong | String expression of lat, long of location | string in ISO 6709 format | No |  _+40.6894-074.0447_/ |
| airport | Airport code associated with location if an airport location | string | No | SFO |
| is\_deleted | Whether the location is deleted/no longer a location | boolean | No | FALSE |
| ota\_code | Location code used by online travel agencies or GDS systems | string | No | EXAMPLE |

#### 4.5 Reservations (or Bookings)



#### 4.6 Rentals (or Contracts)


#### 4.7. Looks or pageviews ??


#### 3.1 Basic Format

An event log is in [JSON](https://en.wikipedia.org/wiki/JSON) format that contains common fields and event
type specific fields in the following format :

```json
{
    $common_fields,
    "properties": $event_specific_fields
}
```

| Field | Description |
|-------------:|:-------------|
| common_fields| fields that are common to any event log |
| event\_specific\_fields | fields that vary depending on event type |

#### 3.2 Common Fields

| Field | Description | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| datetime | date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. This is the beginning time of the aggregation period. | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| event\_type | a string indicating the type of event | Yes | 'pageview', 'add\_to\_cart', etc. See below for more detail. |
| user_id | an anonymized character string uniquely identifying a user, if applicable | Yes | 56d61b53283d19956d60a6fa |
| version | a log version string in “integer.integer.integer” format, default = 1.0.0 | Yes | 1.0.0 |
| event\_id | a string uniquely identifying an event | No | 837840745805772274 |

Standard values of __event\_type__ for different types of events :

| Type | Value |
|-------------:|:-------------|
| product detail | pageview |
| catalog | pageview | 
| cart | pageview |
| search | pageview |
| checkout | checkout |
| add to cart | add\_to\_cart |

For any other type of events, either use custom values for __event\_type__ as needed or do not add collect them.
Some examples of custom __event\_type__ values includes :

| Type | Value |
|-------------:|:-------------|
| tapping a category | tap\_category |
| view a billing page in app | view\_checkout\_billing\_step |

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
    },
    {
        "type": "Size",
        "values": [
            "S",
            "M",
            "L",
            "XL"
        ]
    }
]
```

\* Note that options sent in for product detail page view events includes ALL possible values each option can take, not a particular choice uses chose.

Field "variants" is defined as follows :

```json
[
    {
        "availability": $availability,
        "catalog_discount": $catalog_discount,
        "image_url": $image_url,
        "price": $price,
        "properties": $properties,
        "sku": $sku,
        "title": $full_variant_title,
        "uii": $uii,
        "variant": $variant_title
    },
    ...
]
```
where

| Field | Description | Required |
| -------------:|:-------------|:-------------|
| availability | a boolean value, True if this variant is displayed as available, False otherwise | Yes, if there are no variants. |
| catalog_discount | discount publicly announced and applied automatically to this variant. This may not be directly what's displayed to user, e.g. original price before discount shown along with discounted price, or sale %, etc. In such case, this value must be computed from those values. | No |
| image_url | a URL of the thumbnail image of this variant. | No |
| price | price of this variant, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |
| properties | a dictionary of key value pairs that contain static information about the variant, e.g. brand, width, etc. | No |
| sku | SKU of product. | No |
| full_variant_title | product-level title + ' ' + variant-level title, e.g. 'Adrafinil Capsules' + ' ' + '30 CAPSULES' == 'Adrafinil Capsules 30 CAPSULES' | Yes |
| uii | a string that uniquely identifies a variant, e.g. stock number, SKU, UPC, etc. | Yes |
| url | URL address of variant page user visited. Only necessary when a separate page exists for this variant in addition to a main product page | Yes, if a web page view log. |
| variant_title | variant-level title, e.g. '30 CAPSULES' | Yes |

\* In most cases, variants are presented to users altogether in a single page view. All such variants in a single page must be
stored in the same event log.

\* An _option_ is different from a _property_ in that option is for which value users have freedom to choose among a few possibilities,
e.g. size of T-shirts, while a property is static to a product such that users do not have freedom choose its value, e.g. size, color,
or weight of diamonds.

\* Even if a product comes in variants, a product-level _uii_ must be provided, by which variants for this product could be grouped.

##### Examples

---

