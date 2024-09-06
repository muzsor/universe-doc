# Formulas

> [!IMPORTANT]
> **Document about the game Flyff Universe. <ins>NOT affiliated with GalaLab / Wemade Connect / PlayPark.</ins>**

> [!IMPORTANT]
> **The information used in the document is from the FlyFF Universe API and belong to Gala Lab Corp.**

> [!CAUTION]
> **This repository was created to promote the game. If any inconvenience is caused, please contact me as soon as possible. Thanks you.** 🙏

<!-- Copyright 2024 © Gala Lab Corp. All Rights Reserved. -->

<table><tr><td><details><summary>📁 Table of Contents</summary>

- [Formulas](#formulas)
  - [⚔️ damage](#️-damage)
    - [dps](#dps)
    - [auto attack](#auto-attack)
    - [melee skill](#melee-skill)
    - [magic skill](#magic-skill)
  - [⚔️ critical chance vs critical damage](#️-critical-chance-vs-critical-damage)
  - [⚔️ blade damage](#️-blade-damage)
  - [🗡️ empower weapon](#️-empower-weapon)
  - [🔪 sword vs axe 🪓](#-sword-vs-axe-)
  - [❤️ health](#️-health)
    - [max hp](#max-hp)
  - [⛔ block](#-block)
    - [calculate](#calculate)
    - [block cap](#block-cap)
    - [block penetration](#block-penetration)

</details></td></tr></table>

## ⚔️ damage

<details>
  <summary>📁 damage details</summary>

### dps

```
DamagePerSecond = computeDamage * hitsPerSecond
```

* hitsPerSecond
   ```js
   hitsPerSecond = classHitsPerSecond * attackSpeed * HitRate
   ```

   * HitRate
      ```js
      // ------------------------------------------------------------------------------------
      // If not AUTO_ATTACK, this is always 100.
      // ------------------------------------------------------------------------------------

      factor = 1.6 * 1.5 * ((AttackLevel * 1.2) / (AttackLevel + DefenderLevel))
      hitProb = (AttackDex / (AttackDex + DefenderParry)) * factor
      HitRate = clamp(hitRate + ExtraHitRate, 0.2, 0.96)
      // Limited to 0.2 ~ 0.96
      ```
      ```js
      // simplify formula
      nHitRate = (AttackDex * 2.88 * AttackLevel) / ((AttackDex + DefenderParry) * (AttackLevel + DefenderLevel))
      HitRate = clamp(hitRate + ExtraHitRate, 0.2, 0.96)
      // Limited to 0.2 ~ 0.96
      ```

   * DefenderParry : From Defender's unscaled `parry` `DST_PARRY`.

   * ExtraHitRate : From Attacker's Gear, Buff scales `hitrate` `DST_ADJ_HITRATE`.

### auto attack

<table><tr><td><details><summary>details</summary>

* ATK_TYPE : `ATK_GENERIC`

* computeAttack
   ```js
   computeAttack = (HitPower * AttackMultiplier) + FlatAttack
                 = (HitMinMax * DamagePropertyFactor * (1 + attack% + skillDamage% ) * (1 + PvEPvP%) * (1 + Upcut%)) + FlatAttack
   ```

   * HitPower
      ```js
      HitPower = HitMinMax * DamagePropertyFactor

      // ------------------------------------------------------------------------------------
      // DamagePropertyFactor = ElementMultiplier(UpgradeLevel)
      // Find the increase/decrease factor of ATK and DEF to be used in the GetHitPower function.
      // ------------------------------------------------------------------------------------
      ```

   * HitMinMax
      ```js
      HitMinMax = ((WeaponBaseAttackMinMax * 2) + WeaponAttack + AttackerPlusDamage) * WeaponMultiplier + WeaponUpgradeLevelAdditionalAttack
      // ------------------------------------------------------------------------------------
      // WeaponBaseAttackMinMax = minAttack DST_ABILITY_MIN, maxAttack DST_ABILITY_MAX
      // ------------------------------------------------------------------------------------

      // ------------------------------------------------------------------------------------
      // example (Lusaka's Crystal Axe U+5, Demol Earring U+5, Spirit Fortune) :
      // ((544 ~ 546 * 2) + 3123 + (540 * 2) + 150) * 1.39 + 58.0948 = 7621.0848 ~ 7626.6448
      // ------------------------------------------------------------------------------------
      ```

   * WeaponAttack
      ```js
      WeaponAttack = statAttack + levelAttack + plusWeaponAttack
      ```
      ```js
      // ------------------------------------------------------------------------------------
      statAttack = (AttackerStats - WeaponTypeStatModifer) * ClassWeaponTypeAutoAttackFactors
      // ------------------------------------------------------------------------------------
      // ClassWeaponTypeAutoAttackFactors = autoAttackFactors = GetJobPropFactor( JOB_PROP_TYPE )
      // ------------------------------------------------------------------------------------
      // WeaponTypeStatModifer:
      // sword WT_MELEE_SWD 12
      // axe WT_MELEE_AXE 12
      // staff WT_MELEE_STAFF 10
      // stick WT_MELEE_STICK 10
      // knuckle WT_MELEE_KNUCKLE 10
      // wand WT_MAGIC_WAND 10
      // yoyo WT_MELEE_YOYO 12
      // bow WT_RANGE_BOW 14
      // ------------------------------------------------------------------------------------
      // example (Blade str 500 and use Axe) :
      // (500 - 12) * 5.7 = 2781.6
      // example (Blade str 500 and use Sword) :
      // (500 - 12) * 4.7 = 2,293.6
      // ------------------------------------------------------------------------------------

      // ------------------------------------------------------------------------------------
      levelAttack = AttackerLevel * WeaponTypeLevelFactor
      // ------------------------------------------------------------------------------------
      // WeaponTypeLevelFactor :
      // sword WT_MELEE_SWD 1.1
      // axe WT_MELEE_AXE 1.2
      // staff WT_MELEE_STAFF 1.1
      // stick WT_MELEE_STICK 1.3
      // knuckle WT_MELEE_KNUCKLE 1.2
      // wand WT_MAGIC_WAND 1.2
      // yoyo WT_MELEE_YOYO 1.1
      // bow WT_RANGE_BOW 0.91
      // ------------------------------------------------------------------------------------
      // example (lv160 Blade use Axe) :
      // 160 * 1.2 = 192
      // ------------------------------------------------------------------------------------

      // ------------------------------------------------------------------------------------
      plusWeaponAttack : From Attacker’s Gear, Buff Weapon Type unscaled Additional Attack.
      // ------------------------------------------------------------------------------------
      // swordattack DST_SWD_DMG
      // axeattack DST_AXE_DMG
      // staffattack, stickattck
      // knuckleattack DST_KNUCKLE_DMG
      // wandattack, yoyoattack DST_YOY_DMG
      // bowattack DST_BOW_DMG
      // ------------------------------------------------------------------------------------
      // master skill :
      // DST_KNUCKLEMASTER_DMG
      // DST_YOYOMASTER_DMG
      // DST_BOWMASTER_DMG
      // DST_TWOHANDMASTER_DMG
      // ------------------------------------------------------------------------------------
      // example (Blade Skill Axe) :
      // Smite Axe axeattack + 50 and Axe Mastery axeattack + 100, total = 150
      // ------------------------------------------------------------------------------------


      // ------------------------------------------------------------------------------------
      // example total = 2781.6 + 192 + 150 = 3123
      // ------------------------------------------------------------------------------------
      ```

   * AttackerPlusDamage : From Attacker's Gear, Buff unscaled `damage` `DST_CHR_DMG`.

      * Example : *Demol Earring* `damage`, *Spirit Fortune* `damage` etc.

   * WeaponMultiplier : Weapon Attack Upgrade Level Bonus
      ```js
      // WeaponUpgradeLevel = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, U1, U2, U3, U4, U5
      WeaponMultiplier = 2%, 4%, 6%, 8%, 10%, 13%, 16%, 19%, 21%, 24%,27%, 30%, 33%, 36%, 39%
      ```

   * WeaponUpgradeLevelAdditionalAttack : Weapon Attack Upgrade Level Additional Attack
      ```js
      WeaponUpgradeLevelAdditionalAttack = WeaponUpgradeLevel^1.5
      ```

   * AttackMultiplier
      ```js
      // AttackMultiplier = (1 + DST_ATKPOWER_RATE%) * ( 1 + DST_PVP_DMG%DST_MONSTER_DMG%) * (1 + SM_ATTACK_UP1% || SM_ATTACK_UP%)
      AttackMultiplier = (1 + attack% + skillDamage% ) * (1 + PvEPvP%) * (1 + Upcut%)
      ```

   * FlatAttack : From Attacker's Gear, Buff unscaled `attack` `DST_ATKPOWER`.

      * Example : *Balloons* `attack`, *Power Scroll* `attack` etc.

* computeDamage

   * **The term `critical` here refers to a factor derived from a series of calculations. For detailed calculations, please refer to the section below.**

   ```js
   computeDamage = applyDefense(computeAttack)
                 = applyGenericDefense(computeAttack) * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
                 = damageAfterCritical * blockFactor * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
                 = applyAttackDefense(computeAttack, defense) * critical * blockFactor * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
   ```

   * applyGenericDefense
      ```js
      applyGenericDefense = damageAfterCritical * blockFactor
      ```

   * 💥 damageAfterCritical

      * **The term `critical` here refers to a factor derived from a series of calculations. For detailed calculations, please refer to the section below.**

      ```js
      damageAfterCritical = applyAttackDefense(computeAttack, defense) * critical
                        = damageAfterApplyDefense * critical
      ```

   * defense
      ```js
      defense = computeDefense
              = computeGenericDefense
      ```

   * 💥 criticalChance

      * AttackerCriticalChance : From Attacker's Gear, Buff scales `criticalchance` `DST_CHR_CHANCECRITICAL`.

      ```js
      criticalChance = CriticalResistFactor * (((AttackerDex / 10) * ClassCriticalFactor) + AttackerCriticalChance)
      // ------------------------------------------------------------------------------------
      // ClassCriticalFactor : critical , class.critical, job.critical
      // ClassCriticalFactor = GetJobPropFactor (JOB_PROP_CRITICAL )
      // ------------------------------------------------------------------------------------
      // example (Blade's str 500, dex 60, cc 45) :
      // criticalChance = CriticalResistFactor * (((60 / 10) * 1) + 45)) = CriticalResistFactor * 51
      // ------------------------------------------------------------------------------------
      ```

   * 💥 criticalFactor
      ```js
      // ------------------------------------------------------------------------------------
      // your level <= monster's level
      minCritical = 1.1
      maxCritical = 1.4
      // ------------------------------------------------------------------------------------
      // Average Dps
      criticalFactor = (minCritical + maxCritical) / 2.0 = 1.25
      // ------------------------------------------------------------------------------------

      // ------------------------------------------------------------------------------------
      // monster's level < your level
      minCritical = 1.2
      maxCritical = 2.0
      // ------------------------------------------------------------------------------------
      // Average Dps
      criticalFactor = (minCritical + maxCritical) / 2.0 = 1.6
      // ------------------------------------------------------------------------------------
      ```

   * 💥 criticalDamage

      <img src="./formulas/devblog-2021_critical_damage_formula.png" alt="devblog-2021_critical_damage_formula.png"/>

      ```js
      criticalDamage = damageAfterApplyDefense * criticalFactor * (1 + CriticalDamage%)
      // ------------------------------------------------------------------------------------
      // criticalBonus, CriticalDamage% : From Attacker's Gear, Buff scales criticaldamage, DST_CRITICAL_BONUS
      // ------------------------------------------------------------------------------------
      ```

   * 💥 **damageAfterCritical**
      ```js
      // linearInterpolation
      damageAfterCritical = linearInterpolation(damageAfterApplyDefense, criticalDamage, criticalChance)
                        = damageAfterApplyDefense * ((1 - criticalChance) + criticalChance * criticalFactor * (1 + criticalDamage%))
      ```
      ```js
      // your level <= monster's level, average dps
      damageAfterCritical = damageAfterApplyDefense * ((1 - criticalChance) + criticalChance * 1.25 * (1 + criticalDamage%))
      ```
      ```js
      // monster's level < your level, average dps
      damageAfterCritical = damageAfterApplyDefense * ((1 - criticalChance) + criticalChance * 1.6 * (1 + criticalDamage%))
      ```

   * ElementResistFactor : `0.7`, `1.0`, `1.3`

   * DamageMultiplier
      ```js
      DamageMultiplier = OffhandWeaponAttackFactor * HolycrossSwordcross2x * LevelDifferenceReductionFactor

      // ------------------------------------------------------------------------------------
      // OffhandWeaponAttackFactor : PARTS_LWEAPON 0.75
      // ------------------------------------------------------------------------------------
      // HolycrossSwordcross2x : CHS_DOUBLE
      // ------------------------------------------------------------------------------------
      ```

   * LevelDifferenceReductionFactor
      ```js
      LevelDifferenceReductionFactor = Math.cos((Math.PI * Math.min(nDelta, MAX_OVER_ATK - 1)) / MAX_OVER_ATK * 2)
                                     = Math.cos(Math.PI * Math.min(nDelta, 15) / 32)
      //for ( i = 0; i < 16; i++ ) {
      //  console.log(Math.cos(Math.PI * Math.min(i, 15) / 32))
      //}
      ```

</details></td></tr></table>

### melee skill

<table><tr><td><details><summary>details</summary>

* ATK_TYPE : `ATK_MELEESKILL`, `skill.magic == false`

* computeAttack
   ```js
   computeAttack = (MeleeSkillPower * AttackMultiplier) + FlatAttack
   ```

   * MeleeSkillPower
      ```js
      MeleeSkillPower = (((WeaponAttackPowerMinMax + (SkillMinMaxAttack + WeaponAdditionalSkillDamage) * 5 + ReferStat - 20) * (16 + SkillLevel)) / 13) + PlusWeaponAttack + AttackerPlusDamage
      ```

   * WeaponAttackPowerMinMax
      ```js
      WeaponAttackPowerMinMax = WeaponBaseAttackMinMax * WeaponMultiplier + MainhandWeaponUpgradeLevel^1.5

      // ------------------------------------------------------------------------------------
      // example (Lusaka's Crystal Axe U+5) :
      // (544 ~ 546 * 1.39) + 58.0948 = 814.25 ~ 817.03
      // ------------------------------------------------------------------------------------
      ```

   * WeaponMultiplier : Weapon Attack Upgrade Level Bonus
      ```js
      WeaponMultiplier = 2%, 4%, 6%, 8%, 10%, 13%, 16%, 19%, 21%, 24%,27%, 30%, 33%, 36%, 39%
      ```

   * ReferStat
      ```js
      // If there are two Stats, add them after calculation.
      ReferStat = AttackerStat * ((((PvEPvPSkillStatScale * 50.0) - (SkillLevel + 1)) / 5.0) / 10.0) + ((AttackerStat * SkillLevel) / 50.0)
                = AttackerStat * (((PvEPvPSkillStatScale × 50.0) - 1) / 50)

      // ------------------------------------------------------------------------------------
      // example (Bldae use Armor Penetrate Lv10 PvE str scale 3, dex scale 1.7) :
      // character's str 500, dex 60 :
      // (500 * (((3 * 50.0) - 1) / 50.0)) + (60 * (((1.7 * 50.0) - 1) /50.0)) = 1590.8
      // ------------------------------------------------------------------------------------
      ```

   * SkillMinAttack : skill.minAttack and skill.maxAttack

   * WeaponAdditionalSkillDamage : weapon.additionalSkillDamage

   * PlusWeaponAttack : From Attacker’s Gear, Buff Weapon Type unscaled Additional Attack.

   * AttackerPlusDamage : From Attacker's Gear, Buff unscaled `damage` `DST_CHR_DMG`.

      * Example : *Demol Earring* `damage`, *Spirit Fortune* `damage` etc.

   * AttackMultiplier
      ```js
      // AttackMultiplier = (1 + DST_ATKPOWER_RATE%) * ( 1 + DST_PVP_DMG%DST_MONSTER_DMG%) * (1 + SM_ATTACK_UP1% || SM_ATTACK_UP%)
      AttackMultiplier = (1 + attack% + skillDamage% ) * (1 + PvEPvP%) * (1 + Upcut%)
      ```

   * FlatAttack : From Attacker's Gear, Buff unscaled `attack` `DST_ATKPOWER`.

      * Example : *Balloons* `attack`, *Power Scroll* `attack` etc.

* computeDamage
   ```js
   computeDamage = applyDefense(computeAttack)
                 = applyDefenseParryCritical(computeAttack) * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
                 =  damage * blockFactor * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
   ```

   * applyDefenseParryCritical
      ```js
      applyDefenseParryCritical = applyAttackDefense(computeAttack, defense)
      ```

   * defense
      ```js
      defense = computeDefense
              = computeGenericDefense
      ```

   * ElementResistFactor : `0.8`, `1.0`, `1.4`

      * If the skill and weapon match the element, apply `10%` more damage; otherwise, apply `-10%` damage.

   * DamageMultiplier
      ```js
      DamageMultiplier = SkillDamageMultiplier * SkillAwakeBonus * OffhandWeaponAttackFactor * HolycrossSwordcross2x * LevelDifferenceReductionFactor

      // ------------------------------------------------------------------------------------
      // OffhandWeaponAttackFactor : PARTS_LWEAPON 0.75
      // ------------------------------------------------------------------------------------
      // HolycrossSwordcross2x : CHS_DOUBLE
      // ------------------------------------------------------------------------------------
      ```

   * LevelDifferenceReductionFactor
      ```js
      LevelDifferenceReductionFactor = Math.cos((Math.PI * Math.min(nDelta, MAX_OVER_ATK - 1)) / MAX_OVER_ATK * 2)
                                     = Math.cos(Math.PI * Math.min(nDelta, 15) / 32)
      //for ( i = 0; i < 16; i++ ) {
      //  console.log(Math.cos(Math.PI * Math.min(i, 15) / 32))
      //}
      ```

   * SkillDamageMultiplier : `skill.levels.damageMultiplier * skill.levels.probability(probabilityPVP) * BuffSkillDamageMultiplier`

   * BuffSkillDamageMultiplier : Damage caused by specific skills in different states.

      * Example : *If it's a Silent Shot, the damage is doubled, and if it's Dark Illusion, it's removed.*

</details></td></tr></table>

### magic skill

<table><tr><td><details><summary>details</summary>

* ATK_TYPE : `ATK_MAGICSKILL`, `skill.magic == true`

* computeAttack
   ```js
   computeAttack = (MagicSkillPower * AttackMultiplier) + FlatAttack
                 = (MeleeSkillPower MeleeSkillPower * (1+ magicattack%) * (1 + ElementMastery%) * AttackMultiplier) + FlatAttack
   ```

   * MagicSkillPower
      ```js
      // MagicSkillPower = MeleeSkillPower * (1 + DST_ADDMAGIC%) * ( 1 + DST_MASTRY_ELEMENT%)
      MagicSkillPower = MeleeSkillPower * (1+ magicattack%) * (1 + ElementMastery%)
      ```

   * ElementMastery% : From Attacker's Gear, Buff scales `firemastery` `DST_MASTRY_FIRE`, `watermastery` `DST_MASTRY_WATER`, `electricitymastery` `DST_MASTRY_ELECTRICITY`, `windmastery` `DST_MASTRY_WIND`, `earthmastery` `DST_MASTRY_EARTH`.

   * AttackMultiplier
      ```js
      // AttackMultiplier = (1 + DST_ATKPOWER_RATE%) * ( 1 + DST_PVP_DMG%DST_MONSTER_DMG%) * (1 + SM_ATTACK_UP1% || SM_ATTACK_UP%)
      AttackMultiplier = (1 + attack% + skillDamage% ) * (1 + PvEPvP%) * (1 + Upcut%)
      ```

   * FlatAttack : From Attacker's Gear, Buff unscaled `attack` `DST_ATKPOWER`.

      * Example : *Balloons* `attack`, *Power Scroll* `attack` etc.

* computeDamage
   ```js
   computeDamage = applyDefense(computeAttack)
                 = applyMagicSkillDefense(computeAttack) * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
                 =  damage * blockFactor * ElementResistFactor * Link/Global * DamageMultiplier * afterDamageFactor
   ```

   * applyMagicSkillDefense
      ```js
      // nATK = nATK - nATK * pDefender->GetParam( DST_RESIST_MAGIC_RATE, 0 ) / 100
      applyMagicSkillDefense = applyAttackDefense((computeAttack * (1 − magicDefense%)), defense)
      // ------------------------------------------------------------------------------------
      // magicDefense% : From Defender's scales magicDefense, DST_RESIST_MAGIC_RATE
      // ------------------------------------------------------------------------------------
      ```

   * defense
      ```js
      defense = computeDefense
              = computeGenericDefense
      ```

   * ElementResistFactor : `0.8`, `1.0`, `1.4`

      * If the skill and weapon match the element, apply `10%` more damage; otherwise, apply `-10%` damage.

   * DamageMultiplier
      ```js
      DamageMultiplier = SkillDamageMultiplier * SkillAwakeBonus * OffhandWeaponAttackFactor * HolycrossSwordcross2x * LevelDifferenceReductionFactor

      // ------------------------------------------------------------------------------------
      // OffhandWeaponAttackFactor : PARTS_LWEAPON 0.75
      // ------------------------------------------------------------------------------------
      // HolycrossSwordcross2x : CHS_DOUBLE
      // ------------------------------------------------------------------------------------
      ```

   * LevelDifferenceReductionFactor
      ```js
      LevelDifferenceReductionFactor = Math.cos((Math.PI * Math.min(nDelta, MAX_OVER_ATK - 1)) / MAX_OVER_ATK * 2)
                                     = Math.cos(Math.PI * Math.min(nDelta, 15) / 32)
      //for ( i = 0; i < 16; i++ ) {
      //  console.log(Math.cos(Math.PI * Math.min(i, 15) / 32))
      //}
      ```

   * SkillDamageMultiplier : `skill.levels.damageMultiplier` * `skill.levels.probability(probabilityPVP)` * `BuffSkillDamageMultiplier`

      * `skill.levels.probability(probabilityPVP)` `dwProbability` : The skill's probability. Will calculate damage factor upon success.

      * BuffSkillDamageMultiplier : Damage factor caused by specific skills in different buffs.

         * Example : *If it's a Silent Shot, the damage is doubled, and if it's Dark Illusion, it's removed.*

</details></td></tr></table>

</details>

## ⚔️ critical chance vs critical damage

<details>
  <summary>📁 critical chance vs critical damage details</summary>

<div align="center"><img src="./formulas/crit_chance&crit_damage1.png" alt="crit_chance&crit_damage1.png"/></div>

<div align="center"><img src="./formulas/crit_chance&crit_damage2.png" alt="crit_chance&crit_damage2.png"/></div>

> source:[@shayminhunter @TeachMeHisty (discord flyff universe)](https://discord.com/channels/778915844070834186/1099736335469781063/1126098066823467030 "@shayminhunter @TeachMeHisty (discord flyff universe)")

* example 1:

   * At `32%` critical chance and `50%` critical damage increase, you get the value `2.73`.

   * If you gain `x%` critical chance from one source, then `2.73` times those `x%` in critical damage increase will do the same for you.

      `10% critical chance == 27.3% critical damage increase(rounded up to 28%).`

   * This multiplier stays constant, no matter the heights of the bonuses with one exception.

      * If current critical chance bonus exceed `100%`, then only the part that's missing to `100%` must be multiplied and compare.

* example 2:

   * At `96%` critical chance and `120%` critical damage increase, you get the value `1.64`.

      * `10% critical chance == 16.4% critical damage increase(rounded up to 16%).`

      * then normally you'd opt for critical chance.

</details>

## ⚔️ blade damage

<details>
  <summary>📁 blade damage details</summary>

* Attack calculation:
   1. main hand
   2. main + offhand (dual)
   3. main hand
   4. main + offhand (dual)
   - repeat

> dual and main distribution is split 50/50, offhand never attacks alone.

> 主手攻擊和雙手攻擊是各為一半，副手從不單獨攻擊。


> dual hit is 100% main hand + 75% off hand damage.

> 雙手攻擊是 `100%` 主手傷害 + `75%` 副手傷害。

> upgrading offhand does affect actual damage when hitting with that weapon.

> 副手基礎傷害和屬性等級加成會影響使用該武器擊中(雙手攻擊)時的實際傷害。

> Each hit's damage is calculated independently based on which weapon is being used for that hit.

> 每次攻擊的傷害都是根據該攻擊所使用的武器獨立計算的。

> source:[@shayminhunter @TeachMeHisty (discord flyff universe)](https://discord.com/channels/778915844070834186/999269862260084736/1032237394856001556 "@shayminhunter @TeachMeHisty (discord flyff universe)")

<div align="center"><img src="./formulas/blade_damage.png" alt="blade_damage.png"/></div>

> source:[@frostiae @[Dev] Frostiae (discord flyff universe)](https://discord.com/channels/778915844070834186/999269862260084736/1000695721990815744 "@frostiae @[Dev] Frostiae (discord flyff universe)")

</details>

## 🗡️ empower weapon

<details>
  <summary>📁 empower weapon details</summary>

* `Empower Weapon` adds to weapons element upgrade level (literally), it is not a direct damage boost.

* The current max element is `+10`, and since you are forced to have at least `+1` on weapon to activate the skill, `Empower Weapon` can only contribute `+9` max.

* The stat window only shows empower weapon and weapon element + bonus separately.

* Only on actual damage (auto attack) calculation are both merged into one and result in a `+10` element.

> source:[@shayminhunter @TeachMeHisty (discord flyff universe)](https://discord.com/channels/778915844070834186/999269862260084736/1034085511754678303 "@shayminhunter @TeachMeHisty (discord flyff universe)")

</details>

## 🔪 sword vs axe 🪓

<details>
  <summary>📁 sword vs axe details</summary>

* The crit chance from the axe is stronger than the increase critical damage by default and going from `5.7` to `4.7` is a `17.5%` damage loss from `STR` portion of the damage alone, which makes up around halve of total attack.

* `8.78%` loss from the lower scaling + less damage from `10 crit chance` to `10 critical damage` and you're at around `10%` total dps loss.

> source:[@shayminhunter @TeachMeHisty (discord flyff universe)](https://discord.com/channels/778915844070834186/999269862260084736/1102990787186262136 "@shayminhunter @TeachMeHisty (discord flyff universe)")

</details>

## ❤️ health

<details>
  <summary>📁 hp details</summary>

* `DST_HP`

* Vagrant
   ```js
   hp = 150 + (level * 18) + (sta * level * 0.18)
      = 150 + level * (18 + (0.18 * sta))
   ```

* Mercenary, Blade, Jester, Psykeeper, Elementor
   ```js
   hp = 150 + (level * 30) + (sta * level * 0.3)
      = 150 + level * (30 + (0.3 * sta))
   ```

* Assist, Acrobat, Magician
   ```js
   hp = 150 + (level * 28) + (sta * level * 0.28)
      = 150 + level * (28 + (0.28 * sta))
   ```

* Knight
   ```js
   hp = 150 + (level * 40) + (sta * level * 0.4)
      = 150 + level * (40 + (0.4 * sta))
   ```

* Ringmaster
   ```js
   hp = 150 + (level * 34) + (sta * level * 0.34)
      = 150 + level * (34 + (0.34 * sta))
   ```

* Billposter
   ```js
   hp = 150 + (level * 36) + (sta * level * 0.36)
      = 150 + level * (36 + (0.36 * sta))
   ```

* Ranger
   ```js
   hp = 150 + (level * 32) + (sta * level * 0.32)
      = 150 + level * (32 + (0.32 * sta))
   ```

### max hp

* hp

   * flatMaxHp : From Character's Gear, Buff unscaled `maxhp`.

   ```js
   hp = (baseHealth * (1 + maxHp%)) + flatMaxHp
   ```

</details>

## ⛔ block

<details>
  <summary>📁 block details</summary>

>

> source:[@shayminhunter @TeachMeHisty (discord flyff universe)](https://discord.com/channels/778915844070834186/1000058902576119878/1266532805651726346 "@shayminhunter @TeachMeHisty (discord flyff universe)")

* You will still get hit, but you'll take significantly less damage. Secondary effects such as crowd control, debuffs, or Sword Cross can still be triggered even if the hit is blocked.

> source:[v1.2.0 Reborn is coming on March 13!](https://universe.flyff.com/news/reborn120 "v1.2.0 Reborn is coming on March 13!")

* Blocked hits no longer deal 1 damage at the minimum, but 20% of the initial damage instead.

> source:[Flyffulator/src/calc/mover.js/getBlock](https://github.com/Frostiae/Flyffulator/blob/7e6b38dc458bffd9edb5e5e6e96237bfe6ae3b51/src/calc/mover.js#L103 "Flyffulator/src/calc/mover.js/getBlock")

### calculate

* Defender is Player.

* block chance

   * block failure : `6 / 80 = 7.5%`

   * block success : `5 / 80 = 6.25%`

   * Further calculate the block rate : `69 / 80 = 86.25%`

   * If reaching the maximum block%, the block chance is **`6.25% + 86.25% = 92.5%`.**

* blockRate (random values is `6 ~ 74`, total of `69` possible values)
   ```js
   blockRate = ((PlayerDex / 8.0) * classBlockModifier) + fAdd + ExtraBlock
   // if blockRate < 0.0 , then 0.0

   // ------------------------------------------------------------------------------------
   // classBlockModifier = GetJobPropFactor( JOB_PROP_BLOCKING )
   ```

   * fAdd
      ```js
      fblockA = PlayerLevel / ((PlayerLevel + AttackerLevel) * 15.0)
      fblockB = clamp(Math.floor((PlayerDex + AttackerDex + 2) * ((PlayerDex - AttackerDex) / 800.0)), 0, 10)
      // fblockB Limited to 0.0 ~ 10.0

      fAdd = fblockA + fblockB
      // if fAdd < 0.0 , then 0.0
      ```

   * ExtraBlock
      ```js
      block% + DST_BLOCK_RANGE%DST_BLOCK_MELEE%
      // ------------------------------------------------------------------------------------
      // block%
      // if IsRangeAttack = rangedblock%, DST_BLOCK_RANGE%
      // if not IsRangeAttack = meleeblock%, DST_BLOCK_MELEE%
      // ------------------------------------------------------------------------------------
      ```

* character window block

   ```js
   CharacterWindowBlock = ((PlayerDex / 8.0) * classBlockModifier) + fblockB + ExtraBlock
   ```
   * The block rate displayed in the character window assumes that your enemies's level is the same as yours and that they have 15 dex, which can make your block rate seem higher than it really is.
      ```js
      // simple formula in Excel
      // A1 : Player's Dex
      // A2 : classBlockModifier
      // A3 : Attacker's Dex (same level enemies Dex, in character window is always 15)

      CharacterWindowBlock =MIN(MAX(MIN(MAX(ROUNDDOWN((A1+A3+2)*((A1-A3)/800), 0), 0), 10)+ROUNDDOWN(((A1/8)*A2), 0), 0), 100)
      ```

### block cap

<div align="center"><img src="./formulas/block_rate_translation_table.png" alt="block_rate_translation_table.png"/></div>

> source:[@bluechromed @[Dev] Blukie (discord flyff universe)](https://discord.com/channels/778915844070834186/1000058902576119878/1085622720575852654 "@bluechromed @[Dev] Blukie (discord flyff universe)")

* 75% is still the block cap. For those reading this and wondering why you may see a higher % in your stat window, it’s because you can technically have more block % but it caps at 75%. Block is rolled out of 80, so 75% block = 75/80 = 93.75% chance to block.

> source:[@bluechromed @[Dev] Blukie (discord flyff universe)](https://discord.com/channels/778915844070834186/1076577520301903984/1174839023383085080 "@bluechromed @[Dev] Blukie (discord flyff universe)")

* The cap is 75% and it’s divided by 80 instead of 100. So you end up with 92.5% block (even though it says 75%). Anything above that is only useful again enemies that have block penetration.

### block penetration

> source:[@frostiae @[Dev] Frostiae (discord flyff universe)](https://discord.com/channels/778915844070834186/867043266162458654/1272345376720158841 "@frostiae @[Dev] Frostiae (discord flyff universe)")

* It makes your target's block rate `block rate * (1 - your block penetration)`

* Block penetration only affects PvP damage.

</details>
