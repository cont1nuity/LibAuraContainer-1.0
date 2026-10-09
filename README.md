# LibAuraContainer-1.0

A LibStub library that lets addons written against Blizzard's retail `AuraContainer` run on
classic clients too. Create containers through the library and use Blizzard's API unchanged.

| client | what `LAC:CreateContainer` returns |
|---|---|
| retail 12.1.0, 12.1.5, WoW Forever | Blizzard's native `CustomAuraContainerTemplate` container, untouched |
| MoP Classic, TBC Anniversary, Classic Era | the library's own implementation, driven by `UNIT_AURA` |

The backend is chosen once at load, by whether the client knows Blizzard's template, never by
build number. The classic container accepts the union of the 12.1.0 and 12.1.5 APIs and
behaves like 12.1.5 where they differ.

## Embedding

Ship the folder inside your addon (for example under `Libs/LibAuraContainer-1.0/`) and load
LibStub first, then the library's embed file, `LibAuraContainer.xml`:

```xml
<Script file="LibStub\LibStub.lua"/>
<Include file="LibAuraContainer-1.0\LibAuraContainer.xml"/>
```

Use your own LibStub if you have one; the bundled copy is for standalone loading.
`LibAuraContainer-1.0.toc` is only for loading the library by itself while developing.

## API in brief

```lua
local LAC = LibStub("LibAuraContainer-1.0")
local container = LAC:CreateContainer(name, parent [, extraTemplates])
```

- `LAC.IsNative` is true when you got Blizzard's container.
- Enums and defaults are exported under Blizzard's key names and values: `LAC.SortMethod`,
  `LAC.SortDirection`, `LAC.AuraDataType`, `LAC.Defaults.Layout`, `LAC.FlowDirection`,
  `LAC.UpdateMode`, `LAC.Inbound` and so on. On native clients each one is Blizzard's own table.
- The container and button methods are Blizzard's; use them as the retail documentation
  describes.

## Differences on classic

Classic lacks some of the APIs the retail container uses, so a few things behave differently:

- **Cancel on click** is not available: `SetCancelAuraButtons` is accepted and does nothing.
  Classic only lets secure code cancel auras and weapon enchants.
- **Pandemic window:** the last 30 % of the aura's duration, worked out from the aura's own
  times, for every aura with an expiry (retail opens one only for auras that have pandemic).
- **Element indices:** `AddDispelTypeTexture` and `AddPandemicRegion` also return 12.1.0's index.
- **Unit swaps:** classic sends no `UNIT_AURA` when a unit token comes to name another unit (a
  new target, say); the container watches the events that re-point its unit and compares the
  unit's GUID instead.
- **Filter tokens:** tokens the client's engine lacks or ignores are decided from aura and spell
  data. On MoP that includes `CROWD_CONTROL`, `BIG_DEFENSIVE`, `EXTERNAL_DEFENSIVE`,
  `RAID_PLAYER_DISPELLABLE` and `RAID_IN_COMBAT` (MoP lists them but does not filter on them). MoP's spell data marks
  no defensive as one, so `BIG_DEFENSIVE` and `EXTERNAL_DEFENSIVE` match nothing there in
  practice. Bleeds carry no dispel type on MoP.
- **Spell-ID filters:** classic's `UnitCanAssist` takes no immune / uninteractable arguments, so
  the identity gate may differ on such units.
- **Caster names** are not shown for a non-player caster that has no unit token (the caster
  GUID is then nil).
- **Templates:** a button or container template's `OnLoad` runs before the library's methods
  exist; configure buttons in `initializeFrame` and containers after `CreateContainer` returns.
- **Container object:** `GetObjectType()` is `"Frame"`, and `GetChildren()` also lists a
  private driver frame.
- **Dispel atlases** fall back to the next atlas the client has, then to the border colour.
- **Duration text:** the colour curve is applied on a timer; the default formatter uses the
  client's own rounding; `SetDurationText`'s `binding` option copies the binding's expired and
  zero-duration texts, time modifier and update interval, not its formatter or text format.
- **Item enchantments** come from `GetWeaponEnchantInfo` (no permanent-enchant flag).
- **Tooltip style options** are checked by the library's own rules.
- **Edit-mode preview** reads the client's fake aura provider through `AuraUtil.ForEachAura`.
- Private auras are never shown (classic has none).
