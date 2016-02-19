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

Response for this call is :

```json
{
    "status": 0,
    "message": "Success",
    "data": {
        "ppu": "e87e63b6f59a5ec6e0dc6bf472988d78",
        "time": "1446411036"
    }
}

```

where

| Field | Description |
| -------------: |:------------- |
| status | an integer valued status code, which is 0 if POST request is successfully processed, or non-zero values if there was any backend error. Details of possible error status codes and what must be done is explained in the following section. |
| message | a string containing a brief note about what happened. |
| data | additional meta information : |
| data/ppu | a unique hash assigned to this user, which is usually added into the cookie for key “ppu”. |
| data/time | a timestamp that records logging time. |


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

### 2.1 Common Format
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
        "default status": "",
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

For single page apps, page view events include not only the initial page load, but also subsequent pages re-rendering due to clicks. For each of a list of specified such page views in single page apps, event must be sent by The Company according to this format, or Perfect Price, Inc. if our tracking Javascript is used.


### 2.2 page\_specific\_data Format

The format of this data depends on the content of pages that it is generated from. For our purpose, most e-commerce or retail pages fall into one of the following list of categories :

| Category | type\_of\_log | Description |
| -------------: |:-------- |:------------- |
| ITEM_DETAIL | "pageview" | a detailed page __per__ individual item |
| CATALOG | "pageview" | a page that contains a list / matrix of items as small cells, possibly paged, for any specific category or search result |
| CART | "pageview" | a cart view page |
| OTHER | "pageview" | all other pages |


##### ITEM_DETAIL Page

Format of page\_specific\_data and type\_of\_log (Note that other common fields are not shown here for simplicity but must be present in real logs) :

```json
{
    "page": {
        "product": {
            "categories": $categories,
            "currency": $currency,
            "discount": $discount,
            "image_url": $image_url,
            "market": $market,
            "meta": $meta_og_properties,
            "price": $price,
            "options": $options,
            "sku": $sku,
            "title": $title,
            "variations": $variations
        }
    }
    "type": "pageview"
}
```

where

| Field | Description |
| -------------: |:------------- |
| categories | a list of categories name(s), e.g. "Home > Groceries > Vegetables > Organic" will be ```["Home", "Groceries", "Vegetables", "Organic"]``` |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. |
| discount | the amount discounted from price, if this is also shown to user. | 
| image_url | a URL of the thumbnail image of this product. |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. |
| meta\_og\_properties | a list of meta-tag contents for og:xxx properties. See below for details. |
| options | a list of option type and possible values each option can take for the given product. See below for details. |
| price | the price, AS DISPLAYED on item detail page. |
| sku | SKU of product. |
| title | the title of the product, which MUST BE the same as what’s shown in item detail view logs. |
| variations | a list of variations of this product. This is different from options in that options is price-agnostic, while each variation is associated with different prices. |


__product\_option\_array__ is defined as follows :

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

An example of product\_option\_array :

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

__variations__ is defined as follows :

```json
[
    {
        ""
    },
    ...
]
```

| Field | Description |
| -------------: |:------------- |
| meta\_og\_key | property name of a meta-og tag, e.g. \<META PROPERTY="og:title" CONTENT="This is title."\> __without__ "og:" prefix. |
| meta\_og\_value | content of a meta-og tag |


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
            "app code name": "Mozilla",
            "app name": "Netscape",
            "app version": "5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36",
            "cookie enabled": true,
            "language": "ko",
            "online": true,
            "platform": "MacIntel",
            "product": "Gecko",
            "user agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36"
        },
        "page": {
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
            "available height": 1413,
            "available width": 2560,
            "colod depth": 24,
            "height": 1440,
            "pixel depth": 24,
            "width": 2560
        },
        "window": {
            "closed": false,
            "default status": "",
            "inner height": 609,
            "inner width": 1239,
            "outer height": 1324,
            "outer weight": 1239,
            "screen x": 169,
            "screen y": 39,
            "status": ""
        }
    },
    "version": "1.2.0"
}
```

##### Add To Cart Log

When an item is added to cart, this must be sent. Example:

A user clicks “Add To Cart” button.



Note that a user chose options : Waist : 36, Length: 36.



A pop up opens up to show what’s added and what are in current cart.




In this example, page\_specific\_data is in red below :

```json
{
    "document": {
        "base URI": "https://www.betabrand.com/",
        "cookie": "optimizelyEndUserId=oeu1446252480923r0.7751143390778452;... ",
        "document URI": "https://www.betabrand.com/mens/pants/mens-navy-cordarounds-corduroy-pants.html",
        "domain": "www.betabrand.com",
        "title": "Nesting-Doll Cordarounds | Men's Navy Horizontal Corduroy Pants | Betabrand",
        "url": "https://www.betabrand.com/mens/pants/mens-navy-cordarounds-corduroy-pants.html"
    },
    "id": "97da7ff7f0e157b0874d839cfd79876e",
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
    "cart": {
        "items": [
            {
                "quantity": 1,
                "name": "Nesting-Doll Cordarounds",
                "price": "100.00",
                "discount": "10.00",
                "product_options": [
                    "Waist": "36",
                    "Length": "36"
                ],
                "added": true
            },
            {
                "quantity": 1,
                "name": "Some shirt",
                "price": "70.00",
                "discount": "8.00",
                "product_options": [
                    "Color": "Blue",
                    "Size": "M"
                ]
            }
        ],
        "tax": "15",
        "total": "170",
        "total discount": "18",
        "currency": "USD",
        "market": "en-US"
    },
    "screen": {
        "available_height": 832,
        "available_width": 1440,
        "color_depth": 24,
        "height": 900,
        "pixel_depth": 24,
        "width": 1440
    },
    "type": "add_to_cart",
    "window": {
        "closed": false,
        "default status": "",
        "inner_height": 206,
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
| items | a list of items contained in the cart, which contains sub fields : |
| items/quantity | an integer value that contains the count of this item in cart |
| items/name | the name of the product, which MUST BE the same as what’s shown in item detail view logs. |
| items/price | the price, AS DISPLAYED on item detail page. |
| items/discount | the amount discounted from price, if applied.  |
| product_options | a list of option type and value(s) selected for this item in cart. |
| added | an optional Boolean value which must be set to true if this item is added in this event, default = false. |
| tax | total tax applied. |
| total | sum of the item price x quantity for all items. |
| total\_discount | sum of discounts applied. |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. |

2.2.3. Purchase (Checkout) Log
When an item is purchased, this must be sent. Example:

A user clicks “Checkout” button, e.g. in the pop up



or in the cart slide section :




Then in the checkout page user clicks “Place Order” in order page.




In this example, page\_specific\_data is in red below :

```json
{
    "document": {
        "base URI": "https://www.betabrand.com/",
        "cookie": "optimizelyEndUserId=oeu1446252480923r0.7751143390778452;... ",
        "document URI": "https://www.betabrand.com/mens/pants/mens-navy-cordarounds-corduroy-pants.html",
        "domain": "www.betabrand.com",
        "title": "Nesting-Doll Cordarounds | Men's Navy Horizontal Corduroy Pants | Betabrand",
        "url": "https://www.betabrand.com/mens/pants/mens-navy-cordarounds-corduroy-pants.html"
    },
    "id": "97da7ff7f0e157b0874d839cfd79876e",
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
    "checkout": {
        "items": [
            {
                "quantity": 1,
                "name": "Nesting-Doll Corarounds",
                "price": "100.00",
                "discount": "10.00",
                "product_options": [
                    "Waist": "36",
                    "Length": "36"
                ]
            },
            {
                "quantity": 1,
                "name": "Some shirt",
                "price": "70.00",
                "discount": "8.00",
                "product_options": [
                    "Color": "Blue",
                    "Size": "M"
                ]
            }
        ],
        "shipping": {
            "option": "Fast (3 Days)",
            "cost": "16.95"
        },
        "tax": "15",
        "total": "170",
        "total_discount": "18",
        "currency": "USD",
        "market": "en-US"
    },
    "screen": {
        "available_height": 832,
        "available_width": 1440,
        "color_depth": 24,
        "height": 900,
        "pixel_depth": 24,
        "width": 1440
    },
    "type": "checkout",
    "window": {
        "closed": false,
        "default status": "",
        "inner_height": 206,
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
| items | a list of items contained in the cart, which contains sub fields : |
| items/quantity | an integer value that contains the count of this item in cart |
| items/name | the name of the product, which MUST BE the same as what’s shown in item detail view logs. |
| items/price | the price, AS DISPLAYED on item detail page. |
| items/discount | the amount discounted from price, if applied. |
| items/product_options | a list of option type and value(s) selected for this item in cart. |
| shipping | shipping method containing sub fields : |
| shipping/option | string containing the shipping option, e.g. Fast (3 Days), Standard (5-7 Days), etc. |
| shipping/cost | the shipping cost. |
| tax | total tax applied. |
| total | sum of the item price x quantity for all items. |
| total\_discount | sum of discounts applied. |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. |
