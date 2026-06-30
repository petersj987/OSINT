# OSINT Geolocation with Overpass Turbo

## What is Overpass Turbo?

Overpass Turbo is a web-based query interface for the **Overpass API**, which sits on top of the OpenStreetMap (OSM) database. Unlike a normal map search where you need to already know a place's *name*, Overpass Turbo lets you search by **tagged attributes** — brand, building type, road class, amenity type, etc. This makes it a core OSINT geolocation tool: when investigating an image or video with no place name attached, you can search by the *visual/structural clues* instead (a storefront brand, a distinctive building shape, a road type) and get every matching candidate location plotted on a map.

This is the reverse of normal navigation:
- **Normal map search:** "I know the name → show me the place."
- **Overpass Turbo:** "I know the pattern/shape/brand → tell me where it is."

### Why it matters for geolocation OSINT
If given a video frame showing a distinctive gas station canopy and an unusual road junction, scrolling Google Maps hoping to visually spot it is slow and unreliable. Querying for that brand/shape inside a bounded area returns every candidate location instantly, which you then cross-reference against the junction shape and surrounding building footprints visible in the source image.

---

## Key Concepts

### Query Language: Overpass QL
Queries are written in Overpass QL, not natural language. Core structure:

1. **Output settings** — format (`json`/`xml`) and timeout.
2. **Search area definition** — either a named admin area or a bounding box.
3. **Query body** — a union block of `node` / `way` / `relation` filters by tag.
4. **Output directives** — what to return (tags only, full geometry, etc).

### OSM Geometry Types
- `node` — a single point (e.g. a shop location)
- `way` — a line or polygon (e.g. a road, a building outline)
- `relation` — a grouped set of nodes/ways (e.g. a multi-building complex)

### Useful Tags for OSINT
| Tag | Use case |
|---|---|
| `amenity` | Schools, hospitals, fuel stations, places of worship |
| `shop` / `brand` | Retail chains — fast way to narrow a region/country |
| `building` | Building type/shape matching |
| `highway` | Road classification — useful for matching road markings/lane patterns |
| `man_made` | Towers, masts, water towers, chimneys |
| `leisure` | Stadiums, parks, sports pitches |

### Bounding Box vs Named Area
Named-area queries (`area["name"="X"]`) rely on OSM having a clean admin boundary for that name. For informal areas like a city's "CBD," this often doesn't exist as a tagged boundary, so a **manual bounding box** (`south, west, north, east` coordinates) is more reliable for tightly scoped urban searches.

---

## Exercise 1: Stadiums in Kenya (intro exercise)

Stadiums make a good first exercise because they have very distinctive aerial shapes you can cross-check against satellite imagery.

```overpassql
[out:json][timeout:25];
area["name"="Kenya"]["admin_level"="2"]->.searchArea;
(
  way["leisure"="stadium"](area.searchArea);
  relation["leisure"="stadium"](area.searchArea);
);
out body;
>;
out skel qt;
```

### Breakdown
- `area["name"="Kenya"]["admin_level"="2"]->.searchArea;` — defines the country as the search boundary and stores it as a reusable variable.
- `way[...]` / `relation[...]` — stadiums are typically mapped as polygons (way) or grouped complexes (relation), not single points.
- Output trio (`out body; >; out skel qt;`) returns tags, recurses to pull in referenced geometry, then outputs a compact skeleton for map rendering.

---

## Exercise 2: KFC Branches in Nairobi CBD (case study)

### Goal
Find all KFC locations within the Nairobi Central Business District using brand tags, with a fallback name-match for any branches not properly tagged with `brand`.

### Query Used

```overpassql
[out:json][timeout:25];
(
  node["brand"="KFC"](-1.295,36.815,-1.275,36.835);
  way["brand"="KFC"](-1.295,36.815,-1.275,36.835);
  node["name"~"KFC",i](-1.295,36.815,-1.275,36.835);
);
out body;
>;
out skel qt;
```

### Query Breakdown
- `(-1.295,36.815,-1.275,36.835)` — bounding box (south, west, north, east) covering Nairobi CBD and slightly beyond, to catch boundary-adjacent branches.
- `node["brand"="KFC"]` / `way["brand"="KFC"]` — primary search using the properly tagged franchise brand field.
- `node["name"~"KFC",i]` — regex fallback (case-insensitive via `,i`) to catch entries where contributors only filled the `name` field and skipped `brand`.
- `out body; >; out skel qt;` — standard output trio: return full tag data, recurse to pull in referenced geometry, then output a compact skeleton suitable for map rendering.

### Result
Executed in the Overpass Turbo browser UI (overpass-turbo.eu). Status bar confirmed:

```
Loaded — nodes: 2, ways: 0, relations: 0
Displayed — pois: 2, lines: 0, polygons: 0
```

Two KFC branches were returned as point markers within the CBD bounding box:
1. Near the **Kimathi Street / Tubman Road** junction.
2. Near **Mama Ngina Street / Standard Street**, close to KCB & Diamond Trust Bank.

Both locations were tagged as `nodes` (single-point POIs), with no `way` or `relation` results — meaning no KFC location in this area had a mapped building footprint, only a point marker.

### Observations
- All results came back via the `brand` tag match; the `name~` fallback didn't surface additional untagged entries in this case, suggesting decent OSM tagging consistency for this brand in Nairobi.
- A 2-result return for a CBD this size is a reasonable real-world count, not a sign of incomplete OSM coverage — worth cross-checking against Google Maps to confirm no branches are missing from OSM data.
- This pattern (brand/name dual-query inside a manual bounding box) is reusable for any franchise/POI hunt in an urban core where named-area boundaries are unreliable.

---

## Summary: When to Use What

| Task | Tool/Method |
|---|---|
| Exploratory query-building, visual confirmation | Overpass Turbo browser UI |
| Validated query, automation, scripting, batch runs | `curl` against `https://overpass-api.de/api/interpreter` |
| Country/region-level search | Named `area["name"="X"]` |
| City district / informal boundary search | Manual bounding box (south, west, north, east) |
| Matching a brand/storefront seen in an image | `brand` tag + `name~` regex fallback |
| Matching building/structure shape | `way`/`relation` + `out geom;` for full polygon coordinates |

---

## Next Steps for Practice
- Swap `brand=KFC` for other chains (e.g. fuel stations, banks) to compare OSM tagging completeness across categories.
- Try `out geom;` instead of `out skel qt;` to pull full polygon geometry for `way`/`relation` results — useful when matching building *shape*, not just location.
- Cross-reference returned coordinates against satellite imagery (Google Earth / Sentinel Hub) to validate hits visually, the way a real geolocation investigation would.
- Build a small script to pipe Overpass JSON results into a Folium/Leaflet map for local interactive review.

## Tools
- **Overpass Turbo** — https://overpass-turbo.eu (browser UI)
- **Overpass API** — `https://overpass-api.de/api/interpreter` (raw HTTP endpoint, same query language)
```
