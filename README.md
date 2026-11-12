# YORIGAMI SYNDICATE (GTA San Andreas Mod)
By **EagleRaviOrange** (grotobuvic on Reddit)

<p align="center">
  <img width="572" height="490" alt="image" src="https://github.com/user-attachments/assets/eb2ea791-a90a-47a2-bf23-0f9d18e23606" />
  <br>
  <em>"Even though it's just the two of them..."</em>
</p>

A mod that brings Jo'on and Shion Yorigami from *Touhou Project* into GTA San Andreas as playable characters with complete dialogue and companion systems.

I initially built this to fix the problem of money becoming completely useless in late-game San Andreas, but it ended up spiraling into a full-blown RPG-lite perk tree and an *Inigo*-inspired companion mod with about 3,000 lines of contextual banter.

📺 **[Some clips of the mod (YouTube Playlist)](https://www.youtube.com/playlist?list=PLXo8zqvaGgzU)**  
📖 **[Full In-Depth Documentation & Technical Reference](https://github.com/grotovu/YORIGAMI-SYNDICATE/blob/main/modloader/YORIGAMI%20SYNDICATE/cleo/documentation/Documentation.md)**

---

## ⚙️ Installation

**Prerequisites:**  
You will need a standard downgraded, mod-ready GTA SA setup:
1. **CLEO 4** (or CLEO 5)
2. **CLEO+** (Absolute requirement for scripting opcodes)
3. **Modloader**
4. **EffectLoader.asi** (Must be in your root GTA SA folder so custom particles work)
5. **A limit adjuster** (fastman92limitAdjuster or Open Limit Adjuster)
6. **[Animation Fix: New Car Anim + LCS/VCS Sprinting](https://libertycity.net/files/gta-san-andreas/222557-animation-fix-new-car-anim-lcs-vcs-sprinting.html)** (Required so anime skin meshes sit inside vehicles properly without clipping/distorting, and includes Stories-style sprinting animations)
7. **Essentials Pack / SilentPatch** (Recommended standard setup)

---

### 📥 First-Time Install (New Players)
1. **Initial Setup:** Open the `_FIRST TIME INSTALL (New Players)` folder and drag the `modloader` folder inside it directly into your root GTA San Andreas directory.  
   *(This configures required default `.ini` settings, base structure, and initial data folders).*
2. **Main Mod Content:** Drag the main `modloader` folder from the release zip into your root GTA San Andreas directory.
3. **Important (Particle Effects):** Drag the `models` folder (containing `models/effects/joon_gold` and `models/effects/shion_misfortune`) straight into your base GTA San Andreas directory.  
   *Note: EffectLoader does not read from Modloader, so custom particle textures must be placed directly into your base game's `models/effects` folder.*
4. **EffectLoader:** Ensure `EffectLoader.asi` is in your main GTA SA root directory if you do not have it already.
5. **Recommended (`SkinArmor`):** In `_Optional Tweaks`, drop the `SkinArmor` folder straight into your `modloader`. This prevents anime player models from warping into body-stretched monstrosities during missions and cutscenes that force clothing changes on CJ (such as Burglary, 555 We Tip, Key to Her Heart, etc.).

---

### 🔄 Updating from an Earlier Version
1. Drag the `modloader` (and `models` if effects were updated) folder into your root GTA San Andreas directory.
2. Choose **"Replace files in the destination"** when prompted.
3. Check the `_UPDATED INIs` folder: copy any new keys or updated configuration settings into your existing `.ini` files located in `modloader/YORIGAMI SYNDICATE/cleo/` so you do not miss out on new features.
4. Check the `_Optional Tweaks` folder for new scripts or optional patches.
5. **Your Saves are Safe:** Active saves, perk points, and memory ledgers inside `cleo/savedata/` remain 100% intact.

---

## 🎮 Controls Quick Reference
*(Almost all of these can be changed in `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`)*

* **Cheat Codes:**
  * Type `JOON` — Swap to Jo'on
  * Type `POVERTY` — Swap to Shion _(you're all alone... at least there's no one stopping you from rummaging dumpsters.)_
  * Type `SHION` — Spawn Shion as a companion (if you're playing as Jo'on)
  * Type `BRIBE` — Pay off your wanted level using cash
* **Hotkeys:**
  * **Hold `Y` + `L`** — Instant swap between Jo'on and Shion (Perfect Possession)
  * **Hold `Y` + `H`** — Toggle Shion between physical ped and floating Hitodama (ghost) form
  * **Hold `Y` + `B`** — Toggle Shion stance (Peaceful / Hostile)
  * **`H`** — Stay put / hold ground on foot (or stay inside vehicle when dismounting)
  * **`G`** — Recall Shion (single-tap to break combat lock, double-tap to warp on foot; warp into vehicle or swap passenger seats while driving)
  * **Hold `Y` + `M`** — Open the Prestige perk menu  
    *(Press **`SPACE`** while hovering over any perk for an in-depth gameplay guide & controls)*
  * **Hold `Y` + `J`** — Didier Sachs Flex (throw money to buy Y-Points directly)
  * **Hold `Y` + `G`** — Toggle Money Danmaku (Jo'on)
  * **Hold `Y` + `C`** — Cast Pestilence Fever (Jo'on)
  * **Hold `Y` + `Z`** — Tell Shion to scavenge nearby trash bins / dumpsters
  * **Hold `Y` + `R`** — Swap weapons and ammo back and forth with Shion
  * **Hold `Y` + `T`** — Talk to Shion (triggers contextual banter on command)
  * **Hold `TAB`** — Contextual interactions (open Shion's settings in safehouses, order at diners, or hand off burglary loot to Shion)

---

## 🛠️ What This Mod Actually Does

<p align="center">
  <img width="484" height="250" alt="image" src="https://github.com/user-attachments/assets/f23e2d34-89fa-4d26-a7c7-567127126f76" />
</p>

### 1. Playing as the Sisters & The Companion System
* You can play as CJ, Jo'on, or Shion.
* If you play as Jo'on, Shion acts as a follower. To fix GTA's notoriously bad follower AI, she can turn into a Hitodama (a floating blue orb). As a ghost, she floats over your shoulder, acts as a bullet-deflecting shield, and warps into the car (press `G` or drive away at a certain distance) with you so she never gets lost or stuck on fences.
* You can swap guns with her (which runs out of ammo, make sure to restock), tell her to haul your stolen burglary items, and have her take hits for you.

### 2. Yorigami Prestige (The Sink for Useless Late-Game Cash)
Every $10,000 you spend or lose in the game earns you 1 **Y-Point** (praying to the gods through reckless spending). You allocate these in the `Y+M` menu to level up perks:

* **Jo'on (Greed & Wealth):**
  * *Jo'on's Bling:* Fists damage boost. Locking onto someone and punching lets you do a fast gap-closing dash with 2 seconds of bullet immunity ("graze").
  * *Vehicle Smasher:* Lets you punch cars until they blow up, _because cars are nothing but containers of petroleum._
  * *Taxi Extortion:* Vastly increases the payout of Taxi sub-mission fares. (Limitless scaling. Once you're maxed out, dump your extra Y-Points here for stupid amounts of money extorted per fare.)
  * *Midas Magnet:* Pulls cash pickups straight to you from far away. No more running to pick up money. _Wealth naturally comes to you._
  * *Drip Worship:* Raises sex appeal and charms civilians, drug dealers, and rival gang members into your entourage (`Aim` + `G`).
  * *Street Shakedown:* Rob pedestrians on foot; marks rich targets on the map for extra cash (`Aim` + `Y`).
  * *Pestilence Fever:* An active skill that forces nearby pedestrians to drop their cash and panic (`Y` + `C`).
  * *Money Danmaku:* Pay-to-win shooting. Your guns cost actual cash per shot from your wallet, but shred armor and vehicles instantly (`Y` + `G` to toggle).

<p align="center">
  <img width="742" height="523" alt="image" src="https://github.com/user-attachments/assets/d486bf28-0b00-47dc-a2e2-4b66d4712a1e" />
  <br>
  <em>"Extortioner of the 80s biiiiiiihh"</em>
</p>

* **Shion (Misfortune & Poverty):**
  * *Scavenger Feast:* Shion will dig through bins and dumpsters to pull out food or discarded weapons (Tec-9s, Shotguns, Molotovs) (`F` near dumpsters, type `ADDBIN` to register custom props).
  * *Shion Resonance:* If you're completely broke ($0 or in debt) and on foot, cops will ignore 1-star wanted levels after 6 seconds—provided you are **unarmed or only using brass knuckles**. Pulling out a weapon immediately blows your cover.
  * *Misfortune Aura:* Enemies near Shion randomly trip over their own feet or suffer sudden weapon jams.
  * *Traffic Hex & Aviation Calamity:* Chasing cop cars stall out and pop tires; pursuit police choppers and Hydra jets spontaneously explode in mid-air.
  * *Debt Detonation:* Dropping low on health sets off a shockwave that knocks everyone down and disarms them.
  * *Grim Pity:* If you take lethal damage while completely flat broke, Shion's curse cancels your death, heals you, and gives temporary invulnerability. (Requires having $0 or in debt.)
  * *Misery Loves Company:* Reflects incoming damage back to whoever shot you. (Requires having $0 or in debt. Scalable over 100% for lethal retribution.)

<p align="center">
  <img width="622" height="605" alt="image" src="https://github.com/user-attachments/assets/da975a93-4b9c-43f0-8292-a46af8b3fdd7" />
  <br>
  <em>"She didn't do anything..."</em>
</p>

* **Yorigami Contract:**
  * Instantly plunge yourself into $1,000,000 of crippling debt in exchange for a massive payout of Y-Points. Perfect for instantly activating Shion's poverty-reliant perks (or just ruining your own life).
  * _You can configure this up to 1,000 Y-Points for $10,000,000 debt in the menu or INI if you want to max out all the perks and suffer the consequences..._

<p align="center">
<img width="1264" height="706" alt="image" src="https://github.com/user-attachments/assets/fe8860a6-c714-4ac2-96b6-b8497e4486f2" />
  <br>
  <em>"When our balance is zero, you're in my domain, Jo'on."</em>
</p>

### 3. Food, Hunger & Wallet Drain
* Shion gets hungry every two in-game days. You can pull into drive-thrus (Burger Shot, Cluckin' Bell), stop by diners, or hit up street carts to feed her. If you don't have cash, she'll beg vendors for scraps or look through the garbage. (Type `ADDFOOD` to register custom food locations).
* Optional combat resource — `Wallet Drain`: Shion's shooting drains your wallet. Instead of being limited to physical ammo, she fires until you're broke, paying per bullet. If you run out of money, she gets disarmed until you earn cash. (Type `DURGASTRA` to enable limitless debt mode for Wallet Drain).
* Idle cigarette smoking for Jo'on (Press `F11` to smoke on command. Costs $1 by default. Fully configurable in `Shion.ini`).

### 4. Ambient Banter & Lore (The 3,000+ Lines)
A massive pool of contextual dialogue reactive to what you do:
* **Side-missions:** Unique reactions for Vigilante, Paramedic, Firefighter, Taxi, Pimping, Burglary, and Stadium events.
* **Radio:** Dedicated commentaries for every single radio station in the game.
* **World awareness:** Specific dialogue depending on where you are (Rodeo Drive, the desert, hoods, inside clothing stores, gun shops, casinos).
* **Physics/Driving:** Shion complains if you drive like a maniac, do flips, end up underwater, or punch random civilians.
* **Sister Talk (31 Lore Episodes):** Unlocked through milestones (spending money, surviving on $0, doing paramedic jobs, hitting Mt. Chiliad, etc.). Replayable anytime via the in-game Lore Archive menu.

### 5. Persistent Tracker (`ShionMemories.ini`)
The mod saves lifetime stats per save slot in the `savedata/` folder:
* Tracks time spent broke, favorite radio stations, odometer per vehicle model, casino losses, cars wrecked, and total cash spent feeding Shion.

---

## 📖 Technical Documentation

For developers, modders, or anyone who wants an exhaustive look under the hood:
* Check out the **[Full Mod Documentation & Memory Map](https://github.com/grotovu/YORIGAMI-SYNDICATE/blob/main/modloader/YORIGAMI%20SYNDICATE/cleo/documentation/Documentation.md)** for detailed breakdowns on:
  * Master Heap layout & IPC architecture (`Y_Brain`).
  * Dialogue priority channel arbitration (`MEM_PRIORITY_LOCK`).
  * Complete subsystem interactions, register safety maps, and INI keys.

---

## ⚙️ Configuration & Savegame Safety

### 🎮 Customization & Balance Knobs
If you want to rebind hotkeys, adjust perk caps, change scan intervals (`ScanIntervalSeconds`), disable hunger, or toggle wallet drain, open up `cleo/Shion.ini` and `cleo/YorigamiPerks.ini`. Every setting is documented with plain-English comments.

### 💾 Zero Save Corruption & Data Isolation
* **Safe for Existing Saves:** Nothing from this mod is baked into your vanilla GTA San Andreas save files (`GTASAsf*.b`). No vanilla globals or script tables are modified in ways that cause savegame bloat or corruption.
* **Complete Data Isolation:** Mod progression, perk points, and sister banter milestones are saved strictly in the mod's own `savedata/` folder, completely segregated from the base game's save system.
* **Safe on 100% Saves:** You can safely load this mod on a fresh game or an existing 100% completion save file without polluting CJ's base statistics. If you ever uninstall the mod, your base saves remain completely clean.
* **Open & Editable:** All saved data is stored in plain-text `.ini` files that you can inspect or modify at will.

---

## 📜 Credits & Attributions

* **ZUN / Team Shanghai Alice & Twilight Frontier** — *Touhou Project* and the Yorigami Sisters (*Touhou 15.5: Antinomy of Common Flowers*, *Touhou 17.5: Sunken Fossil World*).
* **schwarz** — Original Jo'on & Shion MMD Models:
  * [Yorigami Jo'on MMD Model](https://3d.nicovideo.jp/works/td32247)
  * [Yorigami Shion MMD Model](https://3d.nicovideo.jp/works/td31923)
* **tudhsbf123** — Jo'on MMD model port to GTA SA.
* **Misaki Yutsuki** — Shion MMD model port to GTA SA.
* **WarButler, DK22Pac, Junior_Djjr, Ezekiel, Function-X** — *CombatFX* (Base shootlight FX modified into custom `joon_gold` and `shion_misfortune` particle systems via texture and keyvalue edits).
* **Lighthoneygirl** — *Smoking Mod* (The original F11 smoking mod that served as a reference for playing secondary animations and cigarette mechanics).
* **Artem.1.9.9.6** — _Busy Pedestrians: Peds Events Project_ (the Para Jumper.txt as reference for skydiving and parachute related scripts).

---

## 🛠️ Development & Transparency

A lot of the coding, CLEO opcode debugging, and string/data table formatting throughout this development process was heavily AI-assisted. 

For the dialogue and narrative: I provided the specific character context, lore, and desired tone (referencing source material and the Yorigami sisters' established dynamics). I used AI as an assistive tool to generate initial dialogue drafts, which were then **heavily reviewed, edited, and curated by me**. As an ESL (English as a Second Language) developer, these tools helped bridge linguistic gaps to ensure the rough, abrasive, and distinct slang of the characters was localized accurately for an English-speaking audience. I personally revised every block of dialogue to ensure it adhered to my specific vision for Jo'on and Shion and to eliminate any OOC (out-of-character) behavior. 

All creative direction, systems architecture, mechanical balance (the prestige tree, companion logic, economy sinks), and dialogue concepts were directed and finalized by **EagleRaviOrange**.

---

## 📖 Further Reading & Design Notes
**Note:** These links leads to my personal blog. It contains unfiltered analysis and embraces the darker, messy, and taboo elements of Japanese doujin culture. Proceed accordingly.
* [An explanation for Jo'on's dialogue:](https://eagleraviorange.bearblog.dev/explanation-and-ramblings-about-joons-dialogue-in-my-mod-yorigami-syndicate/) 
  *Ramblings about linguistic inspirations, localizing Japanese Shitamachi speech patterns into English.*
* [A text regarding Jo'on Yorigami's ability to adapt to the current era and environment, and what that means in a criminal sandbox game](https://eagleraviorange.bearblog.dev/a-text-regarding-joon-yorigamis-ability-to-adapt-to-the-current-era-and-environment-and-what-that-means-in-a-criminal-sandbox-game/)
* [A text regarding the thought process on building Shion Yorigami as a companion in GTA San Andreas](https://eagleraviorange.bearblog.dev/a-text-regarding-the-thought-process-on-building-shion-yorigami-in-gta-san-andreas/)

---

*Touhou Project belongs to ZUN / Team Shanghai Alice. Mod is open-source and free on GitHub.*
