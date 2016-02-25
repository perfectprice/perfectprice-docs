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

# 3. Providing Data

Please email us the CSV or JSON file, until an alternative method is specified here.
