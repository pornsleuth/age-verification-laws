# Examples: using the table at the edge

Built from `age-verification-laws.csv`, snapshot of 25 Nov 2026 (112 rows). Laws change: the [live tracker](https://pornsleuth.com/age-verification-laws) is checked daily and is always newer than these lists. Regenerate them from the CSV before you rely on them.

General information, not legal advice.

## 1. Cloudflare WAF custom rule: places that require an age check

27 US states and 8 countries with `status` = `Age check required`. Use it to block, challenge or route visitors to your age-check flow.

```text
(ip.src.country eq "US" and ip.src.subdivision_1_iso_code in {"US-AL" "US-AR" "US-AZ" "US-FL" "US-GA" "US-IA" "US-ID" "US-IN" "US-KS" "US-KY" "US-LA" "US-MO" "US-MS" "US-MT" "US-NC" "US-ND" "US-NE" "US-OH" "US-OK" "US-SC" "US-SD" "US-TN" "US-TX" "US-UT" "US-VA" "US-WV" "US-WY"}) or (ip.src.country in {"AU" "BR" "DE" "FR" "GB" "IT" "PH" "ZA"})
```

US visitors whose state can't be looked up have an empty `ip.src.subdivision_1_iso_code`. Decide how to treat them; `(ip.src.country eq "US" and ip.src.subdivision_1_iso_code eq "")` matches them.

## 2. Countries where adult sites are prohibited

32 countries with `status` = `Adult sites prohibited`:

```text
(ip.src.country in {"AE" "AF" "BD" "BH" "BY" "CN" "ID" "IN" "IQ" "IR" "JO" "KG" "KP" "KR" "KW" "KZ" "LB" "MY" "NP" "OM" "PK" "QA" "RU" "SA" "SG" "TH" "TM" "TR" "TZ" "UG" "UZ" "VN"})
```

## 3. Watchlist: law passed, not yet in force

- `CH` Switzerland (JSFVG)

## 4. JSON for Workers or other edge middleware

```json
{
  "source": "https://pornsleuth.com/age-verification-laws",
  "license": "CC-BY-4.0",
  "snapshot": "2026-11-25",
  "us_states_age_check": [
    "AL", "AR", "AZ", "FL", "GA", "IA", "ID", "IN", "KS", "KY",
    "LA", "MO", "MS", "MT", "NC", "ND", "NE", "OH", "OK", "SC",
    "SD", "TN", "TX", "UT", "VA", "WV", "WY"
  ],
  "countries_age_check": ["AU", "BR", "DE", "FR", "GB", "IT", "PH", "ZA"],
  "countries_prohibited": [
    "AE", "AF", "BD", "BH", "BY", "CN", "ID", "IN", "IQ", "IR",
    "JO", "KG", "KP", "KR", "KW", "KZ", "LB", "MY", "NP", "OM",
    "PK", "QA", "RU", "SA", "SG", "TH", "TM", "TR", "TZ", "UG",
    "UZ", "VN"
  ],
  "countries_pending": ["CH"]
}
```

In a Cloudflare Worker, `request.cf.country` gives the country and `request.cf.regionCode` the US state (`TX`, no prefix).

## 5. Regenerate the lists from the CSV

```bash
python3 - <<'PY'
import csv
rows = list(csv.DictReader(open("age-verification-laws.csv")))
for status in ["Age check required", "Adult sites prohibited", "Law passed, not yet in force"]:
    states = sorted(r["region"] for r in rows if r["status"] == status and r["region"])
    countries = sorted(r["country"] for r in rows if r["status"] == status and not r["region"])
    print(status, "| states:", " ".join(states), "| countries:", " ".join(countries))
PY
```

## Credit

CC BY 4.0. Please credit: PornSleuth, "[Age verification laws for adult sites, by country and US state](https://pornsleuth.com/age-verification-laws)".
