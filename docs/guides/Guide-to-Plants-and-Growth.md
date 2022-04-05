---
tags: [Guides]
---

# Guide to Plants and Growth

Plants are defined with multiple distinct Growth Stages, which require specific Growth Actions to advance through. Different plants have different needs that may require special tools or goods, and will take different amounts of time to grow. Plant definitions are presently available for all plants, but in the future you may need to research new seeds to learn about their growth stages.

## Plots

Plots have a size, which determines how many plants they can hold and of what size. If a plot has Huge size, it has 64 'slots'. So, you can plant 64 miniature, 32 tiny, 16 small, 8 modest, 4 average, 2 large, or 1 huge plant in it.

The plot holds the `growth_complete_timestamp` field which has the unix timestamp for when the next action can be taken. This is set to `Now() + GrowthTime` when you complete a Growth Action.

It also has information on the plant planted inside it, if applicable. This will include the name, size, and numeric index for the current stage of the plant, as well as the yield_modifier. Any added_yield is added to this field, which is used when calculating harvests.

## Plant Definitions

Plants have definitions that you can get from the server at the `GET: /plants` endpoint. As always, the `GET: /plants/{plant-name}` endpoint gets a specific plant. Also of interest may be the `GET: /plants/{plant-name}/stage/{stage-num}` endpoint which gets a specific growth stage, specified by its numeric index.

A plant definition has several fields:

- **name** Obviously, tells the name of the plant. The `{plant-name}` path variable uses this, and is case insensitive (and spaces may be replaced with underscores)

- **description** This field contains lore as well as potential hints at the difficulty or features of the plant

- **min_size** and **max_size** All plants can only be planted within a certain range of sizes, for example, the `Vocatus Zahra` can only be average

- **growth_stages** This field is an ordered list of growth stages (if ordered isn't preserved in your language, let me know, I can switch this over to a map with integer keys)

Certain plants will benefit from different strategies, which may greatly affect profit. For example, sometimes growing more of a smaller size plant will be more profitable as you will get more seeds. Conversely, other plants may be best to grow at larger sizes for better produce prices or more goods per harvest. Some plants need more added yield than otherse to have a reasonable shot at returning your investment.

## Growth Stages

Growth stages are defined in the plant definition, and always require the use of a tool mapped 1-1 with an action verb (except the Wait and Skip stages which are always tool-less). A growth stage may have some or all of the following:

- **name** In-lore name of the growth stage

- **description** In-lore description of the stage

- **action** An action verb that must be sent in the `PATCH: /my/plots/{plot-id}/interact` request to complete this stage

- **skippable** A boolean field, if true the "Skip" action may be sent instead of the normal stage action. See skippable stages below

- **repeatable** A boolean field, if true the growth stage will be repeated until skipped or the plot is cleared. See repeatable stages below

- **consumable_options** An unordered list of every consumable option. Unless the stage is skippable, one and only one consumable option must be selected in the interact request body to advance this stage. Some options may require more or less quantity, and may have an added_yield (see below for more info). Quantity should be multiplied by plant size and number of plants in the plot to get the actual quantity needed. Skipping a stage does not consume any items

- **added_yield** This field contains the amount of yield added by this stage. While typically zero, optional stages (where skippable is true) may have a non-zero value for this field. When consumables are included this field is always zero, as optional consumables include added_yield on the consumables themselves. Ignored when skipped. See plant yield below

- **growth_time** This field contains the time in seconds that the plant will be growing and uninteractable after completing the given stage. Skipping a stage also skips the growth time

- **harvestable** This field is present when completing the given stage will harvest the plant. See harvestable stages below

- - **goods/produce/seeds/tools** These fields are optionally present based on what the harvest gives, and map the string name of the material to the float harvest chance

- - **final_harvest** A boolean field, if true then harvesting the stage will automatically clear the plot afterwards. If the stage is skippable, skipping the harvest will not clear the plot

### Skippable Stages

If a growth stage includes `skippable: true` then you can call the `PATCH: /my/plots/{plot-id}/interact` with the "skip" action to skip the stage, e.g.:

```json
{
    "action": "skip"
}
```

Skipping a stage means consumables are not used, added_yield is not added, harvest is ignored, and growth_time does not apply. Skipping a stage can be helpful to skip an expensive set of consumables or want to bypass an early final_harvest (e.g. with the Gulb plant you can optionally harvest it early or let it continue to grow for additional yield).

You cannot skip an action with `skippable: false`

### Repeatable Stages

If a growth stage includes `repeatable: true` then interacting with the `action: ...` will not advance to the next stage. This enables several types of stages like yield boosting that can be repeated indefinitely, or orchard-like infinite harvest cycles.

If the stage has consumable options, note that you will use these on every repeat of the cycle. Similarly, added_yield will be added every cycle if applicable, as will harvestables.

If the stage also has `skippable: true` then you can escape this loop with the skip action. If not, you must clear the plot with the `PUT /my/plots/{plot-id}/clear` endpoint, removing the plant entirely.

### Harvestable Stages

IF a growth stage includes a `harvestable: {}` field then interacting with the `action: ...` will deposit the specified harvestables into the local warehouse, based on the harvest chance specified in the harvestable object. Take for example the following harvestable:

```json
{
  "harvestable": {
    "produce": {
      "Spinosa Flower": 0.1
    },
    "seeds": {
      "Spinosa Seeds": 0.1
    },
    "goods": {
      "Enchanted Water": 0.1,
      "Spinosa Meat": 0.75,
      "Water": 2
    },
    "final_harvest": true
  }
}
```

The produce, seeds, and enchanted water all have 10% harvest chance, while the spinosa meat has a 75% chance, and water has a 200% chance (meaning it will give double yield). Note this is also a final_harvest, so interacting will clear the plot after harvest. See plant yield for more info on yield and harvest chance

### Plant Yield

What you harvest from a given plant depends entirely on both the harvestable object and the yield you accumulate for it during growth. Optional stages and better quality consumables increase the yield. Yield both improves harvest chance and harvest quantity.

Yield impacts quantity less than harvest chance. The total yield is used to calculate a QuantityModifier like such: `QM = 1 + ((TotalYield - 1) / 2)`, in other words, only half of the added yield applies to harvest quantity.

A QuantityRNG value is rolled between 0.8 and 1.2 for the plot as a whole, which is used to increase or decrease the quantity harvested randomly.

A HarvestRNG value is rolled between 0 and 1 for every plant in the plot (so if plant_quantity is 5 that is 5 rng values). This is used for each category of harvestable in a different way.

#### Produce

Produce is impacted only by yield, not by size. Produce if harvested if the harvest chance from the harvestable object multiplied by the total yield is greater than the HarvestRNG for that plant. So for the Spinosa Meat example above, if the RNG value was 0.9, and total yield was 1.5, you'd get: `0.75*1.5=1.125 > 0.9` so harvest is successful. If the total yield were 1.0, however, you'd have `0.75*1.0=0.75 !> 0.9` so harvest would fail.

Harvest quantity is calculated from the QuantityModifier, the HarvestRNG, and the HarvestChance (if greater than 1). So using the Spinosa Meat example from above, with a total yield of 1.5 and QuantityRNG of 0.9, you'd have `QM = 1 + ((1.5 - 1) / 2) = 1.25` and since harvest chance <= 1, `QM * HRNG = 1.25 * 0.9 = 1.125`. This is the harvest quantity from one plant. Assuming there are more plants in the plot, sum their harvest quantities to get the total harvest quantity. For example, say this plot had a total quantity of 24.78, this value is floored (rounded down) to 24. Note that if the value is < 1, for example, a total quantity of 0.3, the value is rounded up to 1. Any time the harvest is successful, at least one item will be harvested regardless of QM or HRNG.

Produce size is always equal to that of the plant, so if you plant a Large plant in the plot, the produce will be Large.

#### Seeds

Seeds are not impacted by yield nor by size, and are always harvested successfully (though they are not guaranteed to have at least 1 quantity). In other words, if the Spinosa Seeds from the above example are harvested, the harvest quantity for the plot is `NumberOfPlants * HarvestChance * QuantityRNG`. So for a plot with 8 spinosa plants, with the 0.1 harvest chance above, and QuantityRNG of 0.8, you'd get `8 * 0.1 * 0.8 = 0.64` which is floored to 0. Alternatively, if you had 10 plants with a 1.1 QRNG you'd have `10 * 0.1 * 1.1 = 1.1` which is floored to 1 harvested seed.

#### Goods

Goods are impacted by both yield and size. Harvest chance is calculated in the same way as produce. Harvest quantity includes a SizeQuantityModifier calculated as: `QuantityModifier  * (Size * 0.5)`. You'll note this includes the QuantityModifier from above, which is based on total yield. Here you can see the impact of size is halved. If SizeQuantityModifier is < 1, it is rounded up to 1. The main take away about this formula is that smaller sizes, miniature in particular, are bad for goods. Miniature size will halve the already reduced QuantityModifier, so added yield is useless for miniature plants until 4.0 or greater.

The total quantity is the floored sum of `SQM * QRNG * HC` when HarvestChance is >= 1, and `SQM * QRNG` when it is less than 1, so for water from above with a total yield of 1.5 and a quantity rng of 1.0 on an average size plant, you'd have: `QM = 1.25; SQM = 1.25 * (16 * 0.5) = 10; SQM * QRNG * HC = 10 * 1.0 * 2.0 = 20` so you'd harvest 20 from one plant.

As another example, take enchanted water from the above with the same total yield, quantity rng, and size: `SQM * QRNG = 10 * 1.0 = 10` so you'd harvest 10 from one plant.

## Example

In this example I will plant Potatoes.

First, call `GET: /plants/potato` to get the information on the plant. Note that it can be planted in miniature through huge sizes. If you call `GET: /my/plots` you can see what plots you have available. I have a huge plot that I will use, so let's plant 4 average size potatoes.

I will call `POST: /my/plots/{plot-id}/plant` where plot-id is `TS-PR-HF!Plot-0`, my huge plot. The request body I send is:

```json
{
    "name": "Potato Chunk",
    "quantity":4,
    "size": "Average"
}
```

Note the name is the seed name, not the plant name. Figuring out what seeds go to what plants is part of the fun, or at least it is supposed to be - complain on discord if you don't like that.

The response (shown below) contains a lot of information. First the warehouse, which you'll note has had 4 Potato Chunks removed. You always use the same quantity of seeds regardless fo plant size. Next the plot object, which has a growth_complete_timestamp in the past, because planting always takes 0 time, and the plant object showing the current_stage as 0 and a yield_modifier of 1. You always start with a base yield of 1. Finally, the next stage is included, which shows us an unskippable, unrepeatable "Water' action with consumables required.

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "warehouse": {
            "uuid": "Greenitthe|Warehouse-TS-PR-HF",
            "location_symbol": "TS-PR-HF",
            "tools": {
                "Drying Rack": 1,
                "Hoe": 1,
                "Knife": 1,
                "Liquid Tap": 1,
                "Pestle and Mortar": 1,
                "Pitchfork": 1,
                "Rake": 1,
                "Scroll of Bind Evil": 1,
                "Scroll of Hyperspecific Cloud Cover": 1,
                "Shears": 1,
                "Sickle": 1,
                "Spade": 1,
                "Spirit Flute": 1,
                "Sprouting Pot": 1,
                "Water Wand": 1
            },
            "produce": {
                "Potato|Tiny": 1000
            },
            "seeds": {
                "Cabbage Seeds": 1000,
                "Convocare Bulb": 1000,
                "Grape Seeds": 1000,
                "Gulb Bulb": 1000,
                "Potato Chunk": 996,
                "Shelvis Fig Seeds": 1000,
                "Spectral Grass Seeds": 1000,
                "Spinosa Seeds": 1000,
                "Uona Spore": 1000
            },
            "goods": {
                "Dragon Fertilizer": 1000,
                "Enchanted Dragon Fertilizer": 1000,
                "Enchanted Fertilizer": 1000,
                "Enchanted Water": 1000,
                "Fertilizer": 1000,
                "Vocatus Blossom": 1000,
                "Vocatus Blossom In Perfect Bloom": 1000,
                "Wagyu Fungus Steak": 1000,
                "Water": 1000
            }
        },
        "plot": {
            "uuid": "Greenitthe|Farm-TS-PR-HF|Plot-0",
            "location_symbol": "TS-PR-HF",
            "id": 0,
            "size": "Huge",
            "plant_quantity": 4,
            "growth_complete_timestamp": 1649176671,
            "plant": {
                "name": "Potato",
                "size": "Average",
                "current_stage": 0,
                "yield_modifier": 1
            }
        },
        "next_stage": {
            "name": "Seed",
            "description": "Potatoes require water to sprout",
            "action": "Water",
            "skippable": false,
            "repeatable": false,
            "consumable_options": [
                {
                    "name": "Water",
                    "quantity": 1,
                    "added_yield": 0
                },
                {
                    "name": "Enchanted Water",
                    "quantity": 1,
                    "added_yield": 0.1
                }
            ],
            "added_yield": 0,
            "growth_time": 30
        }
    }
}
```

I'll go with the cheap option here, and just use plain water. Note that if I used enchanted water I would add 0.1 to the yield. The growth time after I submit this request will be 30 seconds. To execute growth actions, hit the interact endpoint: `PATCH: /my/plots{plot-id}/interact` with a body like:

```json
{
    "action": "Water",
    "consumable": "water"
}
```

Notice that the action and consumable selection is case insensitive. The response will be similar to before with a growth_complete_timestamp in the future and 64 less water in the warehouse. This is because consumables are multiplied by size AND plant quantity (16 * 4). Take a look at the next_stage below:

```json
"next_stage": {
	"name": "Seedling",
	"description": "Potatoes may be fertilized to increase yield",
	"action": "Fertilize",
	"skippable": true,
	"repeatable": false,
	"consumable_options": [
		{
			"name": "Fertilizer",
			"quantity": 1,
			"added_yield": 0.5
		},
		{
			"name": "Enchanted Fertilizer",
			"quantity": 1,
			"added_yield": 0.75
		},
		{
			"name": "Dragon Fertilizer",
			"quantity": 1,
			"added_yield": 1.5
		},
		{
			"name": "Enchanted Dragon Fertilizer",
			"quantity": 1,
			"added_yield": 3
		}
	],
	"added_yield": 0,
	"growth_time": 60
}
```

Notice the next stage has `skippable: true` and is the Fertilize action. If I didn't have 64 fertilizer or a pitchfork I could skip this stage. Note that each type of fertilizer has different added yield. Sometimes the cheapest options won't have any added yield or will require more quantity than more expensive options. Since I do have some fertilizer and the required tool, I'll fertilize here: `PATCH: /my/plots/{plot-id}/interact`:

```json
{
    "action": "Fertilize",
    "consumable": "Fertilizer"
}
```

This gives me the following for the next stage:

```json
"next_stage": {
    "name": "Seedling",
    "description": "Potatoes may be fertilized to increase yield",
    "action": "Fertilize",
    "skippable": true,
    "repeatable": false,
    "consumable_options": [
        {
            "name": "Fertilizer",
            "quantity": 1,
            "added_yield": 0.5
        },
        {
            "name": "Enchanted Fertilizer",
            "quantity": 1,
            "added_yield": 0.75
        },
        {
            "name": "Dragon Fertilizer",
            "quantity": 1,
            "added_yield": 1.5
        },
        {
            "name": "Enchanted Dragon Fertilizer",
            "quantity": 1,
            "added_yield": 3
        }
    ],
    "added_yield": 0,
    "growth_time": 60
}
```

This is also skippable, and the growth time is a bit longer, so let's skip this step (though notice there is a lot of added_yield for this step - in the real game you might not want to skip this step if you have the right tool). After the growth time from the previous step is finished, send the Skip action: `PATCH: /my/plots/{plot-id}/interact`:

```json
{
    "action": "skip"
}
```

Notice that the growth_complete_timestamp is not in the future because we skipped this step and its growth_time. Now take a look at the next stage:

```json
"next_stage": {
    "name": "Adult Plant - 1st Harvest",
    "description": "Potatoes must be harvested, and may be harvested again after a while",
    "action": "Dig",
    "skippable": false,
    "repeatable": false,
    "added_yield": 0,
    "growth_time": 120,
    "harvestable": {
        "produce": {
            "Potato": 1
        },
        "final_harvest": false
    }
}
```

It includes a harvestable object, and final_harvest is false. Potatoes can be harvested 3 times before they are cleared. Send the request to harvest it now: `PATCH: /my/plots/{plot-id}/interact`: 

```json
{
    "action": "dig"
}
```

The response will show a number of "Potato|Average" added to your local warehouse. My yield_modifier is 1.5 and I got 5 potatoes on this harvest. Note that even when harvesting the growth time applies. Send the same request a 2nd time after the growth_time is up and we'll harvest a second time. Now notice that final harvest has been set to true:

```json
"next_stage": {
    "name": "Adult Plant - 3rd Harvest",
    "description": "Potatoes die after their third harvest",
    "action": "Dig",
    "skippable": false,
    "repeatable": false,
    "added_yield": 0,
    "harvestable": {
        "produce": {
            "Potato": 1
        },
        "final_harvest": true
    }
}
```

You'll also note growth_time is null. This is the case for final_harvests, as there can be no proceeding steps after. Some steps have a growth_time of 0, which is explicitly set and not the same. Send the same request one last time to harvest and clear the plot. You'll get a response like:

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful",
    "data": {
        "warehouse": {
            "uuid": "Greenitthe|Warehouse-TS-PR-HF",
            "location_symbol": "TS-PR-HF",
            "tools": {
                "Drying Rack": 1,
                "Hoe": 1,
                "Knife": 1,
                "Liquid Tap": 1,
                "Pestle and Mortar": 1,
                "Pitchfork": 1,
                "Rake": 1,
                "Scroll of Bind Evil": 1,
                "Scroll of Hyperspecific Cloud Cover": 1,
                "Shears": 1,
                "Sickle": 1,
                "Spade": 1,
                "Spirit Flute": 1,
                "Sprouting Pot": 1,
                "Water Wand": 1
            },
            "produce": {
                "Potato|Average": 14,
                "Potato|Tiny": 1000
            },
            "seeds": {
                "Cabbage Seeds": 1000,
                "Convocare Bulb": 1000,
                "Grape Seeds": 1000,
                "Gulb Bulb": 1000,
                "Potato Chunk": 996,
                "Shelvis Fig Seeds": 1000,
                "Spectral Grass Seeds": 1000,
                "Spinosa Seeds": 1000,
                "Uona Spore": 1000
            },
            "goods": {
                "Dragon Fertilizer": 1000,
                "Enchanted Dragon Fertilizer": 1000,
                "Enchanted Fertilizer": 1000,
                "Enchanted Water": 1000,
                "Fertilizer": 936,
                "Vocatus Blossom": 1000,
                "Vocatus Blossom In Perfect Bloom": 1000,
                "Wagyu Fungus Steak": 1000,
                "Water": 936
            }
        },
        "plot": {
            "uuid": "Greenitthe|Farm-TS-PR-HF|Plot-0",
            "location_symbol": "TS-PR-HF",
            "id": 0,
            "size": "Huge",
            "plant_quantity": 0,
            "growth_complete_timestamp": 1649178554,
            "plant": null
        },
        "next_stage": null
    }
}
```

Note the plot has been reset like it was prior to planting, and the next_stage is null. Alternatively, at any point we could have cleared the plot (which removes the plant without refunding any materials used), by calling `PUT: /my/plots/{plot-id}/clear` with no body. This returns the reset plot object:

```json
{
    "code": 1,
    "message": "[Generic_Success] Request Successful | Successfully cleared plot: Greenitthe|Farm-TS-PR-HF|Plot-0",
    "data": {
        "uuid": "Greenitthe|Farm-TS-PR-HF|Plot-0",
        "location_symbol": "TS-PR-HF",
        "id": 0,
        "size": "Huge",
        "plant_quantity": 0,
        "growth_complete_timestamp": 1649178627,
        "plant": null
    }
}
```