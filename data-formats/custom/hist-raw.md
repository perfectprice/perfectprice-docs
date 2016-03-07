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
| product detail | a single product detail, or its preview | [Example](http://www.bluenile.com/build-your-own-ring/diamond-engagement-ring-14k-white-gold_20305?elem=img&track=product) |
| catalog | a matrix of products and/or subcategories in certain category | [Example](http://www.bluenile.com/build-your-own-ring/settings?track=TitleVintage) |
| cart | Details (price, quantity, discounts, etc.) of products added to cart/basket | [Example](https://secure.bluenile.com/basket.html) |
| checkout | Result page loaded after successfully placing an order | N/A |
| adjustment | Changes in purchases, e.g. refund, canellation | N/A |
| search results | a list of products that match query | [Example](http://www.bluenile.com/build-your-own-ring/diamonds) |

#### 2.2 Click / Tap

For web services, a __click__ event is specifically defined as an action of clicking certain area of
a page that fires subsequent actions, e.g. form submission, pull down select/options menu, etc.
For apps, a __tap__ event is touching an area inside an app that results in similar types of subsequent
actions.

| Type | Description | Example |
|-------------:|:-------------|:-------------|
| add to cart | add an item into cart/basket | click or tap on 'Add to Cart' buttom. |
| search | search with a query | type search keywords and/or options and click or tap on 'Search' button |


### 3. Event Logs

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
    "sku": $sku,
    "title": $title,
    "uii": $uii,
    "url": $url,
    "variants": $variants
}
```

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| availability | a boolean value, True if product is displayed as available, False otherwise | Yes, if there are no variants. |
| catalog_discount | discount publicly announced and applied automatically. This may not be directly what's displayed to user, e.g. original price before discount shown along with discounted price, or sale %, etc. In such case, this value must be computed from those values. | No |
| categories | a list of categories name(s), e.g. "Home > Groceries > Vegetables > Organic" will be ["Home", "Groceries", "Vegetables", "Organic"] | No |
| currency | an optional three capital letter currency code (ISO 4217), default = “USD”. | No |
| image_url | a URL of the thumbnail image of this product. | No |
| market | an optional string, which is a concatenation of two letter language code (ISO 639-1) and two capital letter country code (ISO 3166-1 alpha-2), default = “en-US”. | No |
| options | an array of options for this product. See below for more detail. | No |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc. | Yes, if there are no variants. |
| sku | SKU of product. | No |
| title | name of a product e.g. Earl Grey Tea. In case variants exist, this is the represenatative product name for all variants. | Yes |
| uii | a string that uniquely identifies a product, e.g. stock number, SKU, UPC, etc. | Yes, if there are no variants. |
| url | URL address of page user visited | Yes, if a web page view log. |
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
| availability | a boolean value, True if product is displayed as available, False otherwise | Yes, if there are no variants. |
| catalog_discount | discount publicly announced and applied automatically. This may not be directly what's displayed to user, e.g. original price before discount shown along with discounted price, or sale %, etc. In such case, this value must be computed from those values. | No |
| image_url | a URL of the thumbnail image of this product. | No |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc. | Yes |
| properties | a dictionary of key value pairs that contain static information about the variant, e.g. brand, width, etc. | No |
| sku | SKU of product. | No |
| title | product-level title + ' ' + variant-level title, e.g. 'Adrafinil Capsules' + ' ' + '30 CAPSULES' == 'Adrafinil Capsules 30 CAPSULES' | Yes |
| uii | a string that uniquely identifies a product, e.g. stock number, SKU, UPC, etc. | Yes |
| url | URL address of page user visited | Yes, if a web page view log. |
| variant_title | variant-level title, e.g. '30 CAPSULES' | Yes |

\* In most cases, variants are presented to users altogether in a single page view. All such variants in a single page must be
stored in the same event log.

\* An _option_ is different from a _property_ in that option is for which value users have freedom to choose among a few possibilities,
e.g. size of T-shirts, while a property is static to a product such that users do not have freedom choose its value, e.g. size, color,
or weight of diamonds.

##### Examples

[ A product without options or variants ]

```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "pageview",
    "properties":
    {
        "catalog_discount": "2.05",
        "categories":
        [
            "Home",
            "Bundle",
            "E-Cigarette",
            "Juice"
        ],
        "currency": "USD",
        "image_url": "http://www.myfreedomsmokes.com/media/catalog/product/cache/1/image/750x/040ec09b1e35df139433887a97daa66f/k/s/ksub_platinu_bundle_1-2_copy.jpg",
        "market": "en-US",
        "price": "37.95",
        "sku": "KBOX-PLATINUM-SUBOX-BUNDLE",
        "title": "Petite Solitaire Engagement Ring",
        "uii": "KBOX-PLATINUM-SUBOX-BUNDLE",
        "url": "http://www.myfreedomsmokes.com/shop-by-brand/kangertech/ksub-platinum-bundle.html"
    }
}
```


[ A product with variants ]

```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "pageview",
    "properties":
    {
        "currency": "USD",
        "market": "en-US",
        "title": "Petite Solitaire Engagement Ring",
        "variants":
        [
            {
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

[ A product with options ]

```json
{
    "datetime": "2016-01-23T22:49:05+00:00",
    "userid": "d61b53283d19956d60a6fa",
    "event_type": "pageview",
    "properties":
    {
        "price": "118.80",
        "catalog_discount": "11.88",
        "options":
        [
            {
                "type": "Colors",
                "values": [
                    "Navy",
                    "Black",
                    "Gray"
                ]
            }
        ],
        "sku": "BEST-TRAVEL-PANTS-GRAY",
        "uii": "80192301293",
        "url": "https://www.betabrand.com/category/favorites/mens-gray-wrinkle-resistant-best-travel-pants.html",
        "title": "Best Travel Pants (Gray)"
    }
}
```

---

##### Catalog Page View

This event occurs when a user views a page that contains a list / matrix of items as small cells, possibly paged, for any specific
category or search result. For this event type, event_specific_fields is defined as follows :

```json
{
    "categories": $categories,
    "currency": $currency,
    "market": $market,
    "products": $products
}
```

__products__ is an array of product information in this catalog view, which is defined as follows :

```json
[
    {
        "catalog_discount": $catalog_discount,
        "image_url": $image_url,
        "price": $price,
        "sku": $sku,
        "title": $title,
        "uii": $uii
    }    
]
```

where

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |


---

##### Cart Page View

When a user visits a cart view page, which shows a list of items in the shopping cart, a cart log must be sent. *Note that a cart page view is different from an add to cart event log, which is sent when a user takes an action to add item to a cart.

Format of event\_specific\_fields is defined as follows :

```json
{
    "cart": {
        "currency": $currency,
        "items": $items,
        "market": $market,
        "total": $total,
        "total_discount": $total_discount
    }
}

```

__items__ is a list of products added to cart, which is defined as follows :

```json
[
    {
        "cart_discount": $cart_discount,
        "catalog_discount": $catalog_discount,
        "options": $options,
        "price": $price,
        "quantity": $quantity,
        "sku": $sku,
        "title": $title,
        "uii": $uii,
        "url": $url
    },
    ...
]
```

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| options | chosen options for an added item, which is different from options in product detail pages, where options contain _all_ possible values each option can take. | No |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |
| quantity | an integer value that contains the purchase count of this item | Yes |

---

##### Checkout

This event occurs when items are _actually_ purchased. This therefore is a log that gets sent not on a "page load" event, but
a successful completion of "purchase", e.g. finishing placing orders, returning from Stripe or Paypal checkout page after making
payments. Note that event_specific_fields for this event is almost identical to that of 'Thank You' (Checkout) page views. See
end of this section for an important note on how to handle this issue.

For this event type, event_specific_fields is defined as follows :

```json
{
    "checkout": {
        "currency": $currency,
        "items": $items,
        "market": $market,
        "order_id": $order_id,
        "shipping": $shipping,
        "tax": $tax,
        "total": $total,
        "total_discount": $total_discount
    }
}
```

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| order_id | a string that is uniquely identifies this checkout | Yes |
| shipping | shipping method. See below for more details. | No |
| tax | total tax applied. | No |

__items__ is a list of products purchased, which is defined as follows :

```json
[
    {
        "cart_discount": $cart_discount,
        "catalog_discount": $catalog_discount,
        "options": $options,
        "price": $price,
        "quantity": $quantity,
        "sku": $sku,
        "title": $title,
        "uii": $uii
    },
    ...
]
```

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| options | chosen options for an added item, which is different from options in product detail pages, where options contain _all_ possible values each option can take. | No |
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |
| quantity | an integer value that contains the purchase count of this item | Yes |

__shipping__ is defined as follows :


```json
{
    "option": $option,
    "cost": $cost
}
```

where

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| option | string containing the shipping option, e.g. Fast (3 Days), Standard (5-7 Days), etc. | Yes |
| cost | the shipping cost. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |

___Adjustments on Checkouts___
>
> Note that we do not back propagate subsequent changes in purchase information due to reasons such as refunds or canellations
> in our current scope of analytical service. Please contact us via email support@perfectprice.io for further discussion.


---

##### Search Result Page View

This event occurs when users view search results for items after issueing a search using fields like keywords or options.
This is very similar to a cart page view event except that it is associated with a search, rather than a specific set of
categories.

For this event type, event\_specific\_fields is defined as follows :

```json
{
    "currency": $currency,
    "market": $market,
    "query": $query,
    "products": $products
}
```

__query_parameter__ is defined as an [HTTP query string](https://en.wikipedia.org/wiki/Query_string) containing query parameters
used in the search. Note that this does not contain the host and domain names.

__products__ is an array of product information in this catalog view, which is defined as follows :

```json
[
    {
        "catalog_discount": $catalog_discount,
        "image_url": $image_url,
        "price": $price,
        "sku": $sku,
        "title": $title,
        "uii": $uii
    }    
]
```

where

| Field | Description | Required |
|-------------:|:-------------|:-------------|
| price | price, __after__ catalog discount if applicable. __No currency symbol. Represented as string type with digit separators, e.g. ',' or '.', etc.__ | Yes |
