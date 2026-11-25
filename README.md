# Malaysia Neighbourhood Features

Open neighbourhood boundaries for Malaysia: the taman, bandar and kampung polygons with the
names and aliases locals use. Starting in the Klang Valley — Kuala Lumpur, Putrajaya and
Selangor — and growing.

**Licence:** [CC BY 4.0](LICENSE) · **Format:** GeoJSON (WGS 84) · Corrections and additions welcome

## Why this exists

Malaysia's published boundary layers stop at *negeri*, *daerah* and *mukim*. The units people
actually use to say where they live — Taman Melawati, Bandar Sri Damansara, Kampung Batu Belah
— have names in every listing and no geometry anywhere public. Anyone building a property site,
a delivery zone, a census mashup or a school-catchment map has to draw them again from scratch.

This is that layer, released so nobody has to redraw it.

## What's in the file

A single `FeatureCollection`, `malaysianeighbourhoodsfeatures.geojson`, in **CRS84 / WGS 84**
(longitude, latitude — the GeoJSON default). Every feature is a `MultiPolygon`.

| Property | Type | Example | Notes |
|---|---|---|---|
| `id` | integer | `1559` | Stable across releases. Cite this when reporting a correction. |
| `name` | string | `KAMPUNG BATU BELAH` | The neighbourhood's primary name, upper case. |
| `alias` | array of strings | `["ARA DAMANSARA"]` | Other names the same place is known by. Often empty — see *Known gaps*. |
| `area` | string | `BATU BELAH, KLANG UTARA, KAPAR` | The larger locality this sits inside — see below. |
| `state` | integer | `10` | See the state codes below. |

### What `area` means

`name` answers *what is this place called*. `area` answers *whereabouts is it* — and that
answer is cultural, not official. **An area is the place people commonly associate the
neighbourhood with: where a local would say it is.** No authority defines these, no line is
drawn anywhere, and the naming is entirely by common usage. Taman Perindustrian OUG is in
`JALAN PUCHONG`; Presint 12 is in `PUTRAJAYA`; Kampung Batu Belah is in
`BATU BELAH, KLANG UTARA, KAPAR, OFF JALAN SUNGAI PUTUS`.

Four things worth knowing about it:

- **A neighbourhood can belong to more than one area, and often does.** This is not a hierarchy
  and the order carries no meaning: it is a list of the places people associate it with. Two
  residents of the same street can place it differently, and both are right — a taman on the
  edge of Kapar and Klang Utara is in both. Most neighbourhoods carry one area; 812 carry two,
  and a few carry up to five, comma-separated.
- **It is a controlled vocabulary, not free prose.** Across the whole dataset there are 353
  distinct area names, so you can group, filter and facet by them.
- **An area can be a town, a locality or a council zone.** They are not *daerah*, *mukim* or
  postcodes. Some are towns (`RAWANG`, `KAJANG`, `SUNGAI BULOH`), some are localities everyone
  knows (`BATU CAVES`, `SETAPAK`, `MONT KIARA`), and some are council zones that happen to be
  what people say (`SHAH ALAM ZON UTARA`, `KLANG SELATAN`, `PETALING JAYA UTARA`). The only
  thing they have in common is that people use them.
- **Where a locality straddles a state line, the side is marked in brackets.** Cheras is one
  continuous place, but the Kuala Lumpur–Selangor boundary runs straight through it, so areas
  read `CHERAS (KL)` and `CHERAS (SELANGOR)`. The bracket says which side of the line a
  neighbourhood sits on; it does not mean there are two different Cheras.
- **Some entries include an access road.** Where a place is genuinely hard to locate any other
  way, the area records how you get to it — `OFF JALAN SUNGAI PUTUS`. About 50 entries do this.

**Confidence varies, and it is lowest in the kampung and rural entries.** In built-up areas the
association is usually settled — everyone agrees Mont Kiara is Mont Kiara. Out in the kampung
and the rural districts, both things get soft at once: which area a place is associated with,
and where the neighbourhood itself begins and ends. Names are used loosely, edges run into
plantation and smallholding rather than a road, and the people who could settle it do not
always agree with each other. Treat those entries as the best current reading rather than a
fact, and tell us when you know better — that is exactly where local knowledge beats anything
we can do from a desk.

Two neighbourhoods currently have no area. If you know the locality they belong to, that is a
useful correction to file.

### State codes

`state` is a numeric code. [`states.csv`](states.csv) lists all sixteen Malaysian states and
federal territories with their codes and whether they appear in this release, so the codes stay
stable as coverage grows. The three present today:

| Code | State |
|---|---|
| `10` | Selangor |
| `14` | Kuala Lumpur |
| `16` | Putrajaya |

## Coverage

This release covers the Klang Valley and the rest of Selangor. Everything else is not started.

| Area | Neighbourhoods | Status |
|---|---|---|
| Kuala Lumpur | 479 | good coverage |
| Putrajaya | 19 | good coverage |
| Selangor — Petaling | 597 | good coverage |
| Selangor — Hulu Langat | 565 | good coverage |
| Selangor — Gombak | 381 | good coverage |
| Selangor — Klang | 372 | good coverage |
| Selangor — Sepang | 152 | good coverage |
| Selangor — Kuala Langat | 83 | partial |
| Selangor — Hulu Selangor | 44 | thin |
| Selangor — Kuala Selangor | 41 | thin |
| Selangor — Sabak Bernam | 1 | barely started |
| Everywhere else | — | not started |

Counts are for the current release and will change. The goal is national coverage; the order
depends on where the demand and the help come from.

## Using it

**QGIS** — drag the `.geojson` onto the canvas. No projection setup needed.

**Convert to GeoPackage or Shapefile**

```bash
ogr2ogr -f GPKG neighbourhoods.gpkg malaysianeighbourhoodsfeatures.geojson
ogr2ogr -f "ESRI Shapefile" neighbourhoods_shp malaysianeighbourhoodsfeatures.geojson
```

**Python**

```python
import geopandas as gpd

gdf = gpd.read_file("malaysianeighbourhoodsfeatures.geojson")
klang_valley = gdf[gdf["state"].isin([10, 14, 16])]
print(gdf.loc[gdf["name"].str.contains("MELAWATI"), ["id", "name", "area"]])
```

**PostGIS**

```bash
ogr2ogr -f PostgreSQL PG:"dbname=yourdb" malaysianeighbourhoodsfeatures.geojson \
  -nln neighbourhoods -lco GEOMETRY_NAME=geom -nlt MULTIPOLYGON
```

**Point lookup** — the common case is "which neighbourhood is this coordinate in":

```sql
SELECT name, area
FROM neighbourhoods
WHERE ST_Covers(geom, ST_SetSRID(ST_MakePoint(101.7121, 3.1570), 4326));
```

## Attribution

CC BY 4.0 asks for credit. Use this, with a link back to the repository:

> Neighbourhood boundaries © Trovaprop, CC BY 4.0

Short enough for a map corner. On a web map, the data attribution control is the right place.

## What this is *not*

- **Not official boundaries.** These are not JUPEM, cadastral, administrative or electoral
  units, and carry no legal standing.
- **Not survey-accurate.** Edges sit where people say a neighbourhood ends, which is frequently
  vague and occasionally disputed. Two neighbours can disagree about which taman a corner lot
  belongs to, and both can be right.
- **Not safe for legal, valuation or boundary-dispute use.** Use the official sources for that.

What it is good for: grouping, searching, filtering, display, catchment analysis and anything
else where "roughly this area, by the name people use" is the honest requirement.

## Corrections and additions

Local knowledge is the only way this gets accurate, and you have more of it than we do for your
own neighbourhood. Open an issue:

- **Wrong name or spelling** — give the `id` and the name it should be.
- **Missing alias** — the other name people call it by.
- **Boundary wrong** — describe where it should run (roads, rivers, landmarks). A screenshot
  with the line drawn on it is ideal.
- **Neighbourhood missing** — name, roughly where, and what bounds it.
- **Duplicate or merged** — two `id`s for one place, or one polygon covering two.

Evidence helps: a signboard photo, a local council map, a developer's site plan, or simply
"I've lived here fifteen years and it's called this". Say which, so we can weigh it.

**Please don't send pull requests that edit the geometry.** A boundary change in a GeoJSON diff
is thousands of changed coordinates that no reviewer can verify. The master copy lives in a
PostGIS database; corrections are applied there and ship in the next release, with the reporter
credited in the release notes. Pull requests for the README, examples or tooling are very
welcome.

Contributions are accepted under the same licence as the project, CC BY 4.0.

## Roadmap

Coverage grows state by state. The Klang Valley came first because that is where the work
started, not because the rest matters less.

If you want a particular state prioritised — or your organisation needs it — say so in an
issue. Sponsorship that funds a state is credited in that state's release.

## Releases

Each release is tagged, and `id` values are stable across them, so you can diff two releases to
see what changed. Watch the repository to be told when a state lands.

## Licence

The data in this repository is licensed under
[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).
You may use it commercially, modify it and redistribute it, provided you give credit.

Version 4.0 is used deliberately: it is the first Creative Commons version that explicitly
covers *sui generis database rights*, which matters for a dataset rather than a creative work.
