# YORIGAMI SYNDICATE (GTA San Andreas Mod)
By **EagleRaviOrange** (grotobuvic on Reddit)

<p align="center">
  <img width="572" height="490" alt="image" src="https://github.com/user-attachments/assets/eb2ea791-a90a-47a2-bf23-0f9d18e23606" />
  <br>
  <em>"Even though it's just the two of them..."</em>
</p>

A mod that brings Jo'on and Shion Yorigami from *Touhou Project* into GTA San Andreas as playable characters with complete dialogue and companion systems.

I initially built this to fix the problem of money becoming completely useless in late-game San Andreas, but it ended up spiraling into a full-blown RPG-lite perk tree and an *Inigo*-inspired companion mod with about 3,000 lines of contextual banter.

---

## ⚙️ Installation

**Prerequisites:**
You’ll need a standard downgraded, mod-ready GTA SA setup:
1. **CLEO 4** (or CLEO 5)
2. **CLEO+** (Absolute requirement for the script opcodes)
3. **Modloader** 
4. **EffectLoader.asi** (Put this in your root GTA SA folder so the custom particles work)
5. A limit adjuster (I personally use fastman92limitAdjuster, Open Limit Adjuster works too)

**How to Install:**
1. Grab the latest release zip.
2. Put `EffectLoader.asi` in your main GTA SA directory if you don't already have it.
3. Drop the `YORIGAMI SYNDICATE` folder into your `modloader` folder (this loads the scripts, character models, etc.).
4. **Important:** Merge the `models/effects` folder from the zip into your game's base `models` directory. *EffectLoader does not read from Modloader*, so the custom particle effects for Jo'on's gold and Shion's misfortune must be placed directly into your base game's `models/effects` folder.
5. That's pretty much it. Check the `optional` folder if you want extra compatibility stuff like taxi boarding, high-stakes bets, or quick fist swaps.

---

## 🎮 Controls
*(Almost all of these can be changed in `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`)*

* **Cheat Codes:**
  * Type `JOON` - Swap to Jo'on
  * Type `POVERTY` - Swap to Shion _(you're all alone... at least there's no one stopping you from rummaging dumpsters.)_
  * Type `SHION` - Spawn Shion as a companion (if you're playing as Jo'on)
  * Type `BRIBE` - Pay off your wanted level using cash
* **Hold `Y` + `L`** - Instant swap between Jo'on and Shion (Perfect Possession)
* **Hold `Y` + `H`** - Toggle Shion between physical ped and floating Hitodama (ghost) form
* **Hold `Y` + `M`** - Open the Prestige perk menu
* **Hold `Y` + `J`** - Didier Sachs Flex (throw money to build Y-Points)
* **Hold `Y` + `G`** - Toggle Money Danmaku (Jo'on)
* **Hold `Y` + `C`** - Cast Pestilence Fever (Jo'on)
* **Hold `Y` + `Z`** - Tell Shion to scavenge nearby trash bins / dumpsters
* **Hold `Y` + `R`** - Swap weapons and ammo back and forth with Shion
* **Hold `Y` + `T`** - Talk to Shion (triggers contextual banter on command)
* **Hold `TAB`** - Contextual interactions (open Shion's settings in safehouses, order at diners, or hand off burglary loot to Shion)

---

## 🛠️ What This Mod Actually Does

<p align="center">
  <img width="484" height="250" alt="image" src="https://github.com/user-attachments/assets/f23e2d34-89fa-4d26-a7c7-567127126f76" />
</p>

### 1. Playing as the Sisters & The Companion System
* You can play as CJ, Jo'on, or Shion.
* If you play as Jo'on, Shion acts as a follower. To fix GTA's notoriously bad follower AI, she can turn into a Hitodama (a floating blue orb). As a ghost, she floats over your shoulder, acts as a bullet-deflecting shield, and warps into the car (press G or drive away at a certain distance) with you so she never gets lost or stuck on fences.
* You can swap guns with her (which runs out of ammo, make sure to restock), tell her to haul your stolen burglary items, and have her take hits for you.

### 2. Yorigami Prestige (The Sink for Useless Late-Game Cash)
Every $10,000 you spend or lose in the game earns you 1 **Y-Point** (praying to the gods through reckless spending). You allocate these in the `Y+M` menu to level up perks:

* **Jo'on (Greed & Wealth):**
  * *Jo'on's Bling:* Fists damage boost. Locking onto someone and punching lets you do a fast gap-closing dash with 2 seconds of bullet immunity ("graze").
  * *Vehicle Smasher:* Lets you punch cars until they blow up, _because cars are nothing but containers of petroleum._
  * *Taxi Extortion:* Vastly increases the payout of Taxi sub-mission fares. (Limitless scaling. Once you're maxed out, dump your extra Y-Points here for stupid amounts of money extorted per fare.)
  * *Midas Magnet:* Pulls cash pickups straight to you from far away. No more running to pick up money. _Wealth naturally comes to you._
  * *Drip Worship:* Raises sex appeal and charms civilians and rival gang members into your entourage (`Aim`+`G`).
  * *Street Shakedown:* Rob pedestrians on foot; marks rich targets on the map for extra cash (`Aim`+`Y`).
  * *Pestilence Fever:* An active skill that forces nearby pedestrians to drop their cash and panic (`Y`+`C`).
  * *Money Danmaku:* Pay-to-win shooting. Your guns cost actual cash per shot from your wallet, but shred armor and vehicles instantly (`Y`+`G` to toggle).

<p align="center">
  <img width="742" height="523" alt="image" src="https://github.com/user-attachments/assets/d486bf28-0b00-47dc-a2e2-4b66d4712a1e" />
  <br>
  <em>"Extortioner of the 80s biiiiiiihh"</em>
</p>

* **Shion (Misfortune & Poverty):**
  * *Scavenger Feast:* Shion will dig through bins and dumpsters to pull out food or discarded weapons (Tec-9s, Shotguns, Molotovs) (`F` near dumpsters, type `ADDBIN` to register a dumpster if the object is a static prop or addition in another mod).
  * *Shion Resonance:* If you're completely broke ($0), cops will ignore 1-star wanted levels out of pure pity.
  * *Misfortune Aura:* Enemies near Shion randomly trip over their own feet.
  * *Traffic Hex & Aviation Calamity:* Chasing cop cars stall out, pop tires, and police choppers can spontaneously explode mid-air.
  * *Debt Detonation:* Dropping low on health sets off a shockwave that knocks everyone down and disarms them.
  * *Grim Pity:* If you take lethal damage while completely flat broke, Shion's curse cancels your death, heals you, and gives temporary invulnerability. (Requires having $0 or in debt.)
  * *Misery Loves Company:* Reflects a chunk of incoming damage back to whoever shot you. (Requires having $0 or in debt.)

<p align="center">
  <img width="622" height="605" alt="image" src="https://github.com/user-attachments/assets/da975a93-4b9c-43f0-8292-a46af8b3fdd7" />
  <br>
  <em>"She didn't do anything..."</em>
</p>

* **Yorigami Contract:**
  * Instantly plunge yourself into $1,000,000 of crippling debt in exchange for a massive payout of Y-Points. Perfect for instantly activating Shion's poverty-reliant perks (or just ruining your own life).
  * _You can configure this up to 1,000 Y-Points for $10,000,000 debt if you're impatient like me and want to max out all the perks and suffer the consequences..._

<p align="center">
<img width="1264" height="706" alt="image" src="https://github.com/user-attachments/assets/fe8860a6-c714-4ac2-96b6-b8497e4486f2" />
  <br>
  <em>"When our balance is zero, you're in my domain, Jo'on."</em>
</p>

### 3. Food, Hunger & Wallet Drain
* Shion gets hungry every two in-game days. You can pull into drive-thrus (Burger Shot, Cluckin' Bell), stop by diners, or hit up street carts to feed her. If you don't have cash, she'll beg vendors for scraps or look through the garbage. (Type `ADDFOOD` to register custom food locations, which are saved in the savedata folder).
* Optional combat resource - `Wallet Drain`: Shion's shooting drains your wallet. Instead of being limited to the ammo you gave her, she can fire until you're broke, retroactively paying per bullet. If you run out of money, her gun disappears until you get cash again. (Type `DURGASTRA` to enable limitless debt mode for Wallet Drain, or enable `DURGASTRA` mode in Shion's Settings Menu in-game).
* Idle cigarette smoking for Jo'on (Press `F11` to smoke on command. Costs $1 by default. Can be configured and disabled in `Shion.ini`).

### 4. Ambient Banter & Lore (The 3,000+ Lines)
I wrote a ton of contextual dialogue based on what you're doing in the game:
* **Side-missions:** Unique reactions for Vigilante, Paramedic, Firefighter, Taxi, Pimping, Burglary, and Stadium events.
* **Radio:** Commentaries for every single radio station in the game.
* **World awareness:** Specific dialogue depending on where you are (Rodeo Drive, the desert, hoods, inside clothing stores, gun shops, casinos).
* **Physics/Driving:** Shion complains if you drive like a maniac, do flips, end up underwater, or punch random people.
* **Sister Talk (31 Lore Episodes):** Unlocked through in-game milestones (spending money, surviving on $0, doing paramedic jobs, hitting Mt. Chiliad, etc.). You can read and replay them anytime via the Lore Archive menu.

### 5. Persistent Tracker (`ShionMemories.ini`)
The mod saves lifetime stats in the savedata folder, and each save has its own memory:
* Tracks things like total time spent broke, favorite radio station, favorite car model, casino losses, cars wrecked, and total cash spent feeding Shion.

---

## ⚙️ Tweaking Stuff
If you want to rebind keys, change how much money 1 Y-Point costs, disable the hunger mechanic, or turn off the wallet drain, open up `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`. Everything is commented and highly configurable.

CJ's base game stats are tracked completely separately from the experiences accumulated by Jo'on and Shion. So you can play on a 100% save and Shion acts as if Jo'on has yet to accomplish anything.

There is nothing encrypted in the savedata folder. You may alter as you wish.

---

## 🛠️ Development & Transparency
A lot of the coding, CLEO opcode debugging, and string/data table formatting throughout this 60-day development process was heavily AI-assisted.

For the dialogue and narrative: I provided the specific character context, lore, and desired tone (referencing source material and the Yorigami sisters' established dynamics). I used AI as an assistive tool to generate initial dialogue drafts, which were then heavily reviewed, edited, and curated by me. As an ESL (English as a Second Language) developer, these tools helped bridge linguistic gaps to ensure the rough, abrasive, and distinct slang of the characters was localized accurately for an English-speaking audience. I personally revised every block of dialogue to ensure it adhered to my specific vision for Jo'on and Shion and to eliminate any OOC (out-of-character) behavior.

All creative direction, systems architecture, mechanical balance (the prestige tree, companion logic, economy sinks), and dialogue concepts were directed and finalized by **EagleRaviOrange**.

---
*Touhou Project belongs to ZUN / Team Shanghai Alice. Mod is open-source and free on GitHub.*
