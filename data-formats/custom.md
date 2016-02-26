# 1. Cost



### Introduction


"Cost" here refers to __variable__ cost, an amount that is to be deducted from the price of
each *product* or *stock unit* to compute [contribution margin](https://en.wikipedia.org/wiki/Contribution_margin),
which does not include cart-level costs like shipping or processing fee. A *product* is an
individual item that's priced, listed, and sold, which is a unit that we collect and aggregate
behavior metrics.

If what could be gathered and provided is not __variable__ costs in the traditional definition,
still the format of data remains the same, while the interpretation of computed margin could vary.
Discussion with our expert is recommended should this need arise.

### Format

| Field | Description | Required | Example |
|-------------:|:-------------|:------------|:------------|
| __datetime__ | Date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| __title__ | Name of product. | Yes | Youde UD Prebuilt Clapton Coils |
| __sku__ | SKU of product. | Yes | K-MCRW40 |
| __amount__ | Cost represented without any currency symbol, but possibly with digit separators e.g. ',' or '.', etc. | Yes | 321.01 |
| currency_code | A three capital letter currency code in [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) format. Default value is “USD”. | No | USD or KRW |
| type | A short description on what type of cost this is. This is useful to list different types of cost for each item. Default value is “total”. | No | Material, Commission, Credit Card Fee, etc. |
| id | A unique product ID, e.g. [UPC](https://en.wikipedia.org/wiki/Universal_Product_Code) or [GTIN](http://www.gs1.org/gtin), etc., other than SKU, if available. | No | 036000291453 |
| description | Any note about cost in particular, if needed. | No | ... |

which could be provided either in CSV or in JSON as follows :

##### CSV

[ Example 1 - All columns provided with empty placeholders for optional columns with no values ]

```
datetime,title,sku,amount,currency_code,type,id,description
2016-01-23,Youde UD Prebuilt Clapton Coils,K-MCRW40,321.10,USD,total,036000291453,Total cost as of today
2016-01-23,Jasmine Tea Pearl Dragon,JASPERL-2202,5.99,USD,,,Missing type and UPC
...
```

[ Example 2 - Non-required fields omitted. ]

```
datetime,title,sku,amount
2016-01-23,Youde UD Prebuilt Clapton Coils,K-MCRW40,321.10
2016-01-23,Jasmine Tea Pearl Dragon,JASPERL-2202,5.99,USD
...
```

##### JSON

[ Example - Non-required fields with no values could be omitted. ]

```json
[
    {
        "datetime": "2016-01-23",
        "title": "Youde UD Prebuilt Clapton Coils",
        "sku": "K-MCRW40",
        "amount": "321.10",
        "currency_code": "USD",
        "type": "total",
        "id": "036000291453",
        "description": "Total cost as of today"
    },
    {
        "datetime": "2016-01-23",
        "title": "Jasmine Tea Pearl Dragon",
        "sku": "JASPERL-2202",
        "amount": "5.99",
        "currency_code": "USD",
        "description": "Missing type and UPC"
    },
    ...
]
```

# 2. Unique ID

### Introduction

Unique ID here refers to standard codes assigned to each *product* such as : 
[UPC](https://en.wikipedia.org/wiki/Universal_Product_Code), [GTIN](http://www.gs1.org/gtin), etc.

### Format

| Field | Description | Required | Example |
|-------------:|:-------------|:------------|:------------|
| __title__ | Name of product. | Yes | Youde UD Prebuilt Clapton Coils |
| __sku__ | SKU of product. | Yes | K-MCRW40 |
| __id__ | A unique product ID, e.g. [UPC](https://en.wikipedia.org/wiki/Universal_Product_Code) or [GTIN](http://www.gs1.org/gtin), etc., other than SKU, if available. | Yes | 036000291453 |
| type | Name of unique ID standard. This is useful to list different types of unique IDs if more than one are assigned to a *product*. Default value is “UPC”. | No | UPC or GTIN, etc. |

which could be provided either in CSV or in JSON as follows :


##### CSV

[ Example 1 ]

```
title,sku,id,type
Youde UD Prebuilt Clapton Coils,K-MCRW40,036000291453,UPC
Jasmine Tea Pearl Dragon,JASPERL-2202,00110234223,GTIN
...
```

[ Example 2 - Multiple unique IDs in multiple lines ]

```
title,sku,id,type
Youde UD Prebuilt Clapton Coils,K-MCRW40,036000291453,UPC
Youde UD Prebuilt Clapton Coils,K-MCRW40,192381209312,GTIN
Jasmine Tea Pearl Dragon,JASPERL-2202,00110234223,GTIN
...
```

##### JSON

[ Example - Unique IDs are in "ids" array. ]

```json
[
    {
        "title": "Youde UD Prebuilt Clapton Coils",
        "sku": "K-MCRW40",
        "id": [
            {
                "type": "UPC",
                "value": "036000291453"
            },
            {
                "type": "GTIN",
                "value": "192381209312"
            }
        ]
    },
    {
        "title": "Jasmine Tea Pearl Dragon",
        "sku": "JASPERL-2202",
        "id": [
            {
                "type": "GTIN",
                "value": "00110234223"
            }
        ]
    },
    ...
]
```

# 3. Historical Data

### Introduction

Historical data for our analysis consists of :

    1) aggregations of core metrics,
    2) at varying combinations of attributes,
    3) for each *product*.

##### 1) Metrics

| Field | Description | Required |
|-------------:|:-------------|:-------------|:-------------|
| __view_count__ | Number of times a product is viewed, e.g. web page view, app page load, etc. | Yes | 
| __purchase\_count__ | Number of times a product is purchased. | Yes | 
| __purchase\_quantity__ | Quantity a product is purchased. | Yes | 
| __add\_to\_cart\_count__ | Number of times a product is added to cart. | Yes | 
| __add\_to\_cart\_quantity__ | Quantity a product is added to cart. | Yes | 

\* W.o.l.g. above metrics is aggregated by 2) and 3).

##### 2) Attributes

| Field | Description | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| __title__ | title of the product. | Yes | Phoenix Dan Cong |
| __sku__ | SKU of product. | Yes | K-MCRW40 |
| __datetime__ | Date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. This is the beginning time of the aggregation period. | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| __duration__ | An integer that indicates the number of seconds in the aggregation period. | Yes | 86400 |
| __price__ | Price represented to user, a string value with no currency symbol, but possibly with digit separators e.g. ',' or '.', etc. This is usually a price after subtracting catalog discount. | Yes | 321.20 or 1,234.56 |
| catalog_discount | discount applied only when certain action is taken on purchased items, e.g. entering coupon codes., a string value with no currency symbol, but possibly with digit separators e.g. ',' or '.', etc. | No | 5.99 |
| cart_discount | discount publicly displayed and applied automatically | No | 1.99 |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. | No | USD |
| option_type | type of option | No | Size |
| option_value | value of option | No | XL |

If more one combination of option_type and option_value exist for a product, then aggregation metrics could be broken down into each combination. However, in most such cases, those options could have been displayed in the same page, i.e. the view_count applies to all those options equally. For such cases, repeat view_count over multiple lines for each combintation.

##### 3) Product Info

| Field | Description | Required | Example |
| -------------: |:------------- |:------------- |:------------- |
| __title__ | title of the product | Yes | Phoenix Dan Cong |
| __sku__ | SKU of product. | Yes | K-MCRW40 |
| categories | a list of categories name(s) | No | "Home > Groceries > Vegetables > Organic" will be ```Home|Groceries|Vegetables|Organic``` in CSV (with '\|' escaped if part of category name) or ```["Home", "Groceries", "Vegetables", "Organic"]``` in JSON |
| image_url | a URL of the thumbnail image of this product. | No | https://101vape.com/3639-large_default/fruit-whip-by-kilo-e-liquids.jpg |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. | No | en-US |

which could be provided either in CSV or in JSON as follows :

##### CSV

If provided as a CSV, then provide two files such that *attribute and metric file* that contains 1) + 2) and the *product info file* that contains 3). The product info file is not required if no other product info than what are in attribute and metric file.

[ Example - Showing two different display prices for Phoenix Dan Cong, and two different lines of Barilla Pasta Sause for different sizes ]

##### Attributes and Metrics File

```
title,sku,datetime,duration,price,catalog_discount,cart_discount,currency,option_type,option_value,view_count,purchase_count,purchase_quantity,add_to_cart_count,add_to_cart_quantity
Phoenix Dan Cong,K-MCRW40,2016-01-23T22:49:05+00:00,86400,39.99,5.99,1.99,USD,,,100000,900,1200,1500,2000
Phoenix Dan Cong,K-MCRW40,2016-01-23T22:49:05+00:00,86400,49.99,,,USD,,,10000,90,100,150,200
Barilla Pasta Sause,BPS-1123,2016-01-23,86400,19.99,,,USD,1000,Size,24oz,1000,20,25,30,35
Barilla Pasta Sause,BPS-1123,2016-01-23,86400,19.99,,,USD,1000,Size,12oz,1000,40,56,50,80
...
```
Note that view_count of 1000 is shared between two different sizes of 24oz and 12oz.

##### Product Info File

```
title,sku,categories,image_url,market
Phoenix Dan Cong,K-MCRW40,Home|Groceries|Vegetables|Organic,https://101vape.com/3639-large_default/fruit-whip-by-kilo-e-liquids.jpg,en-US
...
```

##### JSON

```json
[
    {
        "title": "Phoenix Dan Cong",
        "sku": "K-MCRW40",
        "datetime": "2016-01-23T22:49:05+00:00",
        "duration": 86400,
        "price": "39.99",
        "catalog_discount": "5.99",
        "cart_discount": "1.99",
        "currency": "USD",
        "view_count": 100000,
        "purchase_count": 900,
        "purchase_quantity": 1200,
        "add_to_cart_count": 1500,
        "add_to_cart_quantity": 2000
    },
    {
        "title": "Phoenix Dan Cong",
        "sku": "K-MCRW40",
        "datetime": "2016-01-23T22:49:05+00:00",
        "duration": 86400,
        "price": "49.99",
        "currency": "USD",
        "view_count": 10000,
        "purchase_count": 90,
        "purchase_quantity": 100,
        "add_to_cart_count": 150,
        "add_to_cart_quantity": 200
    },
    {
        "title": "Barilla Pasta Sause",
        "sku": "BPS-1123",
        "datetime": "2016-01-23",
        "duration": 86400,
        "price": "19.99",
        "currency": "USD",
        "option_type": "Size",
        "option_value": "24oz",
        "view_count": 1000,
        "add_to_cart_quantity": 20,
        "purchase_count": 25,
        "purchase_quantity": 30,
        "add_to_cart_count": 35
    },
    {
        "title": "Barilla Pasta Sause",
        "sku": "BPS-1123",
        "datetime": "2016-01-23",
        "duration": 86400,
        "price": "19.99",
        "currency": "USD",
        "option_type": "Size",
        "option_value": "12oz",
        "view_count": 1000,
        "purchase_count": 40,
        "purchase_quantity": 56,
        "add_to_cart_count": 50,
        "add_to_cart_quantity": 80
    }
]
```



# 4. Providing Data

Please email us the CSV or JSON file, until an alternative method is specified here.
