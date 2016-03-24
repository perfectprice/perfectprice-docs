
menu document type
==================

items
-----

```json
{
    "type":"item",
    "id":"2b0c479d20497f768081c81a037b36fb",
    "ts":"1447946268",
    "title":"Boot-Cut | Travel Yoga Pants",
    "parent_title": "Yoga Pants",
    "uii":"W0398-BK-XS0P",
    "sku":"W0398-BK-XS0P",
    "price":"78",
    "url":"https://www.betabrand.com/womens/pants.html",
    "currency":"USD",
    "market":"en-US",
    "image url":"https://www.betabrand.com/media/catalog/product/cache/1/thumbnail/684x400/0dc2d03fe217f8c83829496872af24a0/b/o/boot_cut___travel_yoga_pants_1.jpg",
    "categories": [
        "category1",
        "category2",
        "category3"
    ]
}
```

category
--------

```json
{
    "type": "category",
    "title":"Flavored Organic Tea"
}
```

summary document type
=====================

```json
{
    "date": "2015/11/19",
    "id": "fef187a9f30865d4e252587dbb4fdf21",
    "uii": "0318-LG",
    "title": "Meat Feet Three-Pack Sampler",
    "currency": "USD",
    "market": "en-US",
    "views": {
        "count": 4,
        "detail": [
            {
                "count": 1,
                "price": "30.00",
                "detail": [
                    {
                        "count": 1,
                        "title": "Meat Feet Three-Pack Sampler"
                    }
                ]
            },
            {
                "count": 2,
                "price": "40.00",
                "detail": [
                    {
                        "count": 1,
                        "title": "Meat Feet"
                    },
                    {
                        "count": 1,
                        "title": "Meat Feet Three-Pack Sampler"
                    }
                ]
            },
            {
                "count": 1,
                "price": "0.00",
                "detail": [
                    {
                        "count": 1,
                        "title": "Meat Feet Three-Pack Sampler"
                    }
                ]
            }
        ]
    },
    "carts": {
        "revenue": 209.0,
        "cost": 80.0,
        "discount": 21.0,
        "count": 4,
        "quantity": 6,
        "detail": [
            {
                "price": "30.00",
                "count": 1,
                "quantity": 1,
                "costs": [
                    {
                        "cost": "10.00",
                        "count": 1,
                        "quantity": 1
                    }
                ],
                "discounts": [
                    {
                        "discount": "3.00",
                        "count": 1,
                        "quantity": 1
                    }
                ]
            },
            {
                "price": "40.00",
                "count": 3,
                "quantity": 5,
                "costs": [
                    {
                        "cost": "10.00",
                        "count": 2,
                        "quantity": 3
                    },
                    {
                        "cost": "20.00",
                        "count": 1,
                        "quantity": 2
                    }
                ],
                "discounts": [
                    {
                        "discount": "4.00",
                        "count": 2,
                        "quantity": 3
                    },
                    {
                        "discount": "3.00",
                        "count": 1,
                        "quantity": 2
                    }
                ]
            }
        ]
    },
    "purchases": {
        "revenue": 209.0,
        "cost": 80.0,
        "discount": 21.0,
        "count": 4,
        "quantity": 6,
        "average_order_info": {
            "sum_value": 40.0,
            "sum_quantity": 20,
            "sum_count": 10,
            "order_count": 5
        },
        "detail": [
            {
                "price": "30.00",
                "count": 1,
                "quantity": 1,
                "costs": [
                    {
                        "cost": "10.00",
                        "count": 1,
                        "quantity": 1
                    }
                ],
                "discounts": [
                    {
                        "discount": "3.00",
                        "count": 1,
                        "quantity": 1
                    }
                ]
            },
            {
                "price": "40.00",
                "count": 3,
                "quantity": 5,
                "costs": [
                    {
                        "cost": "10.00",
                        "count": 2,
                        "quantity": 3
                    },
                    {
                        "cost": "20.00",
                        "count": 1,
                        "quantity": 2
                    }
                ],
                "discounts": [
                    {
                        "discount": "4.00",
                        "count": 2,
                        "quantity": 3
                    },
                    {
                        "discount": "3.00",
                        "count": 1,
                        "quantity": 2
                    }
                ]
            }
        ]
    }
}
```

events document type
====================

```json
{
    "date": "2015/07/11",
    "id": "51a65936c574ed4f0a000015",
    "price": "19.99",
    "title": "Kraft Macaroni and Cheese 18 x 7.25 oz.",
    "type": 1
}
```

definition of event type:

| value | definition         |
|-------|--------------------|
|   0   |  price increase    |
|   1   |  price decrease    |
|   2   |  item first appear |
|   3   |  item disappear    |
|   4   |  item reappear     |
|   5   |  no change         |
