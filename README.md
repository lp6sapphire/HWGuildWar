# ⚔️ GW Planner — Hero Wars: Alliance (Mobile)

Standalone HTML tool for Guild War attack planning. No server, no install — open the `.html` file in any browser.

---

## Quick Start

```
1. Import JSON or fill roster manually (name, hero K, titan K, E5, risk)
2. Enter enemy defender power on 🗡 Enemy tab
3. Configure settings ⚙️ (boost, penalty, margins)
4. Press ⚔️ Calc
5. Review plan on 📊 Attack tab
6. During war: mark spent attacks (0/1/2) + closed slots (✓)
```

---

## Features

### Roster (collapsible)
- Up to **20 players** × **2 attacks** = 40 attacks (Gold League)
- Per player: hero power ⚔️, titan power ⚡, E5 checkbox, Risk 🚧 flag
- ± buttons with hold-to-increment on all power fields
- Sort by max / hero / titan power
- Auto-select on focus for all inputs

### Settings ⚙️
| Setting | Description |
|---------|-------------|
| Pts per win | Points per battle won (typically 20) |
| E5 boost % | Power bonus for E5 titans (both ours and enemy) |
| Non-E5 penalty % | Power reduction for non-E5 titans (both ours and enemy, symmetric) |
| Hero margin % | Minimum power advantage required for confident hero win |
| Titan margin % | Minimum power advantage required for confident titan win |
| 👻 Ghost Bridge | Force Barracks + Lighthouse + Mage Academy capture (T2 path insurance) |

All settings have ± buttons with hold, Enter key navigation, and auto-recalc.

### Tabs

**🗡 Enemy** — enter enemy defender power per building slot. Titan slots have E5 checkbox (default on). Non-E5 enemies get penalty applied automatically. 4-column layout: T1 | T2 | T2 | T3.

**🛡 Defense** — assign players to our defense. Auto-distribute button fills by building priority. Defense and attack are independent (same player can do both).

**📊 Attack** — calculation results with two plans:
- **🗡 Max** — aggressive: all possible wins including risky (margin below threshold)
- **🛡 Safe** — conservative: only confident wins (margin ≥ threshold). May lose buildings but every attack is reliable.

---

## Algorithm (v4)

### Phase 1: Capture Plan
- Enumerates all 1024 (2^10) building combinations
- Filters by unlock chain: T2 requires Bridge OR Ghost Bridge, Citadel requires Fire/Nature/Ice
- Ghost Insurance: if enabled, any T2 building requires all 3 Ghost Bridge buildings
- **mustCapture**: buildings with already-closed slots cannot be dropped
- Feasibility check (greedy): capture slots first (scarcity sort, weakest sufficient), then extras
- Score = wins × fightPts + capture bonuses
- Tiebreaker: strategic value (Citadel > CIT_UNLOCK > Ghost Req > Bridge)

### Phase 2: Assignment (R1 → R2 → R3 → R4 → R5)
| Round | Slots | Player selection | Notes |
|-------|-------|-----------------|-------|
| R1 | Capture hero slots | Strongest hero, scarcity-first | Skips titan-critical players |
| R1b | Remaining capture hero | Strongest hero (all eligible) | Fallback if R1 left gaps |
| R2 | Capture titan slots | Weakest sufficient titan | — |
| R3 | Extra hero slots | Strongest hero | Non-capture buildings |
| R4 | Extra titan slots | Weakest sufficient titan | Non-capture buildings |
| R5 | Unassigned titan slots | Non-E5 last resort | Only if nothing else fits |

**Titan-critical protection**: players who are sole candidate (≤2 options) for a heavy capture titan slot are protected from R1 hero assignment.

**Risk avoidance**: on CIT_UNLOCK buildings and Citadel, risky players are deprioritized if alternatives exist.

### Phase 3: Swap Optimization
Pairwise swaps within same type (hero↔hero, titan↔titan). Accept if:
1. Fewer risky results (priority)
2. Same risk + more same-building consolidation (both attacks on one building)
3. Same risk + same SB + lower max margin in the pair (margin evening)

### Power Formulas
| | E5 | Non-E5 |
|---|---|---|
| **Our titan** | tPow × (1 + boost%) | tPow × (1 − penalty%) |
| **Enemy titan** | power × (1 + boost%) | power × (1 − penalty%) |
| **Hero** | hPow (raw) | hPow (raw) |

Comparison is always boosted-vs-boosted (symmetric).

---

## Battle Tracking

During war, mark progress without recalculating from scratch:
- **Spent** (0/1/2) per player — how many attacks used
- **Closed** (✓) per building slot — enemy slot defeated

Plan auto-recalculates with remaining attacks and open slots. Buildings with closed slots become **mustCapture** (won't be dropped).

### Building Accessibility (🔒)
| Tier | Accessible when |
|------|----------------|
| T1 | Always |
| T2 | Bridge captured (all ✓) OR Ghost Bridge (Barracks + Lighthouse + Mage all ✓) |
| T3 (Citadel) | T2 accessible AND any of Fire/Nature/Ice captured |

Locked buildings show slots and planned attackers but checkmarks are disabled (🔒).

---

## Building Map

| Tier | Building | Type | Slots | Bonus |
|------|----------|------|-------|-------|
| 1 | Bridge | ⚡ Titans | 4 | +60 |
| 1 | Barracks | ⚔️ Heroes | 3 | +40 |
| 1 | Lighthouse | ⚔️ Heroes | 3 | +40 |
| 1 | Mage Academy | ⚔️ Heroes | 3 | +40 |
| 2 | Foundry | ⚔️ Heroes | 4 | +60 |
| 2 | Bastion of Fire | ⚡ Titans | 4 | +60 |
| 2 | Gates of Nature | ⚡ Titans | 4 | +60 |
| 2 | Bastion of Ice | ⚡ Titans | 4 | +60 |
| 2 | Spring of Elements | ⚡ Titans | 4 | +60 |
| 3 | Citadel | ⚔️ Heroes | 7 | +120 |

**Max score**: 40 wins × 20pts + 600 bonus = **1400 pts**

---

## Score Bar (sticky)

| Metric | Meaning |
|--------|---------|
| PTS | Total: wins × fightPts + capture bonuses |
| W | Win count (guaranteed) |
| R | Risky count (margin < threshold) |
| 🏴 | Captured buildings / 10 |
| Fact | Actual points from closed slots during war |

---

## Export / Import

- **💾 Export** — saves full state to `gw_YYYY-MM-DD.json`
- **📂 Import** — loads JSON (v4 and v6 compatible, auto-migrates settings)
- **Auto-save** — localStorage (`gwp_v6`), restored on page load
- **🗑 Reset** — clears defense + enemy + progress, keeps roster

### JSON format (v6)
```json
{
  "v": 6,
  "roster": [...],
  "ourDef": {...},
  "enmDef": {...},
  "cfg": {
    "fightPts": 20,
    "titanBoost": 10,
    "titanPenalty": 10,
    "heroMargin": 20,
    "titanMargin": 5,
    "ghostInsurance": true
  },
  "spentAttempts": {},
  "closedSlots": [],
  "nextId": 21
}
```

Powers in **K units** (e.g. `450` = 450K).

---

## In-App Help

Press **?** button next to title. Bilingual: RU (default) / EN toggle. Does not reset any state.
