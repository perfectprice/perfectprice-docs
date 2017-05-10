# Car rental raw data standard format

### 1 Introduction

This document covers booking (reservation) and rental data coming __into__ Perfect Price. This applies both when you are getting set up with Perfect Price and on an ongoing basis, so the AI can constantly learn from experience and react to market changes. 

For market data (competitor prices, price shops, etc.) please see [the specifications for market data here](https://github.com/perfectprice/perfectprice-docs/blob/master/data-formats/competition.md). 

We are __sometimes__ able to customize various data structures or import your data in the format you can easily export it in, and transform it into the format we need. So please ask us if your data does not line up perfectly with our formats! We don't expect it to. 

It is __usually__ worthwhile to send us an export of data in the format you have easy access to. Sometimes we may be able to [transform](https://en.wikipedia.org/wiki/Extract,_transform,_load) it into the format we need more easily or reliably than your systems can. 

### 2 Architecture Overview

Our system stores data in a hadoop cluster in the cloud, in [JSON](https://en.wikipedia.org/wiki/JSON). This is then stored and used for charting, modeling, or other purposes. Wherever possible we do all computations on the raw data, rather than importing aggregated data. 

Models are typically run daily, with burst or surge models run intraday (up to every 5 minutes). The more rapidly we receive data updates from your system, the more responsive and "human like" the AI can be. However, we have flexibility around frequency of updates, and not everything needs to be updated at high frequency. 

We duplicate all data on our side. If you have a small data set and a data warehouse that allows it, we can re-read all data in for several years each time we run the model. This enables us to pull in updates that you may retroactively make to your data and include that auotmatically in our models. However that can be an expensive and inefficient process and therefore we can also simply pull and write new data, which is how most customers prefer to run the system. It is important to have well maintained ids so that, for example, if a booking is modified, we can update it on our side as well.

#### 2.1 Data categories

The broad categories of data we need are as follows:

| Type | Description | Can be manually updated? | Can be batch updated? | Can be connected to an API? |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| Vehicles | Data on each vehicle in your fleet used for availability, location and cost calculations | No | Yes | Yes |
| Locations | Data on each location or virtual location, used for cost calculations | Yes | Yes | Yes |
| Bookings | Raw bookings in bulk or as they come in | No | Yes | Yes |
| Rentals | Raw data on actual rentals (contracts) as they happen | No | Yes | Yes |
| Costs | Any model you may already have on operating costs for optimizations, or raw data such as acquisition/disposal/mileage of vehicles, variable cost for pickup/delivery, or commissions by channel  | Yes | No | No |

#### 2.2 Note on PII

Please do *not* send personally identifiable information (PII) such as email, real name, phone numbers, etc. Customer specific data needs to be either hashed (using [SHA-1](https://en.wikipedia.org/wiki/SHA-1) for example) or not sent at all. 

### 3 Formats

The data will end up in the below formats to be consumed and used by our AI. Some customizations are possible. Remember, we can help get the data into the right format even if your system cannot.

#### 3.1 Historical exports

Historical exports serve to either backfill months or years of data or to give Perfect Price's team a sample of data so we can provide better guidance in architecting a scalable, stable method of sending data. 

For one time, historical exports, we can accept 3 formats:
* JSON (preferred)
* XML
* CSV (best for direct SQL exports)

If a CSV, our system will map your fields to the fields that are expected. This can be a more brittle method of data transfer and we recommend against it for complex data structures or any data structure to be used on an ongoing basis that may change. 

#### 3.2 Ongoing updates

For ongoing updates, we can accept the same 3 formats, with conditions:
* JSON (preferred) - ideally, in API/webservice
* XML - API/webservice preferred
* CSV - Direct export or FTP dump.

The ideal method is always an API or webservice to send us incremental data, or for us to query for data updated since the last query. This is less efficient than a 1-time data export, but a well architected API or Webservice could support large batch transfers. 

Examples of when you would push new data to us, or what we would be querying to find updates for include:
* new bookings (reservations)
* new rentals or pickups (pickups or contracts)
* new returns
* changes in availability of a vehicle (recall, repair, delivery, acquisition, disposal, etc.)
* vehicles added or removed to the system 

Batch updates or individual updates are both ok. The closer to real time, the better the AI can react. 

#### 4 Raw Data Requirements

The AI needs at a minimum the same information a human would look at, but it needs it to be accurate and complete. For example if you have a clever way of removing vehicles for repair by, say, setting up a rental for $0 to a John Repair, it will affect the AI (yes, true story!). 

In all cases we __strongly prefer raw data__. That is, the actual transactions. It is easier and preferable for us to make computations on our side rather than work from already computed numbers. For example, giving us the start and end mileage on each reservation is much better than just giving an average mileage per rental across your entire fleet. Our system is capable of optimizing using mileage on a vehicle, day, day of week, month of year, and location basis in a way that humans cannot possibly achieve. 

We need comprehensive information across several categories, which follow. Note that some information is duplicative or not required at all, but we include everything for comprehensiveness. Easier to set this up once than to have to go in and redo it.

#### 4.1 Vehicles

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| vehicle\_unique\_id | Unique identifier for that vehicle | string | Yes | EXAMPLE |
| vehicle\_name | Human readable name of vehicle | string | Yes | Sprinter Bench |
| body\_style | Body style of vehicle | string | No | custom_van |
| car\_classes | Standardized 4- or 5-letter car classes vehicle is associated with can have multiple | string | No | XXAR,CCAR |
| make | Brand or make of car | String | Yes | Mercedes |
| model | Manufacturer name of car | String | Yes | Sprinter |
| year  | Model year of car | integer | Yes | 2016|
| color | Exterior color of car in cents | integer | Yes | Black |
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
| autonomous\_level | Whether car is autonomous capable and if so what [level, 0-5](https://en.wikipedia.org/wiki/Autonomous_car) | integer | No | 5 |
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

Rate codes can be manually configured with the proper inputs during setup and configuration, or imported programmatically. The id's must be used in other data sets (eg Bookings) that match. 

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| rate\_code\_id | id for this rate code, in the event the code changes   
| rate\_code | The rate code key in your system | string | Yes | EXAMPLE |
| lor | Length of rent | integer | Yes | 3 | 
| lor\_units | Units for LOR | char one of h, d, w, m | Yes | d |
| default_price | The default price for this rate code, or the current price, in cents | Integer | No | 2000 |
| default_price_currency | Currency for default price | string | No | USD |
| vehicles | Array of vehcile IDs associated with this rate code at the current time | string | No | EXAMPLE |
| dates | Array of tuples when rate code was in effect in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) If left blank then new rate code overwrites old rate code for future and past.| string | No | EXAMPLE |

#### 4.3 Car Classes (Vehicle types)

Car classes can be manually configured with the proper inputs during setup and configuration, or imported programmatically. The id's must be used in other data sets (eg Bookings) that match. 

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

Locations can be manually configured with the proper inputs during setup and configuration, or imported programmatically. The id's must be used in other data sets (eg Bookings) that match. 

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
| latlong | String expression of lat, long of location | string in [ISO 6709](https://en.wikipedia.org/wiki/ISO_6709) format | No |  \_+40.6894-074.0447\_/ |
| airport | Airport code associated with location if an airport location | string | No | SFO |
| is\_deleted | Whether the location is deleted/no longer a location | boolean | No | FALSE |
| ota\_code | Location code used by online travel agencies or GDS systems | string | No | SFO |
| shuttle | Whether the location provides a shuttle from the Airport it is associated with | BOOLEAN | No | FALSE |
| off\_airport | Whether the location is associated with an airport but is off-airport | BOOLEAN | No | FALSE |

#### 4.5 Reservations (or Bookings)

Reservations or bookings are confirmed requests for a car, which may or may not require prepayment or deposit. They may or may not chnage. The renter may or may not show up. If the renter shows up and picks up the car, this will result in a Rental (or contract, below). Much is unknown at the time of a reservation, which is fine. 

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
| car\_class | Car or vehicle class associated with reservatoin, must be joinable with one of your car classes, preferalby by car\_class\_id | string | No | EXAMPLE |
| vehicle\_id | Vehicle assigned to this specific reservation | string, specific id from vehicles | No | 2345h2DAHE |  
| total\_amount | Total amount of booking cost in cents | integer | No | 13400 |
| fees\_amount | Array of name and amount of tax or fee for each tax or fee charged | string | No | EXAMPLE |
| ancillary\_services | Array of ancillary services and charges for those services | string | No | EXAMPLE |
| ancillary\_amount | Cost of ancillary charges in cents | integer | No | 13400 |
| deposit\_amount | Amount of deposit in cents | integer | No | 10000 | 
| prepaid\_amount | Amount of nonrefundable prepaid payment | No | 13400 | 
| refundable | Whether or not the prepayment or deposit amount paid is refundable | integer | No | TRUE | 
| currency | Currency for all revenue and rate amounts | string | No | USD | 
| customer\_id | Unique identifier for customer, possibly a hashed email address, that can be joined with other data to inform segmentation, etc. | string | No | 2k31j4lkhago9h08h34 |

#### 4.6 Rentals (or Contracts)

When a car is  picked up and, generally, when a contract is issued it becomes a rental. A rental can be open (ie, still out and on rent) or closed (the car has been picked up and returned). For rentals out on rent, they will simply be missing an actual dropoff time. 

| Field name |  Description | Type | Required | Example |
|-------------:|:-------------|:-------------|:-------------|:-------------|
| reservation\_id | unique id of reservation, joinable with other tables | string | Yes | 32lkjghew2 |
| rental\_id | unique id of rental or contract, joinable with other tables | string | Yes | 32lkjghew2 |
| created\_time | Time and date rental was created in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| last\_modified\_time | Time and date rental was last modified in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_time | Time and date vehicle was picked up in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| pickup\_location | Location vehicle was picked up at, should be linked to your Locations | string | Yes | SFO |
| dropoff\_time | Time and date vehicle will be dropped off in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string 
| actual\_dropoff\_time | Time and date vehicle was actually dropped off in UTC using [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | string | Yes | 2017-04-13T22:00:20+00:00 |
| dropoff\_location | Location vehicle will be dropped off, should be linked to your Locations | string | Yes | LAX |
| rate\_codes | Array of rate codes for reservation, must be an ID of one of your rate code(s) | string | No | CCAR_1_day, CCAR_1_week |
| car\_classes | Car classes associated with reservation, must be an id of one of your car classes | string | No | BMW_Mini |
| vehicle\_id | Vehicle assigned to this specific reservation | string, specific id from vehicles | No | 2345h2DAHE |  
| total\_amount | Total amount of cost in cents | integer | No | 13400 |
| fees\_amount | Array of name and amount of tax or fee for each tax or fee charged | string | No | EXAMPLE |
| ancillary\_services | Array of ancillary services and charges for those services | string | No | EXAMPLE |
| ancillary\_amount | Cost of ancillary charges in cents | integer | No | 13400 |
| deposit\_amount | Amount of deposit in cents | integer | No | 10000 | 
| prepaid\_amount | Amount of nonrefundable prepaid payment | No | 13400 | 
| refundable | Whether or not the amount paid is refundable | integer | No | TRUE | 
| currency | Currency for all revenue and rate amounts | string | No | USD | 
| customer\_id | Unique identifier for customer, possibly a hashed email address, that can be joined with other data to inform segmentation, etc. | string | No | 2k31j4lkhago9h08h34 |

#### 4.7. Looks or pageviews (Optional)

When a user views a booking page, or you get a request from a GDS or online booking tool for a price quote, knowing that is important data in determining whether there is demand for a particular vehicle. 

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

### 5. Method of transfer

We can provision an API for you to push updates to using the above formats, or in some limited cases, a custom format. 

Alternatively, you can drop batch updates in a folder to which we are granted access, and we will poll that folder to look for updated files on a regular basis. The best practice is to create a subfolder for each day, and add raw files (json, CSV, etc.) labeled with an agreed upon naming format including time into the folder. For example:

/$org_id/type_of_file/$yyyy/$mm/$dd/$hh/type_of_file.csv

where:

| Type | Value |
|-------------:|:-------------|
| $org_id | the parent folder for your organizaiton (or our organization/customer if on your server) |
| type_of_file | the type of data being transferred, e.g., rentals, bookings, etc. | 
| $yyyy | 4 digit year, in UST |
| $mm | 2 digit month |
| $dd | 2 digit day of month | 
| $hh | 2 digit hour of the day in 24 hr clock |
| type_of_file.csv | name of the file, preferably something descriptive matching the data like 'reservations.csv' | 

Note that all times should be UST, or discussed with us in advance. 
