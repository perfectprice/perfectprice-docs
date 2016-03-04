# Historical Data

### Introduction

Historical data for our analysis consists of :

    1) aggregations of core metrics,
    2) at varying combinations of attributes,
    3) for each *product*.

##### 1) Metrics

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| __view_count__ | number of times a product is viewed, e.g. web page view, app page load, etc. | Yes | 
| __purchase\_count__ | number of times a product is purchased. | Yes | 
| __purchase\_quantity__ | quantity a product is purchased. | Yes | 
| __add\_to\_cart\_count__ | number of times a product is added to cart. | Yes | 
| __add\_to\_cart\_quantity__ | quantity a product is added to cart. | Yes |
| average\_cart\_quantity | average quantity of items per checkout, this is avereage of sum of quantities for all items in checkouts. | No |
| average\_cart\_value | average amount spent per checkout. | No |


\* W.o.l.g. above metrics is aggregated by 2) and 3).

##### 2) Attributes

| Field | Description | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| __title__ | title of the product. | Yes | Phoenix Dan Cong |
| __sku__ | SKU of product. | Yes | K-MCRW40 |
| __datetime__ | date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. This is the beginning time of the aggregation period. | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| __duration__ | an integer that indicates the number of seconds in the aggregation period. | Yes | 86400 |
| __price__ | price represented to user, a string value with no currency symbol, but possibly with digit separators e.g. ',' or '.', etc. This is usually a price after subtracting catalog discount. | Yes | 321.20 or 1,234.56 |
| catalog_discount | discount publicly displayed and applied automatically. | No | 1.99 |
| cart_discount | discount applied only when certain action is taken on purchased items, e.g. entering coupon codes., a string value with no currency symbol, but possibly with digit separators e.g. ',' or '.', etc. | No | 5.99 |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. | No | USD |
| option_type | type of option | No | Size |
| option_value | value of option | No | XL |

\* If more one combination of option\_type and option\_value exist for a product, then aggregation metrics could be broken down into each combination. However, in most such cases, those options could have been displayed in the same page, i.e. the view\_count applies to all those options equally. For such cases, repeat view\_count over multiple lines for each combintation.

\* Similar to the case with view\_counts being shared for varying option\_value, view\_count cannot be aggregated for attribute cart\_discount, because cart\_discount only shows up in purchase events. Hence, the view\_count applies to all those options equally. For such cases, repeat view\_count over multiple lines for each combintation.

\* For promotional analysis using coupons, catalog and/or cart (coupon) discounts must be provided, if applied. Omitting thse values are interpreted as having no promotions.


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
