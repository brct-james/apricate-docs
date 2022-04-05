---
tags: [Guides]
---

# Guide to Caravans and Assistants

This document will guide you through sending assistants to different locations via caravans with or without wares, using ports, fog of war, and briefly cover summoning new assistants (see guide to magic, rites, and rituals for more detail)

## Assistants

Assistants are the 'ships' of Apricate, and are how you interact with the world outside your farm. Assistants are summoned and are responsible for moving items between locations as well as interacting with NPCs and markets.

You can get them with `GET: /my/assistants` or `GET:/my/assistants/{assistant-id}`. You will note the Archetype field which determines the base value for the speed and carry cap fields. The improvements field has any improvements that have been applied to the assistant (future), and the location field which holds either the location symbol or the Caravan UUID for where the assistant is located. The numeric ID field is typically used in url parameters rather than the UUID.

### Fog of War

Assistants clear the fog of war, allowing you to see markets, npcs, and other additional info at locations in the world. You cannot interact with locations that lack assistants.

### Summoning

Assistants are summoned via Rituals following Rites. Read the [Guide to Magic, Rites, and Rituals](https://apricate.stoplight.io/docs/apricate/ZG9jOjQ4MTg2MjQz-guide-to-magic-rites-and-rituals) for more info on the exact process. Generally, you'll need some materials and so arcane flux to conduct the summoning ritual. More difficult rites are specified for better assistants. To start out with, look to the Familiar, and eventually the Imp.

## Travel

Assistants travel in groups called Caravans between locations. There is a carrying capacity bonus for using more than 1 assistant in a caravan, so larger groups are generally most efficient. Caravans only exists between two endpoints, and must be unpacked and reformed after each destination. Caravans may or may not include wares that are transported with the caravan. Ports are used to travel between islands and have set travel times as well as a fare price in Coins that must be paid per caravan.

### Caravans

Caravans can be formed with one or many assistants. They may bring wares of any type (goods, produce, tools, and seeds) with them up to a carrying capacity limit.

- Note: All goods use `capacity = quantity` EXCEPT produce, which uses `capacity = int(size) * quantity`

Forming a caravan is done with `POST: /my/caravans` and caravans can be gotten with `GET: /my/caravans` or `GET: /my/caravans/{caravan-id}` (caravan IDs are listed on the user object)

Caravans travel at the speed of their slowest assistant, and travel time is based on either the speed and distance in the x-y plane of the island or based on the static travel time specified by the port connection.

Caravans have a carrying capacity equalt o the sum of their member assistants' caps multiplied by the caravan Team Factor

- The team factor is `1 + (0.1 * (NumberOfAssistants - 1))` so caravans with more assistants get a larger bonus to their total carrying capacity, and even small assistants contribute to team factor, so they retain relevance throughout the game

Caravans must be manually disbanded at their destination with `DELETE: /my/caravans/{caravan-id}`

When disbanded, all held wares are deposited into the local warehouse (one is created if it does not already exist)

If you wish to continue with the same caravan, simply reform it for the next location in your route

### Ports

Ports are predefined locations that connect two islands. The mapping is 1-1 so each port is directly tied to one port on another island

Ports have a fare in coins that must be paid per caravan, so make larger caravans to save on port fare

Travel time is independent of caravan speed, and always takes a static amount of time defined by the port

## Travel Example

For this example, we'll form a caravan taking fertilizer from the Homestead Farm on the island Pria to Port Nayanahd on the island of Veldis. This involves an over land travel step as well as travel between one set of ports.

To start, form the caravan in HF with `PATCH: /my/caravans`:

```json
{
  "origin": "TS-PR-HF",
  "destination": "TS-PR-PSH",
  "assistants": [0],
  "wares": {
    "goods": {
      "Fertilizer": 5
    }
  }
}
```

Note how the caravan charter object can have multiple assistant IDs specified in the assistants array. The wares object can have any of the categories specified like in warehouses, or none. Here we've specified the Fertilizer good, though it would be equally valid to use this request body if we wanted to send multiple assistants and no wares. Feel free to mix and match:

```json
{
  "origin": "TS-PR-HF",
  "destination": "TS-PR-PSH",
  "assistants": [0, 1, 2]
}
```

When you form that caravan with the fertilizer, you should get a reponse like:

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "uuid": "Greenitthe|Caravan-1649180782042960975",
        "id": 1649180782042960975,
        "origin": "TS-PR-HF",
        "destination": "TS-PR-PSH",
        "assistants": [
            0
        ],
        "wares": {
            "goods": {
                "Fertilizer": 5
            }
        },
        "arrival_time": 1649180966,
        "seconds_till_arrival": 184
    }
}
```

The id field is used in url parameters, rather than the UUID. You also will want to note the arrival time which is a unix timestamp and the alternative representation: seconds_till_arrival.

After arrival, unpack the caravan in PSH with `DELETE: /my/caravans/{caravan-id}` (caravan-id here is 1649180782042960975). The response should be like:

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful | Caravan unpacked at TS-PR-PSH",
    "data": {
        "assistants_released": [
            0
        ]
    }
}
```

Now you are free to form another caravan. We're the Port Shoos in Pria, which connects to Port Nayanahd in Veldis, so construct the body like:

```json
{
	"origin": "TS-PR-PSH",
    "destination": "TS-VD-PNN",
    "assistants": [0],
    "wares": {
        "goods": {
            "Fertilizer": 5
        }
    }
}
```

You'll note the travel time is exactly 30 seconds, regardless of caravan speed. You will also note that the fare was deducted from your ledger (or the request returned a validation failure if you didn't have enough coins). Once in Port Nayanahd, unpack the caravan.