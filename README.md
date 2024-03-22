# yPlants

## How It Works

### Crops

From a technical standpoint, crops use tripwire blocks, where the generated resource pack makes all tripwires look
invisible.
These tripwires are simply the hit-box that crops use.
An armor stand is spawned on top of that tripwire that uses an item
on it's head with a *custom model data* to actually display the custom model.
When a crop grows the *custom model data* of that item is changed to reflect the new growth stage!

Crops also behave exactly like vanilla crops except they have some additional options to allow control of which biomes
they can grow in, what light levels they prefer, and the type of farmland they can be planted on. They will also break
if pushed by a piston or if the block they are planted on is pushed by a piston. Similarly, flowing water or lava also
destroys a crop.

A crop will be randomly selected to grow and will only drop its `drops` when it is fully grown.

### Vegetation

Similarly, vegetation also uses tripwires and armor stands in the exact same way. Except the *custom model data* is not
updated since vegetation doesn't 'grow'. There is an option for vegetation in configurations to `use-full-block-hitbox`.
What this means is that instead of a tripwire, a mushroom stem is used instead. This allows the vegetation to fill an
entire
block instead of the smaller hit-box that a tripwire occupies. The main issue however, is that any adjacent blocks to an
invisible mushroom stem allow a player to essentially x-ray and see underground. As such if your vegetation DOES NOT
fill an entire block, then leave this option to blank.

### Nodes

Nodes behave quite differently, as they are instead represented by a barrier block, where an armour stand is also
spawned to represent the custom model. The difference is that nodes are not instantly breakable and have a configurable
hardness value. Additionally, nodes can be configured to regenerate after a set amount of time. Nodes have two states,
a harvested state and a non-harvested state - where the model used is different for each state.

## Configuration

### Custom Crops

Custom crops are defined in `.yml` files in the `yPlants/crops` folder. You can create any number of files here, you can
create one file for each crop or have one file contain multiple crops.
Each `section` in the `.yml` file should be a unique identifier for a crop.

````
magical_crop:
  seed-item: "mmoitem:material:magical_crop_seed"
  height: 1
  allow-bone-meal: true
  bone-meal-per-stage: 1
  destroy-particle: green_wool
  destroy-sound: block_grass_break
  ...
````

In the example above, `magical_crop` is the
unique identifier. This must be different to any other specified crop.
*The plugin will warn you if it encounters any duplicates :)*

**Items**
In all the configurations when referring to an item whether it be the seed item or a plant's
drops, they all use the same format. There are two types of configurations;
**MMOItems**
Requires the type and the identifier for the item in the format `mmoitem:type:identifier`. For example;
`mmoitem:material:magical_crop_seed`
**Vanilla**
Requires simply the material used in the format `vanilla:material`. For example;
`vanilla:lapis_lazuli`

### Custom Vegetation

Custom vegetation is defined in the `yPlants/vegetation` folder. Similar to crops, any file ending with `.yml` will be
read as a vegetation type for each sub-section in that file.

Vegetation itself, will attempt to randomly generate through a configured world, matching block and biome predicates
which are configured below.

*Please note that vegetation will ONLY generate if a given world is present in the `allowed-worlds` in `config.yml`.
This also means that you cannot FORCE create a vegetation if it is not present in that configuration option.*

````
special_bush:
  block-types: [ "dandelion" ]
  weight: 5.0
  chance: 50%
  required-biomes: [ "plains" ]
  destroy-particle: grass
  destroy-sound: block_grass_break
  drops:
    - id: "mmoitem:material:magical_crop_seed"
      amount: 1-2
      chance: 20%
````

Vegetation uses a very similar configuration to crops. Items are specified the exact same way.
The `block-types` option contains a list of materials, where if that material is found as the highest block in
a world, then providing the biome matches and the chance rolls successfully - then that block will be replaced by this
custom vegetation. Vegetation does not regenerate, once it is destroyed it is gone forever.

### Custom Nodes

Nodes are functionally similar to vegetation. Where you are able to configure them to randomly spawn. The difference is
that nodes can regenerate after a specific amount of time, and have a configurable hardness meaning it takes time to
harvest them.

````
mithril_ore:
  display-name: "&6Mithril Ore"
  weight: 5.0
  chance: 50%
  required-biomes: [ "plains" ]
  adjacent-blocks: [ "stone" ]
  regeneration-time: 5min
  hardness: 60
  experience: 20
  required-tools:
    - "mmoitem:tool:mithril_pickaxe"
  destroy-particle: diamond_ore
  destroy-sound: block_stone_break
  hit-sound: block_stone_hit
  drops:
    - id: "mmoitem:material:mithril"
      amount: 1-2
      chance: 50%
  model:
    display-type: "model_engine"
    model-id: "mithril_ore"
    alt-model-id: "rock"
````

*Note the extra option `alt-model-id` which is ONLY present for nodes.*

## Models

When configuring the model for any crop, vegetation or node, you first must decide whether you are using custom model
data or integration with ModelEngine. The `display-type` argument should be either 'default ' or 'model_engine'. The
rest of the arguments are different depending on what model display type you require.

### **Default Display Type**

````
model:
  display-type: 'default'
  source-path: "magical_crop"
  destination-path: "custom/crops"
````

The default display type takes json and texture files directly placed in yPlants/resources and generates a resource pack
containing the formatted files. This option does not require any additional plugins.
The following options allow you to control where to place the various json and texture files for this crop in the
generated resource pack.

**source-path**
This is the file path where the plugin expects the `.json` file for the given plant. For this example, for the
*magical_crop*,
let's say this crop has 3 stages. The plugin then expects to find a **file** called `magical_crop_stage_1.json`,
`magical_crop_stage_2.json` *AND* `magical_crop_stage_3.json` in the `yPlants/resources` **folder**. It important that
you **MUST** name
these `.json` files in the same manner, such that it's the `<source-path>_stage_X.json`

If the option looked more like `source-path: crops/magical_crop` then the plugin would expect to find the `.json` *
*files**
at `yPlants/resources/crops/magical_crop_stage_X.json`.

When generating a resource pack, the plugin will then read that `.json` file and get the corresponding texture files.
Now
this is the **important** part.
In most cases, you should place a `.json` model's textures in the same folder the model is.
For example, if the
`magical_crop_stage_X.json` files are simply in the `yPlants/resources` folder then place the `.png` texture files in
there as well! See the example below for the `magical_crop_stage_1.json` file;

````
"textures": {
	"0": "magical_crop",
	"1": "magical_crop2",
	"particle": "magical_crop"
}
````

The plugin will read each of these textures and try to find 3 `.png` files in the `yPlants/resources` folder. For
example,
it will try to find `yPlants/resources/magical_crop.png` **AND** `yPlants/resources/magical_crop2.png`. Based on the
next
`destination-path` option, the textures new path will be automatically changed by the plugin so there is no need to
worry about that!

**HOWEVER**, lets look at another example of a `.json` file.

````
"textures": {
	"0": "blocks/magical_crop"
}
````

In the case such as this, the plugin expects to find the texture at `yPlants/resources/blocks/magical_crop.png`. If the
plugin
spits out some errors, saying "can't find texture file", then this MIGHT be the reason why. So double-check your `.json`
files
if you get this error.

Keep in mind that for **vegetation** types using a default display, they will simply only require one json file.
For **node** types, the plugin will look for a file pointing to `<source-path>_0.json` for the *un-harvested* state of
a node. Whilst the harvested state of the node will look for a file pointing to `<source-path>_1.json`. Similar to
crops,
where the numbers sort of mean a stage, however since a node only has two stages, only 0 and 1 are used.

**destination-path**
Once the plugin has read all `.json` model files and `.png` texture files, it will then attempt to form the resource
pack.
The destination-path means the destination folder in other terms, since the `.json` and `.png` files will be copied to
whatever
folder is specified here.
For example `destination-path: "custom/crops"` will cause our *magical_crop* example to have all `.json` models copied
to
the folder `assets/minecraft/models/custom/crops`. Whilst all `.png` files will be copied
to `assets/minecraft/textures/custom/crops`

### Model Engine Display Type

With Model Engine integration, configuring this section becomes a lot simpler. You simply need to specify the identifier
(aka the name of the `.bbmodel` file name to link the two models). However keep in mind crops, vegetation and nodes are
handled a bit differently.

**Crop and Vegetation Model Configuration**

````
model:
    display-type: "model_engine"
    model-id: grape
````

**model-id**
For vegetation, the vegetation will look for a model file called `<model-id>.bbmodel` exactly.
For crops, this is similar to the 'default' display type, where the stage of the crop determines its model such that
`<model-id>_stage.bbmodel`. For a crop called 'grape' with 4 stages, the plugin will try and look for files called
`grape_1.bbmodel`, `grape_2.bbmodel`, `grape_3.bbmodel`, and `grape_4.bbmodel`. As such make sure that your files are
named correctly!

**Node Model Configuration**

````
  model:
    display-type: "model_engine"
    model-id: "mithril_ore"
    alt-model-id: "rock"
````

**model-id**
The model to use for a node when it has not been harvested

**alt-model-id**
The model to use if a node has been harvested and is regenerating
