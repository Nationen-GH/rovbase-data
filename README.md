# Rovbase — carnivore records as one GeoJSON

Norwegian and Swedish large-carnivore records (lynx, wolverine, bear, wolf, golden eagle),
enriched with the damage-assessment text that [rovbase.no](https://www.rovbase.no) shows in its
map tooltip, as a single standardized GeoJSON for client-side maps.

**Updated daily, automatically.** Rebuilt whenever [Rovbase's GBIF
dataset](https://ipt.gbif.no/resource?r=rovbase) publishes a new version. Each commit is tagged
with the GBIF version it came from (`gbif-v1.352`).

## Files

| File | Size | What |
|---|---|---|
| `rovbase.geojson` | ~87 MB | RFC 7946 FeatureCollection, EPSG:4326, one Point feature per record |
| `rovbase.geojson.gz` | ~5 MB | The same bytes, gzip -9. Fetch this one |
| `rovbase.csv` | ~31 MB | Same records and properties as a flat table with `lat`/`lon` columns |
| `versions.jsonl` | — | One line per upstream publish: date, GBIF version, row count, checksum |

```
https://raw.githubusercontent.com/bergea1/rovbase-data/main/rovbase.geojson.gz
```

There are **no pre-filtered variants** (the old `rovbase-no-1y.csv` and friends are gone). Filter
client-side on `land`, `dato`/`aar`, `art_id` and `datatype`.

## FeatureCollection

The collection carries metadata as top-level members alongside `features`:

| Member | Example |
|---|---|
| `name` | `rovbase` |
| `generated` | `2026-09-08T13:35:12Z` — build time, UTC |
| `gbif_version` | `1.352` |
| `count` | `91089` — equals `features.length` |
| `source` | `https://ipt.gbif.no/resource?r=rovbase` |
| `attribution` | `Rovbase / Miljødirektoratet` |
| `licence`, `licence_url` | `CC BY 4.0` |

Each feature has `id` set to the Rovbase catalogue number (same as `properties.id`) and a Point
geometry in `[lon, lat]` order, 6 decimals.

## Properties

Names are Norwegian snake_case. Every raw enum id (`*_id`) sits next to its Norwegian label.
Values are typed; anything absent or empty upstream is `null`, never `""`. In the CSV, `null` is
an empty cell and booleans are `true`/`false`.

| Property | Type | Notes |
|---|---|---|
| `id` | string | Rovbase catalogue number. `K…` = livestock damage, `M…` = dead carnivore |
| `datatype` | string | `Rovviltskade` (K) or `DodeRovdyr` (M) |
| `title` | string/null | e.g. `Rein skadet av gaupe`, `Død gaupe` |
| `comment` | string/null | Full assessment, e.g. `Ett dyr er  drept. Skaden er undersøkt av SNO. …` |
| `art_id`, `art` | int/null, string/null | Carnivore: 1 Ulv, 2 Bjørn, 3 Gaupe, 4 Jerv, 6 Kongeørn, 7 Ukjent fredet rovdyr. **No 5** |
| `bytte_id`, `bytte` | int/null, string/null | Animal damaged (K only): 1 Sau, 2 Rein, 3 Hund, 4 Geit, 5 Storfe |
| `vurdering_id`, `vurdering` | int/null, string/null | Assessment: 1 Dokumentert, 2 Antatt sikker, 3 Usikker, 4 Feilmelding |
| `tilstand_id`, `tilstand` | int/null, string/null | State of the damaged animal (K only): 1 drept, 2 skadd og avlivet, 3 skadd og ikke avlivet, 4 savnet |
| `skadested_id`, `skadested` | int/null, string/null | Where (K only): 1 Utmark, 2 Innmark, 3 Inngjerdet innmark, 4 Område med rovviltavvisende gjerde |
| `undersokt_av_id` | int/null | Inspected by (K only): 1 SNO, 2 Sysselmester, 3 Länsstyrelsen, 4 Sameby, 5 not inspected in the field |
| `hendelse_id` | string/null | Rovbase incident id (K only), `V…` |
| `dodsarsak_id` | int/null | Cause of death (M only): 1 lisensfelling, 2 skadefelling, 3 nødverge (dyr), 4 SNO-oppdrag, 5 dyrevelferd, 6 politibeslutning, 7 drept av dyr, 8 forskning, 9 ulovlig avliving, 10 påkjørt tog, 11 påkjørt bil, 12 nødverge, 13 ulykke, 14 sykdom, 15 ukjent, 16 kvotejakt |
| `utfall_id` | int/null | Outcome (M only): 1 felt/død, 2 skadeskutt bekreftet, 3 skadeskutt ukjent |
| `kjonn_id`, `kjonn` | int/null, string/null | Sex (M only): 1 Hann, 2 Hunn, 3 Ukjent |
| `alder` | int/null | Age in years when upstream gave a plain integer (M only) |
| `alder_tekst` | string/null | Age exactly as recorded upstream; free text like `1-2` or `unge` in some records |
| `helvekt`, `slaktevekt` | number/null | Whole / carcass weight in kg (M only). `null` when 0 or missing |
| `individ_id`, `individ_navn` | string/null | Identified individual, e.g. `BI040180` / `BD022 +` (M only) |
| `yngling` | boolean/null | Breeding animal (M only) |
| `dato` | string/null | Event date, `YYYY-MM-DD` |
| `aar` | int/null | Year of `dato` |
| `modified` | string/null | Upstream last-modified date, `YYYY-MM-DD` |
| `kommune` | string/null | Municipality, without the `(N)`/`(S)`/`(F)` suffix |
| `land` | string/null | `NO`, `SE` or `FI` |
| `funnsted` | string/null | Locality |
| `url` | string | `https://www.rovbase.no/search?T=<id>` |

Fields that belong to the other record type are always `null`: a dead carnivore has no `bytte`,
a damage record has no `kjonn`.

Two quirks worth knowing:

- The double space in `Ett dyr er  drept` is upstream's own — reproduced deliberately so the text
  matches rovbase.no character-for-character.
- ~74 records carry `null` `title`, `comment`, `kommune` and `land`. They exist in the Darwin Core
  export but were withdrawn from the live Rovbase API. They're kept so the record set stays
  complete; coordinates and locality names place them in Sweden.

## Licence

Data: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), as published by Rovbase /
Miljødirektoratet through GBIF. Credit "Rovbase / Miljødirektoratet".
