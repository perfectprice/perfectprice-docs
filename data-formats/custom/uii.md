# Unique ID

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
