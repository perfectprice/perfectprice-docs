competitor_menu document type
=============================

```json
{
    "categories": [
        "CANDY",
        "Gummy Candies"
    ],
    "competition": {
        "highest": {
            "competitors": [
                {
                    "competitor": "amazon.com: Shoe Annex",
                    "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]"
                },
                {
                    "competitor": "amazon.com: Stellar Depot",
                    "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]"
                },
                {
                    "competitor": "amazon.com: Quality Knobs and Hardware",
                    "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]"
                }
            ],
            "price": 18.07
        },
        "lowest": {
            "competitors": [
                {
                    "competitor": "amazon.com: TA National Trading",
                    "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]"
                }
            ],
            "price": 5.99
        }
    },
    "competitiveness": 2,
    "upc": "873983003210",
    "id": "53c60a860d069f4272000457",
    "number of competitors": 6,
    "price": 7.99,
    "rank": 3,
    "title": "Hi-Chew 17.64 oz. - Variety Pack",
    "type": "item",
    "ts": 1443325300,
    "date": "2015/10/20"
}
```

definition of competitiveness:

best(0),
competitive(1),
uncompetitive(2),
nocompetitors(3);


| value | definition     |
|-------|----------------|
|   0   |  best          |
|   1   |  competitive   |
|   2   |  uncompetitive |
|   3   |  nocompetitors |

competitor_summary document type
================================

```json
{
  "date": "2015/09/15",
  "title": "Hi-Chew 17.64 oz. - Variety Pack",
  "id": "53c60a860d069f4272000457",
  "upc": "873983003210",
  "prices": [
    {
      "price": 7.99,
      "url": "http://boxed.com/product/960/hi-chew-17.64-oz.-variety-pack/"
    },
    {
      "price": 8.49,
      "url": "http://boxed.com/product/960/hi-chew-17.64-oz.-variety-pack/"
    }
  ],
  "number of competitors": 6,
  "rank": 3,
  "competitiveness": 2,
  "lowest price": 5.99,
  "highest price": 18.07,
  "difference": {
    "percent": 33.3889816360601,
    "price": 2.0
  },
  "expected loss": 0,
  "competition": [
    {
      "name": "amazon.com: Shoe Annex",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 18.07,
          "url": "http://www.amazon.com/dp/B00EWT1M4S"
        },
        {
          "price": 19.07,
          "url": "http://www.amazon.com/dp/B00EWT1M4S/variant"
        }
      ],
      "ts": 1441632900,
      "date": "2015/09/07"
    },
    {
      "name": "amazon.com: SBH2000",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 13.54,
          "url": "http://www.amazon.com/dp/B0088W0DG8"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    },
    {
      "name": "amazon.com: TA National Trading",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 5.99,
          "url": "http://www.amazon.com/dp/B0088W0DG8"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    },
    {
      "name": "amazon.com: Stellar Depot",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 18.07,
          "url": "http://www.amazon.com/dp/B00EWT1M4S"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    },
    {
      "name": "boxed.com",
      "title": "Hi-Chew 17.64 oz. - Variety Pack",
      "prices": [
        {
          "price": 7.99,
          "url": "http://boxed.com/product/960/hi-chew-17.64-oz.-variety-pack/"
        },
        {
          "price": 8.49,
          "url": "http://boxed.com/product/960/hi-chew-17.64-oz.-variety-pack/"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    },
    {
      "name": "amazon.com: Quality Knobs and Hardware",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 18.07,
          "url": "http://www.amazon.com/dp/B00EWT1M4S"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    },
    {
      "name": "amazon.com: Clearance Bargains",
      "title": "Morinaga Hi -Chew Assorted Flavored 17.64 oz 100+ Individually Wrapped Fruit Chews Mango Grape Strawberry Green Apple [1 Bag]",
      "prices": [
        {
          "price": 6.74,
          "url": "http://www.amazon.com/dp/B0088W0DG8"
        }
      ],
      "ts": 1441532900,
      "date": "2015/09/06"
    }
  ]
}
```
