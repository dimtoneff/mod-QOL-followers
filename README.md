# mod-QOL-followers

**mod-QOL-followers** is a standalone AzerothCore WotLK module that gives real players summonable utility companions for services that are normally tied to cities and NPC hubs.

The module currently provides a **Banker**, an **Auctioneer**, and **Medivh** as a combined trainer, profession helper, talent-service NPC, dual-spec trainer, and teleporter. Followers physically accompany their owner, use configurable movement hysteresis, and can safely catch up when the player gets too far away.

No custom client patch or SQL import is required.

## Followers

| Follower | Main services |
| --- | --- |
| **Banker** | Bank, guild bank, and dismiss gossip options. Uses Pack Kodo / Pack Mule appearances by default. |
| **Auctioneer** | Opens the Auction House, inherits the owner's faction so the correct auction house is used, and offers an "Open Shop" gossip option backed by a configurable vendor entry. |
| **Medivh** | Class training, profession trainers, talent reset services, pet talent reset for hunters, dual specialization, and teleportation. |

### Auctioneer shop

The Auctioneer's "Open Shop" gossip option sells items from a standard vendor's `npc_vendor` list at their normal gold prices. The catalog is configurable and requires no SQL:

```ini
UtilityFollowers.Shop.Enable = 1
UtilityFollowers.Shop.VendorEntry = 1285
```

`Enable` toggles the gossip option; `VendorEntry` defaults to `1285` (Thurman Mullby, Stormwind general goods). Any creature entry with `npc_vendor` rows can be used.

### Banker guild bank

The Banker's "Open Guild Bank" gossip option opens the player's guild bank through a temporary guild vault GameObject:

```ini
UtilityFollowers.Banker.GuildBank.Enable = 1
UtilityFollowers.Banker.GuildBank.VaultEntry = 187293
UtilityFollowers.Banker.GuildBank.DisplayId = 1907
```

`Enable` toggles the gossip option. `VaultEntry` defaults to `187293` and must be a type-34 Guild Vault GameObject
entry whose `IconName` is not `"Point"` (validated on first use); `0` falls back to `187293`. The player's own guild
rank and tab permissions still apply. `DisplayId` overrides the temporary object's model: the default `1907` is the
small WotLK mailbox instead of the giant Guild Vault model, and `0` falls back to `1907`. An invalid non-zero id is
rejected at spawn with a warning and falls back to the configured vault template's display id. Interaction range
uses the template model; collision/LOS follows the display override. The object stays a type-34 guild vault, so
bank functionality is unchanged.

### Medivh teleportation

Medivh can teleport players to:

- Dalaran and Shattrath
- Alliance and Horde capitals
- Classic, TBC, and WotLK dungeon entrances
- Classic, TBC, and WotLK raid entrances
- Extra world destinations such as Karazhan, Caverns of Time, Gadgetzan, and Booty Bay

Teleport use, cost, minimum level, Dalaran level requirement, dungeon/raid menus, and extra destinations are configurable.

## Summoning

By default, eligible real players learn three native WotLK spells at level 10:

| Follower | Spell |
| --- | --- |
| Banker | `67368` — Bank Errand |
| Auctioneer | `54614` — Steam-Powered Auctioneer |
| Medivh | `62978` — Summon Guardian |

Casting a follower spell toggles that follower:

- not active → summon
- already active → dismiss

The module can prevent automatically learned follower spells from being placed on the action bar. Players can still drag them from the spellbook manually.

## Chat commands

Player commands are also available:

```text
.utility banker
.utility auctioneer
.utility medivh
.utility dismiss
.utility
```

The first three commands toggle their matching follower, `.utility dismiss` removes all active utility followers, and `.utility` displays command help.

## Movement behavior

The follower AI is designed to avoid constant micro-adjustments around the player.

Default movement settings:

| Setting | Default | Purpose |
| --- | ---: | --- |
| `UtilityFollowers.FollowDistance` | `3.5` | Preferred resting distance from the owner |
| `UtilityFollowers.StartFollowingDistance` | `7.0` | Distance that starts follow movement |
| `UtilityFollowers.CatchUpDistance` | `30.0` | Distance that triggers safe catch-up/teleport behavior |
| `UtilityFollowers.FollowUpdateInterval` | `300` ms | Movement update cadence |
| `UtilityFollowers.TeleportWhenStuck` | `1` | Allows safe catch-up when stuck or too far away |

The default active-follower limit is one:

```ini
UtilityFollowers.MaxActive = 1
UtilityFollowers.MaxActivePolicy = "ReplaceOldest"
```

Both the capacity and replacement policy are configurable.

## Environment controls

Followers can be enabled or disabled independently for different content:

```ini
UtilityFollowers.AllowInWorld = 1
UtilityFollowers.AllowInDungeons = 1
UtilityFollowers.AllowInRaids = 1
UtilityFollowers.AllowInBattlegrounds = 0
UtilityFollowers.AllowInArenas = 0
```

Battlegrounds and arenas are disabled by default.

## Installation

Clone the module into your AzerothCore `modules/` directory:

```bash
cd ~/azerothcore-wotlk/modules
git clone https://github.com/Ildourol/mod-QOL-followers.git
```

Then rebuild/install AzerothCore using your normal build workflow. For the mod-playerbots AzerothCore fork, for example:

```bash
cd ~/azerothcore-wotlk
./acore.sh compiler all
```

After installation, the distributed configuration is installed with the module configs. Create the live config if your setup does not already do this automatically:

```bash
cd ~/azerothcore-wotlk/env/dist/etc/modules
cp mod_qol_followers.conf.dist mod_qol_followers.conf
```

Restart `worldserver` after rebuilding or changing module configuration.

## Configuration

The full configuration template is:

```text
conf/mod_qol_followers.conf.dist
```

Notable configuration groups include:

- master enable/disable
- per-follower enable switches
- movement distances and update interval
- maximum active follower count and replacement policy
- world / dungeon / raid / PvP restrictions
- follower creature entries, display IDs, and scale
- summon spell IDs and auto-learning
- auctioneer shop enable switch and vendor entry
- banker guild bank enable switch and vault GameObject entry
- trainer costs and level restrictions
- profession-training mode
- teleport cost, combat restriction, level requirements, and destination menus

## Compatibility

The module metadata declares AzerothCore `^3.0.0` compatibility for both the standard `master` branch and the mod-playerbots `Playerbot` branch.

This module is self-contained and does not require database schema changes or custom client files.

## Project layout

```text
mod-QOL-followers/
├── conf/
│   └── mod_qol_followers.conf.dist
├── src/
│   ├── UtilityFollowerAI.cpp
│   ├── UtilityFollowerAI.h
│   ├── UtilityFollowerCommon.h
│   ├── UtilityFollowerConfig.cpp
│   ├── UtilityFollowerConfig.h
│   ├── UtilityFollowerMgr.cpp
│   ├── UtilityFollowerMgr.h
│   ├── UtilityFollowerScripts.cpp
│   └── mod_QOL_followers_loader.cpp
├── acore-module.json
├── CMakeLists.txt
└── include.sh
```

## License

The module source files currently carry **GNU AGPL v3** license headers.

---

Built for AzerothCore WotLK 3.3.5a.
