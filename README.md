# Global App Store Price Index

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23052645.svg)](https://doi.org/10.5281/zenodo.23052645) [![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

The Global App Store Price Index provides a transparent reference point for cross-border digital price parity. Built on a basket of leading software subscriptions compared like-for-like, it measures how local market pricing deviates from the dollar-standard baseline.

The latest index snapshot (2026-09-30) reveals clear global tiers:

Egypt is the cheapest App Store we track, with the typical subscription 56.2% below its US price. Turkey (−49.0%) and Nigeria (−48.2%) come next. At the other end, Denmark (+25.1%), United Kingdom (+23.7%) and Switzerland (+20.3%) charge more than the US. The data covers 18 countries, 2,320 subscription prices in the index and 3,120 prices in total, compiled by [OpenTheRank](https://opentherank.com/).

Interactive version with a world map: https://opentherank.com/app-price-index/ · How it's calculated: https://opentherank.com/app-price-index/#method · Every version of this dataset: https://opentherank.com/open-data/app-price-index/

## Grab the data

| If you want… | Use |
|---|---|
| One row per country: the index, its spread, the purchasing-power ratio | `data/country_index_OpenTheRank.csv` |
| Which categories (AI, streaming, productivity…) are cheap or dear where | `data/country_category_OpenTheRank.csv` |
| Every individual price | `data/prices_OpenTheRank.csv`, or one country at a time in `data/countries/` |
| The country table across snapshots | `data/country_index_history_OpenTheRank.csv` |
| Everything in one file | `data/app-price-index_OpenTheRank.json` |
| Column types for your tooling | `datapackage.json` (Frictionless Data Package) |

The CSVs are UTF-8 with a byte-order mark, so ₺, ₦ and E£ show up correctly in Excel too. The copies on opentherank.com allow cross-origin requests, so you can load them straight into a notebook:

```python
import pandas as pd
index = pd.read_csv("https://opentherank.com/open-data/app-price-index/files/country_index_OpenTheRank.csv")
prices = pd.read_csv("https://opentherank.com/open-data/app-price-index/files/prices_OpenTheRank.csv")
```

## A few things that stand out

- In 10 of the 18 countries, more than half of the subscriptions we track cost less than in the US.
- Dollar prices are only half the story. Adjust for local price levels using IMF purchasing-power data and India looks very different: its subscriptions are 27% cheaper than in the US in dollars, yet 3.4 times what the country's general price level would suggest. Australia sits closest to its own price level (×1.04).
- Each country is measured on 113 to 145 matched subscriptions. On top of that, 800 App Store game prices feed the mobile-game column of the category table.

## How the index works

Every country goes through the same three steps.

1. **Match the plan.** Each app is tracked at one plan, the one its own pricing page leads with (ChatGPT Plus, Netflix Standard, Spotify Individual and so on). A country's price only counts if its store sells that plan on the same billing cycle as the US store. Prices come from each country's own App Store, or from the vendor's own pricing page for the handful of subscriptions sold direct. Nothing is estimated from another country's price.
2. **Convert and compare.** The local price is converted to US dollars at that day's reference rate and compared with the US price of the same plan: `price_usd / us_price_usd - 1`.
3. **Take the median.** A country's index is the middle value across all its matched apps. One oddly priced app can drag an average around; it barely moves a median. The 10th to 90th percentiles and the share of apps cheaper than the US sit next to it, so you can see how spread out a country is.

A few rules decide what gets in:

- A country needs at least 30 matched subscriptions to be ranked. Below that, a median describes a handful of apps rather than the country. 18 countries clear the bar right now.
- Free listings are dropped. A price more than three times the app's median across countries, or one that doesn't match what the store itself shows, is set aside as "to verify" and left out of every number, and out of these files.
- Sometimes a store lists several entries under one plan name. In those cases `price_note` says which one we used, in the same words as the product page (659 prices in this snapshot).

The purchasing-power column (`ppp_ratio`) compares prices with each country's general price level instead of the US dollar. The price level is the IMF's implied PPP conversion rate divided by the market exchange rate (IMF World Economic Outlook, April 2026). A ratio of 1 means subscriptions are priced in line with that level; 2 means twice what it would suggest. The IMF numbers are annual estimates, so they lag a sudden currency drop.

## Before you use it

- Outside the US and Canada, store prices include VAT or GST. US and Canadian prices are before sales tax. Card fees and your bank's exchange margin aren't included either.
- Each app is tracked at one plan. Its other tiers can be discounted more, or less.
- The basket is a fixed set of widely used AI, streaming, photo and video, productivity, education and social subscriptions, all weighted equally. A country can be cheap for these apps and not for others.
- Exchange rates move every day, so a country's number can change without a single price changing.
- Argentina lists most App Store prices in US dollars, often at exactly the US price, which pulls the median towards 0% (see `usd_billed`).
- Prices are read on a rolling schedule: the biggest apps daily, watched ones every 3 days, the rest every 15 days. `checked_at` tells you when each product was last read.
- This is about prices, not access. Paying in another country's store usually needs a local payment method and billing address.

## Files

| File | Rows | Contents |
|---|---:|---|
| `data/country_index_OpenTheRank.csv` | 18 | Country index |
| `data/country_category_OpenTheRank.csv` | 126 | Category medians |
| `data/prices_OpenTheRank.csv` | 3,120 | Prices, all 18 countries |
| `data/country_index_history_OpenTheRank.csv` | 18 | Country index, every snapshot |
| `data/app-price-index_OpenTheRank.json` | — | All tables and the snapshot metadata as JSON |
| `data/countries/` | 18 files | The prices table split by country: Egypt, Turkey, Nigeria, Pakistan, India, Brazil, Japan, Canada, Mexico, South Korea, Argentina, United States, Australia, France, Germany, Switzerland, United Kingdom, Denmark |

## Columns

### country_index_OpenTheRank.csv

| Column | Type | Description |
|---|---|---|
| `snapshot_date` | date | Day this snapshot was taken (UTC). |
| `country_code` | string | ISO 3166-1 alpha-2 code of the App Store storefront. |
| `country` | string | Country name in English. |
| `rank` | integer | Rank among the 18 countries, 1 = furthest below US prices. |
| `index_pct_vs_us` | number | The index: median percentage difference between this storefront's subscription prices and the US prices of the same plans, both converted to USD at the market rate. Negative = cheaper than the US. |
| `p10_pct` | number | 10th percentile of the same per-plan percentage differences. |
| `p25_pct` | number | 25th percentile. |
| `p75_pct` | number | 75th percentile. |
| `p90_pct` | number | 90th percentile. |
| `cheaper_share` | number | Share of the basket priced below the US (0 to 1). |
| `ppp_ratio` | number | Median of price_usd / (PPP price level x us_price_usd), with the price level from IMF PPP conversion factors over the market rate (US = 1). 1 = in line with the country's general price level; above 1 = dearer than that level would suggest. |
| `basket_n` | integer | Subscriptions in the median: App Store and vendor-website plans with a US price, one plan per product. |
| `usd_billed` | boolean | True when this App Store storefront prices in US dollars, which pulls the index towards 0. |
| `prices_as_of` | date | Most recent day a price in the basket was checked. |
| `fx_date` | date | Day of the exchange rates used for the USD conversions. |
| `ppp_source` | string | Edition of the IMF World Economic Outlook the PPP factors come from. |

### country_category_OpenTheRank.csv

| Column | Type | Description |
|---|---|---|
| `snapshot_date` | date | Day this snapshot was taken (UTC). |
| `country_code` | string | ISO 3166-1 alpha-2 code of the App Store storefront. |
| `category` | string | ai, streaming, photo-video, productivity, education, social or mobile-game. Only categories with at least 5 products in a country are listed. |
| `median_pct_vs_us` | number | Median percentage difference vs the US within the category. Mobile games are App Store games and are not part of the index itself. |
| `n` | integer | Products in the category median. |

### prices_OpenTheRank.csv (and the per-country files)

| Column | Type | Description |
|---|---|---|
| `snapshot_date` | date | Day this snapshot was taken (UTC). |
| `country_code` | string | ISO 3166-1 alpha-2 code of the storefront. |
| `product_slug` | string | Stable product identifier on opentherank.com. |
| `product` | string | Product name. |
| `category` | string | ai, streaming, photo-video, productivity, education, social or mobile-game. |
| `source` | string | app_store: the App Store storefront of that country. vendor_website: the vendor's own pricing page for that country. |
| `plan` | string | The plan compared across countries: one representative plan per product; for mobile games, the in-app item or game purchase compared. |
| `period` | string | Billing period: week, month or year for subscriptions; one-time for most mobile-game rows (an in-app item or a paid game). |
| `price_local` | string | The price exactly as the storefront printed it, in local currency. |
| `currency` | string | ISO 4217 code of price_local. |
| `price_usd` | number | price_local converted to USD at the snapshot's exchange rates (fx_date in the country index and the JSON file). |
| `us_price_usd` | number | The same plan's US price in USD. |
| `pct_vs_us` | number | Percentage difference: (price_usd / us_price_usd - 1) x 100. |
| `in_index` | boolean | True for the subscriptions in index_pct_vs_us; false for App Store games, which only feed the mobile-game category median. |
| `price_note` | string | Set when the storefront lists several purchase entries matching the plan: which entry is published and why, in the words of the product page. Empty otherwise. |
| `checked_at` | date | Day the product's prices were last read from the store or vendor. |
| `page_url` | string | The opentherank.com page this price is published on. |

## License and credit

The data is free to use under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), commercial use included. If you publish something with it, credit OpenTheRank and link to the dataset page when you can. If you changed the data, say so.

Short credit: *Source: OpenTheRank Global App Store Price Index (opentherank.com), CC BY 4.0*

Full citation: OpenTheRank (2026). Global App Store Price Index, snapshot 2026-09-30 (v2026.09.30) [Data set]. https://opentherank.com/open-data/app-price-index/. CC BY 4.0. https://doi.org/10.5281/zenodo.23052645

The license covers our compilation and the figures we derive from it. The prices themselves are set by the stores and vendors, and product names and trademarks belong to their owners. OpenTheRank isn't affiliated with Apple or any of the apps listed. `ppp_ratio` is derived from IMF data (Source: International Monetary Fund, World Economic Outlook Database, April 2026). The exchange rates used for the conversions aren't included. Full terms: https://opentherank.com/terms/#data-license

## About OpenTheRank

[OpenTheRank](https://opentherank.com/) tracks what the same digital product costs from one country to the next: AI subscriptions, streaming, apps, mobile games, Steam and Nintendo eShop titles, and Apple hardware. Every price is read from the store that sells it in that country. You can see where each price comes from, and when every storefront was last read, at https://opentherank.com/data-sources/.

Country reports: [Egypt](https://opentherank.com/app-price-index/egypt/) · [Turkey](https://opentherank.com/app-price-index/turkey/) · [Nigeria](https://opentherank.com/app-price-index/nigeria/) · [Pakistan](https://opentherank.com/app-price-index/pakistan/) · [India](https://opentherank.com/app-price-index/india/) · [Japan](https://opentherank.com/app-price-index/japan/) · [South Korea](https://opentherank.com/app-price-index/south-korea/)

## Versions and corrections

We take a new snapshot when there's reason to, not on a fixed schedule. Each one is a git tag and a GitHub release, archived on Zenodo with its own DOI (the badge above always points to the latest). Published versions are never edited; a fix arrives as a new version. All of them stay downloadable at https://opentherank.com/open-data/app-price-index/.

Spotted a wrong price? Email support@opentherank.com with the app and the country.
