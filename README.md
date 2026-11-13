# Forever Leveling BiS API

Forever Leveling BiS gives other addons read access to its stat weights and item
scores through the global table `ForeverLevelingBiS_API`. Bag addons, leveling
guides and gear tools can show the same numbers as its tooltips, for the spec
and weights the player picked in the addon.

Current version: **2**

## Getting started

Add the addon as an optional dependency, so it loads before yours when it is
installed:

```
## OptionalDeps: ForeverLevelingBiS
```

Check that the API is there and new enough before you call it:

```lua
local FLB = ForeverLevelingBiS_API
if FLB and (FLB.version or 1) >= 2 then
    local gain, loading = FLB.GetGain(itemLink)
    if gain and gain > 0 then
        -- an upgrade for the player's main spec
    end
end
```

Every function is safe to call at any time. Before `PLAYER_LOGIN`, or for an
unknown class or spec, it returns `nil`. Every table it returns is a new copy,
so you can keep or change it.

## Functions

### IsUpgrade(link) *(v1)*

Returns `true` when the item gets the addon's green bag arrow: an upgrade for
the player's main spec. It returns `false` while the player has **Bag upgrades**
turned off in Settings, and while the item is still loading.

### OnChanged(func) *(v1)*

Calls `func()` whenever the answers may have changed: gear, level, spec, weapon
setup, stat weights or item data loading. Use it to refresh what you show.

### GetSpec() *(v2)*

Returns `spec, setup, classFile`: the main spec the player picked in the addon,
and its weapon setup. Forever has no API for the talent spec, so this is the
spec the addon scores for.

- `spec`: a spec key from `GetSpecs`, like `"fury"`
- `setup`: `"twohand"`, `"dualwield"`, `"shield"`, `"held"` or `"onehand"`
- `classFile`: like `"WARRIOR"`

### GetSpecs(classFile) *(v2)*

Returns the class's spec keys in talent-tab order:

| Class | Specs |
|---|---|
| WARRIOR | arms, fury, protection |
| PALADIN | holy, protection, retribution |
| HUNTER | beastmastery, marksmanship, survival (ranged), survivalmelee |
| ROGUE | assassination, combat, subtlety |
| PRIEST | discipline, holy, shadow |
| SHAMAN | elemental, enhancement, restoration |
| MAGE | arcane, fire, frost |
| WARLOCK | affliction, demonology, destruction |
| DRUID | balance, feral (cat), bear, restoration |

### GetWeights(classFile, spec, level, setup) *(v2)*

Returns `weights, info`.

`weights` is a table of stat key to weight, the same weights the addon scores
with. Example: `{ str = 2.00, agi = 1.37, sta = 1.17, ... }`. Keys the spec
doesn't value are missing, which means 0.

Every argument is optional. The defaults are the player's class, main spec,
level and weapon setup. For another class or spec, the default setup is that
spec's default.

If the player picked a weight profile of their own in the addon for that spec,
those are the weights you get. Otherwise you get the addon's own weights.

`info` describes the weights:

| Field | Meaning |
|---|---|
| `source` | `"model"` (the addon's leveling model), `"placeholder"`, or the name of the player's profile |
| `profile` | `true` when the weights are a player's profile |
| `fromLevel`, `toLevel` | the level range these weights cover; they change at other levels |

### GetStatKeys() *(v2)*

Returns every weight key with its label in the game's language:
`{ { key = "str", label = "Strength" }, ... }`.

### ScoreItem(link, offHand) *(v2)*

Returns the item's score for the player's main spec, the same score its
tooltip uses. The score counts the item's stats (random suffix included),
weapon DPS and speed, and racial weapon bonuses. With `offHand` true, a weapon
is scored as an off-hand weapon.

`link` is an item link, or an item ID (with an ID you get the item's base
stats, without a random suffix).

It returns `nil` for an item that isn't gear. The second result is `true` when
the score is missing because the item hasn't loaded yet. Ask again on
`GET_ITEM_INFO_RECEIVED`, or when `OnChanged` fires.

### GetGain(link) *(v2)*

Returns what the item gains the player's main spec over the gear it's compared
with, the same as the green or red number on its tooltip. By default, the
item is compared with what the player wears.

- `0` for an item the player wears
- `nil` when the player can't wear it now, it isn't gear, or it is lighter
  armor while the player has **Armor type: Class type** set
- the second result is `true` when that can't be told yet (the item, or the
  worn gear, is still loading)

The **Bag upgrades** switch doesn't affect it.

## Stat keys

Most weights are per point of the stat.

| Key | Stat |
|---|---|
| `str`, `agi`, `sta`, `int`, `spi` | primary stats |
| `atkpwr`, `rgdatkpwr`, `feratkpwr` | attack power, ranged attack power, feral attack power |
| `splpwr`, `spldmg`, `splheal` | spell power, spell damage, healing power |
| `arcsplpwr`, `firsplpwr`, `frosplpwr`, `holsplpwr`, `natsplpwr`, `shasplpwr` | spell power of one school |
| `exprtng`, `splpen`, `defrtng` | expertise, spell penetration, defense |
| `block`, `armor`, `armorbonus` | block value, base armor, bonus armor |
| `manargn`, `healthrgn` | mana and health per 5 seconds |
| `dps`, `ohDps`, `rangedDps` | per point of weapon DPS: main hand, off hand, ranged |
| `speed`, `ohSpeed`, `rangedSpeed` | per 0.1 s that the weapon is slower than the reference speed |
| `speedRef`, `ohSpeedRef`, `rangedSpeedRef` | the reference speeds, in seconds; when missing: two-handers 3.3, ranged 2.8, others 2.4 |
| `critPct`, `hitPct`, `dodgePct`, `parryPct`, `blockPct` | per 1%: Forever gear has percentage stats, not ratings |

A weapon's DPS and speed score together never goes below 0. Unless you need
the raw weights, use `ScoreItem` and `GetGain`: they apply these rules, random
suffixes and racial bonuses for you.

## Terms

You may call this API from any addon, free of charge. Credit is welcome.
Copying the addon's data or code, the weights included, still needs written
permission (see LICENSE).

## Versions

- **2**: GetSpec, GetSpecs, GetWeights, GetStatKeys, ScoreItem, GetGain
- **1**: IsUpgrade, OnChanged

Functions are never removed or changed in ways that break callers. New ones
raise the version.
