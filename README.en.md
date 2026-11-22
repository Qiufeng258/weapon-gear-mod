# Weapon Gear — DoL Weapon / Firearm / Armor System Mod

> 📥 Latest release: https://github.com/Qiufeng258/weapon-gear-mod/releases

A mechanic-only equipment expansion: **no rendered clothing images, purely stats & gameplay**. Ships with an in-game builder and a web editor, plus a shareable custom weapon pack ecosystem.

## Features

- **Weapons** — 4 types (blunt / blade / gun / unarmed), 37 built-in items (18 melee + 13 guns + 6 armors); shop with categories / pagination / stat cards; wardrobe equip / unequip / discard / secondary weapon; live sidebar display.
- **Guns** — 16 ammo calibers (bullet icons + armor-piercing tier 0–6 colors; penetration is a display-only placeholder for now — no concrete mechanic yet); the first shot of each battle costs 1 ammo box; auto-falls back to your secondary weapon when out of ammo; guns & military gear unlock at £6,000 cumulative spend + cash; the PG-7VM rocket deals AOE damage.
- **Armor** — pain / stress / trauma / control resistances (direction-aware: bad stats only reduce increases, control only reduces losses); durability wear (permanently destroyed at 0); repair service (returns after 1 day, tiered rates by max durability).
- **Builders** —
  - In-game workbench: create custom weapons; **since v2.5.20 you can also edit loaded weapon stats** (built-in / weapon-pack shop templates; overrides are saved with your save file, with one-click restore to default; already-owned copies stay as snapshots).
  - Web editor: import / export weapon pack zips, browser autosave, live preview, combat reference panel.
- **Weapon pack ecosystem** — export `weapon_pack_*.zip` to share; packs depend on `weapon_gear ^2.0.0`; install the base mod first.
- **Localization friendly** — the shop entry is injected at runtime; all patch anchors are language-independent (anchored to code, never to visible text), so Chinese-localized builds work too.

## Compatibility

| Target | Status | Notes |
|---|---|---|
| DoL 0.5.12.13 (Lyra 1.0.1a / goose 1004.1) | ✅ Full support | **Since v2.5.22**; damage anchor follows the new build (see "0.5.12.13 adaptation" below) |
| DoLP v0.768 goosefem_mys 0.5.11.9 | ✅ Full support | All mechanics (including the brawling / footwork / striking / kicking skills) |
| Official DoL 0.5.11.9 (mod-loader builds) | ✅ Full support | Stats narrowed to the vanilla trio: physique(0–15000) / athletics(0–1000) / handskill(0–1000); unarmed weapons are auto-mapped |
| Other versions / other forks | 🟡 Try it | Generally no major issues; run anchor validation if something misbehaves |

**What we change, and what we don't:**

1. Guard-style hooks at stat entry/exit points (purely additive):
   - Wraps the `pain / stress / trauma / arousal / control` stat functions — armor reduction applies only while armor is worn and the damage direction matches; nothing is intercepted or rewritten;
   - Patches the combat `<<violence>>` pain / stress lines, the two struggle damage sites, and NPC damage splitting (AOE) — again, only effective while armor is worn.
2. High Street entry: the weapon shop link is **injected at runtime** — it is anchored to DOM structure (`a[data-passage="Office Lobby"]` and other `data-passage` attributes), not to translated text, so it works in localized builds; the label switches by language (Chinese "武器店 (0:01)" / English "Weapon Shop (0:01)").

The mod is patched in memory by the mod loader; **game source files are never modified** — uninstalling restores the original game.

## Installation

1. Pick the zip for your game version:
   - **DoL 0.5.12.13 (Lyra / goose builds)**: `dist/DoL0.5.12.13/武器系统mod_v2.5.22_适配DoL0.5.12.13.zip`
   - DoLP v0.768: `dist/DoLP/武器系统mod_v2.5.20_适配DoLP0.768.zip`
   - Vanilla DoL 0.5.11.9: `dist/原版DoL/武器系统mod_v2.5.20_适配原版DoL0.5.11.9.zip`
2. In-game ModLoader → Add / Remove Mods → choose file → sideload → refresh.
3. (Optional) Install weapon packs: the base mod must be installed first; packs depend on `weapon_gear ^2.0.0`, and installing them in the wrong order disables the pack.
4. In game: High Street → available places → Weapon Shop / Weapon Workbench / Weapon Wardrobe.

## Editors

- **Web editor** (open in a browser):
  - `dist/DoL0.5.12.13/武器编制器.html` — **DoL 0.5.12.13 version**: hookable stats are narrowed to physique / athletics / hands kill (the new build has no `brawling`-style variables)
  - `dist/DoLP/武器编制器.html` — DoLP version (stats include brawling / footwork / striking / kicking)
  - `dist/原版DoL/武器编制器-原版.html` — vanilla DoL version (three stats / physique cap 15000)

  Usage (same for all three):

  1. Enter a pack name → ＋add weapons and edit each one (name / type / damage / multiplier / stat hooks / price / caliber / armor stats / image) → 📦 export the weapon pack zip;
  2. 📥 You can also import an existing weapon pack to modify or merge packs;
  3. Your work is autosaved in the browser; the exported zip is the real backup.
- **In-game workbench**: create custom weapons; click 修改 on any loaded weapon to edit its stats, 恢复默认 to restore, and custom weapons can be deleted.

## 0.5.12.13 adaptation (v2.5.22)

Against 0.5.12.13 (Lyra 1.0.1a / goose 1004.1), **13 of the mod's 15 patches still match unchanged**. Only two things moved:

1. **The core damage anchor was rewritten.** In the `Widgets` / `defiance` component the old line was
   `<<set _thedamage to (2 * _defiance * (currentSkillValue('physique') / 4000))>>`, the new build uses
   `<<set _thedamage to (2 * _defiance * ($physique / 4000))>>`. The weapon boost is injected right after the
   `_thedamage` assignment and before health is subtracted, so only the anchor changed (gun first-shot ammo
   consumption / damage multiplier / numeric guard are untouched).
2. **The `struggle_defense_result` component was removed.** The old "enemy runs out of steam after 3 blocks"
   mechanic (`$struggle.enemy[_i].dodge`) no longer exists anywhere in 0.5.12.13, so that struggle-bonus patch had
   no anchor left and was dropped; **the remaining two struggle damage sites still live in `Widgets Struggle`**
   and still receive the bonus.

Post-adaptation full audit (item-by-item against 0.5.11.9):

- All 5 JS stat hooks (`pain / stress / trauma / arousal / control`) still hit;
- Direct `$pain +=` (6 sites) and `$stress +=` (16 sites) are identical to the old build → armor hook coverage is
  unchanged, no new "bypasses armor" outlet was introduced;
- The player→enemy damage outlets differ by exactly one line (`Math.round(_thedamage)` → `_thedamage`) inside the
  same patched component → still covered;
- `printmoney` (pennies ÷100), `linkifyDivs`, `createInternalLink`, `Links.generateLinkNumbers`, `Time.pass` and
  `currentSkillValue` all still exist, so the High Street entry injection and currency conversion need no change;
- No passage / widget name collisions (16,087 passages, 5,101 widgets in the build).

**Known (not a regression):** 5 of the 37 built-in weapons hook `brawling` (the 5 unarmed ones: brass knuckles / tactical knuckles / single-blade / double-blade / wrist claws).
Those never had a backing variable even in 0.5.11.9 (`currentSkillValue` returns 0), and 0.5.12.13 also dropped the `$brawling` feat variable → the hook is worth +0 and breaks nothing else.

- The **web editor** is already narrowed for the new build: `dist/DoL0.5.12.13/武器编制器.html` only offers physique / athletics / hands kill, so newly created weapons can't hook dead stats.
- The **in-game workbench** still lists the old stats (brawling / footwork / striking / kicking, also +0), and those 5 built-in unarmed weapons still hook `brawling`. Aligning both (and re-mapping the 5 items to `athletics`, which would raise unarmed damage by 20–100) is **left for you to decide** — not changed yet.

**Older builds are not replaced:** DoLP 0.768 and vanilla 0.5.11.9 use different anchors — keep using their
v2.5.20 / v2.5.21 packages and **do not mix them**.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## Repository layout

```
dist/DoL0.5.12.13/  DoL 0.5.12.13 mod zip + web editor (stats: physique/athletics/handskill)
dist/DoLP/       DoLP mod zip + web editor
dist/原版DoL/      vanilla DoL mod zip + web editor
docs/            release notes (quick install guide)
```

## Notes

- Weapon packs depend on `weapon_gear ^2.0.0`; install the base mod first.
- Editing a shop template only affects **future** purchases; owned weapons are snapshots taken at purchase time.
- Web editor autosave lives in the browser; clearing browser data loses it (the exported zip is the real backup).
- Guns unlock at £6,000 by default (edit `weaponGearUnlockGuns` in the source and repack if you want to change it).
