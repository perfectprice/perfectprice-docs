# Raw Data

### 1. Introduction

Generally speaking, raw data must provide basic information so that we could start from
__aggregations__ such as number of views, purchases, revenue, CVRs, etc. (using __event logs__),
at varying combinations of __attributes__ including date, time, region, user, etc (using __product data__).

### 2. Event Types

#### 2.1 View / Load

For web services, a __page view__ event is specifically defined as the completion of rendering the page
content in browsers for the accessed URL. For app services, it is not straight forward to define a
__page view__ event as there is no standardized notion of completion of rendering contents in apps.
Hence, for apps, we define '__tapping__' an area inside an app that results in loading relevant contents
such as product details, or catalogs, etc., is considered as a __page view__ event.

| Type | Description | Example |
|-------------:|:-------------|:-------------|
| product detail | a single product detail | [Example](http://www.bluenile.com/build-your-own-ring/diamond-engagement-ring-14k-white-gold_20305?elem=img&track=product) |
| catalog | a matrix of products and/or subcategories in certain category | [Example](http://www.bluenile.com/build-your-own-ring/settings?track=TitleVintage) |
| cart | Details (price, quantity, discounts, etc.) of products added to cart/basket | [Example](https://secure.bluenile.com/basket.html) |
| thank you | Result page loaded after successfully placing an order | |
| search | a list of products matching query | [Example](http://www.bluenile.com/build-your-own-ring/diamonds) |

#### 2.2 Click / Tap

For web services, a __click__ event is specifically defined as an action of clicking certain area of
a page that fires subsequent actions, e.g. form submission, pull down select/options menu, etc.
For apps, a __tap__ event is touching an area inside an app that results in similar types of subsequent
actions.

| Type | Description | Example |
|-------------:|:-------------|:-------------|
| product detail preview | showing product detail preview | click 'Detail' or 'Preview' to display a sliding sub-window or pop-up modal dialog | 
| checkout | place an order or make payment | click or tap on 'Submit' or 'Payment' button. |
| add to cart | add an item into cart/basket | click or tap on 'Add to Cart' buttom. |


### 3. Event Log Format

An event log contains common fields and event type specific fields in the following format :

```json
{
    $common_field_key1: $common_field_value1,
    $common_field_key2: $common_field_value2,
                    :,
    "properties": $event_specific_fields
}
```

#### 3.1 Common Fields

| Field | Description | Required | Example |
|-------------:|:-------------|:-------------|:-------------|
| __datetime__ | date and time in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format. If only date is provided, then default value of time is 00:00:00. This is the beginning time of the aggregation period. | Yes | 2016-01-23T22:49:05+00:00 or 2016-01-23 |
| __user_id__ | an anonymized character string uniquely identifying a user, if applicable | Yes | 56d61b53283d19956d60a6fa |
| __event\_type__ | a string indicating the type of event | Yes | 'page\_view', 'add\_to\_cart', etc. See below for more detail. |
| event\_id | a string uniquely identifying an event | No | 837840745805772274 |

Standard values of __event\_type__ for different types of events :

| Type | Value |
|-------------:|:-------------|
| product detail | page\_view |
| catalog | page\_view | 
| cart | page\_view |
| thank you | page\_view |
| search | page\_view |
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
    "event_type": "page_view",
    "properties": $event_type_specific_data
}
```

#### 3.2 Event Type Specific Fields

When a product is presented to users with properties that can take one or more than among a variety of values,
e.g. size 'Small' from a set of choices [ 'Small', 'Medium', 'Large' ], or color 'Red' among [ 'Blue', 'Green',
'Purple', 'Red', 'White' ], etc., a particulr set of choices of those properties may result in changes in prices
depending on the choice. For instance, a tea product may be sold in a 1Oz tin can at $10.00 USD, or as a 20 sachet
bags at $20.00 USD, or a T-shirt comes in different colors and sizes, while they all are sold at $15.00 USD.

When a variation incurs change in price, each variation is called a __variant__, while if not, it is called an
__option__. Distinguishing variants from options is important as our analysis is tightly coupled with prices.

##### product detail page views

Format of event\_type\_specific\_data is shown here, without common fields for simplicity but must be presented in real logs.

```json
{
    "price": $price,
    "title": $title,
    "uii": $uii,
    "url": $url,
    "options": $options,
    "sku": $sku,
    "variants": $variants
}
```

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| __price__ | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |
| __title__ | name of a product e.g. Earl Grey Tea | Yes |
| __uii__ | a string that uniquely identifies a product, e.g. stock number, SKU, UPC, etc. | Yes |
| __url__ | URL address of page user visited | Yes, if a web page view log. |
| catalog_discount | discount publicly announced and applied automatically. This may not be directly what's displayed to user, e.g. original price before discount shown along with discounted price, or sale %, etc. In such case, this value must be computed from those values. | No |
| image_url | a URL of the thumbnail image of this product. | No |
| options | an array of options for this product. See below for more detail. | No |
| sku | SKU of product. | No |
| variants | an array of variants for this product. See below for more detail. | No |

Field "options" is defined as follows :

```json
[
    {
        "type": $option_name,
        "values": $option_value_array
    },
    ...
]
```

where

| Field | Description |
| -------------: |:------------- |
| option_name | a string that indicates the name of this option, e.g. "size", "weight", "clarity", "color", etc. |
| option\_value\_array | an array of values one or more of which this option can take |

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
        ]
    },
    {
        "type": "Length",
        "values": [
            "32",
            "36"
        ]
    }
]
```

\* Note that options sent in for product detail page view events includes ALL possible values each option can take, not a particular choice uses chose.

Field "variants" is defined as follows :

```json
[
    {
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

| Field | Description |
| -------------: |:------------- |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |
| full_variant_title | product-level title + ' ' + variant-level title, e.g. 'Adrafinil Capsules' + ' ' + '30 CAPSULES' == 'Adrafinil Capsules 30 CAPSULES' |
| variant_title | variant-level title, e.g. '30 CAPSULES' |

\* In most cases, variants are presented to users altogether in a single page view. All such variants in a single page must be stored in the same event log.

Example of a product detail page view log :

[ A product with options ]

```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "page_view",
    "properties": {
        "title": "Petite Solitaire Engagement Ring",
        "variants": [
            {
                "currency": "USD",
                "market": "en-US",
                "price": "830",
                "sku": "19010",
                "image_url": "http://img.bluenile.com/is/image/bluenile/-petite-solitaire-ring-platinum-/setting_template_main?$phab_detailmain$&$diam_shape=is{bluenile/main_RD_standard_100}&$diam_position=0,50&$ring_position=0,0&$ring_sku=is{bluenile/19010_setmain}",
                "title": "Petite Solitaire Engagement Ring Pt",
                "uii": "19010",
                "url": "http://www.bluenile.com/build-your-own-ring/white-gold-engagement-ring-setting_19287?action=18kWhiteGoldSelect&track=alternate-metalsCustomizer",
                "variant": "Pt"
                "properties": {
                    "Width": "1.9mm",
                    "Prong Metal": "Platinum",
                    "Rhodium Plated": "Yes"
                }
            },
            {
                "currency": "USD",
                "market": "en-US",
                "price": "690",
                "sku": "19287",
                "image_url": "http://img.bluenile.com/is/image/bluenile/-petite-solitaire-ring-platinum-/setting_template_main?$phab_detailmain$&$diam_shape=is{bluenile/main_RD_standard_100}&$diam_position=0,50&$ring_position=0,0&$ring_sku=is{bluenile/19010_setmain}",
                "title": "Petite Solitaire Engagement Ring 18k",
                "uii": "19010",
                "url": "http://www.bluenile.com/build-your-own-ring/white-gold-engagement-ring-setting_19287?action=18kWhiteGoldSelect&track=alternate-metalsCustomizer",
                "variant": "18k"
            }
        ]
    }
}

```
```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "page_view",
    "properties": {
        "price": "23.99",
        "title": "Adrafinil Capsules",
        "uii": "ANDRAFINIL-CAPSULES-30",
        "options": [
            {
                "type": "Size",
                "values": [
                    "15 Grams",
                    "30 Grams"
                ]
            }
        ],
        "url": "http://www.powdercity.com/products/adrafinil-capsules",
        "sku: "ANDRAFINIL-CAPSULES-30",
    }
}

```

[ A product with variants ]

##### catalog page views

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| catalogs

### Attributes

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
