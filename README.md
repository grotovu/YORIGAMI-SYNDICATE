# YORIGAMI SYNDICATE (GTA San Andreas Mod)
By **EagleRaviOrange** (grotobuvic on Reddit)

A mod that brings Jo'on and Shion Yorigami from *Touhou Project* into GTA San Andreas as playable characters and a fully voiced (via subtitles) companion system. 

I initially built this to fix the problem of money becoming completely useless in late-game San Andreas, but it ended up spiraling into a full-blown RPG-lite perk tree and an *Inigo*-inspired companion mod with over 3,000+ lines of contextual banter.

---

## ⚙️ Installation

**Prerequisites:**
You’ll need a standard downgraded, mod-ready GTA SA setup:
1. **CLEO 4** (or CLEO 5)
2. **CLEO+** (Absolute requirement for the opcodes I used)
3. **Modloader** 
4. **EffectLoader.asi** (Put this in your root GTA SA folder so the custom particles work)
5. A limit adjuster (I personally use fastman92limitAdjuster, Open Limit Adjuster works too)

**How to Install:**
1. Grab the latest release zip.
2. Put `EffectLoader.asi` in your main GTA SA directory if you don't already have it.
3. Drop the `YORIGAMI SYNDICATE` folder into your `modloader` folder.
4. Merge the `models/effects` folder into your game's `models` directory (these are the custom particle effects for Jo'on's gold and Shion's misfortune).
5. That's pretty much it. Check the `optional` folder if you want extra compatibility stuff like taxi boarding, high-stakes bets, or quick fist swaps.

---

## 🎮 Controls
*(Almost all of these can be changed in `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`)*

* **Cheat Codes:**
  * Type `JOON` - Swap to Jo'on
  * Type `POVERTY` - Swap to Shion
  * Type `SHION` - Spawn Shion as a companion (if you're playing as Jo'on)
* **Hold `Y` + `L`** - Instant swap between Jo'on and Shion (Perfect Possession)
* **Hold `Y` + `H`** - Toggle Shion between physical ped and floating Hitodama (ghost) form
* **Hold `Y` + `M`** - Open the Prestige perk menu
* **Hold `Y` + `J`** - Didier Sachs Flex (throw money to build Y-Points)
* **Hold `Y` + `G`** - Toggle Money Danmaku (Jo'on)
* **Hold `Y` + `C`** - Cast Pestilence Fever (Jo'on)
* **Hold `Y` + `Z`** - Tell Shion to scavenge nearby trash bins / dumpsters
* **Hold `Y` + `R`** - Swap weapons and ammo back and forth with Shion
* **Hold `Y` + `T`** - Talk to Shion (triggers contextual banter or lore episodes)
* **Hold `TAB`** - Contextual interactions (eat at food carts/diners, or hand off burglary loot to Shion)

---

## 🛠️ What This Mod Actually Does

### 1. Playing as the Sisters & The Companion System
* You can play as CJ, Jo'on, or Shion.
* If you play as Jo'on, Shion acts as a follower. To fix GTA's notoriously bad follower AI, she can turn into a Hitodama (a floating blue orb). As a ghost, she floats over your shoulder, acts as a bullet-deflecting shield, and automatically clips into the car with you so she never gets lost or stuck on fences.
* You can swap guns with her (which runs out), tell her to haul your stolen burglary items, and have her take hits for you.

### 2. Yorigami Prestige (The Sink for Useless Late-Game Cash)
Every $10,000 you spend or lose in the game earns you 1 **Y-Point** (praying to the gods through reckless spending). You spend these in the `Y+M` menu to level up perks:

* **Jo'on (Greed & Wealth):**
  * *Jo'on's Bling:* Huge melee damage boost. Locking onto someone and punching lets you do a fast gap-closing dash with 2 seconds of bullet immunity ("graze").
  * *Vehicle Smasher:* Lets you punch cars until they blow up, eventually making you blast-proof.
  * *Midas Magnet:* Pulls cash pickups straight to you from far away.
  * *Drip Worship:* Charms civilians and rival gang members into your entourage based on your sex appeal.
  * *Street Shakedown:* Rob pedestrians on foot; marks rich targets on the map for extra cash.
  * *Pestilence Fever:* An active skill that forces nearby pedestrians to drop their cash and panic.
  * *Money Danmaku:* Pay-to-win shooting. Your guns cost actual cash per shot from your wallet, but shred armor and vehicles instantly.

* **Shion (Misfortune & Poverty):**
  * *Scavenger Feast:* Shion will dig through bins and dumpsters to pull out food or discarded weapons (Tec-9s, Shotguns, Molotovs).
  * *Shion Resonance:* If you're completely broke ($0), cops will ignore 1-star wanted levels out of pure pity.
  * *Misfortune Aura:* Enemies near Shion have their guns jam and randomly trip over their own feet.
  * *Traffic Hex & Aviation Calamity:* Chasing cop cars stall out, pop tires, and police choppers can spontaneously explode mid-air.
  * *Debt Detonation:* Dropping low on health sets off a shockwave that knocks everyone down and disarms them.
  * *Grim Pity:* If you take lethal damage while completely flat broke, Shion's curse cancels your death, heals you, and gives temporary invulnerability.
  * *Misery Loves Company:* Reflects a chunk of incoming damage back to whoever shot you.

### 3. Food, Hunger & Wallet Drain
* Shion gets hungry every two in-game days. You can pull into drive-thrus (Burger Shot, Cluckin' Bell), stop by diners, or hit up street carts to feed her. If you don't have cash, she'll beg vendors for scraps or look through the garbage.
* Optional setting: Shion's shooting drains your wallet. If you run out of money, her guns disappear until you get cash again.
* Custom idle cigar smoking for Jo'on, and eating/scavenging animations for Shion.

### 4. Ambient Banter & Lore (The 3,000+ Lines)
I wrote a ton of contextual dialogue based on what you're doing in the game:
* **Side-missions:** Unique reactions for Vigilante, Paramedic, Firefighter, Taxi, Pimping, Burglary, and Stadium events.
* **Radio:** Commentaries for every single radio station in the game.
* **World awareness:** Specific dialogue depending on where you are (Rodeo Drive, the desert, hoods, inside clothing stores, gun shops, casinos).
* **Physics/Driving:** Shion complains if you drive like a maniac, do flips, end up underwater, or punch random people.
* **Sister Talk (31 Lore Episodes):** Unlocked through in-game milestones (spending money, surviving on $0, doing paramedic jobs, hitting Mt. Chiliad, etc.). You can read and replay them anytime via the Lore Archive menu.

### 5. Persistent Tracker (`ShionMemories.ini`)
The mod saves lifetime stats that carry over between saves:
* Tracks things like total time spent broke, favorite radio station, favorite car model, casino losses, cars wrecked, and total cash spent feeding Shion.

---

## ⚙️ Tweaking Stuff
If you want to rebind keys, change how much money 1 Y-Point costs, disable the hunger mechanic, or turn off the wallet drain, open up `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`. Everything is commented and configurable.

---
*Touhou Project belongs to ZUN / Team Shanghai Alice. Mod is open-source and free on GitHub.*