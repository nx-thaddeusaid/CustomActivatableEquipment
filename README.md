# CustomActivatableEquipment

A [ModTek](https://github.com/BattletechModders/ModTek) mod for [HBS BattleTech](https://harebrained-schemes.com/battletech/) that adds an activatable equipment state machine. Equipment such as MASC, Stealth Armor, and C3 networks can be toggled on and off mid-combat via a UI dialog, with configurable failure chances, tonnage restrictions, and status effects.

**Upstream repo:** [BattletechModders/CustomActivatableEquipment](https://github.com/BattletechModders/CustomActivatableEquipment)

---

## Features

- **Activation dialog** — Ctrl+click the Move button in the mech HUD to open the activation/deactivation dialog.
- **Heat sink control** — Ctrl+click the Brace button to manually toggle dedicated heat sinks.
- **Failure mechanics** — Per-component fail chance that escalates each round of active use, with configurable damage and critical hit effects on failure.
- **Tonnage/slot restrictions** — Optional chassis tonnage range restrictions per component.
- **Auto-activation by heat** — Components can automatically activate or deactivate when a mech's heat crosses a threshold.
- **Charge-based activation** — Components can consume charges on each use instead of toggling.
- **Auras** — Activatable aura effects (C3, ECM) with AI support.

---

## Building

Requires **.NET Framework 4.7.2** and a local BattleTech game install.

```bash
dotnet build --configuration Release -p:BattleTechGameDir="/path/to/BATTLETECH"
```

Format check (CI enforces this):
```bash
dotnet format whitespace --verify-no-changes source/CustomActivatableEquipment.csproj
```

---

## Mod integration

To make a component activatable, add an `ActivatableComponent` block inside the `Custom` field of the component's JSON def:

```json
{
  "Custom": {
    "Category": [{ "CategoryID": "Activatable" }],
    "ActivatableComponent": {
      "ButtonName": "MASC",
      "FailFlatChance": 0.3,
      "FailRoundsStart": 1,
      "FailChancePerTurn": 0.5,
      "FailISDamage": 10,
      "FailCrit": true,
      "FailDamageLocations": ["LeftLeg", "RightLeg"],
      "statusEffects": [ ... ]
    }
  }
}
```

Key fields:

| Field | Type | Description |
|---|---|---|
| `ButtonName` | string | Label shown in the activation dialog. Keep short. |
| `FailFlatChance` | float | Base failure chance on activation (0.0–1.0). |
| `FailRoundsStart` | int | Round (from activation) when per-round fail checks begin. |
| `FailChancePerTurn` | float | How much fail chance grows per active round. |
| `FailISDamage` | int | Internal structure damage dealt on failure. |
| `FailCrit` | bool | Whether failure also triggers a critical roll. |
| `FailDamageLocations` | string[] | Locations eligible for crit damage on failure. |
| `AutoActivateOnHeat` | int | Heat level that triggers automatic activation. |
| `AutoDeactivateOnHeat` | int | Heat level that triggers automatic deactivation. |
| `ChargesCount` | int | If > 0, uses charge-based activation instead of toggling. |
| `MechTonnageWeightMult` | int | Restricts to chassis tonnage range based on component tonnage. |

The `statusEffects` array uses standard BattleTech status effect JSON and is applied while the component is active.

Full documentation is in the [`Readme.md`](Readme.md) file (legacy format with complete field reference and worked examples).
