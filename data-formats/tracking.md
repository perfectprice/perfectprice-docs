# 1. Introduction

Standard format of log that tracking JS should send to logging server is provided in this document. Here, “The Company” refers to the company that signs up for Perfect Price, Inc. service, i.e. a client company of Perfect Price, Inc.

### Common Log Format
For certain events, we collect logs in JSON format, which is transferred to our log server as a POST request. Any data sent must be in the following form :

```json
{
    "json_text": $stringified_json_data,
    "version": $version
}
```

where :

| Field | Description |
| -------------: |:------------- |
| stringified\_json\_data | a Javascript dictionary variable that contains all data we collected first converted into JSON object and second converted into a string |
| version | a log version string in “integer.integer.integer” format |

Details of what must be collected at which event for stringified\_json\_data is explained in following sections. Version string starts from “1.0.0”, which is also the default value for logs from our old clients when version is omitted.

### Trasmitting Logs

Once constructed, make a POST request to :

```
http://log.pfpr.co/log/$company_hash
```

where :

| Field | Description |
| -------------: |:------------- |
| company_hash | a hash of client company, currently MD5 |

An example response from [log.pfpr.co](log.pfpr.co) :

```json
{
    "status": 0,
    "message": "Success",
    "data": $data
}

```

where

| Field | Description |
| -------------: |:------------- |
| status | an integer valued status code, which is 0 if POST request is successfully processed, or non-zero values if there was any backend error. Details of possible error status codes and what must be done is explained in the following section. |
| message | a string containing a brief note about what happened. |
| data | additional logging information |

__data__ is defined as follows :

```json
{
    "ppu": $ppu,
    "time": $timestamp
}
```
where

| Field | Description |
| -------------: |:------------- |
| ppu | a unique hash string assigned to this user, which is usually added into the cookie for key “ppu”, e.g. "e87e63b6f59a5ec6e0dc6bf472988d78" |
| timestamp | a timestamp that records logging time, a string value, e.g. "1446411036" |


### Handling Errors

##### Connection Errors
If our log server [log.pfpr.co](log.pfpr.co) is not responding or connectable, then logs sent will not be saved in our backend. Unless such logs are queued either at browser or some other server and resent, logs will be permanently lost. For safety, it is recommended to implement any actions to avoid log loss as much as possible, such as local queueing. 

##### Backend Errors
All backend errors happened in [log.pfpr.co](log.pfpr.co), while sending logs is internally handled so no specific error handling is necessary from the browser side.

##### Frontend Errors
Any frontend errors except for connection exceptions must be handled appropriately to avoid any log loss. Unless tracking Javascript code is provided by us, this is the responsibility of The Company.


### Coding Rules

##### Namespacing
To avoid conflict with any other existing JS on The Company's page, apply proper namespacing, e.g. prefixing functions or variables, etc.

##### JQuery
If JQuery is necessary, take an appropriate action to avoid version conflict.

##### JSON Field Naming
Default naming scheme for fields in any JSON data we generate is __lower\_case\_with\_underscords__.

 
# 2. Log Data
This section covers specifics of __stringified\_json\_data__ explained in previous section.

### 2.1 Logs from Page Load Events

Any log sent at __page load event__ must be of this format :

```json
{
    "document": {
        "base_uri": "https://www.betabrand.com/",
        "cookie": "",
        "document_uri": "https://www.betabrand.com/",
        "domain": "www.betabrand.com",
        "title": "Betabrand - Pants, Jackets, Hoodies, Breakthroughs",
        "url": "https://www.betabrand.com/"
    },
    "id": $company_hash,
    "navigator": {
        "app_code_name": "Mozilla",
        "app_name": "Netscape",
        "app_version": "5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36",
        "cookie_enabled": true,
        "language": "en-US",
        "online": true,
        "platform": "MacIntel",
        "product": "Gecko",
        "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36"
    },
    "page": $page_specific_data,
    "screen": {
        "available_height": 832,
        "available_width": 1440,
        "color_depth": 24,
        "height": 900,
        "pixel_depth": 24,
        "width": 1440
    },
    "type": $type_of_log,
    "window": {
        "closed": false,
        "default_status": "",
        "inner_height": 286,
        "inner_width": 1270,
        "outer_height": 832,
        "outer_weight": 1270,
        "screen_x": 22,
        "screen_y": 23,
        "status": ""
    }
}
```

where


| Field | Description |
| -------------: |:------------- |
| company_hash | a hash of client company, currently MD5 |
| page\_specific\_data | data that varies based on the page it is in. More dtail is to follow in the following sections. |
| type\_of\_log | a string that categorizes the context this log is generated, e.g. "pageview" if generated when page is loaded, or "checkout" if generated during cart checkout. |


\* page\_specific\_data and type\_of\_log differ on different pages and events.

\* Assume that values for fields in the event logs for keys that are not explicitly specified but only given as examples, e.g. "document", "navigator", etc. are trivial from the example values to be prepared using standard Javascript library.

\* For single page apps, page view events include not only the initial page load, but also subsequent pages re-rendering due to clicks. For each of a list of specified such page views in single page apps, event must be sent by The Company according to this format, or Perfect Price, Inc. if our tracking Javascript is used.


### page\_specific\_data Format

The format of this data depends on the content of pages that it is generated from. For our purpose, most e-commerce or retail pages fall into one of the following list of categories :

| Category | type\_of\_log | Description |
| -------------: |:-------- |:------------- |
| ITEM_DETAIL | "pageview" | a detailed page __per__ individual item |
| CATALOG | "pageview" | a page that contains a list / matrix of items as small cells, possibly paged, for any specific category or search result |
| CART | "pageview" | a cart view page |
| OTHER | "pageview" | all other pages |

\*Fields contained in these types of data may include but are not limited to what are defined in the formats below.

\*Certain fields may be omitted if __not__ available or applicable based on the specific content of pages data is scraped.

\*Specifics of fields common to various types of logs are separately provided in "2.3 Common Fields."

##### ITEM_DETAIL Page

Format of page\_specific\_data and type\_of\_log. \*Note that other common fields are not shown here for simplicity but must be present in real logs.

```json
{
    "page": {
        "meta": $meta_og_properties,
        "product": {
            "categories": $categories,
            "currency": $currency,
            "catalog_discount": $catalog_discount,
            "image_url": $image_url,
            "market": $market,
            "price": $price,
            "options": $options,
            "sku": $sku,
            "title": $title,
            "variations": $variations
        }
    },
    "type": "pageview"
}
```

where

| Field | Description |
| -------------: |:------------- |
| options | a list of option type and possible values each option can take for the given product. See "2.3 Common Fields" for details. |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |
| variations | a list of variations of this product. This is different from options in that options is price-agnostic, while each variation is associated with different prices. |


__variations__ is defined as follows :

```json
[
    {
        "catalog_discount": $catalog_discount,
        "image_url": $image_url,
        "price": $price,
        "sku": $sku,
        "title": $title
    },
    ...
]
```
where

| Field | Description |
| -------------: |:------------- |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |


Example of a ITEM_DETAIL log :

```json
{
    "json_text": {
        "document": {
            "base_uri": "https://www.harney.com/phoenix-dan-cong.html",
            "cookie": "ppu=1f8f43905772406cea79332fe7ba1840; uid=349602443; SLIBeacon_368182260=ECZNEOCSE1455834894808TQHIRIIKZ; SLI2_368182260=ECZNEOCSE1455834894808TQHIRIIKZ.1455834894808.1.1; SLI4_368182260=1455834894808; _gat=1; __utmt=1; VIEWED_PRODUCT_IDS=5074%2C5075; _ga=GA1.3.341494139.1455834867; __utma=11410477.341494139.1455834867.1455834868.1455846369.2; __utmb=11410477.2.10.1455846369; __utmc=11410477; __utmz=11410477.1455834868.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none); _sbtk=e30%3D",
            "document_uri": "https://www.harney.com/phoenix-dan-cong.html",
            "domain": "www.harney.com",
            "title": "Phoenix Dan Cong",
            "url": "https://www.harney.com/phoenix-dan-cong.html"
        },
        "id": "2f3e8b1797e471de254457b753ff433b",
        "navigator": {
            "app_code_name": "Mozilla",
            "app_name": "Netscape",
            "app_version": "5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36",
            "cookie_enabled": true,
            "language": "ko",
            "online": true,
            "platform": "MacIntel",
            "product": "Gecko",
            "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36"
        },
        "page": {
            "meta": {
                "description": "A nice twisted oolong that has subtle roasted peach flavors.",
                "image": "https://livemedia-harneyteas.netdna-ssl.com/media/catalog/product/cache/1/small_image/200x200/9df78eab33525d08d6e5fb8d27136e95/h/-/h-s150805_0012_phoenix_dan_cong.jpg",
                "title": "Phoenix Dan Cong | Harney & Sons Fine Teas",
                "type": "product",
                "url": "https://www.harney.com/phoenix-dan-cong.html"
            },
            "product": {
                "categories": [
                    "Tea",
                    "Phoenix Dan Cong"
                ],
                "variations": [
                    {
                        "image_url": "https://livemedia-harneyteas.netdna-ssl.com/media/catalog/product/cache/1/image/370x424/0dc2d03fe217f8c83829496872af24a0/h/-/h-s150805_0012_phoenix_dan_cong.jpg",
                        "price": "20.00",
                        "sku": "44615",
                        "title": "Phoenix Dan Cong - loose 2 oz tin"
                    },
                    {
                        "image_url": "https://livemedia-harneyteas.netdna-ssl.com/media/catalog/product/cache/1/image/370x424/0dc2d03fe217f8c83829496872af24a0/h/-/h-s150805_0012_phoenix_dan_cong.jpg",
                        "price": "156.00",
                        "sku": "41509",
                        "title": "Phoenix Dan Cong - loose 1 lb bag"
                    }
                ],
                "title": "Phoenix Dan Cong"
            }
        },
        "screen": {
            "available_height": 1413,
            "available_width": 2560,
            "colod_depth": 24,
            "height": 1440,
            "pixel_depth": 24,
            "width": 2560
        },
        "window": {
            "closed": false,
            "default_status": "",
            "inner_height": 609,
            "inner_width": 1239,
            "outer_height": 1324,
            "outer_weight": 1239,
            "screen_x": 169,
            "screen_y": 39,
            "status": ""
        }
    },
    "version": "1.2.0"
}
```

##### CATALOG Page

Format of page\_specific\_data and type\_of\_log. \*Note that other common fields are not shown here for simplicity but must be present in real logs.

```json
{
    "page": {
        "categories": $categories,
        "currency": $currency,
        "market": $market,
        "products": $products
    },
    "type": "pageview"
}
```

__products__ is defined as follows : 

```json
[
    {
        "catalog_discount": $catalog_discount,
        "image_url": $image_url,
        "price": $price,
        "sku": $sku,
        "title": $title
    },
    ...
]
```
where

| Field | Description |
| -------------: |:------------- |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |


##### CART Page

When a use visits a cart view page, which shows a list of items in the shopping cart, a cart log must be sent. \*Note that a cart page view is different from an __add to cart__ event log, which is sent when a use takes an action to add item to a cart. 

Format of page\_specific\_data and type\_of\_log. \*Note that other common fields are not shown here for simplicity but must be present in real logs.


```json
{
    "cart": {
        "currency": $currency,
        "items": $items,
        "market": $market,
        "total": $total,
        "total_discount": $total_discount
    },
    "type": "add_to_cart"
}
```

__items__ is defined as follows :

```json
[
    {
        "added": true
        "cart_discount": $cart_discount,
        "catalog_discount": $catalog_discount,
        "options": $options,
        "price": $price,
        "quantity": $quantity,
        "sku": $sku,
        "title": $title
    },
    ...
]
```
where

| Field | Description |
| -------------: |:------------- |
| added | an optional Boolean value which must be set to true if this item is added in this event, default = false. |
| price | price, __after__ any type of discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |

##### OTHER Page

There is no specific fields needed for this category of page logs. In this case, page\_specific\_data could have a general information like meta. For instance,

```json
{
    "meta": $meta_og_properties
}
```

### 2.2 Logs from Checkout Events
When an item is *actually* purchased, this log must be sent. This therefore is  a log that gets sent not on a "page load" event, but a successful completion of "purchase", e.g. finishing placing orders, returning from Stripe or Paypal checkout page after making payments.

In this example, page\_specific\_data is in red below :

```json
{
    "id": $company_hash,
    "checkout": {
        "items": $items,
        "shipping": $shipping,
        "tax": $tax,
        "total": $total,
        "total_discount": $total_discount,
        "currency": $currency,
        "market": $market
    }
}
```

where

| Field | Description |
| -------------: |:------------- |
| items | a list of items purchased. |
| shipping | shipping method. See below for more details. |
| tax | total tax applied. |

__items__ is defined as follows :

```json
[
    {
        "cart_discount": $cart_discount,
        "catalog_discount": $catalog_discount,
        "options": $options,
        "price": $price,
        "quantity": $quantity,
        "sku" : $sku,
        "title": $title
    },
    ...
]
```
where
        
| Field | Description |
| -------------: |:------------- |
| price | price, __after__ any type of discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |
| quantity | an integer value that contains the purchase count of this item |


__shipping__ is defined as follows :

```json
{
    "option": $option,
    "cost": $cost
},     
```
where

| Field | Description |
| -------------: |:------------- |
| option | string containing the shipping option, e.g. Fast (3 Days), Standard (5-7 Days), etc. |
| cost | the shipping cost. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ |


### 2.3 Common Fields


| Field | Description |
| -------------: |:------------- |
| categories | a list of categories name(s), e.g. "Home > Groceries > Vegetables > Organic" will be ```["Home", "Groceries", "Vegetables", "Organic"]``` |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. |
| cart_discount | discount applied only when certain action is taken on purchased items, e.g. entering coupon codes. |
| catalog_discount | discount publicly displayed and applied automatically |
| image_url | a URL of the thumbnail image of this product. |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. |
| meta\_og\_properties | a list of meta-tag contents for og:xxx properties. See below for details. |
| options | a list of option type and possible values each option can take for the given product. See below for details. |
| sku | SKU of product. |
| title | title of the product, which MUST BE the same as what’s shown in ITEM_DETAIL logs. |
| total | total amount for products with catalog or cart discounts applied.  |
| total\_discount | sum of catalog and cart discounts applied. |

__meta\_og\_properties__ is defined as follows :

```json
{
    $meta_og_key: $meta_og_value,
              :
}
```
where

| Field | Description |
| -------------: |:------------- |
| meta\_og\_key | property name of a meta-og tag, e.g. \<META PROPERTY="og:title" CONTENT="This is title."\> __without__ "og:" prefix. |
| meta\_og\_value | content of a meta-og tag |


__options__ is defined as follows :

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
| option_name | a string that indicates the name of this option, e.g. "size", "weight", etc. |
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
