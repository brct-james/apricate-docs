---
tags: [Guides]
---

# Guide to Plants and Growth

Plants are defined with multiple distinct Growth Stages, which require specific Growth Actions to advance through. Different plants have different needs that may require special tools or goods, and will take different amounts of time to grow. Plant definitions are presently available for all plants, but in the future you may need to research new seeds to learn about their growth stages.

## Plant Definitions

Plants have definitions that you can get from the server at the `GET: /plants` endpoint. As always, the `GET: /plants/{plant-name}` endpoint gets a specific plant. Also of interest may be the `GET: /plants/{plant-name}/stage/{stage-num}` endpoint which gets a specific growth stage, specified by its numeric index.

A plant definition has several fields:

- **name** Obviously, tells the name of the plant. The `{plant-name}` path variable uses this, and is case insensitive (and spaces may be replaced with underscores)

- **description** This field contains lore as well as potential hints at the difficulty or features of the plant

- **min_size** and **max_size** All plants can only be planted within a certain range of sizes, for example, the `Vocatus Zahra` can only be average

- **growth_stages** This field is an ordered list of growth stages (if ordered isn't preserved in your language, let me know, I can switch this over to a map with integer keys)

## Growth Stages

Growth stages are defined in the plant definition, and always require the use of a tool mapped 1-1 with an action verb (except the Wait and Skip stages which are always tool-less). A growth stage may have some or all of the following:

- **name** In-lore name of the growth stage

- **description** In-lore description of the stage

- **action** An action verb that must be sent in the `PATCH: /my/plots/{plot-id}/interact` request to complete this stage

- **skippable** A boolean field, if true the "Skip" action may be sent instead of the normal stage action. See skippable stages below

- **repeatable** A boolean field, if true the growth stage will be repeated until skipped or the plot is cleared. See repeatable stages below

- **consumable_options** An unordered list of every consumable option. Unless the stage is skippable, one and only one consumable option must be selected in the interact request body to advance this stage. Some options may require more or less quantity, and may have an added_yield (see below for more info). Quantity should be multiplied by plant size and number of plants in the plot to get the actual quantity needed

- **added_yield** This field contains the amount of yield added by this stage. While typically zero, optional stages (where skippable is true) may have a non-zero value for this field. When consumables are included this field is always zero, as optional consumables include added_yield on the consumables themselves. See plant yield below

- **growth_time** This field contains the time in seconds that the plant will be growing and uninteractable after completing the given stage. Skipping a stage also skips the growth time

- **harvestable** This field is present when completing the given stage will harvest the plant. See harvestable stages below

- - **goods/produce/seeds/tools** These fields are optionally present based on what the harvest gives, and map the string name of the material to the float harvest chance

- - **final_harvest** A boolean field, if true then harvesting the stage will automatically clear the plot afterwards. If the stage is skippable, skipping the harvest will not clear the plot

### Skippable Stages

### Repeatable Stages

### Harvestable Stages

### Plant Yield

## Plots

## Example