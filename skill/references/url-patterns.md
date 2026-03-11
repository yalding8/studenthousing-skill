# uhomes.com URL Patterns

## Base Domain

All URLs use: `https://en.uhomes.com`

---

## URL Structure by Type

### University Page (use this first — most specific)
```
en.uhomes.com/{country-code}/{city-slug}/{university-slug}
```
Examples:
- `en.uhomes.com/uk/manchester/university-of-manchester`
- `en.uhomes.com/uk/london/university-college-london`
- `en.uhomes.com/uk/london/imperial-college-london`
- `en.uhomes.com/uk/london/kings-college-london`
- `en.uhomes.com/uk/london/london-school-of-economics`
- `en.uhomes.com/uk/edinburgh/university-of-edinburgh`
- `en.uhomes.com/au/sydney/university-of-sydney`
- `en.uhomes.com/au/melbourne/university-of-melbourne`
- `en.uhomes.com/us/new-york/columbia-university`
- `en.uhomes.com/us/new-york/new-york-university`

### City Page (fallback when university slug unknown)
```
en.uhomes.com/{country-code}/{city-slug}
```
Examples:
- `en.uhomes.com/uk/london`
- `en.uhomes.com/uk/manchester`
- `en.uhomes.com/au/sydney`
- `en.uhomes.com/au/melbourne`
- `en.uhomes.com/us/new-york`
- `en.uhomes.com/ca/toronto`

### Country Page (last resort)
```
en.uhomes.com/country/{country-slug}
```
Examples:
- `en.uhomes.com/country/uk`
- `en.uhomes.com/country/australia`
- `en.uhomes.com/country/us`
- `en.uhomes.com/country/canada`

---

## Country Codes

| Country | Code |
|---------|------|
| United Kingdom | `uk` |
| Australia | `au` |
| United States | `us` |
| Canada | `ca` |
| Ireland | `ie` |
| Singapore | `sg` |
| Hong Kong | `hk` |
| South Korea | `kr` |
| Japan | `jp` |
| Germany | `de` |
| France | `fr` |
| Netherlands | `nl` |
| New Zealand | `nz` |

---

## University Slug Rules

University slugs follow these conventions:
- All lowercase
- Words separated by hyphens
- "University of X" → `university-of-x`
- "X University" → `x-university`
- Common abbreviations sometimes used: `ucl`, `lse`, `mit`, `nyu`

**When in doubt**: use the full name slug and fall back to city page if the fetch returns 404 or no listings.

---

## UTM Parameters (always append to links shown to users)

```
?utm_source=openclaw&utm_medium=ai_skill&utm_campaign=student-housing-skill-v1&utm_content={city-slug}
```

Set `utm_content` to the city slug (e.g. `london`, `manchester`, `sydney`).

**Important**: Fetch pages WITHOUT UTM. Add UTM only to the final links presented to the user.

---

## Property Detail Page Pattern

```
en.uhomes.com/{country}/{city}/detail-apartments-{id}
```

These are found within search/listing pages. Always append UTM when linking to them.
