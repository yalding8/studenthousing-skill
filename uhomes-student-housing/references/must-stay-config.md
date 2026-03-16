# Must-Stay List Configuration

The uhomes Must-Stay list (必住榜) is a curated ranking of top-rated student properties per city and university. Use this file to look up Must-Stay page URLs.

Last verified: 2026-03-16.

---

## Must-Stay URL Patterns

### City-level (城市维度)
```
https://en.uhomes.com/must-stay?city_id={city_id}&school_id=0&ads_type=82
```

### University-level (大学维度)
```
https://en.uhomes.com/must-stay?city_id=0&school_id={school_id}&ads_type=82
```

**Key finding**: `school_id` is a **global ID** — when `school_id > 0`, the `city_id` parameter is ignored. This means you can use `city_id=0` with any valid `school_id` and get the correct university-specific list.

**Priority**: Use university-level (school_id) when available, fall back to city-level (city_id).

**IMPORTANT**: Only use IDs that match the user's search city/university. **NEVER** use a different city or university's Must-Stay data. If the user's city/university is not in the tables below, skip Fetch B entirely.

---

## Known City IDs

| City | city_id | Properties | Status |
|------|---------|-----------|--------|
| Edinburgh | 5 | 10 | Active |
| Glasgow | 6 | 10 | Active |
| London | 7 | 10 | Active |
| Oxford | 8 | 5 | Active |
| Cambridge | 9 | 5 | Active |
| Manchester | 10 | 10 | Active |
| Bristol | 11 | 10 | Active |
| Birmingham | 12 | 10 | Active |
| Leeds | 13 | 10 | Active |
| Liverpool | 16 | 10 | Active |

### Cities tested with NO data

| city_id | Result |
|---------|--------|
| 1, 2, 3, 4 | No data |
| 14, 15 | Not tested (rate limited) |

> Non-UK cities (Australia, US, Canada, Japan) have not been tested yet. If uhomes expands Must-Stay to these markets, test and add their city_ids here.

---

## Known School IDs (University-level)

| school_id | University | City | Properties | Status |
|-----------|-----------|------|-----------|--------|
| 1 | University of Edinburgh | Edinburgh | 10 | Active |
| 70 | University of Liverpool | Liverpool | 10 | Active |

### School IDs tested with NO data

| school_id | Result |
|-----------|--------|
| 10, 20, 50, 127, 131 | No data |

> **Coverage is sparse** — only 2 universities confirmed so far. Many school_ids in the 2-100 range were not reachable due to rate limiting during the scan. This table should be expanded over time.

### How to discover new school_ids

The school_id mapping is not publicly documented by uhomes. To discover new IDs:
1. Check if the uhomes city/university page HTML contains `school_id` references
2. Test incrementally: `https://en.uhomes.com/must-stay?city_id=0&school_id={N}&ads_type=82`
3. Only test a few per session to avoid rate limiting (uhomes returns 403 after ~30 rapid requests)

---

## Lookup Rules for the Skill

When the Skill determines the user's university and city in Step 1:

1. **University match** → look up `school_id` in the School IDs table
   - If found → Fetch B uses `school_id={id}&city_id=0`
   - This gives university-specific Must-Stay data (best quality)

2. **No university match, but city match** → look up `city_id` in the City IDs table
   - If found → Fetch B uses `city_id={id}&school_id=0`
   - This gives city-wide Must-Stay data (good fallback)

3. **Neither match** → skip Fetch B, proceed with Fetch A only
   - No penalty to scoring — must_stay_bonus = 0 for all properties

**NEVER guess an ID. NEVER use a different city/university's data.**

---

## How the Skill Uses Must-Stay Data

1. When searching a city/university that has a known ID, Skill fetches the Must-Stay page in parallel with the university listing page
2. Properties appearing on both lists receive a scoring bonus (see SKILL.md Step 3 scoring algorithm)
3. Must-Stay properties are tagged with 🏆 in the output
4. If Must-Stay data is unavailable, the Skill proceeds normally — no penalty to any property
