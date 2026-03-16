# Must-Stay List Configuration

The uhomes Must-Stay list (必住榜) is a curated ranking of top-rated student properties per city. Use this file to look up Must-Stay page URLs.

Last verified: 2026-03-16.

---

## Must-Stay URL Pattern

```
https://en.uhomes.com/must-stay?city_id={city_id}&school_id=0&ads_type=82
```

- `city_id`: city identifier (see table below)
- `school_id=0`: city-wide ranking — **always use 0**. University-level filtering (`school_id>0`) is not yet supported by uhomes (returns empty).
- `ads_type=82`: student accommodation category

**IMPORTANT**: Only use the city_id that matches the user's search city. Never fall back to a different city's Must-Stay list. If the user's city is not in the table below, skip Fetch B entirely.

## Known City IDs

| City | city_id | Properties | Status |
|------|---------|-----------|--------|
| London | 7 | 10 | Active |
| Manchester | 10 | 10 | Active |
| Edinburgh | 5 | 10 | Active |
| Glasgow | 6 | 10 | Active |
| Oxford | 8 | 5 | Active |
| Cambridge | 9 | 5 | Active |
| Bristol | 11 | 10 | Active |
| Birmingham | 12 | 10 | Active |
| Leeds | 13 | 10 | Active |

### Cities tested with NO data (as of 2026-03-16)

| city_id | Result |
|---------|--------|
| 1 | No data |
| 2 | No data |
| 3 | No data |
| 4 | No data |

> Non-UK cities (Australia, US, Canada, Japan) have not been tested. If uhomes expands Must-Stay to these markets, test and add their city_ids here.

## City-to-city_id Lookup Rule

When the Skill determines the user's city in Step 1, look up the city name in the table above:
- **City found** → use its city_id for Fetch B
- **City NOT found** → skip Fetch B, proceed with Fetch A only
- **NEVER guess a city_id** — using the wrong city's Must-Stay list is worse than having no Must-Stay data

## School-level Must-Stay (Future)

As of 2026-03-16, `school_id` filtering is **not functional** on the uhomes Must-Stay endpoint. Tested values (`school_id=127`, `school_id=131` for London universities) all return "No ranking data yet". When uhomes enables this feature:

1. Map university slugs to school_ids
2. Change Fetch B URL to include the correct school_id
3. This will make Must-Stay recommendations university-specific rather than city-wide

## How the Skill Uses Must-Stay Data

1. When searching a city that has a known city_id, Skill fetches the Must-Stay page **for that specific city** in parallel with the university listing page
2. Properties appearing on both lists receive a scoring bonus (see SKILL.md Step 3 scoring algorithm)
3. Must-Stay properties are tagged with 🏆 in the output
4. If Must-Stay data is unavailable for the city, the Skill proceeds normally — no penalty to any property
