# System Architecture and Interconnection Documentation: Yorigami Mod Ecosystem

---

## 1. Master Heap & IPC Architecture

Inter-process communication (IPC) across all compiled CLEO threads is mediated through a single unmanaged 1024-byte memory buffer allocated on boot by `Y_Brain.cs`.

```
                  +-----------------------------------+
                  |         Y_Brain.cs (0AC8)         |
                  | Allocates 1024-byte Shared Buffer |
                  +-----------------+-----------------+
                                    |
                    0AAA Script Struct Lookup (+0x3C)
                                    |
            +-----------------------+-----------------------+
            |                                               |
+-----------v-----------+                       +-----------v-----------+
|    Joon.cs (Y_Joon)   |                       | SpawnShion.cs (Spw)   |
| Controls Player Form, |                       | Manages Companion Ped |
| Disarming & Gang Rel. |                       | Lifecycle & Spawning  |
+-----------+-----------+                       +-----------+-----------+
            |                                               |
            +-----------------------+-----------------------+
                                    |
                       Master Heap Pointer (31@)
                                    |
        +---------------------------+---------------------------+
        |                           |                           |
+-------v-------+           +-------v-------+           +-------v-------+
|  yorigami.cs  |           | Shion Systems |           | Perk Daemons  |
| Point Economy |           | Combat, Gear, |           | Bling, Fever, |
| & Save Events |           | Scav, Lore... |           | Danmaku, etc. |
+---------------+           +---------------+           +---------------+
```

### Memory Acquisition Protocol
Every secondary script executes an identical connection routine upon initialization:
1. `0AAA: 31@ = get_script_struct_named 'Y_Brain'`
2. Reads the script structure's internal variable array at offset `+0x3C`: `0A8D: 31@ = read_memory 31@ size 4 virtual_protect 0`
3. Stores the returned base pointer in local variable `31@`.
4. Reads/writes specific fields via pointer arithmetic: `BasePointer + Offset`.

### Critical Memory Map Offsets

| Offset | Constant Name | Type | Description | Primary Writers | Primary Readers |
|---|---|---|---|---|---|
| `0` | `MEM_SAVE_SLOT` | int | Active GTA SA save slot (0–8). | `yorigami.cs`, `Joon.cs` | All persistent scripts |
| `4` | `MEM_SHION_BUSY` | int (bool) | Locks companion AI during atomic actions (scavenge, dine, parachute). | `ShionScav`, `ShionChute`, `ShionBurglary`, `foodDiner` | `SpawnShion`, `ShionControl`, `ShionFollow` |
| `8` | `MEM_INTENTIONAL_DISM` | int (bool) | Flags that companion dismissal was intentional (suppresses death handling). | `Joon.cs`, `Entourage.cs` | `ShionFollow`, `SpawnShion` |
| `12` | `MEM_PLAYER_FORM` | int | Active protagonist identity: `0` = CJ, `1` = Jo'on, `2` = Shion. | `Joon.cs` | `SpawnShion`, `yorigami`, `ShionCombat`, `ShionRetaliation` |
| `16` | `MEM_QUICK_DISM_LATCH` | int (bool) | Window timer flag to catch rapid dismissals after spawn. | `SpawnShion` | `SpawnShion` |
| `20` | `MEM_PRIORITY_LOCK` | int | Hierarchical audio/subtitle mutex (`0` to `4`). | Any dialogue/action script | `ShionChat`, `ShionSandbox`, `ShionLore`, `ShionControl` |
| `24` | `MEM_LORE_MENU_SIGNAL` | int (bool) | One-frame trigger to open the Sister Talk Archive menu. | `ShionMenu` | `loreMenu` |
| `28` | `MEM_TOTAL_POINTS` | int | Lifetime Y-Points accumulated for the active save slot. | `yorigami` | `Menu`, `yorigami` |
| `32` | `MEM_AVAIL_POINTS` | int | Unallocated spendable Y-Points remaining. | `yorigami`, `Menu` | `Menu`, `yorigami` |
| `36–64` | `MEM_ALLOC_*` | int | Allocated point ranks for Jo'on perks (Bling, Smasher, Taxi, Drip, Shakedown, Midas, Fever, Danmaku). | `Menu`, `yorigami` | `Bling`, `Lifestyle`, `Shakedown`, `Fever`, `Danmaku`, `Entourage` |
| `68` | `MEM_CONTRACT_ESCROW` | int | Debt taken via Yorigami Contract, consumed by `yorigami.cs` to shield against wallet rollover exploits. | `Menu` | `yorigami` |
| `72` | `MEM_DURGASTRA_FLAG` | int (bool) | Toggle for Limitless Debt protocol. | `ShionCombat`, `ShionMenu` | `ShionCombat`, `ShionControl` |
| `76` | `MEM_DANMAKU_TOGGLE` | int (bool) | State of Money Danmaku mode (`1` = ON, `0` = OFF). | `Danmaku` | `FX_ShionMoneyDanmaku`, `FX_JoonFlames`, `FX_ShionFlames` |
| `80–108` | `MEM_ALLOC_*` | int | Allocated point ranks for Shion perks (Resonance, Scavenger, Misfortune, TrafficHex, Aviation, DebtDet, GrimPity, Misery). | `Menu`, `yorigami` | `Misfortune`, `Pursuit`, `Scavenger`, `DebtDet`, `GrimPity`, `Misery` |
| `112` | `MEM_GRIMPITY_BURST` | int (bool) | Signals `FX_GrimPity.cs` to trigger outer resurrection flame ring. | `GrimPity` | `FX_GrimPity` |
| `116` | `MEM_SHION_HANDLE` | int | Entity state of Shion: `0` = Absent/Dead, `-1` = Hitodama Spirit, `>0` = Live Ped Handle. | `SpawnShion`, `ShionFollow`, `ShionRetaliation` | All Shion subsystems |
| `120` | `MEM_SHION_STANCE` | int | Companion stance: `0` = Peaceful (unarmed), `1` = Hostile (armed). | `ShionControl`, `ShionCombat` | `ShionCombat`, `ShionGear`, `ShionControl`, `ShionCombatRedMarker` |
| `124` | `MEM_SHION_WEAPON_ID` | int | Active assigned weapon ID for Shion. | `ShionGear`, `ShionControl`, `Joon` | `ShionControl`, `ShionCombat`, `Joon` |
| `128` | `MEM_SHION_AMMO_COUNT` | int | Ammo count assigned to Shion's weapon. | `ShionGear`, `ShionControl`, `Joon` | `ShionControl`, `Joon` |
| `132` | `MEM_COMBAT_RES_MODE` | int | `0` = Wallet Drain (infinite ammo, costs $/shot), `1` = Gear Swap (physical ammo). | `ShionMenu`, `ShionTrack` | `ShionCombat`, `ShionGear`, `ShionControl` |
| `136` | `MEM_JOON_STASH_WEAPON`| int | Preserved weapon ID from Jo'on's shared slot during tag-team swaps. | `Joon`, `ShionGear` | `Joon` |
| `140` | `MEM_JOON_STASH_AMMO` | int | Preserved ammo count from Jo'on's shared slot during tag-team swaps. | `Joon`, `ShionGear` | `Joon` |
| `144–152` | `MEM_HITODAMA_X/Y/Z` | float | World coordinates of floating Hitodama spirit form. | `FX_ShionHitodama` | `SpawnShion`, `ShionFriendlyFire`, `FX_ShionMoneyDanmaku` |
| `156` | `MEM_CMBT_HITODAMA_FX` | int (bool) | Toggle for decorative combat hitodama orbs around physical Shion. | `ShionMenu`, `ShionCombat` | `FX_Hitodama` |
| `160` | `MEM_SHION_PRES_HP` | int | Preserved health for Shion across form dematerialization and rematerialization. | `SpawnShion`, `FX_ShionHitodama` | `SpawnShion`, `FX_ShionHitodama`, `ShionDebug` |
| `164` | `MEM_SHION_PRES_ARMOUR`| int | Preserved armour for Shion across form changes. | `SpawnShion`, `Joon` | `SpawnShion`, `ShionDebug` |
| `168–172` | `MEM_JOON_BASE_HP/ARM` | int | Baseline health/armour snapshot of Jo'on used to reverse damage when Hitodama absorbs hits. | `FX_ShionHitodama` | `FX_ShionHitodama` |
| `176` | `MEM_HITO_DEFLECT_TMR` | int (ms) | Active countdown for blue electric deflection flare on Hitodama absorption. | `FX_ShionHitodama` | `FX_ShionHitodama` |
| `180` | `MEM_HITO_SHIELD_BROKE`| int (bool) | Flag indicating Hitodama shield energy dropped to floor (100 HP). | `FX_ShionHitodama` | `FX_ShionHitodama` |
| `184` | `MEM_DMG_RECURSION_LCK`| int (bool) | Global mutex preventing infinite script damage loops in `0851` damage callbacks. | `Misery`, `FX_ShionHitodama`, `ShionRetaliation` | `Misery`, `FX_ShionHitodama`, `ShionRetaliation` |
| `220` | `MEM_JOON_CIGAR_OBJ` | int | Prop handle of attached cigarette object during idle smoking. | `JoonSmoke` | `Joon`, `JoonSmoke` |
| `224` | `MEM_SPOKEN_TOPICS_BIT`| int (bitmask)| Tracks which of the 9 evaluation topics were discussed in the current safehouse review session. | `ShionMem` | `ShionMem`, `ShionTrack` |
| `228` | `MEM_LAST_FED_HOURS` | int | Cumulative in-game world hours when Shion was last fed. | `ShionFood`, `foodDiner`, `scavLoot` | `ShionFood`, `ShionLore`, `ShionTrack` |
| `232–328` | Performance Ledger | int | Metrics tracked for `ShionMemories.ini` (Kills, Wrecks, Fares, Cash Given, Bribes, Time Broke, etc.). | `ShionTrack`, `ShionCombat`, `ShionFollow` | `ShionMem`, `ShionLore`, `ShionTrack` |
| `340` | `MEM_SAFEHOUSE_DOOR_ID`| int | ID (1–32) of the safehouse doorway EnEx marker currently entered. | `ShionFollow` | `ShionMenu`, `ShionLore` |
| `344` | `MEM_ACTIVE_MARK` | int | Entity handle of the current High-Roller Mark. | `Mark` | `Shakedown`, `Mark` |
| `348` | `MEM_HITO_PAIN_MUTED` | int (bool) | Tracks whether Jo'on's pain screams are suppressed during Hitodama shield absorption. | `FX_ShionHitodama` | `FX_ShionHitodama` |
| `372` | `MEM_CAR_SMASH_SIGNAL` | int (bool) | Signals that a vehicle was detonated via Jo'on's Car Smasher melee punch. | `Bling` | `ShionDeathTracker`, `ShionSandbox` |
| `376` | `MEM_FLEX_SIGNAL` | int (bool) | Signals that Jo'on performed a cash flex (`Y + J`). | `yorigami` | `ShionSandbox` |
| `380` | `MEM_SHION_DEATH_FLAG` | int (bool) | Latch recording that Shion has died in the current session. | `ShionDeathTracker`, `SpawnShion` | `SpawnShion`, `Bling` |
| `384` | `MEM_SHION_KILLER_IS_JOON`| int (bool) | Flag recording if Shion's killer was Jo'on. | `ShionDeathTracker` | `chatFollow` |
| `388` | `MEM_SHION_DEATH_WEAPON`| int | Weapon ID that dealt the lethal blow to Shion. | `ShionDeathTracker` | `chatFollow` |
| `392` | `MEM_SHION_DEATH_IN_CAR`| int (enum) | Vehicle explosion context: `0` = On foot outside via gunfire/bullet detonation, `1` = Inside vehicle, `2` = On foot outside via Car Smasher punch detonation. | `ShionDeathTracker` | `chatFollow`, `chatReunion`, `chatSpawned` |
| `396` | `MEM_SHION_STAY_PUT` | int | Stay-Put state: `0` = Follow, `1` = Ground Stay-Put (`H`), `2` = In-Vehicle Hold (`H`). | `ShionFollow`, `ShionVehicle` | `ShionFollow`, `ShionControl`, `ShionCombatRedMarker` |
| `400` | `MEM_SHION_RECRUITED` | int (bool) | Recruitment state: `0` = Ambient Loiterer, `1` = Entourage Member. | `SpawnShion`, `Joon`, `Entourage` | `ShionChat`, `shionHangoutIcon`, `ShionFood`, `JoonSmoke` |
| `404` | `MEM_SHION_HUNGRY` | int (bool) | Starvation state: `0` = Satisfied, `1` = Starving / Injured. | `ShionFood`, `foodDiner` | `foodDiner`, `ShionFood` |
| `408` | `MEM_SHION_CHUTE_OBJ` | int | Object handle of active parachute canopy (`#PARACHUTE` / `3131`). | `ShionChute`, `SpawnShion` | `SpawnShion`, `ShionChute` |
| `412` | `MEM_SHION_DM` | int | Handle of persistent Zero-Panic Decision Maker allocated once on boot. | `SpawnShion` | `ShionControl`, `Entourage`, `chatDismiss`, `SpawnShion` |

---

## 2. Priority Channel Arbitration (`MEM_PRIORITY_LOCK`)

To prevent text corruption, audio overlaps, and conflicting subtitles across concurrent scripts, all dialogue and screen-space display routines arbitrate through `MEM_PRIORITY_LOCK` (Heap Offset `20`).

```
Tier 4 (Value: 4)  ──>  Sister Talk Theater, Diner, Safehouse Evaluation, Lore Menus
                           ▲ Preempts
Tier 3 (Value: 3)  ──>  Combat Reaction, Form Swap Quips, Friendly Fire, Scavenge Action
                           ▲ Preempts
Tier 2 (Value: 2)  ──>  Ambient World Banter, Idle Conversations, Vehicle Observations
                           ▲ Preempts
Tier 1 (Value: 1)  ──>  Idle Smoking Banter, Minor Quips
                           ▲ Preempts
Tier 0 (Value: 0)  ──>  Channel Clear / Available
```

* **Preemption Rule:** A lower-tier script cannot write to subtitles (`00BC`, `0AD1`, `03E5`) or play dialogue sound (`1058`) if `MEM_PRIORITY_LOCK` is set to a higher tier. A higher-tier script immediately terminates conflicting running threads (`Sh_Zone`, `Sh_Mny`, `Sh_Veh`, etc.) using `0ABA` and overrides the lock.
* **Orphan Watchdog:** `ShionChat.txt` contains an automated watchdog polling every 250ms. If Tier 4 is claimed but no theater structs (`Sh_Thea`, `Sh_Dine`, `Sh_Mem`, `Sh_Menu`, `Sh_Reun`) exist in engine memory, it clears the lock to `0` to prevent softlocks. For Tiers 1–3, if no transient thread is identified, it enforces a 12-second timeout.

---

## 3. Core Orchestration Systems

```
                                 +-----------------------+
                                 |       Y_Brain.cs      |
                                 |  Master Heap Manager  |
                                 +-----------+-----------+
                                             |
                     +-----------------------+-----------------------+
                     |                                               |
         +-----------v-----------+                       +-----------v-----------+
         |      Joon.cs (Core)   |                       |    SpawnShion.cs      |
         +-----------+-----------+                       +-----------+-----------+
                     |                                               |
     +---------------+---------------+               +---------------+---------------+
     |               |               |               |               |               |
+----v----+     +----v----+     +----v----+     +----v----+     +----v----+     +----v----+
| Skin/Tag|     | Weapons |     | Gangs & |     | Spawning|     | Form    |     | Worker  |
| Swap    |     | Stash   |     | Audio   |     | & Blips |     | Toggles |     | Threads |
+---------+     +---------+     +---------+     +---------+     +---------+     +---------+
```

### `Joon.txt` (`Y_Joon`)
* **State Machine:** Governs `MEM_PLAYER_FORM` (`0` = CJ, `1` = Jo'on, `2` = Shion).
* **Tag-Team Swapping (`Y + L` / Cheats "JOON" & "POVERTY"):**
  1. Records current weapon slot via `0470` and a 1-indexed lookup table into local `20@`.
  2. If transforming to Shion: stashes Jo'on's current slot weapon and ammo into heap offsets `MEM_JOON_STASH_WEAPON` (`136`) and `MEM_JOON_STASH_AMMO` (`140`), purges player model with clothes rebuild, loads special actor `'shion'` into `#SPECIAL01`, gives designated weapon stored in `MEM_SHION_WEAPON_ID` (`124`), and restores target weapon slot.
  3. If transforming to Jo'on: saves live weapon stats to Shion's heap pool (`MEM_SHION_WEAPON_ID` and `MEM_SHION_AMMO_COUNT`), strips weapon, reloads special actor `'joon'` into `#SPECIAL01`, restores stashed loadout from `MEM_JOON_STASH_*`, and launches `JoonSmoke.c`.
  4. If reverting to CJ: drops special actors, restores clothes, clears muting, and executes `@Restore_CJ_Gang_Acquaintances`.
* **Gang Hostility Matrix Neutralization:** 
  * While playing as Jo'on, snapshots native gang acquaintances to dynamic RAM (`7@`), then wipes hatred and dislike (`0747: clear_acquaintance 3` and `4`) between gangs (types 7–16) and the player (`0`), making rival gangs neutral unless attacked.
  * **Immediate Boot & Load Execution:** Calls `:Snapshot_And_Neutralize_Gangs` directly within `:Silent_Rebuild_Joon_Model` so gang neutralization takes effect the exact frame a save is loaded or boot startup initializes (`StartupPlayerCharacter == 1`), eliminating the need to toggle cheats back and forth.
  * **Active Hostility Sensor:** Employs an engine-state sensor in `Joon_MainLoop` checking if Ballas (`pedtype 7`) or San Fierro Rifa (`pedtype 10`) have Hate set to the player (`07E8`). If the engine reasserts storyline hostilities during save loads or script resets while Jo'on is active (`0@ == 1`), it automatically re-neutralizes all gangs while keeping the baseline snapshot in `7@` untouched.
  * **Companion Pedtype 23 (Special) Isolation:** Shion operates as `pedtype 23` (`SPECIAL`), fully isolating her entity from the native street gang range (`pedtypes 7–16`). This restores Pedtype 16 back to standard Biker gang neutrality rules in `:Snapshot_And_Neutralize_Gangs`, while Shion herself is pacified via entity-level acquaintance wipes (`077C: clear_actor acquaintance to_all_pedtypes`) and the persistent Zero-Panic Decision Maker (`MEM_SHION_DM`), ensuring rival gangs never target her unprovoked. Restores exact state upon revert to CJ or before writing to a save file.
* **Girlfriend Suppression:** Sets `$391 bit 11` (native date/call suppression) and traverses radar trace memory `0xBA86F0` (175 entries), clearing display bits `0xFC` on sprite ID `21` (Pink Heart) to hide girlfriend blips.
* **Prostitute Solicitation & Subtitle Suppression (`SuppressProstitutes`):**
  * Governed by `SuppressProstitutes = 1` under `[Companion]` in `Shion.ini` and tracked via persistent flag `18@`.
  * **Dynamic GXT Label Blanking (`0ADF` / `0AE0`):** While active in Jo'on or Shion form, overrides native prostitution text keys (`PROS_04` through `PROS_07`, and `PROS_09`) with empty space labels (`0ADF`). This intercepts `CMessages` at the engine text lookup level (CRC32 `0x9A09A4D5`), rendering it physically impossible for "You want a good time, honey?" to draw on screen. Reverting to CJ removes these custom labels (`0AE0`), restoring the native dialogue lines.
  * **Active Task Interception (`@Suppress_Nearby_Prostitutes`):** When driving outside missions, runs a 200ms proximity sweep (`0AE1`, radius 20.0m) monitoring peds:
    * Detects solicitation subtasks `TASK_COMPLEX_PROSTITUTE_SOLICIT` (`1309`), `TASK_COMPLEX_GOTO_VEHICLE_AND_LEAN` (`1216`), and `TASK_COMPLEX_LEAN_ON_VEHICLE` (`1217`) regardless of ped type.
    * Detects `PedType 22` (`CIVFEMALE_PROSTITUTE`) pedestrians initiating car boarding routines (`700`, `800`, `802`, `807`).
    * Instantly invokes `0A43: get_rid_of_player_prostitute`, clears queued small HUD messages (`03EB`), wipes on-foot tasks (`0687: clear_actor task`, `0E96`, `0E97`), and commands ambient sidewalk wandering (`05DE`).
  * **Seat 0 Boarding Failsafe:** Inspects vehicle passenger seat 0 each cycle; if an unauthorized `PedType 22` ped manages to board, immediately clears her tasks, flushes messages (`03EB`), and forces a vehicle exit (`0633: AS_actor exit_car`) while strictly ignoring companion Shion (`1@`).

### `SpawnShion.txt` (`SpwShion`)
* **Companion Lifecycle Engine:** Manages physical companion entity `10@` and Hitodama state.
* **Ambient World Encounter Engine:** When Shion is unrecruited (`MEM_SHION_RECRUITED == 0`), she does not follow the player. The script scans a 50m radius around 3 preset coordinates:
  * Los Santos: Idlewood (behind Well Stacked Pizza) `[2112.7, -1822.6, 13.5]`
  * San Fierro: Doherty (beside Fire Station) `[-2059.3, 98.4, 28.3]`
  * Las Venturas: Old Strip Alley `[2636.4, 1654.8, 11.0]`
  Spawns ped model `299` (`#SPECIAL10`), assigns a custom non-panicking Decision Maker (`65543`), loads animation library `"LOWRIDER"`, and loops `"M_SMKLEAN_LOOP"`. Approaching and pressing `Y` triggers `chatReunion.c` and recruits her into the player's group (`MEM_SHION_RECRUITED = 1`).
* **Hitodama Form Morphing (`Y + H`):**
  * *Solid -> Spirit:* Stores current HP (`MEM_SHION_PRES_HP`) and armour (`MEM_SHION_PRES_ARMOUR`), snapshots weapon/ammo (if in Gear Swap mode), removes Shion from group, writes `-1` to `MEM_SHION_HANDLE`, destroys the ped, and creates `FX_ShionHitodama.c`.
  * *Spirit -> Solid:* Checks vehicle occupancy (identifies free passenger seat 0, 1, or 2), recreates ped model `299` inside vehicle or on foot, writes new ped handle to `MEM_SHION_HANDLE`, restores preserved HP/armour, and reboots physical worker threads.
* **Auto-Dematerialization:** Automatically forces Shion into Hitodama spirit form if the player boards any single-seater vehicle (Rhino, Hydra, Hunter, Rustler, Cropduster, Stuntplane, Sparrow, Seasparrow, Kart, Bandito, Quad, Vortex, Forklift, Mower, Sweeper, Dozer, Dumper, Baggage, Tug).

### `yorigami.txt` (`YPrestige`)
* **Economy Engine:** Monitors player money `010B`. Compares live cash to previous baseline.
* **Universal Interior & Minigame Escrow:** Prevents gambling, clothes shopping, or betting exploits from generating false spending:
  * Detects interior ID > 0, `$ONMISSION == 1`, or active `LOWRIDE` minigame.
  * Freezes the baseline wallet snapshot into `22@`.
  * Upon exiting to the outdoors, calculates net money delta (`Starting - Ending`). If positive, transfers the difference to the spent accumulator `3@`.
* **Point Conversion:** When accumulated spending `3@ >= CashPerPoint` (default $10,000), increments `MEM_TOTAL_POINTS` (`28`) and `MEM_AVAIL_POINTS` (`32`), writing to `yorigami.ini`.
* **Didier Sachs Cash Flex (`Y + J`):** Consumes `$CashPerPoint` from wallet, increments Y-Points directly by +1, triggers particle micro-thread `FX_FLEX.c`, and writes `MEM_FLEX_SIGNAL = 1` for sandbox reactions.

---

## 4. Companion Subsystems

```
                                  +-------------------+
                                  |   SpawnShion.cs   |
                                  +---------+---------+
                                            |
         +----------------------------------+----------------------------------+
         |                                  |                                  |
+--------v--------+                +--------v--------+                +--------v--------+
|   Navigation    |                | Combat & Stance |                | Social & Dialog |
| ShionFollow     |                | ShionControl    |                | ShionChat       |
| ShionVehicle    |                | ShionCombat     |                | ShionLore       |
| ShionChute      |                | ShionGear       |                | ShionMenu       |
| ShionBurglary   |                | RedMarker / Dth |                | ShionSandbox    |
+-----------------+                +-----------------+                +-----------------+
```

### Navigation & Space Management
* **`ShionFollow.txt` (`Sh_Flw`):** 
  * Controls physical companion following behavior, distance-keeping, and auto-catchup.
  * Implements `H` on foot to trigger "Stay Put" mode (`MEM_SHION_STAY_PUT = 1`, removes from group, applies `0638: AS_actor stay_put 1`). Pressing `G` cancels Stay Put globally and reasserts group follow.
  * If Shion was locked waiting inside a vehicle (`MEM_SHION_STAY_PUT == 2`), pressing `G` on foot unlocks the doors, commands an orderly car exit (`0633: AS_actor exit_car`), and streams `chatFollow.c 4` (Mode 4: Exit Vehicle Recall).
  * **Zero-Collision Recall Architecture**: Safely caches previous stay-put states into `29@` (preventing register collisions with the active interior tracker `8@` and eliminating false 2.15s doorway mesh freezes).
  * **Strict Task Execution Ordering**: Clears stay-put and old tasks *prior* to re-registering group membership (`0631`) and follow commands (`07B3`), ensuring native group follow intelligence is never wiped.
  * Single tap `G` triggers an on-foot sprint to Jo'on (`stop_within_radius 1.5m`) and streams Mode 0 recall dialogue; double-tap `G` or long-range recall (> 15m) streams Mode 8 warp dialogue and triggers an instant spatial warp (`FX_Warp.c`).
  * Auto-warps Shion if distance > 45m; initiates despawn if distance > 130m.
  * Detects interior changes via active interior `28@`. Enforces a 2150ms mesh loading delay before reading player coordinates, matching them against 32 known Safehouse EnEx doorways (`MEM_SAFEHOUSE_DOOR_ID`), and placing Shion inside.
* **`ShionVehicle.txt` (`Sh_Sync`):**
  * Controls vehicle passenger seat synchronization.
  * If Shion is outside while player drives, pressing `G` checks vehicle passenger capacity (`01EA`) and warps her into seat 0 (front passenger), 1 (rear left), or 2 (rear right).
  * If Shion is already inside, pressing `G` cycles her through open seats (0 -> 1 -> 2 -> 0) with Hitodama dissolve FX.
  * Pressing `H` in-car sets `MEM_SHION_STAY_PUT = 2`, locks the car door via `039E`, and removes her from the group so she remains seated when the player exits.
  * Monitors vehicle destruction/fire: forces immediate bailout or airborne parachute ejection if car health < 250, burning, or sinking.
* **`ShionCombatRedMarker.txt` (`Sh_RedM`):**
  * Reads the game engine's radar trace pool at address `0xBA86F0` (up to 175 blips).
  * Filters for active red blips (colors 0, 5, 7; blip types 1 = Car, 2 = Ped; non-friendly).
  * Directs Shion's AI tasks (`0672: attack_car` or `05E2: kill_actor`) toward the nearest marked target within 100m. If seated in a car driven by Jo'on, executes native drive-bys (`0713`).
* **`ShionChute.txt` (`Sh_Chut`):** 
  * Triggers when Shion is in freefall outdoors > 15m altitude, strictly filtering against player/companion water immersion (`84AD`, `8965`) and verifying elevation is > 15m above the local water surface (`092E: get_water_height_at`) to prevent misfires while swimming or treading water above deep seabeds.
  * Sets `MEM_SHION_BUSY = 1`, removes from group, grants fall damage immunity (`02AB`), and sets `0575: pinned_position 1` to disable falling flail tasks.
  * Phase 1: High-speed freefall down to 65m altitude.
  * Phase 2: Deducts $800 emergency parachute fee from player wallet (or takes debt), creates attached `#PARACHUTE` object, registers handle in `MEM_SHION_CHUTE_OBJ` (`408`), and glides downward at ~9 m/s.
  * Phase 3: Touchdown landing, releases and destroys canopy object, resets `MEM_SHION_CHUTE_OBJ = 0`, re-adds to group, clears busy flag, and sets `MEM_SKYDIVE_LANDED = 1`.
* **`ShionBurglary.txt` (`Sh_Burg`):**
  * Active during Burglary missions (`$ONMISSION_BURGLARY == 1`).
  * When Jo'on is carrying loot inside a residence, holding `TAB` near Shion initiates a hand-off.
  * Jo'on's carrying task and BURGJB memory (`0xA48AD8`, `0xA48A24`) are cleared. Shion dissolves with Hitodama FX (`MEM_SHION_BUSY = 1`).
  * Runs a 3-second background timer representing a run to the Boxville van, increments van loot memory (`0xA48A20`), plays chime `1058`, and rematerializes Shion beside Jo'on.

### Combat, Stance & Weapons
* **`ShionControl.txt` (`Sh_Ctrl`):**
  * Listens for `Y + B`: toggles `MEM_SHION_STANCE` (`0` = Peaceful, `1` = Hostile).
  * *Dismissal Bailout Watchdog:* Polls `MEM_SHION_RECRUITED` every 50ms and terminates immediately upon dismissal (`0A93`), preventing orphaned companion daemons from persisting.
  * *Peaceful:* Strips all weapons (`048F`), sets target priority to 0, sets decision maker to the persistent Zero-Panic template (`MEM_SHION_DM`), sets acquaintances to "Like" for cops, civilians, and gangs, sets Acquaintance 0 (Respect) to Player, and engages anti-cower and movement watchdogs.
  * *Hostile:* Checks resource mode. In Wallet Drain, equips designated weapon from `MEM_SHION_WEAPON_ID` with 9999 ammo cushion. In Gear Swap, equips exact assigned weapon and ammo from `MEM_SHION_AMMO_COUNT`. Restores aggressive decision maker `65543` and hostility towards rival ped types.
  * **Universal Out-of-Combat Movement Watchdog**: Continuously monitors Shion on foot across *both* Peaceful and Hostile stances. If Shion is out of combat, not ordered to Stay Put (`MEM_SHION_STAY_PUT == 0`), stopped (`02A0`), and > 3.0m away from Jo'on, automatically reasserts group follow commands (`07B3`) and issues a catch-up sprint (`05D9`), preventing armed companion braindeath and stuck states.
* **`ShionCombat.txt` (`Sh_Cmbt`):**
  * **Wallet Drain Mode (`MEM_COMBAT_RES_MODE == 0`):** Monitors live weapon ammo. Each time ammo decreases by `N` rounds, looks up the shot cost from an internal weapon cost table (e.g. 9mm = $7, Desert Eagle = $80, Shotgun = $40, Sniper = $500, Minigun = $7, Rockets = $300) and deducts `N * Cost` directly from player cash.
  * **Bankruptcy Disarm:** If wallet drops <= $0 and cheat "DURGASTRA" is not active, disarms Shion immediately, sets stance to Peaceful (`MEM_SHION_STANCE = 0`), and displays `"CARD DECLINED"`.
  * **Innate Misfortune Aura:** Scans hostiles within 25m every 1s. Rolls a 20% bad luck check: forces victim to suffer gun jams (cower animation) or slip and knockdown animations (`KO_SKID_FRONT/BACK`, `KO_SPIN_L/R`), and spawns an attached `FX_HexFlame.c`.
    * **Cop Retaliation Exemption:** Cops (`PedType 6`) are strictly exempt from both peacetime hostile targeting (`wanted_level == 0`) and post-recovery retaliation assignments (`18@ = 0`), preventing scripted `05E2: kill_actor` orders from re-aggroing pacified officers.
    * **Task Pack Register Isolation:** Uses dedicated scratch register `20@` for AS pack construction (`0615`), fully decoupling sequence creation from the vehicle iterators (`12@`) used in Traffic Hex and Aviation Calamity.
  * **Innate Traffic Hex:** When wanted level > 1, scans police vehicles within 45m every 5s. Rolls 20% chance to force stall (`0477: action 1 time 3500`) and deflates front tires (`04FE`).
  * **Innate Aviation Calamity:** When wanted level > 2, scans helicopters and jets within 300m every 5s. Rolls 20% chance to detonate pursuit aircraft (`020B: explode_car`).
  * **Voodoo Damage Reflection (`OnShionDamage`):** Intercepts damage events on Shion; reflects incoming intensity back to the attacking ped or vehicle driver.
  * **Kill Tracker:** Continuously tallies Shion's confirmed kills (`MEM_SHION_KILLS`) and Jo'on's kills with Shion (`MEM_JOON_KILLS_W_SHION`).
* **`ShionGear.txt` (`Sh_Gear`):**
  * *Dismissal Dormancy:* Polls `MEM_SHION_RECRUITED` and `MEM_SHION_HANDLE`. If Shion is unrecruited or dismissed (`MEM_SHION_RECRUITED == 0`), enters a low-frequency 250ms sleep state that halts real-time ammo tracking and suppresses `[Y + R]` interactions on dismissed wanderers.
  * **Mode Transition Tracking (`13@` / `LastCombatResourceMode`):** Monitors transitions between Wallet Drain (`0`) and Gear Swap (`1`):
    * *Wallet Drain $\rightarrow$ Gear Swap ($0 \rightarrow 1$):* Detects the transition frame and immediately reinforces Shion's untouched physical reserve from `MEM_SHION_AMMO_COUNT` onto her ped actor (`017B: set_actor weapon ammo_to`), stripping the synthetic 9,999 cushion. If the stored physical reserve is empty, disarms her and reverts her stance to Peaceful.
    * *Gear Swap $\rightarrow$ Wallet Drain ($1 \rightarrow 0$):* Equips the actor with the 9,999 round cushion for wallet-drain firing while leaving `MEM_SHION_AMMO_COUNT` strictly isolated in RAM.
  * **Gear Swap Mode (`MEM_COMBAT_RES_MODE == 1`):** Listens for `Y + R` on foot within 4m.
  * If player is unarmed: strips Shion's weapon, pools remaining ammo with any sibling ammo Jo'on holds, restores weapon to Jo'on, and clears Shion's weapon in RAM.
  * If player is holding a firearm (IDs 22–38): removes gun from Jo'on, transfers it to Shion with exact current ammo, updates `MEM_SHION_WEAPON_ID` and `MEM_SHION_AMMO_COUNT`, and switches her stance to Hostile.
  * Continuously checks Shion's live ammo. If she expends all rounds, clears her inventory, sets `MEM_SHION_STANCE = 0`, and plays the out-of-ammo vocal dialogue.
* **`ShionDeathTracker.txt` (`Sh_Dth`):**
  * Attaches to `0EDE: set_script_event_char_damage` specifically targeting Shion's handle and runs an active watchdog loop operating continuously across player state transitions.
  * Resolves damager entity: identifies whether damage originated from Jo'on directly, Jo'on's driven vehicle, an explosive weapon held by Jo'on, a Car Smasher punch detonation (`MEM_CAR_SMASH_SIGNAL`), or environmental drowning.
  * Continuously tracks Shion's vehicle occupancy and Jo'on's driver status while alive, cleanly distinguishing between safe on-foot disembarkation and catastrophic in-vehicle detonation ejections. Uses integer health checks (`002B: 250 >= health`) to accurately detect true burning wrecks without misclassifying healthy 1000 HP vehicles.
  * Clean On-Foot Unlatch: Stepping out of an undamaged vehicle immediately clears the tracked car handle (`17@ = 0`), vehicle occupancy state (`18@ = 0`), and driver flag (`27@ = 0`), flushing any stale explosion pre-arm states back to `-1`. Streamed-out or naturally despawned parked vehicles are safely ignored rather than falsely assumed to have exploded.
  * Confirmed Weapon Shielding: If `OnCharDamageTracker` records a confirmed firearm, bullet, or melee strike (`12@ != -1` and `12@ != 51`), the death loop skips in-vehicle explosion fallbacks entirely (`22@ = 1`), guaranteeing that on-foot gunfight casualties are never overwritten by car explosion telemetry.
  * Comprehensive vehicular fatality classification:
    * **Riding together and exploding**: `MEM_SHION_DEATH_IN_CAR = 1`, `MEM_SHION_KILLER_IS_JOON = 1` $\rightarrow$ Jo'on driving smoking wreck / unbuckle banter.
    * **Waiting inside car and blown up**: `MEM_SHION_DEATH_IN_CAR = 1`, `MEM_SHION_KILLER_IS_JOON = 0` $\rightarrow$ Trapped in car / ears ringing banter.
    * **On foot outside, car punched via Car Smasher**: `MEM_SHION_DEATH_IN_CAR = 2`, `MEM_SHION_KILLER_IS_JOON = 1` $\rightarrow$ Clocked sedan / fireball punch banter.
    * **On foot outside, car detonated via gunfire/minigun**: `MEM_SHION_DEATH_IN_CAR = 0`, `MEM_SHION_KILLER_IS_JOON = 1` $\rightarrow$ Trigger-happy shooting / shrapnel banter.
    * **On foot outside, world/enemy car explosion**: `MEM_SHION_DEATH_IN_CAR = 0`, `MEM_SHION_KILLER_IS_JOON = 0` $\rightarrow$ Sidewalk heat radiating banter.
    * **On foot outside, run over by Jo'on**: Weapon ID 50, `MEM_SHION_KILLER_IS_JOON = 1` $\rightarrow$ Bones snapping like twigs banter.
  * Writes lethal telemetry to `MEM_SHION_DEATH_FLAG`, `MEM_SHION_KILLER_IS_JOON`, `MEM_SHION_DEATH_WEAPON`, and `MEM_SHION_DEATH_IN_CAR`.
* **`ShionRetaliation.txt` (`Sh_Retal`):**
  * Anti-abuse system active exclusively when player is CJ (`MEM_PLAYER_FORM == 0`).
  * **On-Foot Retaliation**: If CJ punches or shoots Shion on foot, intercepts the damage event via `0EDE`, engages recursion lock `MEM_DMG_RECURSION_LCK`, deals 35 HP recoil damage back to CJ, forces him into a hit-stagger animation (`"hit_front"`), renders sapphire burst coronas, displays priority subtitle (`Y_CJRET`), dissolves Shion via `FX_Warp.c`, and terminates companion daemons.
  * **Vehicular Run-Over Retaliation**: Intercepts vehicular damage when CJ is driving (`00DF`), evaluating vehicle pointers, car damage status (`051B`), and impact damage types (`49` *Rammed*, `50` *Run Over*). If CJ attempts to ram or run Shion over, Shion instantly dissolves with hitodama warp FX (`FX_Warp.c`), CJ's vehicle is immediately set ablaze (`health_to 240` and `0325` fire attachment), and companion daemons are cleanly flushed.

### Sustenance, Scavenging & Dining
* **`ShionFood.txt` (`Sh_Food`):**
  * Calculates elapsed in-game hours: `(Days * 24 + CurrentHour) - MEM_LAST_FED_HOURS`.
  * If elapsed >= 48 hours or Shion HP < 950, sets `MEM_SHION_HUNGRY = 1`.
  * Monitors drive-thrus and restaurant counters. When parked near outdoor fast food locations or standing near indoor counters, presents `[TAB]` prompt. Pressing `TAB` claims Tier 4 priority and launches `foodDiner.c`.
* **`foodDiner.txt` (`Sh_Dine`):**
  * Creates native interactive menu panel with bill options: $1, $2, $5, $10, $20, $50, $100.
  * Freezes player movement, deducts cash, increments `MEM_TIMES_TREATED` and `MEM_TOTAL_CASH_GIVEN`.
  * Plays contextual pre-meal dialogue (varying based on hungry vs wounded). Fades screen to black (750ms), restores Shion's health to 1000 and armour to 500, resets player hunger timer (if >= $20 spent), fades in, plays post-meal dialogue, updates `MEM_LAST_FED_HOURS`, and clears `MEM_SHION_HUNGRY = 0`.
* **`ShionScav.txt` (`Sh_Scav`) & `scavLoot.txt` (`Sh_Loot`):**
  * Activated by pressing `Y + Z` or typing cheat "SCAV".
  * `Sh_Scav` scans an 8m radius for dumpsters (1334, 1372, 3035), street cans (849, 1328, etc.), indoor bins, vending machines, and food carts (1340–1342, 588). Checks vendor presence/death status. Commands Shion to walk to the target and perform search/kick animations, then delegates to `Sh_Loot`.
  * `Sh_Loot` rolls probability tables based on perk allocations. If successful, vending/carts grant immediate health refills (`0223: health_to 176`), reset hunger, and increment `MEM_TIMES_DUMPSTER`. Trash bins roll for food or dropped weapons (shotguns, SMGs, pistols, Molotovs) presented via interactive `[Y/N]` prompt.

### Narrative, Menus & Social Dialogue
* **`ShionChat.txt` (`Y_ShChat`):**
  * Central dialogue router running an ambient timer (90–150s pacing) during peacetime free-roam and sub-missions.
  * Listens for manual conversation prompt via `Y + T`.
  * Evaluates health emergencies (critical health triggers high-priority panic lines).
  * **Marine & Aerial Physics Gatekeeper (`Check_Swimming_Or_Falling`)**: Strictly inspects player and companion states for water immersion (`04AD`), active swimming (`0965`), and unassisted on-foot freefall (`0818` with ground clearance > 2.0m, explicitly exempting jetpack flight `0A0C`). Automatically disarms the manual `Y + T` prompt, holds the ambient timer at 0 to avoid immediate trigger spikes upon landing, and blocks out-of-context free-roam scripts (`chatIdle.c`, `chatIdleSniper.c`, `chatMoney.c`, `chatZone.c`, `chatWeather.c`) while explicitly exempting active *Beat the Cock* Triathlon swimming ($Mission_BeatTheCock_Type > 0) so aquatic sub-mission dialogue can be summoned.
  * Evaluates context and dispatches to specialized sub-scripts: `chatWanted.c`, `chatSubMission.c`, `chatZone.c`, `chatRadio.c`, `chatVehicle.c` (strictly partitioned by vehicle category, preventing generic car cruising lines from leaking into boats or planes), `chatWeather.c`, `chatMoney.c`, `chatInterior.c`, `chatIdle.c`, `chatIdleSniper.c`.
* **`chatWeather.txt` (`Sh_Wthr`):**
  * Environmental weather and atmospheric conversation routine arbitrating through Tier 2 Priority Lock (`MEM_PRIORITY_LOCK = 2`).
  * Classifies live weather into 5 distinct atmospheric groups: Rain & Thunderstorms (IDs 8, 16), Clear Skies / Blistering Heatwave (IDs 0, 11, 13, 17, 18), Desert Sandstorms (ID 19), Dense Maritime Fog (ID 9), and Heavy City Smog (IDs 2, 3), split cleanly between daytime and nighttime (21:00–05:59).
  * Form-aware delivery: bifurcates lines between Physical Ped (`11@ == 0`) and floating Hitodama spirit form (`11@ == 1`), featuring Shion's spectral indifference to rain, desert heat, and soot while Jo'on exploits her chill as a personal air conditioner or hair shield.
  * **3-Tier Context Sensor**: Distinguishes between On-Foot (`26@ = 0`), Enclosed Cabin with Roof (`26@ = 1`), and Open-Top / Roofless (`26@ = 2`).
  * **Data-Driven Vehicle Roof Architecture**: Completely bypasses broken native `0EFB: is_car_convertible` handling flags (which erroneously flag 13 hardtop tuners/trucks as convertibles while omitting true convertibles). Queries `[VehicleRoofTypes]` in `ShionVehicles.ini`:
    * *Mode 1 (Permanent Open-Top)*: Always roofless (Comet, Bandito, Kart, Mower, etc.).
    * *Mode 2 (Modular Lowriders)*: Open-top by default, inspects tuning slot 7 (`096D`) for installed hardtop/soft-top upgrades (Blade, Savanna).
    * *Mode 3 (Dynamic Spawn Extras)*: Inspects the `"extra1"` frame node via `0E82: does_car_have_part_node`; treats vehicle as enclosed if spawned with a roof, or roofless if spawned open-top.
    * *Default (Unlisted)*: Enclosed cabin (`26@ = 1`).
* **`ShionLore.txt` (`Sh_Lore`):**
  * Milestone progression engine scanning the 31 bits of `MEM_LORE_PROGRESS_BIT`.
  * Evaluates non-linear gameplay, combat, financial, and environmental milestones across San Andreas:
    * *Combat & Career Milestones*: Shion firearms kills (Ep 1: 25 kills; Ep 5: 100 kills - "The Drug of Poverty"), Pimping Level 5+ (Ep 2), Burglary deliveries (Ep 14), Paramedic saves (Ep 20), vehicle destruction together (Ep 15), and Grim Pity death-defiance saves (Ep 16).
    * *Environmental & Atmospheric Conditions*: Mount Chiliad altitude on foot (Ep 4), dive bar lingering (Ep 12), gas station pump and tanker dwell (Ep 13), train passenger transit > 45s (Ep 22), late-night Rodeo commercial district walks (Ep 26), and alleyway walks in Idlewood/Doherty (Ep 27).
    * *Financial Thresholds*: Casino table losses >= $10k (Ep 11), high-roller wealth thresholds ($100k at LS Tower in Ep 10; $1M along The Strip in Ep 7; $500k in Ep 25), post-wealth bankruptcies ($0 balance in Ep 23 & 28), and safehouse property portfolios >= $500k (Ep 29).
  * Prompts player `[Y: Listen | N: Busy]`. If accepted, streams `loreTheater.c` passing the unlocked episode index.
* **`loreTheater.txt` (`Sh_Thea`):**
  * Claims Tier 4 lock, executes multi-stage cutscene dialogue scripts for Episodes 0–30, and writes `MEM_LORE_PROGRESS_BIT` upon clean completion.
  * Aborts gracefully with priority restoration if interrupted by cell phone calls, combat, player death, or water immersion.
* **`loreMenu.txt` (`Sh_LMen`):**
  * Interactive archive browser (`Y_LMTIT`) displaying 4 pages of 8 episodes each.
  * Reads unlocked state from `MEM_LORE_PROGRESS_BIT` (or debug bypass cheat "ALLORE"). Shows requirement tooltips. Selecting an episode streams `loreTheater.c` for replay.
* **`ShionMenu.txt` (`Sh_Menu`):**
  * Safehouse configuration terminal opened by holding `TAB` while inside any of the 32 safehouses (`MEM_SAFEHOUSE_DOOR_ID > 0`).
  * Page 0 (Arsenal): Launches Safehouse Performance Review (`ShionMem.c`), toggles Combat Resource Mode (`MEM_COMBAT_RES_MODE`), toggles Durgastra Limitless Debt (`MEM_DURGASTRA_FLAG`), toggles Combat Hitodama FX (`MEM_CMBT_HITODAMA_FX`), toggles shot chimes, and bankrupt alerts.
  * Page 1 (Lifestyle): Toggles Food Runs, Sister Talk episodes, opens Lore Archive (`MEM_LORE_MENU_SIGNAL = 1`), cycles startup character (`StartupPlayerCharacter`), and toggles idle smoking. Writes directly to `Shion.ini`.
* **`ShionMem.txt` (`Sh_Mem`):**
  * Safehouse retrospective conversation. Evaluates 9 career categories using performance telemetry:
    * Law & Casualties: Arrest vs death ratio (`MEM_TIMES_WASTED`, `MEM_TIMES_BUSTED`).
    * Combat Kill Share: Jo'on vs Shion kill ratio (`MEM_JOON_KILLS_W_SHION`, `MEM_SHION_KILLS`).
    * Sustenance: Gourmet treats vs dumpster meals (`MEM_TIMES_TREATED`, `MEM_TIMES_DUMPSTER`).
    * Wheels: Mod garage spending vs wrecked cars (`MEM_MOD_CASH_SPENT`, `MEM_CARS_WRECKED`, `MEM_FAV_CAR_MODEL`).
    * Real Estate: Percentage of the state's $979,000 safehouse budget owned (`Property Budget` stat 15).
    * Finances: Net casino table losses (`MEM_CASINO_LOSSES`) and poverty ratio (`MEM_TIME_BROKE_MIN`).
    * Radio: Most and least favorite stations (`MEM_FAV_RADIO_STAT`, `MEM_WORST_RADIO_STAT`).
    * Odd Jobs: Career effort distribution across Taxi, Paramedic, Firefighter, Vigilante, Pimping, and Burglary.
    * Money Danmaku: Pay-to-win expenditure index (`MEM_DANMAKU_CASH_SPENT`).
* **`ShionSandbox.txt` (`Sh_Sndb`):**
  * Environmental event monitor:
    * Detects firetruck water cannon sprayed on physical Shion.
    * Detects vehicle sinking into ocean water (`02BF`).
    * Detects swimming in deep water for > 5 seconds.
    * Detects Pay 'n' Spray repair of a heavily damaged car (< 500 HP restored to >= 990 HP).
    * Detects stunt flips (car flips upside down in air and recovers onto wheels).
    * Intercepts punches or vehicular hit-and-runs on innocent, unarmed civilians.
    * Detects continuous lowrider hydraulic hopping for > 3 seconds.
    * Catches Car Smasher vehicle punch explosions (`MEM_CAR_SMASH_SIGNAL`).
    * Catches cash flex events (`MEM_FLEX_SIGNAL`).
* **`ShionFriendlyFire.txt` (`Sh_FF`):**
  * Intercepts attacks from Jo'on against Shion via `OnCharDamageFF`.
  * Maps damage to specific weapon categories (fists, brass knuckles, golf club, nightstick, knife, bat, shovel, pool cue, katana, chainsaw, dildo, flowers, cane, grenades, rocket launchers, satchel charges, pistols, shotguns, automatics, rifles, spray can, extinguisher).
  * Detects camera viewfinder aim and snapshots on physical Shion or Hitodama spirit (`MEM_HITODAMA_X/Y/Z`) to trigger photo reactions.
* **`chatDismiss.txt` (`Sh_Dism`):**
  * Governs dialogue delivery and post-dismissal ambient life when Shion is dismissed on foot via `Entourage.c`.
  * Immediately applies the shared Zero-Panic Decision Maker (`MEM_SHION_DM`) and locks passive temperament attributes (`actions_uninterupted_by_weapon_fire 1`, `maintain_position_when_attacked 1`, `attack_rate 0`).
  * Priority Arbitration: Gated strictly behind Tier 4 Master Theater (`>= 4`), preventing combat audio locks from killing dismissal dialogue.
  * Claims Tier 2 Priority Lock, launches `ShionFriendlyFire.c`, and plays quick-dismiss easter eggs or randomized banter lines (`Y_DS*`).
  * On dialogue completion, releases Priority Lock, clears stay-put, and commands ambient walking paths (`05DE`).
  * Runs a stalker proximity watchdog (speaks up to 3 complaints via `Y_STK*` if Jo'on tails her within 3m on foot), rejects re-recruitment attempts via `G` (`Y_NORE*`), cleans up actor reference on distance (> 60m) or 60s timeout, and routes death telemetry to `chatFollow.c` if killed.
* **`chatSpawned.txt` (`Sh_Spwn`):**
  * Streamed by `SpawnShion.cs` immediately upon companion ped materialization if `MEM_SHION_DEATH_FLAG == 1`.
  * Claims Tier 3 Priority Lock, consumes and resets `MEM_SHION_DEATH_FLAG = 0`, and delivers weapon-specific post-death complaints across 17 damage categories.
  * Contextual explosion routing: for vehicle detonations (Weapon ID 51), evaluates `MEM_SHION_DEATH_IN_CAR` to voice distinct complaints for being trapped inside an exploding wreck (`1`), getting caught in Jo'on's *Car Smasher* punch detonation (`2`), or being caught in Jo'on's gunfire detonation (`0`).
* **`chatReunion.txt` (`Sh_Reun`):**
  * Streamed when approaching an ambient loitering Shion at a hangout spot and pressing `Y` to recruit her.
  * Claims Tier 4 Master Theater Lock, checks `MEM_SHION_DEATH_FLAG`, and delivers contextual two-person sister banter resolving how she died (including whether Jo'on was the killer).
  * Evaluates vehicle explosions via `MEM_SHION_DEATH_IN_CAR` to debate driving a burning wreck (`1`), glowing Car Smasher fist strikes (`2`), or trigger-happy lead sprays (`0`).
  * If Shion did not die earlier in the session, selects from 6 randomized peacetime free-roam reunion dialogues (`Y_RN0`–`Y_RN5`), sets `MEM_REUNION_SIGNAL = 1`, and releases Tier 4 priority.
* **`ShionDebug.txt` (`Sh_Dbg`):**
  * Centralized quality assurance and testing harness gated behind `[Debug] DebugMode = 1` in `Shion.ini`. Automatically terminates on boot if disabled, ensuring zero testing cheats can conflict during regular gameplay.
  * Summoned in-game via cheat `SHDBG`, running an interactive native panel (`08D4`) strictly controlled via `[SPACE]` (Select) and `[ENTER]` (Cancel/Back), with full actor motion locking (`04D7`) while browsing.
  * Features direct cheat and menu actions:
    * `KILLPOOR`: Strips Shion to 5 HP and 0 Armour across ped and Hitodama forms.
    * `HUNGRY` / `FOOD`: Rewinds `MEM_LAST_FED_HOURS` by 48 hours and asserts `MEM_SHION_HUNGRY = 1` to trigger starvation.
    * `INJURED`: Drops physical Shion to 500 HP and strips armour to test wounded state reactions.
    * `LORE`: Signals `MEM_LORE_MENU_SIGNAL = 1` to open the Sister Talk Archive on demand.
    * `KILLTHEPOOR`: Spawns an aggressive assassin ped equipped with Jo'on's current weapon, monitored by an active thread watchdog that destroys the assassin once Shion dies.
    * `TRIGGERCHAT`: Interactive submenu to stream all chat scripts directly on demand.
* **`chatFollow.txt` (`Sh_FolC`):**
  * Dedicated vocal dispatcher for Jo'on governing movement commands, vehicle transit, and immediate on-scene death reactions across 9 operational modes (`0`–`8`).
  * **Passive Non-Locking Command Barks (Modes 0, 1, 4, 5, 6, 7, 8)**: Tactical order and seat-cycling lines are treated as low-priority quips. They never write to `MEM_PRIORITY_LOCK`, ensuring that rapid command toggles (e.g., calling Shion into a car with `G` and immediately pressing `H` to hold) cannot leave a dirty lock behind if a previous thread is terminated via `0ABA`.
  * **Zero Dialogue Trampling**: All command modes check `MEM_PRIORITY_LOCK > 0` on entry and self-terminate silently (`0A93`) without playing sound or clearing on-screen text (`03EB`) if any higher-tier conversation (smoking, ambient chat, weather, radio, or cutscenes) is active. Companion actions execute mechanically in silence during active chatter.
  * **Tier 3 Reaction Integrity (Modes 2 & 3)**: Companion loss (> 130m away, Mode 2) and immediate death reactions (Mode 3) claim Tier 3 priority lock (`MEM_PRIORITY_LOCK = 3`), preempting ambient free-roam conversations while safely yielding to Tier 4 Master Theater events.
  * Features granular contextual death reaction trees and 2-stage reactive monologues when Jo'on is the killer.

---

## 5. Yorigami Prestige Perks

```
                                  +-------------------+
                                  |    Menu.cs (UI)   |
                                  +---------+---------+
                                            |
         +----------------------------------+----------------------------------+
         |                                                                     |
+--------v--------+                                                   +--------v--------+
|  Jo'on: Wealth  |                                                   |Shion: Misfortune|
| Bling / Smasher |                                                   | Resonance / Pity|
| Taxi Extortion  |                                                   | Traffic / Air   |
| Drip Worship    |                                                   | Misfortune Aura |
| Shakedown / Mark|                                                   | Debt Detonation |
| Fever / Danmaku |                                                   | Misery (Voodoo) |
+-----------------+                                                   +-----------------+
```

### Allocation & Progression UI (`Menu.txt` / `Y_Menu`)
* Bounded to `Y + M`. Provides interactive menu panels for point distribution.
* **Hub Interface:** Routes to Jo'on Extravagance, Shion Misfortune, Yorigami Contract, Mod Preferences, and Respec.
* **Interactive Tooltips & Tutorials:**
  * *Dynamic Tooltips:* Scrolling over any perk row automatically samples `YorigamiPerks.ini` to display live stat bonuses, milestones, and soft/hard cap ceilings in the top-left text box.
  * *Gameplay Tutorials (`Space` / `Sprint`):* Pressing `Space` over any allocated perk in the Jo'on or Shion menus displays a dedicated tutorial help box explaining practical in-game activation, required inputs, and combat mechanics (e.g., aiming + `G` recruitment for Drip Worship, `Y + G` toggle for Money Danmaku, pursuit conditions for Traffic Hex).
* **Stepping Logic:** Normal tap adjusts by ±1. Holding `Shift` adjusts by ±10. Holding `Ctrl` adjusts by ±100. Holding `Shift + Ctrl` for 2.5s dumps or refunds all points.
* **Perk Caps:** Queries dynamic caps from `YorigamiPerks.ini`. If a perk has reached its cap, further increments are rejected with an error buzzer.
* **Yorigami Contract:** Grants an instant loan of configurable points (`ContractPoints`, default 100) in exchange for equivalent negative debt (`Points * CashPerPoint`, default -$1,000,000). Writes to `MEM_CONTRACT_ESCROW` to prevent double-dipping on wallet spending.
* **Respec:** Resets all 16 perk allocations to 0 and refunds all points back to `MEM_AVAIL_POINTS` without clearing current debt.

### Jo'on: Wealth Perks
* **`Bling.txt` (`Y_BLING`):**
  * *Jo'on's Bling (`MEM_ALLOC_JOONBLING`):* Adds scaled melee damage (`Points * BonusDamagePerPoint`, soft-capped by `MaxDamageCap`) to fists and brass knuckles. In Jo'on mode, targeting a ped (> 1.5m, <= 50m) and attacking executes a high-speed 3D dash ("Zippity Zap") with golden afterimage coronas and particle trails, granting temporary bulletproof graze immunity (`FX_JoonGraze.c`). In Shion mode, executes an ethereal remote punch.
  * *Car Smasher (`MEM_ALLOC_SMASHER`):* Punches deal scaled damage to vehicles (`Points * BonusDamage * CarDamageMultiplier`). If car HP drops <= 0, grants blast immunity (if points >= `BlastShieldUnlockPoints`), unhooks damage callbacks to protect the stack, detonates the vehicle, and writes `MEM_CAR_SMASH_SIGNAL = 1`.
  * *Fist-Lock:* Forces unarmed/knuckles during combat approach, breaking out instantly on mouse wheel scroll.
* **`Danmaku.txt` (`Y_Danm`):**
  * Toggled via `Y + G` (`MEM_DANMAKU_TOGGLE`).
  * Intercepts char and vehicle damage via `0EDE` and `0EDF`.
  * Verifies shooter is Jo'on or armed companion Shion using a firearm (IDs 22–38).
  * Deducts cash cost per bullet ($100 default) from player wallet, logs `MEM_DANMAKU_CASH_SPENT`, vaporizes enemy body armor on impact, applies bonus damage, deals 4x damage against vehicles, and creates golden impact sparks.
* **`Fever.txt` (`Y_Fever`):**
  * Activated via `Y + C` on a dynamic cooldown (60s down to 20s based on points).
  * Affects pedestrians within an expanding radius (15m to 35m).
  * Forces all non-gang/non-story peds to drop cash pickups (base $60 up to $300, doubled for businessmen and dealers), attach golden glitter particles, and flee in financial ruin (`05DD: flee_from_actor`).
* **`Shakedown.txt` (`Y_Shake`):**
  * Aim at a ped on foot within 2.8m and press `Y`.
  * Forces both actors into robber/victim animations (`"ROB_Loop_Threat"` / `"SHP_Rob_React"`).
  * Extorts base cash ($150–$400) + scaled bonus per point, plus archetype bonuses (Dealer +$2000, Rich +$1200, Criminal +$800).
  * Calculates resistance rolls based on perk points. If target folds: drops cash pickups, awards respect if rival gang, forces gang members to drop their weapons as physical pickups, and makes target flee. If target resists: target draws weapons and attacks, placing an active bounty on their head.
* **`Mark.txt` (`Y_Mark`):**
  * Periodically spawns High-Roller Marks in free-roam outdoors (every 45–180s).
  * Selects an ambient pedestrian, creates a gold radar blip, attaches a bright red flare (`"smoke_flare"`), and stores handle in `MEM_ACTIVE_MARK`.
  * Extorting or eliminating the mark before escape timeout (60s) awards jackpot cash ($2,500 to $10,000) and bonus respect.
* **`Entourage.txt` (`Y_TROOP`):**
  * Unlocks 7-slot group capacity.
  * Aim at ped and press `G`: rolls recruitment probability based on Drip Worship points. Can charm civilians (up to 90%), drug dealers (unlocked at 25 pts, up to 70%), and rival gang members (unlocked at 60 pts, up to 50%). Converts loyalties to Grove Street, equips with 9mm pistol if unarmed, and adds to group. Cops are immune.
  * Aim at follower and press `N`: dismisses follower, returning them to ambient wander. Special handling for Shion immediately ungroups her, updates `MEM_SHION_RECRUITED = 0`, `MEM_INTENTIONAL_DISM = 1`, and `MEM_SHION_STANCE = 0`, terminates all follower/combat worker daemons (`Sh_Flw`, `Sh_Ctrl`, `Sh_Cmbt`, `Sh_Sync`, `Sh_Chut`, `Sh_Pimp`, `Sh_Burg`, `Sh_RedM`, `Sh_FF`), clears stale priority audio locks (< Tier 4), installs the shared Zero-Panic Decision Maker (`MEM_SHION_DM`) **before** stripping her weapon to prevent unarmed-aim flinches in Hostile mode, zeroes combat aggression parameters, freezes her in place facing Jo'on, and streams `chatDismiss.c`.	
* **`Lifestyle.txt` (`Y_Life`):**
  * *Taxi Extortion (`MEM_ALLOC_TAXIEXT`):* Intercepts taxi mission fares, awarding `Points * BonusPercentPerPoint` extra extortion cut on every drop-off.
  * *Drip Worship (`MEM_ALLOC_DRIPWORSHIP`):* Injects calculated points directly into native memory address `12030948` (Sex Appeal stat), scaling up to 1000 (100% full bar).
  * *Midas Magnet (`MEM_ALLOC_MIDAS`):* Scans the native pickup pool (`0x9788C0` to `0x97D620`, step 32 bytes) for natural cash drops (Type 8). Vacuums pickups within dynamic radius (8m to 35m) directly to CJ's wallet, applies bonus cash multiplier, destroys the pickup, and spawns golden glitter. Excludes scripted mission pickups (Type 19).

### Shion: Misfortune Perks
* **`Misfortune.txt` (`Y_Misfortune`):**
  * *Shion Resonance (`MEM_ALLOC_RESONANCE`):* If allocated >= 5 points and player cash is $0 (or in debt), holding a 1-star wanted level on foot while strictly unarmed or holding brass knuckles (weapon ID `<= 1`) for 6 seconds causes police to lose interest and clears the wanted level (`"TOO BROKE TO PURSUE"`). Drawing any other weapon immediately resets the 6-second timer.
    * **Active Pursuit Pacification Protocol (`@Pacify_Nearby_Cops`):** Upon activation, momentarily pulses `01F7: set_player ignored_by_cops` to purge player threat registers from the police dispatch scanner, then sweeps a 65m radial zone around the player for law enforcement (`PedType 6`).
    * **Vehicle Cruisers:** Switches squad cars to normal driving behavior (`00A9`) and immediately silences sirens (`0397: enable_car siren 0`).
    * **Foot Officers:** Clears primary and secondary tasks (`0792`, `0E96`, `0E97`), aborting active firearm aiming, pursuit sprints, and arrest attempts, then assigns ambient patrol navigation (`05DE: AS_actor walk_around_ped_path`).
  * *Misfortune Aura (`MEM_ALLOC_MISFORTUNE`):* Scans hostiles within 25m on a configurable pacing interval (`[ShionMisfortune] ScanIntervalSeconds`, default 5s, stored in `8@`). Rolls probability for physical stumble/knockdown animations or weapon jams.
    * **Law Enforcement Combat Decoupling:** In peacetime (`wanted_level == 0`), cops (`PedType 6`) are excluded from firing/fighting checks (Check 1) and historical damage memory via `051A` (Check 2), relying strictly on active pursuit stars (Check 3).
    * **Retaliation Suppression:** After recovering from a stumble or weapon jam, cops are filtered out of the kill-target check (`0E44`), guaranteeing `05E2: kill_actor $PLAYER_ACTOR` is never assigned to them.
* **`Pursuit.txt` (`Y_Pursuit`):**
  * *Police Bribe:* Type "BRIBE" to clear 1 wanted star for cash ($20,000 default), incrementing `MEM_BRIBES_PAID`.
  * *Traffic Hex (`MEM_ALLOC_TRAFFICHEX`):* Scans 45m radius around player for pursuit cruisers and bikes when wanted level > 1 on a configurable interval (`[TrafficHex] ScanIntervalSeconds`, default 5s, stored in `18@`). Rolls stall chance, applies engine stall action, deflates tires on 4-wheel cruisers, and uses `20@` as a single-scan audio limiter.
  * *Aviation Calamity (`MEM_ALLOC_AVIATION`):* Scans 300m radius for airborne pursuit aircraft when wanted level > 2 on a configurable interval (`[AviationMisfortune] ScanIntervalSeconds`, default 4s, stored in `11@`). Rolls explosion chance and detonates target.
* **`DebtDet.txt` (`Y_DebtD`):**
  * Triggers when player health drops below threshold (default 50% HP) while having points in Debt Detonation.
  * Launches `FX_GrimBurst.c` (ethereal sapphire screen shockwave).
  * Knocks down all peds within dynamic radius (30m to 60m), strips their weapons, creates ground pickups if `EnableDisarmDrops = 1`, forces back-skid and cower tasks, applies `FX_HexFlame.c`, and engages dynamic cooldown (60s down to 10s). Actively recruited squad members and special actors are protected.
* **`GrimPity.txt` (`Y_PITY`):**
  * Shion's lethal health gate / death defiance.
  * Intercepts fatal damage via `OnGrimPityDamage` when player wallet is <= $0.
  * Nullifies death: overwrites engine ped death state (`CPed + 0x530 = 1`), cancels `TASK_SIMPLE_DIE` via `0687: clear_actor task`, restores health to calculated percentage (25% to 75% max HP), grants 2.5s absolute invulnerability godmode (`BP 1 FP 1 EP 1 CP 1 MP 1`), launches `FX_GrimBurst.c`, triggers radial knockdown shockwave on hostiles, sets `MEM_GRIMPITY_BURST = 1` (igniting outer flame ring in `FX_GrimPity.c`), and increments `MEM_GRIMPITY_SAVES`. Cooldown: 300s down to 60s.
  * **Post-Combat Survival Banter (`chatGrimPity.c` / `Sh_Pity`)**: Streams a 6-variant banter routine between Jo'on and Shion once 3.5 continuous seconds of peacetime calm are detected. Gated behind completion of Sister Talk Episode 16 ("The Aura Beneath the Rags"). Promotes to Tier 3 Priority Lock, incorporating a 4.0-second queue buffer for concurrent Friendly Fire complaints (`Sh_FF`) to conclude before purging transient threads and delivering lines. Aborts cleanly if interrupted by water immersion or freefall.
* **`Misery.txt` (`Y_Misy`):**
  * Intercepts incoming damage on CJ when broke/in debt via `OnCharDamage`.
  * Identifies attacker entity (direct ped or vehicle driver).
  * Uses recursion lock `MEM_DMG_RECURSION_LCK` (`184`) to prevent loopbacks.
  * Calculates reflected damage (`BaseReflectionPercent + Points * ReflectionPerPoint`, soft-capped by `MaxReflectionCap`).
  * Applies script damage (`0851`) to the attacker, plays impact audio, and spawns `shion_misfortune` particles.
* **`Scavenger.txt` (`Y_Scav`):**
  * Enables player on-foot scavenging of dumpsters, street cans, indoor bins, vending machines, and food carts when broke or in debt.
  * Restores health to 176 and resets hunger clock.
  * Handles item extraction and weapon drops with `[Y/N]` prompt.
  * Jo'on identity check: if player is Jo'on, she refuses to scavenge and demands Shion do it.

---

## 6. Visual FX Pipeline

```
                                +-------------------+
                                |    Game Engine    |
                                +---------+---------+
                                          |
         +--------------------------------+--------------------------------+
         |                                |                                |
+--------v--------+              +--------v--------+              +--------v--------+
|  Flame Clusters |              | Burst Coronas   |              |  Combat Shields |
| FX_JoonFlames   |              | FX_SwitchBurst  |              | FX_JoonGraze    |
| FX_ShionFlames  |              | FX_GrimBurst    |              | FX_ShionHitodama|
| FX_Hitodama     |              | FX_Warp         |              | FX_HexFlame     |
| FX_GrimPity     |              | FX_FLEX         |              | FX_ShionMoney   |
+-----------------+              +-----------------+              +-----------------+
```

All persistent visual daemons run 0ms loop cycles normalized against `CTimer::ms_fTimeStep` (`0xB7CB5C * 0.6`) for 60 FPS framerate-independent rendering.

* **`FX_SwitchBurst.txt` (`Sh_Swtc`):** Full-screen camera rumble and triple-layered explosive bloom coronas (Outer Aura, Mid Flare, Core Spark) centered on player torso over 400ms during character swaps. Mode 1 (Jo'on) renders deep gold/sunburst yellow; Mode 2 (Shion) renders electric sapphire/cyan.
* **`FX_Warp.txt` (`Sh_Warp`):** 2-phase bloom and contract corona burst (500ms duration) used for spatial teleportation, companion recruitment, and Hitodama materialization. Supports static world coordinates (Mode 0) or attachment to moving vehicle actors (Mode 1).
* **`FX_Hitodama.txt` (`Sh_Hito`):** 3 erratic teardrop flame clusters orbiting physical Shion. Active during combat or within 6 seconds of damage/shooting. Each cluster consists of 5 sub-coronas (Base Anchor, Radiant Belly, Tapered Neck, Flickering Tip, Diamond Core) modulated by sine/cosine harmonic wave clocks.
* **`FX_ShionHitodama.txt` (`Sh_HitoF`):** Renders the floating Hitodama spirit form when Shion has no physical body (`MEM_SHION_HANDLE == -1`). Renders a central Mother Flame (4-layer corona cluster) and 3 satellite teardrop flames orbiting in 120-degree phase shifts. Follows Jo'on with an exponential 3D lerp. Intercepts incoming gunfire to act as an ablative shield.
* **`FX_JoonFlames.txt` (`Y_FXJ`):** Golden spirit flames trailing Jo'on during combat, Money Danmaku mode, or skill casts, lingering for 3.5 seconds after combat actions.
* **`FX_ShionFlames.txt` (`Y_FXS`):** Blue Hitodama spirit flames trailing player Shion (`MEM_PLAYER_FORM == 2`) during combat or Money Danmaku mode.
* **`FX_GrimPity.txt` (`Y_FXG`):** Dual-layer orbital ring daemon. Inner Ring: 3 clockwise blue flames indicating low-HP debt danger. Outer Ring: 3 counter-clockwise high-speed flames ignited when Grim Pity resurrection activates (`MEM_GRIMPITY_BURST == 1`).
* **`FX_GrimBurst.txt` (`Y_FXGB`):** 750ms 3-layer blinding sapphire/cyan/white shockwave corona bloom deployed at the instant of Grim Pity death defiance or Debt Detonation.
* **`FX_HexFlame.txt` (`Y_HexF`):** Single blue Hitodama teardrop flame that attaches above the head of a hostile affected by Misfortune Aura, Debt Detonation, or Shion combat curses.
* **`FX_JoonGraze.txt` (`Y_Graze`):** Spawns a translucent protective golden corona sphere (1.1m radius) around Jo'on during melee dashes, granting 2000ms bulletproof immunity (`BP 1`).
* **`FX_ShionMoneyDanmaku.txt` (`Sh_FXDn`):** 3 golden satellite orbs orbiting Shion (player, ped, or Hitodama spirit) when Money Danmaku is active.
* **`FX_FLEX.txt` (`Y_FXF`):** 4-wave staggered golden coin and spark burst (`"joon_gold"`) surrounding the player during cash flexing.
* **`YorigamiAimAlpha.txt` (`Y_AIMTR`):** Reads aiming state at the Ammu-Nation shooting range (`$ONMISSION_Shooting == 1`). Sets player alpha to 0 while aiming down sights to prevent model obstruction, restoring alpha to 255 upon release.

---

## 7. Data Persistence Map

```
[Save Game Safehouse Floppy Disk]
                |
     0ED1 Event Callback
                |
   +------------+------------+
   |                         |
+--v------------------+   +--v------------------+
|    yorigami.cs      |   |    ShionTrack.cs    |
| Writes:             |   | Writes:             |
| - yorigami.ini      |   | - ShionMemories.ini |
| - Yorigami_Ledger   |   | - ShionVehicles.ini |
|                     |   | - ShionRadio.ini    |
+---------------------+   +---------------------+
```

Data persistence is segmented across dedicated INI files organized by save slot sections (`[Slot_0]` through `[Slot_8]`), flushed on safehouse floppy disk contact via `0ED1: set_script_event_save_confirmation`.

### File Directory & Roles

```
cleo\
├── Shion.ini                     # Master configuration for companion & controls
├── YorigamiPerks.ini             # Master balancing knobs, caps, scan intervals, & perk rates
└── savedata\
    ├── yorigami.ini              # Progression save: points, buffer, perk ranks
    ├── Yorigami_Ledger.txt       # Plain-text audit ledger: slot, day, points, wallet
	├── ShionMemories.ini         # Performance ledger: kills, treats, wrecks, stats, lore bits
    ├── ShionVehicles.ini         # Vehicle odometer (minutes per model) & [VehicleRoofTypes] registry
    ├── ShionRadio.ini            # Radio station listening minutes (Stations 0–12)
    ├── ShionWeapon.ini           # Stashed weapon ID & ammo for Shion and Jo'on
    ├── ShionFood.ini             # Custom registered restaurant/food coordinates
    └── YorigamiBins.ini          # Custom registered dumpster/bin coordinates
```

### Save / Load Synchronization Flow
1. **Save Game Execution (`OnGameSaved` / `OnTrackSaved`):**
   * `0E2C: get_current_save_slot` identifies active slot (0–8).
   * `yorigami.cs` synchronizes `MEM_TOTAL_POINTS`, `MEM_AVAIL_POINTS`, and `MEM_ALLOC_*` into `yorigami.ini`, writing an entry to `Yorigami_Ledger.txt`.
   * `Joon.cs` snapshots Shion's designated weapon and ammo, zeroes Jo'on's temporary stash if not playing as Shion, and writes to `ShionWeapon.ini`.
   * `ShionTrack.cs` flushes performance telemetry registers (`MEM_TIME_TOGETHER_MIN`, `MEM_SHION_KILLS`, `MEM_LORE_PROGRESS_BIT`, etc.) into `ShionMemories.ini`.
   * `ShionTrack_World.cs` writes the dynamic vehicle odometer array to `ShionVehicles.ini` and radio minutes to `ShionRadio.ini`.
2. **Load Game Execution (Boot / Death / Slot Change Watchdogs):**
   * Scripts monitor `MEM_SAVE_SLOT` and detect when the slot index changes from previous baseline or when player despawns during load screens.
   * `yorigami.cs` reads all perk allocations from `yorigami.ini`, calculates total spent points, sets available points, and synchronizes baseline cash tracker `1@` to live wallet.
   * `Joon.cs` reads `StartupPlayerCharacter` from `Shion.ini`:
     * Mode 0: Reverts player to CJ.
     * Mode 1: Rebuilds player model into Jo'on (`#SPECIAL01`), clears gang hostility, and loads stashed weapon from `ShionWeapon.ini`.
     * Mode 2: Rebuilds player model into Shion (`#SPECIAL01`), equips weapon from `ShionWeapon.ini`, and suppresses companion ped.
   * `ShionTrack.cs` reads all lifetime metrics from `ShionMemories.ini` into master heap offsets `228–328`.
   * `ShionTrack_World.cs` reads `ShionVehicles.ini` into its 4 KB RAM cache `1@`, evaluates the vehicle model with the highest minutes, and crowns it into `MEM_FAV_CAR_MODEL` (`312`). Evaluates radio stations from `ShionRadio.ini` and writes favorite and least favorite IDs to `MEM_FAV_RADIO_STAT` (`332`) and `MEM_WORST_RADIO_STAT` (`336`).

---

## 8. Complete System Interaction Matrix

The following table details the cross-script dependencies for every component in the codebase:

| Component | Source File | Triggers / Activators | Heap Writes | Heap Reads | Target Files / Worker Threads |
|---|---|---|---|---|---|
| **Master Heap** | `Y_Brain.txt` | CLEO boot | Base buffer allocated | None | Connects all scripts via struct `Y_Brain` |
| **Identity & Form** | `Joon.txt` | `Y + L`, "JOON", "POVERTY", Boot | `MEM_PLAYER_FORM`, `MEM_JOON_STASH_*`, `MEM_INTENTIONAL_DISM` | `MEM_SAVE_SLOT`, `MEM_SHION_HANDLE`, `MEM_SHION_WEAPON_ID` | Spawns `FX_SwitchBurst`, `JoonSmoke`; drives `SpawnShion` |
| **Companion Manager** | `SpawnShion.txt` | "SHION", Proximity to hangouts, `Y + H` | `MEM_SHION_HANDLE`, `MEM_SHION_RECRUITED`, `MEM_SHION_PRES_*`, `MEM_SHION_CHUTE_OBJ` | `MEM_PLAYER_FORM`, `MEM_SHION_BUSY`, `MEM_PRIORITY_LOCK`, `MEM_SHION_CHUTE_OBJ` | Spawns `FX_Warp`, `FX_ShionHitodama`, purges stray parachutes, and runs all `Shion*.c` workers |
| **Prestige Engine** | `yorigami.txt` | Wallet change, `Y + J` (Flex), Save disk | `MEM_SAVE_SLOT`, `MEM_TOTAL_POINTS`, `MEM_AVAIL_POINTS`, `MEM_FLEX_SIGNAL` | `MEM_CONTRACT_ESCROW`, `MEM_ALLOC_*` | Spawns `FX_FLEX`, launches all `Yorigami\*.c` perks |
| **Prestige Menu** | `Yorigami\Menu.txt` | `Y + M` on foot, `Space`/`Sprint` for tutorials | `MEM_ALLOC_*`, `MEM_AVAIL_POINTS`, `MEM_TOTAL_POINTS`, `MEM_CONTRACT_ESCROW` | `MEM_AVAIL_POINTS`, `MEM_TOTAL_POINTS` | Reads/writes `YorigamiPerks.ini`, communicates with `yorigami.cs` |
| **Jo'on Bling** | `Yorigami\Bling.txt` | Fists/Knuckles melee attacks, Vehicle punches | `MEM_CAR_SMASH_SIGNAL` | `MEM_ALLOC_JOONBLING`, `MEM_ALLOC_SMASHER`, `MEM_PLAYER_FORM` | Spawns `FX_JoonGraze`, triggers `ShionSandbox` & `ShionDeathTracker` |
| **Money Danmaku** | `Yorigami\Danmaku.txt` | `Y + G` toggle, Bullet impacts | `MEM_DANMAKU_TOGGLE`, `MEM_DANMAKU_CASH_SPENT` | `MEM_ALLOC_DANMAKU`, `MEM_SHION_HANDLE` | Triggers `FX_ShionMoneyDanmaku`, `FX_JoonFlames`, `FX_ShionFlames` |
| **Pestilence Fever** | `Yorigami\Fever.txt` | `Y + C` active cast | None | `MEM_ALLOC_FEVER`, `MEM_SHION_HANDLE` | Spawns cash pickups, forces ped flee tasks |
| **Street Shakedown** | `Yorigami\Shakedown.txt`| Aim + `Y` on ped | `MEM_LAST_EXTORTED_PED` | `MEM_ALLOC_SHAKEDOWN`, `MEM_ACTIVE_MARK` | Cooperates with `Mark.txt`, drops cash/weapons |
| **High-Roller Mark** | `Yorigami\Mark.txt` | Free-roam timer | `MEM_ACTIVE_MARK` | `MEM_ALLOC_SHAKEDOWN`, `MEM_LAST_EXTORTED_PED` | Sets radar blips, communicates with `Shakedown.txt` |
| **Drip Entourage** | `Yorigami\Entourage.txt`| Aim + `G` (recruit) / `N` (dismiss) | `MEM_INTENTIONAL_DISM`, `MEM_SHION_RECRUITED`, `MEM_SHION_STANCE`, `MEM_PRIORITY_LOCK` | `MEM_ALLOC_DRIPWORSHIP`, `MEM_SHION_HANDLE`, `MEM_SHION_DM` | Manages player group capacity, terminates companion daemons, installs `MEM_SHION_DM`, streams `chatDismiss.c` |
| **Dismissal Watchdog**| `Shion\Chat\chatDismiss.txt`| Streamed by `Entourage.c` on Shion dismiss | `MEM_SHION_HANDLE`, `MEM_SHION_DEATH_FLAG`, `MEM_INTENTIONAL_DISM`, `MEM_PRIORITY_LOCK`, `MEM_SHION_STANCE` | `MEM_PLAYER_FORM`, `MEM_QUICK_DISM_LATCH`, `MEM_SHION_RECRUITED`, `MEM_SHION_DM` | Runs dismissal dialogue, stalker watchdog, re-recruitment rejection, cleans up actor references |
| **Midas & Lifestyle** | `Yorigami\Lifestyle.txt`| Cash pickups near player, Taxi fares | None | `MEM_ALLOC_TAXIEXT`, `MEM_ALLOC_DRIPWORSHIP`, `MEM_ALLOC_MIDAS` | Injects Sex Appeal into engine memory, vacuums pickups |
| **Debt Detonation** | `Yorigami\DebtDet.txt` | HP <= threshold while broke | `MEM_DMG_RECURSION_LCK` | `MEM_ALLOC_DEBTDET`, `MEM_SHION_HANDLE` | Spawns `FX_GrimBurst`, `FX_HexFlame`, shockwave disarms peds |
| **Grim Pity** | `Yorigami\GrimPity.txt` | Lethal damage while wallet <= $0 | `MEM_GRIMPITY_BURST`, `MEM_GRIMPITY_SAVES` | `MEM_ALLOC_GRIMPITY`, `MEM_SHION_HANDLE`, `MEM_LORE_PROGRESS_BIT` | Spawns `FX_GrimBurst`, `FX_GrimPity`, streams `chatGrimPity.c` on calm |
| **Grim Pity Dialogue**| `Shion\Chat\chatGrimPity.txt`| Peacetime calm after Grim Pity save | `MEM_PRIORITY_LOCK` | `MEM_SHION_HANDLE`, `MEM_LORE_PROGRESS_BIT` (Bit 16), `MEM_PRIORITY_LOCK` | Claims Tier 3 lock, resolves concurrent friendly fire, delivers post-resurrection sister banter |
| **Misfortune & Resonance** | `Yorigami\Misfortune.txt`| 1-star wanted while unarmed/knuckles, Hostiles near player (timed via `ScanIntervalSeconds`) | None | `MEM_ALLOC_RESONANCE`, `MEM_ALLOC_MISFORTUNE`, `MEM_SHION_HANDLE` | Clears 1-star wanted, pacifies aggro'd cops in 65m radius, stumbles hostiles, spawns `FX_HexFlame` |
| **Pursuit Hexes** | `Yorigami\Pursuit.txt` | "BRIBE", Wanted level > 1 or 2 (timed via `ScanIntervalSeconds`) | `MEM_BRIBES_PAID` | `MEM_ALLOC_TRAFFICHEX`, `MEM_ALLOC_AVIATION`, `MEM_SHION_HANDLE` | Stalls cruisers, pops tires, explodes aircraft |
| **Voodoo Misery** | `Yorigami\Misery.txt` | Damage taken while broke | `MEM_DMG_RECURSION_LCK` | `MEM_ALLOC_MISERY`, `MEM_SHION_HANDLE` | Reflects damage to attacker via `0851` |
| **Player Scavenging** | `Yorigami\Scavenger.txt`| `F` near bin/cart while broke, "ADDBIN" | None | `MEM_ALLOC_SCAVENGER`, `MEM_PLAYER_FORM`, `MEM_SHION_HANDLE` | Gives health/weapons; Jo'on refuses and defers to Shion |
| **Aim Transparency** | `Yorigami\YorigamiAimAlpha.txt`| Aiming in shooting range mission | None | `MEM_PLAYER_FORM` | Toggles player alpha (`0` <-> `255`) |
| **Companion Stance** | `Shion\ShionControl.txt`| `Y + B` (Stance), `Y + Z` (Scavenge) | `MEM_SHION_STANCE`, `MEM_SHION_WEAPON_ID`, `MEM_SHION_BUSY` | `MEM_COMBAT_RES_MODE`, `MEM_DURGASTRA_FLAG`, `MEM_SHION_STAY_PUT` | Spawns `ShionScav.c`, manages companion aggression |
| **Companion Combat** | `Shion\ShionCombat.txt` | Firing weapons, "DURGASTRA" | `MEM_SHION_KILLS`, `MEM_JOON_KILLS_W_SHION`, `MEM_DURGASTRA_FLAG` | `MEM_COMBAT_RES_MODE`, `MEM_SHION_STANCE`, `MEM_SHION_WEAPON_ID` | Deducts shot costs, disarms on bankruptcy, reflects damage |
| **Gear Swap** | `Shion\ShionGear.txt` | `Y + R` on foot near Shion | `MEM_SHION_WEAPON_ID`, `MEM_SHION_AMMO_COUNT`, `MEM_SHION_STANCE` | `MEM_COMBAT_RES_MODE`, `MEM_SHION_HANDLE` | Exchanges weapons between Jo'on and Shion, auto-disarms when dry |
| **Death Telemetry** | `Shion\ShionDeathTracker.txt`| Shion takes damage or dies | `MEM_SHION_DEATH_FLAG`, `MEM_SHION_KILLER_IS_JOON`, `MEM_SHION_DEATH_WEAPON`, `MEM_SHION_DEATH_IN_CAR` | `MEM_CAR_SMASH_SIGNAL`, `MEM_SHION_HANDLE` | Logs cause of death for post-death quips and lockouts |
| **CJ Retaliation**   | `Shion\ShionRetaliation.txt`| CJ attacks or rams Shion with vehicle | `MEM_SHION_HANDLE`, `MEM_DMG_RECURSION_LCK`, `MEM_PRIORITY_LOCK` | `MEM_PLAYER_FORM` | Damages CJ or ignites vehicle, plays reaction, despawns Shion |
| **Follow & Transit** | `Shion\ShionFollow.txt` | `H` (Stay Put), `G` (Recall/Warp), Doors | `MEM_SHION_STAY_PUT`, `MEM_SAFEHOUSE_DOOR_ID`, `MEM_TIMES_WASTED/BUSTED`| `MEM_SHION_BUSY`, `MEM_INTENTIONAL_DISM` | Warps Shion indoors/outdoors, spawns `FX_Warp` |
| **Vehicle Seat Sync** | `Shion\ShionVehicle.txt` | Player driving, `G` (Board/Cycle), `H` | `MEM_SHION_STAY_PUT` | `MEM_SHION_BUSY`, `MEM_SHION_STAY_PUT` | Warps Shion into car seats, cycles seats, bails out on fire |
| **Target Acquisition** | `Shion\ShionCombatRedMarker.txt`| Red blips on radar | None | `MEM_SHION_STANCE`, `MEM_SHION_STAY_PUT` | Commands Shion to attack red-marked targets or execute drive-bys |
| **Hunger & Food** | `Shion\ShionFood.txt` | 48h elapsed, HP < 950, `TAB` at restaurant | `MEM_SHION_HUNGRY`, `MEM_LAST_FED_HOURS` | `MEM_SHION_RECRUITED`, `MEM_PRIORITY_LOCK` | Launches `foodDiner.c` (`MEM_PRIORITY_LOCK = 4`) |
| **Dining Sequence** | `Shion\Food\foodDiner.txt`| Spawned by `ShionFood` | `MEM_LAST_FED_HOURS`, `MEM_TIMES_TREATED`, `MEM_TOTAL_CASH_GIVEN`, `MEM_SHION_HUNGRY` | None | Deducts cash, restores health/armour, resets hunger |
| **Companion Scavenge** | `Shion\ShionScav.txt` | Spawned by `ShionControl` | `MEM_SHION_BUSY` | None | Locates target, plays search anim, spawns `scavLoot.c` |
| **Scavenge Loot** | `Shion\Scav\scavLoot.txt` | Spawned by `ShionScav` | `MEM_TIMES_DUMPSTER`, `MEM_LAST_FED_HOURS`, `MEM_SHION_BUSY` | None | Resolves loot tables, heals Shion/CJ, grants weapons |
| **Sister Talk Lore** | `Shion\ShionLore.txt` | Milestone triggers (Bits 0–30) | `MEM_PRIORITY_LOCK`, `MEM_SKYDIVE_LANDED`, `MEM_LOWRIDER_DONE` | `MEM_LORE_PROGRESS_BIT`, Performance ledger | Prompts `[Y/N]`, streams `loreTheater.c` |
| **Lore Theater** | `Shion\Lore\loreTheater.txt`| Spawned by `ShionLore` or `loreMenu` | `MEM_LORE_PROGRESS_BIT`, `MEM_PRIORITY_LOCK` | None | Executes cutscene dialogue, sets progress bitmask |
| **Lore Archive** | `Shion\Lore\loreMenu.txt`| "ALLORE", `ShionMenu`, or `ShionDebug` ("LORE") | `MEM_LORE_MENU_SIGNAL` | `MEM_LORE_PROGRESS_BIT` | UI panel for reviewing unlocked lore episodes |
| **Safehouse Menu** | `Shion\ShionMenu.txt` | Hold `TAB` inside safehouse | `MEM_PRIORITY_LOCK`, `MEM_COMBAT_RES_MODE`, `MEM_DURGASTRA_FLAG`, `MEM_LORE_MENU_SIGNAL` | `MEM_SAFEHOUSE_DOOR_ID` | Safehouse configuration UI, launches `ShionMem` |
| **Performance Review** | `Shion\ShionMem.txt` | Launched from `ShionMenu` | `MEM_SPOKEN_TOPICS_BIT`, `MEM_PRIORITY_LOCK` | Performance ledger (Kills, Fares, Wrecks, etc.) | Evaluates partnership stats across 9 interactive topics |
| **Sandbox Banter** | `Shion\ShionSandbox.txt`| Water hose, sinking, stunts, civ punches | None | `MEM_SHION_BUSY`, `MEM_CARS_WRECKED`, `MEM_CAR_SMASH_SIGNAL`, `MEM_FLEX_SIGNAL` | Contextual reactions to sandbox actions |
| **Friendly Fire & Cam**| `Shion\ShionFriendlyFire.txt`| Attacking Shion on foot, taking photo | `MEM_PRIORITY_LOCK` | `MEM_PLAYER_FORM`, `MEM_HITODAMA_X/Y/Z` | Reactive dialogue for physical attacks and camera snapshots |
| **Pimping Mission** | `Shion\ShionPimp.txt` | `Y + 2` in 4-door car near prostitute | None | None | Launches native mission 127 (`pimp.sc`) |
| **Hitodama Defense** | `Shion\FX_ShionHitodama.txt`| Shion in Hitodama form (`-1`) | `MEM_HITODAMA_X/Y/Z`, `MEM_SHION_PRES_HP`, `MEM_HITO_DEFLECT_TMR` | `MEM_PLAYER_FORM`, `MEM_JOON_BASE_HP/ARM` | Floats near Jo'on, absorbs bullet damage, grants bike shield |
| **Jo'on Smoking** | `Shion\JoonSmoke.txt` | 10s idle on foot, F11 key | `MEM_JOON_CIGAR_OBJ`, `MEM_PRIORITY_LOCK` | `MEM_PLAYER_FORM`, `MEM_SAVE_SLOT`, `MEM_SHION_HANDLE`, `MEM_SHION_RECRUITED` | Attaches cigarette prop, deducts $1, manages withdrawal lines, gates companion banter to actively recruited Shion |
| **Telemetry (Playtime)**| `Shion\ShionTrack.txt` | 60s timer, Safehouse save | Writes all telemetry to `ShionMemories.ini` | `MEM_SHION_HANDLE`, Performance ledger | Spawns `ShionTrack_World` and `ShionTrack_Missions` |
| **Telemetry (World)** | `Shion\ShionTrack_World.txt`| Driving cars, tuning, radio, casino | `MEM_CASINO_LOSSES`, `MEM_CARS_WRECKED`, `MEM_MOD_CASH_SPENT`, `MEM_FAV_CAR_*` | None | Updates `ShionVehicles.ini` and `ShionRadio.ini` |
| **Telemetry (Jobs)** | `Shion\ShionTrack_Missions.txt`| Sub-missions (Taxi, Medic, Fire, etc.)| `MEM_PARAMEDIC_SAVES`, `MEM_FIREFIGHT_EXTING`, `MEM_VIGILANTE_KILLS`, `MEM_TAXI_FARES`, `MEM_PIMPING_LEVEL`, `MEM_BURGLARIES` | None | Tracks sub-mission milestones with Shion |
| **Hangout Blip Markers**| `Shion\shionHangoutIcon.txt`| Proximity to hangout spots outdoors | None | `MEM_PLAYER_FORM`, `MEM_SHION_RECRUITED` | Manages map skull icons (Marker 23) |
| **Post-Death Complaint**| `Shion\Chat\chatSpawned.txt`| Companion spawned while `MEM_SHION_DEATH_FLAG == 1` | `MEM_SHION_DEATH_FLAG`, `MEM_PRIORITY_LOCK` | `MEM_SHION_DEATH_WEAPON`, `MEM_SHION_KILLER_IS_JOON`, `MEM_SHION_DEATH_IN_CAR` | Voices weapon-specific lethal complaint (including 3-way explosion routing), releases Tier 3 lock |
| **Hangout Reunion**   | `Shion\Chat\chatReunion.txt`| Recruited via `Y` at ambient hangout spot | `MEM_SHION_DEATH_FLAG`, `MEM_REUNION_SIGNAL`, `MEM_PRIORITY_LOCK` | `MEM_SHION_DEATH_WEAPON`, `MEM_SHION_KILLER_IS_JOON`, `MEM_SHION_DEATH_IN_CAR` | Claims Tier 4 theater lock, delivers post-death sister banter or peacetime hangout dialogue |
| **Follow & Transit Vocalizer**| `Shion\Chat\chatFollow.txt`| Triggered by `ShionFollow.c`, `ShionVehicle.c`, or death events (Modes 0–8) | `MEM_PRIORITY_LOCK` (Modes 2 & 3 only) | `MEM_PRIORITY_LOCK`, `MEM_PLAYER_FORM`, `MEM_SHION_STANCE`, `MEM_SHION_KILLER_IS_JOON`, `MEM_SHION_DEATH_WEAPON`, `MEM_SHION_DEATH_IN_CAR` | Delivers low-priority non-locking command barks, seat cycling lines, and Tier 3 stance-aware death reactions |
| **Weather Banter**    | `Shion\Chat\chatWeather.txt`| Streamed by `ShionChat.c` on weather change/timer | `MEM_PRIORITY_LOCK`, `MEM_LAST_WEATHER_TIME` | `MEM_PRIORITY_LOCK`, `MEM_SHION_HANDLE`, `MEM_LAST_WEATHER_TIME` | Reads `[VehicleRoofTypes]` in `ShionVehicles.ini`, classifies open/enclosed cabins, delivers 5-group weather dialogue |
| **Debug Subsystem**   | `Shion\ShionDebug.txt` | Cheats "SHDBG", "KILLPOOR", "HUNGRY", "FOOD", "INJURED", "LORE", "KILLTHEPOOR", "TRIGGERCHAT" | `MEM_SHION_PRES_HP`, `MEM_SHION_PRES_ARMOUR`, `MEM_LAST_FED_HOURS`, `MEM_SHION_HUNGRY`, `MEM_LORE_MENU_SIGNAL` | `MEM_SHION_HANDLE` | Centralized QA harness gated by `DebugMode`, sets HP/armour, triggers hunger, opens lore archive, spawns assassin ped, tests dialogue |