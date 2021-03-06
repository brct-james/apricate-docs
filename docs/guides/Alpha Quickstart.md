---
tags: [Guides]
---

# Alpha Quickstart

Welcome to the Apricate Alpha. This is an extremely early build of the game, and most things are almost guaranteed to change between versions, so I would suggest putting minimal effort into writing SDKs/wrappers for now. That being said, I hope you enjoy getting a taste of the world of Astrid.

My goals with Apricate are to provide a new take on the API game formula established by Space Traders. To that end I've designed Apricate to be more about production than logistics, and developed a setting I hope others will find interesting and engaging. Lore and story are emphasized by providing hand crafted descriptions for each location, as well as (planned) quest lines from hundreds of NPCs, and (planned) ways for players to leave permanent marks on the setting as Seasons progress.

Though the initial learning curve may at first appear steep, the game should allow you to avoid more complex concepts early on (like Optional Growth Stages) and will benefit from a (planned) in-game tutorial quest line. Feedback on the Beginner Guide is *greatly* appreciated.

## v0.6 "Assistants" Update

[Patch notes](https://apricate.io/docs/v0-6)

## Setting Summary

Apricate is set in the world of Astrid, a war-torn fantasy world in the pre-industrial feudal era. In the aftermath of recent fighting, a tragedy known as The Fracturing split apart the continents into thousands of small islands. Magical storms blanket the seas, leaving few navigable routes.

The player is an artificial life form created by an Arch Mage to serve in the war. After winning it for your masters, you grew tired of fighting and retired to a quiet farm on a backwater island called Pria. Here you plan to spend your days bringing life into the world for a change, growing plants of mundane and magical origins, crafting potions and other wares to sell in neighboring towns, and so forth.

## Alpha Goals

The primary goals of this early alpha are:

- Testing endpoints to find bugs and exploits (e.g. calling X endpoint in a particular manner adds coins for free)
- Identifying areas with poor user experience (e.g. the response structure of X endpoint doesn't give enough information OR X endpoint is unwieldy to interact with/automate)
- Finding non-bug design mistakes (e.g. X endpoint returns an incorrect http response code)
- Determining what systems are un-fun and how to improve them (e.g. X game system is tedious even when automated)
- Identify critical game design flaws (e.g. X and Y game system have conflicting interactions)
- Load testing the server with a minimal level of real-world traffic to begin identifying bottlenecks
- Crowdsourcing ideas for improving the game as a whole
- Locating gaps in the documentation
- Soliciting other feedback

## Bugs, Ideas, and Feedback

Please join the Apricate [Discord](https://discord.gg/d9ZCPcAXT6) to report bugs, suggest ideas, or provide other feedback.

You can also contribute changes to the docs directly (or open an Issue) in the [documentation repo](https://github.com/brct-james/apricate-docs).

After completing the beginner guide below you should have a bare minimum knowledge of how to play the game. I've included several "... Advanced" sections in this document with more in-depth information on each major system.

Each advanced section has a list of specific feedback I'm looking for, but don't feel restricted to this if you notice something off-list. Thanks!

## Beginner Guide

TODO: True beginner-level advice on authorization, making requests, etc.

### Registration & Getting Information

- To begin, register a user and ***save your token***: `POST: /api/users/{username}/claim`

- Setup your Bearer token authorization header in whatever client you are using

- Check the warehouse on your starting farm to see what seeds and other materials you have to start with using the secure endpoint: `GET: /api/my/warehouses/TS-PR-HF`

- - Any endpoint under the `/api/my` route is secure, and requires authentication

- Take a moment to observe the structure of the warehouse entity.

- - There are 4 categories, Tools, Goods, Seeds, and Produce.

- - Produce is special, as rather than a simple string:int key value map, Produce also has a size attribute.

- - For now, focus on your tools and seeds:

- - - You should have a couple tools, the Shears and Sickle

- - - You should also have 16 Spectral Grass Seeds, 8 Cabbage Seeds, and 4 Potato Chunks.

- Let's check the plant dictionary to see what the growth requirements are for these seeds: `GET: /api/plants/spectral_grass` (note: "Spectral Grass", "spEcTRal_gRAss" are also valid)

- - The most important part of the plant dictionary entry here is the `growth_stages` array.

- - This array is ordered

- - Each growth stage will always have an `action` that must be completed to advance to the next stage (or harvest)

- - Each growth stage, except the final stage, will have a `growth_time` in seconds. This acts as a cooldown between sending actions to the plot

- - The last stage of Spectral Grass has a `harvestable` field, which gives information on what will be harvested when completing the `action` in that stage

- - Read below in the Growth Stages Advanced section for more info on Growth Stages including Optional Stages, Early Harvests, and Multi Harvests

- We can see the first stage doesn't require any special tools (the Wait action is tool-less) or consumables, but the 2nd stage's Trim action will require Shears, and the 3rd and final harvest action is Reap which uses the Sickle. Since we have all of these tools already, let's plant this.

### Planting

- First, let's get more info on one of our farm's plots. The response we received when we reigstered our user included a list of UUIDs for our starting plots, but we can also get these with more detail using the plots endpoint: `GET: /my/plots`

- - Notice the UUID, this identifies the plot uniquely in the database. To interact with the plot, however, you will simply use the `plot-id`, formatted as '[location_symbol]!Plot-[id]' in the URI

- - The `size` field is Average, which maps to 16 slots. Larger plots hold more plants, or a smaller number of larger plants (see Plots and Plants Advanced)

- - The `plant_quantity` field is 0 because nothing is planted.

- - The `plant` field is null, because nothing is planted.

- Since Plot-0 has 16 slots and we've got 16 Spectral Grass Seeds, let's use it. To plant, hit `POST: /my/plots/TS-PR-HS!Plot-0/plant`. You'll need to specify a request body following the example below:

```json
{
    "name": "Spectral Grass Seeds",
    "quantity": 16,
    "size": "Miniature"
}
```

- - To explain this further, let's look at each field. `name` is the seed name we're planting, `quantity` is the number of seeds to use, and `size` is the intended plant size we want to grow. Take a look at Plots and Plants Advanced for more info on plant sizes, but for now just know Miniature is the smallest.

- The response contains quite a bit of data for your caching convenience.

- - `warehouse` is the local warehouse after planting

- - `plot` is the same plot info that we just saw in a previous step, though you'll notice the `plant` field now has info on the plant we planted (check out the Plots and Plants Advanced section for info on Yield)

- - - Also note `growth_complete_timestamp` is no longer 0 - it should be the current timestamp, as planting is instant

- - Finally, `next_stage` is the next Growth Stage for the plant.

### Growing

- To advance through the remaining stages after planting, we'll use the `PATCH: /my/plots/TS-OR-HS!Plot-1/interact` endpoint, using the request body to specify how we'll handle each stage. Hit is now with a body following the example below:

```json
{
    "action": "wait"
}
```

- Because this is a simple stage, we simply have to use the wait action for the plant to grow. We should receive the same type of response from the server.

- - In the `plot[plant]` response field, you may notice current_stage has advanced to 1, and that `growth_complete_timestamp` is now in the future, as this step had a 30 second cooldown (specified in the previous response's `next_stage` field).

- After the 30 second growth time has passed, we can use the information from the previous response to see we need to send the "Trim" action and will have another 30 seconds wait after.

### Harvesting

- After trimming, we can see the final step has a harvestable field. When we complete the "Reap" action the plot is automatically harvested and cleared for a new plant.

- - The `harvestable` field tells us what to expect when harvesting. There's more info in the Plots and Plants Advanced section, but for now just focus on getting more Spectral Grass Seeds back, and some Spectral Fiber goods.

- - You may realize that the seeds and goods categories map directly to the same categories your warehouse has. You may see other plants with produce as a harvestable. Who knows, in the future there may be a plant that grows tools...

- Send the Reap action once your cooldown is complete. You should see Spectral Fiber added to the Goods category of your warehouse, and that the plot has been cleared.

### Market

- Now that we've harvested some goods, we should sell them at the market so we can afford more seeds, as well as new tools and consumable goods for growing other plants. I'll also give an example of buying some Produce, as transacting it is slightly different from the other categories.

- First, let's get the market on our farm (the only one in-game for now). Hit `GET: /api/my/markets/TS-PR-HF`

- - The response has maps of imports and exports, and within these fields sub-maps for each category (just like warehouses). The string is the name of the item, the number is the price. In this early version of the game, market values are static and have unlimited quantity available. Find Spectral Fiber in the `imports[goods]` section, and see it is worth 3 coins each.

- To sell, we hit the `PATCH /my/markets/TS-PR-HF/order` endpoint with a request body that follows the example below:

```json
{
    "order_type": "MARKET",
    "transaction_type": "SELL",
    "item_category": "GOODS",
    "item_name": "Spectral Fiber",
    "quantity": 1
}
```

- - For now, the only `order_type` is MARKET orders

- - We choose the SELL `transaction_type`

- - The `item_category` is GOODS for Spectral Fiber

- - `item_name` is the name of the item we're transacting

- - Finally, `quantity` is the amount of item to transact

- The response contains the updated user Ledger (which holds the new Coins value) as well as the updated warehouse entity with the new item quantity (if 0, it is removed from the list for brevity)

- As an example of produce, this is what the body would look like to buy some large Potatoes:

```json
{
    "order_type": "MARKET",
    "transaction_type": "BUY",
    "item_category": "PRODUCE",
    "item_name": "Potato|Large",
    "quantity": 1
}
```

- - Here, `transaction_type` is BUY

- - `item_category` is PRODUCE

- - The biggest difference is, for produce, when buying or selling you *MUST* specify a Size after a pipe. So `item_name = [produce_name]|[produce_size]`.

- - It is also important to note that Size multiplies Price.

- - - If potatoes are being exported for 2 coins each, that's the Miniature price, the price for 1 Potato|Large will be 64 coins (2*32).

- - - This works to your advantage when selling, though there are some important notes in Plots and Plants Advanced that you should read before planting larger plants.

- - - See Sizes Advanced for the complete mapping of sizes to integers.

- It will be critical for you to buy new tools from the market to care for harder plants, as well as consumable goods like Water and Fertilizer. See Plots and Plants Advanced when you're ready to move on.

- That concludes the beginner guide for this build. Please continue to explore the different types of plants!

## Locations Advanced

You can get general info on the world around you, but to get details you must have an assistant at the location or it must be a farm you own.

`GET: /api/islands`

`GET: /api/islands/{islandName}`

`GET: /api/regions`

`GET: /api/regions/{regionName}`

`GET: /api/my/nearby-locations`

`GET: /api/my/locations`

`GET: /api/my/locations/{location-symbol}`

### Ports

...TODO...

### Feedback Questions

- Should `/my/locations` return a map rather than array?

## Farms Advanced

Eventually players will be able to get multiple farms, each with different bonuses and unique buildings.

`GET: /api/my/farms`
`GET: /api/my/farms/{location-symbol}`

### Feedback Questions

- Should `/my/farms` return a map rather than array?

## Markets Advanced

I hope to have adaptive, fully-featured markets with `MARKET` `STOP` `LIMIT` and `STOPLIMIT` order types in the near future.

### Feedback Questions

- Are market orders easy to use?
- How could they be improved?

## Assistants Advanced

Assistants are the 'ships' of Apricate, and will cart materials to and from locations for you. They are also how you will interact with NPCs and complete quests. Finally, they provide vision for you in the Fog of War.

### Feedback Questions

- Should `/my/assistants` return a map rather than array?

## Contracts Advanced

Contracts are given by NPCs throughout the world. To be converted to Quests

### Feedback Questions

- Should `/my/contracts` return a map rather than array?

## Sizes Advanced

Sizes are mapped between String and Integer as follows:

```yaml
Miniature: 1
Tiny: 2
Small: 4
Modest: 8
Average: 16
Large: 32
Huge: 64
# --- The sizes below are not able to be grown at this time, in the future they will be used for Trophy class produce
Gigantic: 256
Colossal: 1024
Titanic: 4096
```

### Feedback Questions

- Should there be a `GET` endpoint with a map between String and Integer Size?
- Should users be expected to review the documentation for this instead?
- Is having multiple sizes of Produce worth the extra effort to accommodate their unique data structure?
- Perhaps the middle ground: should I add a 'Slots' Integer field on plots that corresponds to the Integer Size of the plot?

## Warehouses Advanced

Warehouses are automatically created as needed for every user at every location in the game, whenever an item is first added to a location.

### Feedback Questions

- Would it be more user friendly to simply use the summary format i.e. 'Potato|Tiny' in a string:int map rather than using an object/struct for produce? This would bring it in line with the other warehouse fields, at the cost of some user-side parsing and more keys in the map.

## Plots and Plants Advanced

Actions are case-insensitive but space-sensitive

See Sizes Advanced section for map between String and Integer sizes.

### Plot and Plant Size

...TODO...

### Consumable Options

...TODO...

### Feedback Questions

- Duplicate from Sizes questions: should I add a 'Slots' Integer field on plots that corresponds to the Integer Size of the plot?
- Is there too much date in the response when interacting/planting? I could simply send the new quantity of the consumable/seed rather than the whole warehouse, for example. Would that be an improvement?
- Should plot responses include a cooldown_duration field with a seconds Integer? Note: growth_complete_timestamp is already included on the plot object. Would this be a meaningful improvement or more clutter in the response?
- The plot-id (e.g. TS-PR-HS!Plot-0) is not my favorite, but "!" is one of the few symbols that doesn't need to be encoded, otherwise "|" would be my go-to. Any thoughts on improving this selector?

## Growth Stages Advanced

Growth Stages are the building blocks of plant growth. They are one of the deeper parts of the game, so will be reviewed extensively below.

### Growth Actions Advanced

...TODO...

### Consumable Options

...TODO...

### Optional Stages

...TODO...

### Early Harvests

...TODO...

### Multi Harvests

...TODO...

### Feedback Questions

- Should the growth_stages ordered array remain an array, or will this cause problems and should be converted to a map with integer keys to guarantee order

- Should there be a tools or actions endpoint with a map from Action Name to Tool Name, or is it fun to try and figure out the association yourself

- Should ActionToSkip be removed and just use Optional or maybe Skippable bool to indicate?

## Response Codes Advanced

The server returns a numeric code with each response. These are documented here:

...TODO...

## Terminology

The game should follow a consistent terminology. Please report in discord if the documentation or an endpoint uses inconsistent or contradictory terminology.

- **Account** (planned) An account persists between Seasons, and has various meta-progression unlocks to streamline the early game for experienced players.
- **User** A user is the player's primary entity, and resets with the Season.
- **Season** (planned) Seasons are denoted by periodic game state wipes, with leaderboards and notable achievements catalogued by the server.
- TODO: more terminology

## Roadmap

Please check the [Apricate Trello board](https://trello.com/b/Qzr0OVh1/apricate) for the roadmap
