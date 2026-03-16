# Must-Stay List Configuration

The uhomes Must-Stay list (必住榜) is a curated ranking of top-rated student properties per city. Use this file to look up Must-Stay page URLs.

---

## Must-Stay URL Pattern

```
https://en.uhomes.com/must-stay?city_id={city_id}&school_id=0&ads_type=82
```

- `city_id`: city identifier (see table below)
- `school_id=0`: city-wide ranking (no university filter)
- `ads_type=82`: student accommodation category

## Known City IDs

| City | city_id | Must-Stay URL | Status |
|------|---------|---------------|--------|
| London | 7 | `en.uhomes.com/must-stay?city_id=7&school_id=0&ads_type=82` | Active (10 properties) |

> **Note**: Not all cities have Must-Stay data. If a city is not listed above, skip the Must-Stay fetch. As uhomes expands the Must-Stay list to more cities, add their city_id here.

## How the Skill Uses Must-Stay Data

1. When searching a city that has a known city_id, Skill fetches the Must-Stay page in parallel with the university listing page
2. Properties appearing on both lists receive a scoring bonus (see SKILL.md Step 3 scoring algorithm)
3. Must-Stay properties are tagged with 🏆 in the output
4. If Must-Stay data is unavailable, the Skill proceeds normally — no penalty to any property

## Updating This File

When uhomes expands the Must-Stay list to new cities:
1. Visit `en.uhomes.com/must-stay?city_id={N}&school_id=0&ads_type=82` with different city_id values
2. If the page returns property data (not "No ranking data yet"), add the city_id to the table above
3. Update the Skill version and republish
