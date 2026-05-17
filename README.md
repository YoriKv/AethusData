# AETHUS — extracted data

Static dumps of the game's `DT_*` DataTables in three forms, plus a browser
viewer for crafting margins.

**Last updated:** May 16, 2026 · game version **1.029**

## `schemas.json`

Per-DataTable schema for every loaded `DT_*` object. One entry per table,
keyed by table name:

```json
{
  "DT_CraftingRecipes": {
    "struct": "S_CraftingRecipe",
    "size":   232,
    "fields": [
      { "name": "OutputResource_43_A546D54E...", "type": "NameProperty",
        "off": 160, "size": 8, "dim": 1 },
      …
    ]
  }
}
```

`type` is the UE5 reflection class name (`IntProperty`, `MapProperty`,
`StructProperty`, …). `off` is the byte offset within the row struct;
`size` × `dim` is the field's total footprint.

Property names retain their UE editor GUID suffix (`_18_8A1DCC91…`). Strip
with `_(\d+)_[0-9A-Fa-f]+$` if you want semantic names.

## `decoded/`

One JSON file per DataTable. Primitives (`Int*` / `Float` / `Double` / `Bool`
/ `Byte` / `Enum`) decode inline. Complex properties come back as typed
envelopes:

- `FName` → `{ "_type": "FName", "comp_idx": 1595711, "number": 2 }` — the
  in-memory `(ComparisonIndex, Number)` pair. UE5 convention: `number == 0`
  means no suffix; `number == N > 0` renders as `_(N-1)`.
- `FString` → `{ "_type": "FString", "data_ptr": …, "num": …, "max": … }` —
  heap pointer + length, not dereferenced
- `ArrayProperty` / `MapProperty` → `_raw_hex` of the `FScriptArray` /
  `FScriptMap` struct
- `ObjectProperty` / `ClassProperty` → `{ "_type": …, "ptr": <address> }`

Use this when you need typed primitive values or raw pointers / indices.

## `decoded_text/`

The same DataTables, but every field is a fully-resolved export-text string
produced via `FProperty::ExportTextItem`:

- `FName`s render as plain names (`"AntiGravTech_1"`)
- `ArrayProperty` / `MapProperty` expand: `(("Ore_10", 3), ("Coal_1", 2))`
- `StructProperty` recurses
- `TextProperty` keeps its `NSLOCTEXT("ns", "key", "Display Text")` form

Easier to grep, no pointer-chasing required. Use this for analysis and
tooling unless you specifically need typed primitives.

## `pricing.html`

Standalone static viewer for the crafting-margin matrix joined from
`DT_InventoryItems` × `DT_CraftingRecipes`. Open the file in any browser —
no server, no build step. Requires `pricing-data.js` alongside it.

Columns: **item** · **sell** value · output **qty** · **cost / unit** ·
**profit / unit** · expand toggle.

- Click any row marked `CRAFT` to see its recipe(s). Multi-recipe items get
  `SELECTED` / `ALT` badges — `SELECTED` is the cheapest fully-priced recipe
  and drives the row's profit number; `ALT`s are shown for reference.
- Search box matches item name, display name, recipe ID, and ingredient
  names. Press `/` to focus, `Esc` to clear.
- Filter chips (`Craftable` / `Sellable` / `Profitable`) stack with AND.
- Click column headers to sort; numeric columns default to descending.
- `+N unpriced` on a row warns that N ingredient(s) have no sell value, so
  the recipe's full cost can't be computed and `profit / unit` is omitted.

## Provenance

All three JSON datasets are dumped live from a running game via in-process
UE5 reflection — they reflect the current build snapshot.
