# Guide to Magic, Rites, and Rituals

This document will guide you through the intricacies of performing rituals. Presently, their primary function is to facilitate summoning new Assistants. In the future, they may be involved in certain crafting chains, enchanting, summoning free resources, or other beneficial functions.

## Magic

The magic system in Apricate is based on manipulation of the Lattice. It has no at-will casting, only ritual magic. On your User object you will note 3 main fields:

**Arcane Flux** which is an integer field containing the total arcane flux or 'magic power' you have. This number can rise and fall with certain spells, and managing it for your needs will be something you'll eventually want to automate.

**Distortion Tier** is the float value of Log10(flux), floored to 2 decimal places. It tells you roughly the 'tier' of magic you can cast. As your arcane flux rises, so does your distortion tier, allow greater feats of magic. Conversely, with a higher distortion tier, some lower-tier spells are no-longer castable. Think of it as gaining strength but losing the ability to modulate it to the level you used to have - you can now throw a ball 8 miles, but will struggle to toss it *only* 5 feet.

**Lattice Rejection End** is the timestamp for when you may cast another spell. After casting any spell, the Lattice will reject further manipulation from you for a set time. Higher tier magic generally incurs a longer rejection time, though some spells cause less rejection than others even with similar flux requirements. At this time, the research mages of Astrid are unsure as to why that is.

### Distortion Tiers

Here is a list of the distortion tiers, their arcane flux ranges, and a rough approximation of the power you wield at each tier. Note, even if you are ostensibly at a given tier, that does not mean casting spells of that tier is without preparation or cost. Even an archmage will spend many days or weeks preparing for a T7 spell.

- T0 (1-9 flux): Minor conveniences for mundane tasks, like opening an unlocked door
- T1 (10-99 flux): The beginnings of useful magic, like summoning a familiar or opening a mundane lock
- T2 (100-999 flux): Impressive magic for the average person, like accelerating the growth of crops, or summoning an imp or sprite
- T3 (1000-9k flux): Impressive magic for a Military Mage, like defeating a city-level threat or summoning a golem
- T4 (10k-99k flux): Impressive magic for a High Mage, like defeating an island-level threat or summoning an oni
- T5 (100k-999k flux): Impressive magic for a Master Mage, like deafeating a region-level threat or summoning a dragon
- T6 (1m-9m flux): Impressive magic for a Grandmaster Mage, like defeating a shattere-level (a shattere is a group of several regions in Apricate) threat
- T7 (10m-99m flux): Impressive magic for an Archmage, like defeating a world-level threat
- T8 (100-999m flux): Impressive magic for a Dominion Mage, like defeating a domain-level (dimension level) threat
- T9 (1b flux): Impressive magic for a Principality Mage, like defeating a god-level threat

## Rituals

Rituals are how you perform magic in Apricate. They must be performed at a farm, and often require certain materials pulled from the local warehouse and currencies pulled from your global ledger. You perform rituals with the `POST: /farms/{location-symbol}/ritual/{runic-symbol}` endpoint, specifying the Rite template by it's runic-symbol. Validation will fail if you are not of an acceptable distortion tier, don't have enough arcane flux (if the spell consumes arcane flux), or if you lack the required currencies/materials.

## Rites

Rites are the instructions for performing rituals. You can get information on the available rites with `GET: /api/rites` and `GET: /api/rites/{runic-symbol}` to select a particular rite. In the future you may need to unlock rites by discovering them in the world. I'll now walk through the response you'll see for each rite, though you may view the documentation for these endpoints to get the types and other information.

**Runic Symbols** are the identifiers for a given ritual/rite.

**Name** the in-lore name of the ritual.

**Description** tells you what the results of the ritual are, spiced up with some lore. If the results are unclear, to an extent that is likely intended - some will need to be performed to know exactly what and how many of X is gained/lost.

**Required Buildings** explain the buildings and levels required to perform the ritual. In the future farms will be able to have buildings built and upgraded, so for now this is just a placeholder field.

**Min & Max Distiortion Tier** specify lower and upper bounds on the distortion tier your user may have when performing the ritual. Weaker rites become impossible as your distortion tier rises. Managing your distortion tier is part of the game.

**Arcane Flux** is a positive or negative integer describing how much your arcane flux will increase or decrease as a result of the spell. Some spells' only function is to add or remove flux, which can help you manage your distortion tier.

**Lattice Rejection Time** is the time in seconds that further rituals will be on 'cooldown' after performing this rite.

**Currencies and Materials** describe, well, currencies and materials that are consumed in the spell. Currencies pull from your global ledger, and materials pull from your local warehouse. Materials are split out into the same categories as warehouses: goods, produce, tools, seeds. Note: tools are never consumed.

## Summoning a Familiar Example

List the rites with `GET: /api/rites` and find `FLGJ: The Accompaniment of Fylgja`. Note you need a level 1 Summoning Circle, must be between 0 and 4 distortion tier, and that the ritual consumes 50 arcane flux. The rejection time will be 30 seconds, and the materials required are 1 Vocatus Blossom that will be consumed, and 1 Spirit Flute that won't be as it is a tool.

For the purposes of this example, let's assume you've grown or purchased a Vocatus Blossom and a Spirit Flute, and have 200 water. All materials are assumed to be at the home farm.

If you get your user information with `GET: /my/user` you will notice you start with 10 Arcane Flux. We'll need to increase that to atleast 51. To do so, we'll need to cast either `FRVRNG: The Lesser Pentament of Forvrenge` 5 times, or `STRRFRVRNG: The Greater Pentament of Storre Forvrenge` once. The water cost scales linearly between these, so it's down to personal preference. The Greater Pentament will have less total rejection time, and won't put us over the T4 max distortion we need for `FLGJ` so let's go with that one.

We'll be casting this on the home farm, so call `POST: /my/farms/TS-PR-HF/ritual/STRRFRVRNG`. If all is correct, you should get a response with your new user and warehouse data, and your arcane flux should have increased by 100.

Follow this up at least 10 seconds later with `POST: /my/farms/TS-PR-HF/rital/FLGJ` and you will get a similar response. Now your arcane flux will have decreased by 50, and you should see another assistant in the list on your user object. It will be a Familiar archetype assistant, like one of the ones you start with.

## Response Examples

### GET: /rites/FLGJ

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "runic_symbol": "FLGJ",
        "name": "The Accompaniment of Fylgja",
        "description": "Summon a Familiar archetype Assistant using the irresistable Vocatus Blossom, and bind them by playing The Accompaniment of Fylgja on a spirit flute",
        "required_buildings": {
            "Summoning Circle": 1
        },
        "minimum_distortion_tier": 0,
        "maximum_distortion_tier": 4,
        "arcane_flux": -50,
        "lattice_rejection_time": 30,
        "currencies": null,
        "materials": {
            "tools": {
                "Spirit Flute": 1
            },
            "goods": {
                "Vocatus Blossom": 1
            }
        }
    }
}
```

### POST: /my/farms/TS-PR-HF/ritual/FLGJ

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "user": {
            "token": "redacted",
            "username": "Greenitthe",
            "title": "Owner",
            "ledger": {
                "currencies": {
                    "Coins": 1000
                },
                "favor": {
                    "Vince Kosuga": 50
                },
                "escrow": {}
            },
            "arcane_flux": 2150,
            "distortion_tier": 3.33,
            "user-since": 1648395318,
            "achievements": [
                "Owner",
                "Contributor",
                "Noob"
            ],
            "contracts": [
                "Greenitthe|Contract-0"
            ],
            "assistants": [
                "Greenitthe|Assistant-0",
                "Greenitthe|Assistant-1",
                "Greenitthe|Assistant-2"
            ],
            "caravans": [],
            "farms": [
                "Greenitthe|Farm-TS-PR-HF"
            ],
            "plots": [
                "Greenitthe|Farm-TS-PR-HF|Plot-2",
                "Greenitthe|Farm-TS-PR-HF|Plot-0",
                "Greenitthe|Farm-TS-PR-HF|Plot-1"
            ],
            "warehouses": [
                "Greenitthe|Warehouse-TS-PR-HF"
            ],
            "lattice_rejection_end": 1648398089
        },
        "warehouse": {
            "uuid": "Greenitthe|Warehouse-TS-PR-HF",
            "location_symbol": "TS-PR-HF",
            "tools": {
                "Spirit Flute": 1,
                "Sprouting Pot": 1,
                "Water Wand": 1
            },
            "produce": {
                "Potato|Tiny": 1000
            },
            "seeds": {
                "Convocare Bulb": 1000,
                "Grape Seeds": 1000,
                "Uona Spore": 1000
            },
            "goods": {
                "Vocatus Blossom": 999,
                "Vocatus Blossom In Perfect Bloom": 1000,
                "Wagyu Fungus Steak": 1000,
                "Water": 1000
            }
        }
    }
}
```