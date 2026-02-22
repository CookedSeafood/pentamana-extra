# Pentamana Extra

Pentamana Extra is a library providing mana calculation modifying features. Examples include:

- Enchantments.
- Status effects.
- Modifiers.

## Configuration

`config/pentamana-extra.json`:

```json
{
  "enchantmentCapacityBase": 2.0,
  "enchantmentStreamBase": 0.015625,
  "enchantmentManaEfficiencyBase": 0.1,
  "enchantmentPotencyBase": 0.5,
  "statusEffectManaBoostBase": 4.0,
  "statusEffectManaReductionBase": 4.0,
  "statusEffectInstantManaBase": 4.0,
  "statusEffectInstantDepleteBase": 6.0,
  "statusEffectManaPowerBase": 3.0,
  "statusEffectManaSicknessBase": 4.0,
  "statusEffectManaRegenerationBase": 50,
  "statusEffectManaInhibitionBase": 40,
  "shouldConvertExperienceLevel": false,
  "experienceLevelConversionBase": 0.5
}
```

### Usage

`enchantmentCapacityBase`, `statusEffectManaBoostBase`, `statusEffectManaReductionBase`, `shouldConvertExperienceLevel`, `experienceLevelConversionBase` are used by the formula below:

```java
float f = 0.0f;

f += (float)livingEntity.getCustomModifiedValue(PentamanaAttributeIdentifiers.MANA_CAPACITY, capacity.doubleValue());

for (Entry<Holder<Enchantment>> entry : livingEntity.getEnchantments(Enchantments.CAPACITY)) {
    f += CONFIG.enchantmentCapacityBase * (entry.getIntValue() + 1);
}

f += statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_BOOST)
    ? CONFIG.statusEffectManaBoostBase * (statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_BOOST) + 1)
    : 0.0f;
f -= statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_REDUCTION)
    ? CONFIG.statusEffectManaReductionBase * (statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_REDUCTION) + 1)
    : 0.0f;
f += CONFIG.shouldConvertExperienceLevel && livingEntity instanceof ServerPlayer
    ? CONFIG.experienceLevelConversionBase * ((ServerPlayer) livingEntity).experienceLevel
    : 0.0f;
f = Math.max(f, 0.0f);
```

`enchantmentStreamBase`, `statusEffectInstantManaBase`, `statusEffectInstantDepleteBase`, `statusEffectManaRegenerationBase`, `statusEffectManaInhibitionBase` are used by the formula below:

```java
float f = 0.0f;

f += (float)livingEntity.getCustomModifiedValue(PentamanaAttributeIdentifiers.MANA_REGENERATION, regeneration.doubleValue());

for (Entry<Holder<Enchantment>> entry : livingEntity.getEnchantments(Enchantments.STREAM)) {
    f += CONFIG.enchantmentStreamBase * (entry.getIntValue() + 1);
}

f += statusEffectManager.contains(PentamanaStatusEffectIdentifiers.INSTANT_MANA)
    ? CONFIG.statusEffectInstantManaBase * (float) Math.pow(2.0, statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.INSTANT_MANA))
    : 0.0f;
f -= statusEffectManager.contains(PentamanaStatusEffectIdentifiers.INSTANT_DEPLETE)
    ? CONFIG.statusEffectInstantDepleteBase * (float) Math.pow(2.0, statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.INSTANT_DEPLETE))
    : 0.0f;
f += statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_REGENERATION)
    ? 1.0f / Math.max(1, CONFIG.statusEffectManaRegenerationBase >> statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_REGENERATION))
    : 0.0f;
f -= statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_INHIBITION)
    ? 1.0f / Math.max(1, CONFIG.statusEffectManaInhibitionBase >> statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_INHIBITION))
    : 0.0f;
```

`enchantmentManaEfficiencyBase` is used by the formula below:

```java
float f = 0.0f;

f += (float)livingEntity.getCustomModifiedValue(PentamanaAttributeIdentifiers.MANA_CONSUMPTION, consumption.doubleValue());

for (Entry<Holder<Enchantment>> entry : livingEntity.getEnchantments(Enchantments.MANA_EFFICIENCY)) {
    f *= 1.0f - CONFIG.enchantmentManaEfficiencyBase * (entry.getIntValue() + 1);
}
```

`enchantmentPotencyBase`, `statusEffectManaPowerBase`, `statusEffectManaSicknessBase` are used by the formula below:

```java
float f = 0.0f;

f += (float)livingEntity.getCustomModifiedValue(PentamanaAttributeIdentifiers.CASTING_DAMAGE, damage.doubleValue());

for (Entry<Holder<Enchantment>> entry : livingEntity.getEnchantments(Enchantments.POTENCY)) {
    f += CONFIG.enchantmentPotencyBase * (entry.getIntValue() + 1);
}

f += statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_POWER)
    ? (statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_POWER) + 1) * CONFIG.statusEffectManaPowerBase
    : 0.0f;
f -= statusEffectManager.contains(PentamanaStatusEffectIdentifiers.MANA_SICKNESS)
    ? (statusEffectManager.getActiveAmplifier(PentamanaStatusEffectIdentifiers.MANA_SICKNESS) + 1) * CONFIG.statusEffectManaSicknessBase
    : 0.0f;
f *= entity instanceof Witch
    ? 0.15f
    : 1.0f;
f = Math.max(f, 0.0f);
```

## Command

The commands below require premission level 2 to execute.

- `/custom effect give <entities> <effect> [<duration|infinite>] [<amplifier>]` Give custom status effect. `effect` can be `pentamana.mana_boost`, `pentamana.mana_reduction`, `pentamana.instant_mana`, `pentamana.instant_deplete`, `pentamana.mana_regeneration`, `pentamana.mana_inhibition`, `pentamana.mana_power`, `pentamana.mana_sick`.

## Modifier

Modifiers can be added to or removed from items using custom data components. They are active when equipped in the written slot.

![Compound](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/compound.png) **custom_data**: Parent tag.  
&ensp;\\- ![List](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/list.png) **modifiers**  
&emsp;&emsp;\\- ![Compound](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/compound.png)  
&emsp;&emsp;&emsp;&ensp;|- ![String](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/string.png) **attribute**: `namespace:path`. Can be `pentamana:mana_capacity`, `pentamana:mana_regeneration`, `pentamana:mana_consumption`, `pentamana:casting_damage`.  
&emsp;&emsp;&emsp;&ensp;|- ![Double](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/double.png) **base**: Any.  
&emsp;&emsp;&emsp;&ensp;|- ![String](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/string.png) **id**: Any.  
&emsp;&emsp;&emsp;&ensp;|- ![String](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/string.png) **operation**: Can be `add_value`, `add_multiplied_base`, `add_multiplied_total`.  
&emsp;&emsp;&emsp;&ensp;\\- ![String](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/string.png) **slot**: Can be `mainhand`, `offhand`, `feet`, `legs`, `chest`, `head`.

Below is an example modifier which increase mana capacity by 120 when held in offhand.

```component
[
  custom_data={
    modifiers: [
      {
        attribute: "pentamana:mana_capacity",
        base: 120.0d,
        operation: "add_value",
        slot: "offhand"
      }
    ]
  }
]
```

## Status Effect

Status effects can be added to or removed from items using custom data components. They are applied when the item is consumed.

![Compound](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/compound.png) **custom_data**: Parent tag.  
&ensp;\\- ![List](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/list.png) **status_effects**  
&emsp;&emsp;\\- ![Compound](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/compound.png) A status effect.  
&emsp;&emsp;&emsp;&ensp;|- ![String](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/string.png) **id**: `namespace:path`. Can be `pentamana:mana_boost`, `pentamana:mana_reduction`, `pentamana:instant_mana`, `pentamana:instant_deplete`, `pentamana:mana_regeneration`, `pentamana:mana_inhibition`, `pentamana:mana_power`, `pentamana:mana_sickness`.  
&emsp;&emsp;&emsp;&ensp;\\- ![List](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/list.png) **episodes**: Unordered episodes.  
&emsp;&emsp;&emsp;&emsp;&emsp;\\- ![Compound](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/compound.png) An episode.  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;|- ![Int](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/int.png) **amplifier**: Any.  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;\\- ![Int](https://github.com/CookedSeafood/nbtsheet/raw/0cfc19cc5644a82c921d39f9c40729aca3dea33d/int.png) **duration**: Any. In ticks.

Below is an example status effect which increase the mana regeneration by 16 when the item is consumed.

```component
[
  custom_data={
    status_effects: [
      {
        id: "pentamana:instant_mana",
        episodes: [
          {
            duration: 1,
            amplifier: 2
          }
        ]
      }
    ]
  }
]
```

### Mana Boost

Increase mana capacity by `level * statusEffectManaBoostBase`.

### Mana Reduction

Decrease mana capacity by `level * statusEffectManaReductionBase`.

### Instant Mana

Increase mana regeneration by `2 ^ level * statusEffectInstantManaBase`.

### Instant Deplete

Decrease mana regeneration by `2 ^ level * statusEffectInstantDepleteBase`.

### Mana Regeneration

Increase mana regeneration by `manaPerPoint / statusEffectManaRegenerationBase >> level`

### Mana Inhibition

Decrease mana regeneration by `manaPerPoint / statusEffectManaInhibitionBase >> level`

### Mana Power

Increase casting damage by `level * statusEffectManaPowerBase`.

### Mana Sickness

Decrease casting damage by `level * statusEffectManaSicknessBase`.

## Enchantment

### Capacity

- Maximum level: II
- Primary items: Stick
- Secondary items: Axe, Hoe, Mace, Pickaxe, Shovel, Sword, Trident
- Enchantment weight: 2

Capacity adds extra mana capacity `level * enchantmentCapacityBase`.

### Stream

- Maximum level: II
- Primary items: Stick
- Secondary items: Axe, Hoe, Mace, Pickaxe, Shovel, Sword, Trident
- Enchantment weight: 5

Stream adds extra mana regeneration by `level * enchantmentStreamBase`.

### Mana Efficiency

- Maximum level: V
- Primary items: Stick
- Secondary items: Axe, Hoe, Mace, Pickaxe, Shovel, Sword, Trident
- Enchantment weight: 5

Mana Efficiency reduces the casting mana cost by `level * enchantmentManaEfficiencyBase` percent.

### Potency

- Maximum level: V
- Primary items: Stick
- Secondary items: Axe, Hoe, Mace, Pickaxe, Shovel, Sword, Trident
- Enchantment weight: 10

Potency adds the casting damage by `(level + 1) * enchantmentPotencyBase`.
