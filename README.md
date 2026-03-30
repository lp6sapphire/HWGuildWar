# ⚔️ Guild War Planner — Hero Wars: Alliance

A standalone HTML tool for planning Guild War defense and attack assignments. No server, no install — open the `.html` file in any browser.

---

## Features

### Roster
- Up to **20 players**, each with hero power and titan power (in K)
- Per-player flags:
  - **E5** — titan team is Epoch 5 (default: on). E5 titans get an attack boost; non-E5 get a penalty.
  - **Risk** — unreliable player (may not attack). Shown as 🚧 in the attack plan, excluded from guaranteed-capture calculations.
- Sort roster by overall / hero power / titan power
- Enter key navigation: names flow independently, powers flow independently (Tab or click to switch streams)

### 🛡 Our Defense tab
- Assign players to all 10 fortification slots via dropdowns
- **Auto-distribute defense** button: distributes using round-robin by building priority (bonus desc), ensuring every building gets a strong player in its first slot before any building gets a second
- When a player is reassigned, their old slot highlights red in the roster, new slot highlights green — clears after 2 seconds

### 🗡 Enemy Defense tab
- Enter enemy defender power per slot (no names needed)
- Enter key navigates through all slots across all buildings in order
- **Calculate attack plan** button activates once at least one slot is filled

### 📊 Attack Plan tab

#### Assignment by player
Table showing each player's 2 attacks:
- Building name (Russian) + team icon
- Attacker power vs defender power + slot number
- **WIN** — guaranteed (attacker power > defender power)
- **🎲 GAMBLE** — E5 titan with boost applied, may win against stronger opponent
- **⚠️** — non-E5 titan (penalized power)
- **🚧** — risk player (50/50 reliability)
- Points earned per player

#### Assignment by building
Per-building breakdown with defender power, assigned attacker, and result per slot. Buildings highlighted:
- 🟢 Captured (all slots won)
- 🟡 Partial (some slots won)
- 🔴 No wins

#### Copy-paste block
Plain text summary — by player and by building — for Telegram or Discord.

---

## Building Map

| Tier | Building | Type | Slots | Capture Bonus |
|------|----------|------|-------|---------------|
| 1 | Bridge / Мост | ⚡ Titans | 4 | +60 |
| 1 | Barracks / Казармы | ⚔️ Heroes | 3 | +40 |
| 1 | Lighthouse / Маяк | ⚔️ Heroes | 3 | +40 |
| 1 | Mage Academy / Академия Магов | ⚔️ Heroes | 3 | +40 |
| 2 | Foundry / Литейная | ⚔️ Heroes | 4 | +60 |
| 2 | Bastion of Fire / Бастион Огня | ⚡ Titans | 4 | +60 |
| 2 | Gates of Nature / Врата Природы | ⚡ Titans | 4 | +60 |
| 2 | Bastion of Ice / Бастион Льда | ⚡ Titans | 4 | +60 |
| 2 | Spring of Elements / Ист. Стихий | ⚡ Titans | 4 | +60 |
| 3 | Citadel / Цитадель | ⚔️ Heroes | 7 | +120 |

**Unlock chain:**
- Tier 2 unlocks after Bridge is captured **or** Ghost Bridge is active (Barracks + Lighthouse + Mage Academy all captured)
- Tier 3 (Citadel) unlocks after capturing any one of: Bastion of Fire, Gates of Nature, Bastion of Ice

Ghost Bridge is always assumed active in this tool (checked in header).

---

## Attack Rules (Gold League)

- Each player has **2 attack attempts** total per war day
- The same player can defend and attack simultaneously — defense and offense are independent
- Hero teams attack hero buildings; titan teams attack titan buildings
- **E5 titan boost** (default 10%): E5 titans can beat a slightly stronger defender in gamble mode
- **Non-E5 penalty** (default 10%): non-E5 titans have reduced effective power
- **Risk player**: included in plan but flagged — treat as 50% chance to actually attack

---

## Settings (header bar)

| Setting | Description |
|---------|-------------|
| 👻 Ghost Bridge | Always active — Tier 2 is always unlocked |
| Pts per fight win | Points awarded per won battle (default: 20) |
| ⚡ E5 boost % | Attack power multiplier for E5 titans in gamble mode |
| ⚡ non-E5 penalty % | Attack power reduction for non-E5 titans |

---

## Score Bar

| Metric | Meaning |
|--------|---------|
| Safe wins | Guaranteed wins (attacker > defender, no boost) |
| +N 🎲 | Additional gamble wins (E5 boost applied) |
| ×pts | Fight wins × pts per win |
| Bonus | Sum of capture bonuses for fully captured buildings |
| TOTAL | Fight pts + capture bonus |
| Captures | Number of fully captured buildings |

---

## Save / Load

- **Auto-save**: all data is saved to browser localStorage automatically
- **Export**: saves full state to a `.json` file (roster, defenses, enemy data, settings)
- **Import**: loads a previously exported `.json`
- **Reset war**: clears all defense and enemy assignments while keeping the roster

---

## Workflow

```
1. Add players to roster (name, hero power K, titan power K)
   └── Mark E5 (titans) and Risk (unreliable) flags

2. 🛡 Our Defense tab
   └── Click "Auto-distribute defense" or assign manually

3. 🗡 Enemy Defense tab
   └── Enter enemy slot powers (Enter key moves between slots)
   └── Click "Calculate attack plan"

4. 📊 Attack Plan tab
   └── Review assignments by player and by building
   └── Copy-paste summary into Telegram/Discord
```

---

## File format

Exported JSON contains: `roster`, `ourDef`, `enmDef`, `ghostBridge`, `fightPts`, `titanBoost`, `titanPenalty`, `nextId`.

Powers are stored in **K units** (e.g. `450` = 450K).
