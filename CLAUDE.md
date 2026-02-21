# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QuickHeal is a World of Warcraft addon for the **Turtle WoW** private server (patch 1.18). It provides intelligent, automated healing for party/raid members — automatically selecting the best spell rank based on target health, available mana, and class-specific talents.

There is no build, lint, or test system. Development is done by editing Lua/XML files directly and reloading the WoW client (`/reload` in-game) to test changes.

## Key Commands (in-game)

- `/qh cfg` — Open configuration panel
- `/qh toggle` — Switch Normal HPS / High HPS mode
- `/qh dr` — Open Downrank window
- `/qh tanklist` — Toggle tank list UI
- `/qh [mask] [type] [mod]` — Targeted healing (see README for full options)
- `/script QuickHeal(nil,'Spellname')` — Cast max rank of a specific spell on the lowest target

## Architecture

### Entry Point

`QuickHeal()` in `QuickHeal.lua` (line ~2995) is the main function invoked by macros. It orchestrates target selection, spell selection, and cast execution.

### Healing Flow

1. **Target selection** — `FindWhoToHeal()` (line ~2006) picks the lowest-health valid target. `FindWhoToHOT()` does the same for HoT spells.
2. **Spell selection** — Delegated to class-specific modules via `FindHealSpellToUse()` / `FindHoTSpellToUse()`.
3. **Execution** — `ExecuteHeal()` / `ExecuteHOT()` casts the chosen spell.

### Class Modules

Each healer class has its own file containing spell tables (base values, rank scaling), talent modifier detection, and spell-selection logic:

| File | Class | Key spells |
|------|-------|-----------|
| `QuickHealPriest.lua` | Priest | Lesser Heal, Heal, Greater Heal, Flash Heal, PW:Shield, Renew |
| `QuickHealDruid.lua` | Druid | Healing Touch, Regrowth, Swiftmend, Nourish, Lifebloom |
| `QuickHealPaladin.lua` | Paladin | Holy Light, Flash of Light, Holy Shock |
| `QuickHealShaman.lua` | Shaman | Healing Wave, Lesser Healing Wave, Chain Heal |

### Important Supporting Functions (QuickHeal.lua)

- `QuickHeal_GetSpellInfo()` / `QuickHeal_GetSpellIDs()` — Reads available spell ranks from the spellbook at runtime
- `QuickHeal_EstimateUnitHealNeed()` — Predicts how much healing a target needs (accounts for HealComm predictions from other healers)
- `QuickHeal_DetectBuff()` — Scans for active buffs/procs affecting healing (e.g. Divine Favor, Healing Way, Nature's Grace)
- `QuickHeal_GetHealModifier()` — Reads equipment bonuses via ItemBonusLib
- `IsBlacklisted()` / `IsMainTank()` — Role/exclusion checks used in target filtering

### Configuration

Settings are stored per-character in `QuickHealVariables` (SavedVariablesPerCharacter). Defaults are defined in the `DQHV` table near the top of `QuickHeal.lua`. Key settings include health thresholds (`RatioFull`, `RatioHealthy*`, `RatioForceself`), pet priority, notification style, and raid group filters.

### UI

- `QuickHeal.xml` — All frame/button/panel definitions
- `HealingBar.xml` — Real-time healing status bar overlay
- `QuickClick.lua` — Alternative click-based healing interface
- `Bindings.xml` — Keybinding declarations

### Libraries (libs/)

Uses the Ace2 framework. Notable libraries:
- **HealComm-1.0** — Heal prediction across party/raid (used to avoid overheal)
- **ItemBonusLib-1.0** — Equipment healing bonus detection
- **AceDB-2.0** — Saved variables / per-character persistence
- **AceLocale-2.2** — Localization (en/de/fr/cn in `localization.*.lua`)

## Turtle WoW Specifics

- Target game version: **patch 1.18** (vanilla-era WoW with custom content)
- Spell values (base heal, mana cost, cast time) are hardcoded in each class module and must be updated when Turtle WoW patches change spell data
- Talent bonuses are detected at runtime via `QuickHeal_DetectBuff()` and applied as multipliers in each class module
