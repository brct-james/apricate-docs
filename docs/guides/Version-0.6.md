# Version 0.6

## Preface

Version 0.6 brings many new features, including traveling between locations on the home island of Pria and other islands in the Tyldian Spur region. As part of this, several new markets have been opened up on the starting, and other, islands. This allows players to identify and automate profitable trade routes, as well as selling the products of their plants.

Note: The trade/cargo system is fairly different than in space traders. Rather than using individual ships, you will use groups of assistants in Caravans. Caravans combine and buff the carrying capacity of their assistant members. You may benefit to think of assistants as 'pools' of each type rather than individual units.

## New Features

### Caravans

- Form Caravans with one or many assistants

- Caravans may bring wares of any type (goods, produce, tools, and seeds) with them up to a carrying capacity limit

- - All goods use `capacity = quantity`, EXCEPT produce, which uses `capacity = int(size) * quantity`

- Caravans travel at the speed of their slowest assistant

- Travel time is based on caravan speed and distance in the x-y plane of each island

- Caravans have a carry capacity equal to the sum of their assistants' caps multiplied by the caravan Team Factor

- Caravan Team Factor is `1 + (0.1 * (numAssistants - 1))` so caravans with more assistants get a bigger bonus to their total carrying capacity

- Caravans must be manually disbanded at their destination with the `DELETE: /my/caravans/{caravan-id}` endpoint

- When disbanded, all held wares are deposited into the local warehouse (one is created if it does not already exist)

- If you wish to continue with the same assistants and wares, simply form another caravan to the next location on your route

### Ports (Inter-Island Travel)

- Caravans may travel between islands using ports

- Each port has a 1-1 map with a port on another island (except for the unimplemented islands, obviously)

- There is no special endpoint or request needed to use a port, simply set the origin and destination of your charter request as the connected ports

- Traveling between ports has a static travel time and a port fare in coins that must be paid for each caravan, so it is most efficient to group assistants and wares up when using ports to travel

### Markets (More)

- Several markets were added, with goods distributed based on the lore of each location's description

- Not all locations currently have markets. More will be added over time, as every location is expected to have a market

- Some goods were added that are just for trading, and have nothing to do with growing plants

### Magic (Summoning Assistants / Rites & Rituals)

- Assistants must be summoned through the use of magic

- Rites can be found with `GET: /rites` and describe what you need to cast the rite (i.e. conduct the ritual)

- Different types of assistant can be summoned with different rites

- Rites have certain Distortion Tier requirements and an Arcane Flux requirement

- Rituals are conducted with `POST: /my/farms/{farm-symbol}/ritual/{rite-symbol}` and have no body, simply executing the rite as described if you have the requirements locally

- Look at the [Guide to Magic, Rites, and Rituals](https://apricate.stoplight.io/docs/apricate/ZG9jOjQ4MTg2MjQz-guide-to-magic-rites-and-rituals) for more information

### About Endpoints

- Intended to provide basic lore and setting background, as well as a reference tool for the game's systems that isn't as robust as the docs.

- `/about` and `/about/...` for `sizes` `magic` `plants` `world`

### Ratelimiting

Ratelimiting is implemented, you should have 4 calls per second with an additional 4/s burst.

## Changes to Existing Systems

- Plant definitions now have MinSize and MaxSize traits, which means you must plant them at sizes within those bounds

- Added Caravans, ArcaneFlux, DistortionTier, LatticeRejectionEnd fields to User

- Added Speed and Carrying Capacity to assistants, removed Route

- The old 'Regions' are now 'Islands' and the new 'Regions' are groups of islands - you may not notice this much as these weren't really fleshed out before

- Produce in warehouses has been reworked, and now follows the convention of other warehouse categories (see example below)

```json
{
  "produce": {
    "Potato|Large": {
      "name": "Potato",
      "size": "Large",
      "quantity": 2
    }
  }
}
```

becomes

```json
{
  "produce": {
    "Potato|Large": 2
  }
}
```

- Reworked plant harvest calculation (please share feedback in discord)

- New metrics added to `/metrics` for magic stuff

- See Markets (More) section

## Issues To Look Out For (Give Feedback in Discord)

- You have warehouses associated with your user that are empty (usually after selling, conducting a ritual, or chartering a caravan)

- Markets, rituals, or caravans don't correctly add/remove goods from the warehouse

- Ratelimit gives you more or less calls than expected

- Imbalanced plant harvest chances (too much/too little)

- Caravans with multiple assistants having unexpected inventory capacity or speed (too large or small)

- Market prices are too lucrative or too poor to be profitable after accounting for port fare (note: I didn't calculate the margins for every good, some may be very imbalanced in either direction)

- Port fare being per caravan rather than per assistant may be imbalanced in the mid-game where getting more assistants becomes easier

- Rites not working correctly (missing/extra items/flux, etc.)
