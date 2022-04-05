---
tags: [Guides]
---

# Guide to Markets

This document will guide you through interacting with markets. Presently, markets are static, meaning supply and demand are irrelevant. In the future, prices will be fluid, and supplying a market with imports and handling its exports consistently will increase its demand and supply, and may unlock additional imports and/or exports. At the moment, only certain locations have markets. In the future, every location is expected to have a market with goods specific to its in-lore production and needs.

## Market Data

You can only get information on markets where you have an assistant. Similarly, you can only interact with a market when an assistant is at its location. You can start out by calling `GET: /my/markets`, and assuming your assistants are all at the starting location you should get info on the `Homestead Farm Instant Mail Order Catalogue`. As always, the `GET: /my/markets/{location-symbol}` endpoint works to select only one.

For the sake of this guide I'll assume you've already familiarized yourself with the content of the (Guide to Assistants & Travel)[], and know how to travel. So, send a caravan to Yudoa (TS-PR-YD) and unpack it once it arrives. Now call the markets endpoint again and you should see the `Yudoa General Store` added to the response. For the sake of this demonstration, we'll buy 1 average size potato, sell it back, then buy some cabbage seeds. See the example below.

**NOTE:** Market prices for PRODUCE are for the Miniature variety. Prices are multiplied by the numeric size of the produce. So for Average produce, multiply by 16.

## Market Orders

To conduct business at a market, you must call `PATCH: /my/markets/{location-symbol}/order` and send a valid Market Order as the request body. These will always have 5 fields:

- **order_type** which, for now, only supports `MARKET` orders which are executed instantly at the current market price

- **transaction_type** which will either be `BUY` or `SELL`

- **item_category** which will be one of the warehouse categories (`PRODUCE`, `SEEDS`, `GOODS`, or `TOOLS`)

- **item_name** which will be the name of the item, including size if applicable, so for example: `Potato|Average` or `Cabbage Seeds`

- **quantity** which is the number to either buy or sell

## Market Interaction Example

First, we will buy an average size potato from Yudoa. Here is the market order body:

```json
{
    "order_type": "MARKET",
    "transaction_type": "BUY",
    "item_category": "PRODUCE",
    "item_name": "Potato|Average",
    "quantity": 1
}
```

You should receive a response like

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "ledger": {
            "currencies": {
                "Coins": 904
            },
            "favor": {
                "Vince Kosuga": 50
            },
            "escrow": {}
        },
        "warehouse": {
            "uuid": "Greenitthe|Warehouse-TS-PR-YD",
            "location_symbol": "TS-PR-YD",
            "produce": {
                "Potato|Average": 1
            }
        }
    }
}
```

This shows your updated ledger and the local warehouse. Now, we can sell it back with the following body:

```json
{
    "order_type": "MARKET",
    "transaction_type": "SELL",
    "item_category": "PRODUCE",
    "item_name": "Potato|Average",
    "quantity": 1
}
```

A similar response should show your coins increased and your warehouse is now empty. Lastly, let's buy some cabbage seeds to show something other than produce. Try this:

```json
{
    "order_type": "MARKET",
    "transaction_type": "BUY",
    "item_category": "SEEDS",
    "item_name": "Cabbage Seeds",
    "quantity": 1
}
```

A similar response should be sent.