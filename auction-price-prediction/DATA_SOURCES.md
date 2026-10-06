# Auction Price Prediction: Preliminary Data Source Research

*Hedonic models for art, watches and other luxury goods. APMA E4903 seminar project. Research date: 2026-10-06.*

## How this was researched, and confidence tags

All of this research was done with web search. The sandbox's network proxy blocked direct access to christies.com, sothebys.com, phillips.com, ha.com, artnet.com, kaggle.com, jstor.org, columbia.edu and several data vendors. As a result:

- **Website terms of service and robots.txt files were not read directly.** Statements about them come from secondary sources.
- Each item is tagged:
  - **[V]**: verified from a primary or official page.
  - **[S]**: from search snippets or secondary sources.
  - **[U]**: uncertain. Check by hand before relying on it.
- **The reference paper (JSTOR stable/20108831) could not be identified**, because JSTOR is blocked and the ID didn't turn up in search. We need its title and authors to line up our variables with it.

---

## 1. The constraint that decides everything: we must be able to share the data

The APMA 4903 instructions (found via search, since the PDF itself was blocked) require public links for three things:

- the slides,
- the code (Colab or GitHub),
- **the data, so the work can be re-run** (e.g. posted on Google Drive).

That means the key question is not "can we get the data". It is **"can we legally post it publicly?"** Here is where things stand:

- **No auction house or commercial aggregator gives a licence to redistribute.** Artnet, Artprice, MutualArt, askART, Invaluable, LiveAuctioneers, EveryWatch, WatchCharts, Liv-ex, Classic.com and Hagerty all either prohibit export or charge for access with no redistribution rights.
- **Scraping auction house websites is a legal grey area.** Heritage Auctions' terms explicitly ban "database scraping." An arbitrator ordered Christie's subsidiary Collectrium to pay Heritage about $1.8M for scraping roughly 3M listings ([artnet News](https://news.artnet.com/art-world/collectrium-heritage-data-theft-lawsuit-1612843)) [S].
- **Facts and creative content are treated differently.** In the US, bare facts (price, date, dimensions, reference number) generally can't be copyrighted. Catalogue essays and photos can. In the EU there is an additional database right, which affects Dorotheum, Lempertz, Artcurial and Ketterer.

**Proposed pattern (get the instructor to sign off):**

1. Publish the scraper code.
2. Publish a **facts-only derived table**: house, sale, lot, date, artist or brand, reference number, dimensions, medium or material, estimate low/high, price, sold or bought-in flag, and source URL.
3. Do **not** publish descriptions or images.

The other option is to publish only the code and pipeline and keep the raw data private.

---

## 2. Art: what exists

### 2a. Auction houses' own websites (free to view, no API)

| House | What's online | How to get it | Notes |
|---|---|---|---|
| **Christie's** | Results from about 1998 [U]: artist, title, medium, dimensions, estimates, price realised (includes buyer's premium), images, provenance. Unsold lots show no price. | Scrape the internal JSON search endpoint. | Robots.txt reportedly blocks search pages and `apim.christies.com` [S]. Free-text parsing needed. |
| **Sotheby's** | Similar fields. Newer results often need a free login [U]. | JS-heavy scrape. | Hardest of the big three. |
| **Phillips** | **Per-sale results PDFs at predictable URLs** (e.g. `dist.phillips.com/content/web/auction-results/auctionResultsFile_NY010225.pdf`) [S], plus lot pages that show estimates. | PDF parsing plus lot pages. | **Easiest modern source.** The PDF states the premium schedule, so the hammer price can be backed out. |
| **Bonhams, Dorotheum, Lempertz, Artcurial, Ketterer, Swann, Doyle** | Free results archives. | Apify scrapers already exist [S]. | EU database right applies to the European houses. |
| **Heritage** | 4M+ lots, free login. | **Do not bulk-scrape** (see the litigation above). | |
| **Poly / China Guardian** | Chinese-language sites. | Artron/AMMA (paid, Chinese only). | Skip. |

### 2b. Commercial aggregators (buy or subscribe, all non-redistributable)

| Source | Coverage | Cost | Verdict |
|---|---|---|---|
| **Artnet Price Database** | 18M+ results since about 1985, 1,800+ houses [S] | $32.50/day; $450–1,175/yr [S]. **Ask Columbia Libraries whether CLIO or Avery has a licence** [U]. | Good for manual spot-checks. Bulk export is forbidden. |
| **Artprice** | About 30M results [U] | €24/day up to about €500+/yr; data licensing about €7.5k/yr [S] | Aggressive about its IP. Avoid. |
| **MutualArt** | About 20 years of results | $279–2,499/yr [S] | No export. |
| **askART, Invaluable, LiveAuctioneers, Barnebys, Masterworks** | Millions of lots | Free to cheap, manual browsing | No redistribution. Masterworks/Barnebys index levels can be cited. |
| **Artory** | 50M+ records | Business-to-business only | No. |
| **Blouin Art Sales Index** | Was the standard academic source | **Discontinued** ([Princeton library](https://faq.library.princeton.edu/econ/faq/11354)) [S] | Not available. |
| **Artsy API** | Artist metadata only, no prices | Free, but **being retired** [S] | Not a price source. |
| **Sotheby's Mei Moses, ArtTactic, Pi-eX, Art Basel/UBS report** | Index levels and aggregate figures | Free to paid | Use for benchmarks and macro context only. |

### 2c. Free, open or academic datasets (art)

- **Getty Provenance Index: Sales Catalogs** ([github.com/thegetty/provenance-index-csv](https://github.com/thegetty/provenance-index-csv)) **[V]**
  - **Licence: CC0** (public domain).
  - Size: about 1M+ lot records and 22k sales.
  - Fields: artist, title, dimensions, materials, subject, price, estimate, buyer and seller, and **sold / bought-in / passed status**.
  - Downside: historical only. Coverage is Britain, France, Netherlands and Belgium from about 1670 to 1840, and Germany from 1900 to 1945. Prices are in historical currencies.
  - **This is the only large lot-level art dataset we can legally redistribute.**
- **MoMA** ([collection](https://github.com/MuseumofModernArt/collection)) and **Met** ([openaccess](https://github.com/metmuseum/openaccess)) **[V]**
  - CC0 artist and artwork metadata, including Wikidata and ULAN IDs.
  - **Feature enrichment only** (e.g. "artist held by MoMA", artist birth/death, nationality). They contain no prices.
- **AppraiSet** (Mendeley Data, DOI 10.17632/2nfvz8g27c.1) [S]
  - 10k artworks, 22 variables from auction records, built specifically as an open dataset.
  - Licence probably CC BY [U]. **Check this first.**
- **Figshare "Buying a Work of Art or an Artist?"** [S]
  - 34.2k auction sales of 590 living contemporary artists, 1996–2012, with images.
  - Licence [U].
- **Replication packages from published papers**
  - Beggs & Graddy (AER 2009), openICPSR 113297 [S]: link to it, don't re-host.
  - Bocart & Hafner (J. Applied Econometrics 2015), ZBW/JAE data archive [S]: per-artist sets such as Renoir (1.8k) and Matisse (441).
- **Studies whose data is closed** (useful as methods to copy, not as data)
  - Renneboog & Spaenjers "Buying Beauty" (1.1M sales from Blouin).
  - Mei & Moses (AER 2002, plus a 2025 deep-learning follow-up).
  - Korteweg, Kräussl & Verwijmeren (RFS 2016).
  - Aubry, Kräussl, Manso & Spaenjers "Biased Auctioneers" (JF 2023; 1.09M sales with an image neural net; [open-access PDF](https://openaccess.city.ac.uk/id/eprint/35368/)).
- **GitHub and Kaggle student scrapes** [S]
  - Examples: `jasonshi10/art_auction_valuation` (37.6k lots), `marcusrprojects/What-Makes-Art-Valuable` (Christie's and Sotheby's).
  - These have no licence and are ToS-encumbered. Use them for benchmarking or to bootstrap the parser, not to re-host.

---

## 3. Watches: what exists

### 3a. Auction houses

| House | Coverage and fields | Access | Notes |
|---|---|---|---|
| **Phillips** (Geneva/NY/HK, with Bacs & Russo) | **The best structured watch data.** Fields: maker, model, **reference number**, year, case material, movement, dimensions, condition, box and papers, estimates, sold price; unsold flag. Strong from about 2014. Estimated 15–25k lots. | Free website, scrape. | **Rank 1 for watches.** |
| **Christie's** | Watch lots with estimates and price. Reference and box/papers sit in free text. | Scrape the JSON endpoint. | Needs text parsing. |
| **Sotheby's** | Similar to Christie's. | Login and JS. | Harder. |
| **Antiquorum** | Archive going back 50+ years. | Free site (an Apify scraper exists). | Good for long time series. |
| **Monaco Legend, Loupe This, Bonhams, Dr. Crott** | Smaller, specialist sales. | Free sites or PDFs. | Loupe This includes bid histories. Dr. Crott is in German. |
| **Heritage** | Timepieces and luxury accessories. | **Anti-scraping terms, enforced in the litigation above.** | Avoid bulk collection. |
| **eBay sold listings** | High volume, noisy. | **API discontinued** (Finding API shut down Feb 2025; Marketplace Insights covers 90 days and needs business approval) [V]. | Skip. |

### 3b. Aggregators, indices and spec databases

| Source | What it is | Cost / access | Verdict |
|---|---|---|---|
| **EveryWatch** | 500k+ auction results back to 1989, from 250+ houses [S] | About $49/mo or $500/yr, no API | **The closest ready-made lot-level database.** Email them and ask for academic access and permission to redistribute. |
| **WatchCharts API** | Model-level market prices and indices (not individual lots) | API about $5k–8k/yr plus a separate distribution licence [V] | Too expensive. Use the free charts to validate. |
| **Chrono24 ChronoPulse** | Free index of final sale prices for 140 models [S] | No API | Benchmark only. |
| **Subdial / Bloomberg Subdial Index, Bob's Watches Rolex report, Knight Frank Luxury Investment Index** | Index levels | Free to cite | Use as macro covariates or targets. |
| **WatchBase** | Specs for about 41.7k references (calibre, case, complications, production years) | Paid DataFeed API [S] | Useful for **joining specs onto lots by reference number**. Not redistributable. |
| **AWS Data Exchange "Historical Luxury Watch Auction Sales by Lot"** | Christie's, Sotheby's, Bonhams, Phillips | Price and licence unknown [U] | Worth checking on the listing page. |

### 3c. Academic precedents and open data (watches)

- **Ulmer, Schmid & Widenhorn**, *J. Investment Strategies* ([RePEc](https://ideas.repec.org/a/rsk/journ6/7959635.html)) [V]. More than 60k auction results from 1999 to 2020, hedonic model, 5.5% real annual return. **The closest template to follow.** Email the authors about their data.
- **Mayer**, Princeton senior thesis, 2022 ([DataSpace](https://dataspace.princeton.edu/handle/88435/dsp01jw827f83x)) [V]. Repeat-sales index on 9.2k Rolex, AP and Patek auction results. **An undergraduate precedent;** check its data appendix for sources.
- **Kaggle Chrono24 sets** (e.g. `vittoriohaardt/rolex-on-chrono24`, `beridzeg45/watch-prices-dataset`) [S]. These are **asking prices, not auction prices**. Check each licence.

---

## 4. Other luxury goods (if we widen scope)

- **Classic cars: Bring a Trailer.** **The richest open-ish data in any category.**
  - About 230k+ completed auctions since 2014.
  - Fields: year, make, model, VIN, mileage, bid count, comments; includes an **unsold (reserve-not-met) flag**.
  - Public pages, and open-source crawlers already exist on GitHub.
  - Terms of service [U].
  - Cheap API alternative: oldcarsdata.com (free tier, then $49/mo) [V].
- **Sneakers: StockX 2019 Data Contest** ([link](https://stockx.com/news/the-2019-data-contest/)) [V].
  - 99,956 sales, **publicly released for analysis**, widely mirrored.
  - The most shareable luxury resale dataset found.
  - Only two product lines and few features.
- **Wine**
  - Liv-ex and Wine-Searcher APIs are trade-only (€450+/mo) [S].
  - The **Ashenfelter Bordeaux dataset** is a classic teaching set, freely circulated, but tiny (27 vintages).
- **Whisky:** Rare Whisky 101 index levels are free; lot-level data is not.
- **Handbags:** the same houses as watches (Christie's, Sotheby's, Heritage). Rebag Clair has no API.
- **Diamonds:** the ggplot2 `diamonds` set (54k stones). Retail prices; useful only as a teaching baseline for hedonic models.

---

## 5. Cross-reference: which candidate is best?

Scored on the course's needs:

- **Sharable:** can the data be redistributed?
- **Modern:** does it cover the recent market?
- **Hedonic features:** how many characteristics per item?
- **Unsold flag:** do we know which lots failed to sell? This matters for selection bias.
- **Size**
- **Effort:** how hard is it to collect?

| Candidate | Sharable | Modern | Hedonic features | Unsold flag | Size | Effort | Cost |
|---|---|---|---|---|---|---|---|
| **A. Phillips watches (own scrape)** | ◐ facts-only | ✅ 2014–26 | ✅✅ ref, material, year, box/papers | ✅ | 15–25k | Medium | Free |
| **B. Phillips + Christie's art (own scrape, PDFs)** | ◐ facts-only | ✅ | ✅ artist, medium, size, year | ✅ | 50k+ | Medium–high | Free |
| **C. Getty Provenance Index + MoMA/Wikidata** | ✅ CC0 | ❌ historical | ✅ | ✅ | 1M+ | Low–medium | Free |
| **D. Bring a Trailer cars** | ◐ | ✅ | ✅✅ | ✅ | 230k | Low (existing crawlers) | Free |
| **E. AppraiSet / Figshare / JAE art sets** | ✅? (licence to confirm) | ◐ ≤2012–2022 | ✅ | ◐ | 1–34k | Low | Free |
| **F. EveryWatch (academic request)** | ❓ only if they grant it | ✅ | ✅ | ✅ | 500k | Low | ~$500/yr or grant |
| **G. Artnet via library** | ❌ | ✅ | ✅ | ✅ | 18M | Manual only | Library |
| **H. Kaggle Chrono24** | ◐ check licence | ✅ | ✅ | ❌ (asking prices) | 45–280k | Low | Free |

### Recommendation

1. **Primary dataset: Phillips watch auctions (Candidate A).**
   - Watches are the better choice of the two categories for a hedonic model.
     - **Watches are near-fungible:** a reference number pins down most of the characteristics, so the regression is far better identified than for one-of-a-kind paintings.
     - **Phillips data is the cleanest and most structured** of any free source.
   - Parse the free text into features (brand, reference, year, material, complications, box/papers) with regex or an LLM.
   - Join specs on reference number.
   - Use ChronoPulse and Knight Frank index levels as market covariates.
2. **Art component: Phillips and Christie's modern art scrape (Candidate B)**, with a **CC0 backbone (Candidate C)**.
   - The Getty data provides a fully shareable, very large historical hedonic and repeat-sales exercise.
   - It's also a strong fallback if the instructor won't accept scraped modern data.
   - Enrich artists with MoMA, Met and Wikidata features.
3. **Optional third category: Bring a Trailer.** It is the largest structured luxury dataset with an unsold flag, and crawlers already exist.
4. **In parallel, send emails:**
   - **EveryWatch:** academic data access plus permission to redistribute.
   - **Ulmer et al.:** their 60k watch dataset.
   - **Columbia Libraries:** whether we have Artnet or askART for spot-checks.
   - **The professor:** whether "code + facts-only table, raw data private" satisfies the reproducibility rule.

### Modelling notes for later

- **Price definition.** Price realised includes the buyer's premium, and the premium schedule changes over time. Convert to hammer price, or control for the schedule.
- **Selection bias.** Bought-in lots have no price. Use a Heckman or Tobit-style correction, or model "sold" separately. Korteweg et al. (RFS 2016) is the reference.
- **Pre-sale estimates as a benchmark.** The auction house's low/high estimate is a strong baseline forecast. A "beat the estimate" test is a clean, publishable framing.
- **Currency.** Normalise to USD and deflate with CPI.

---

## 6. Using DeepWiki to speed up the scraper work

[DeepWiki](https://deepwiki.com) generates documentation for any public GitHub repo and lets you ask questions about it: swap `github.com` for `deepwiki.com` in the repo URL. *It was blocked in the research sandbox, so it has not been tried on these repos yet.*

Its best use for this project is reading existing scrapers and datasets **before writing our own**: which endpoints they call, which fields they pull, and what they do about pagination and logins. Repos to try:

| Repo | What to ask DeepWiki |
|---|---|
| `thegetty/provenance-index-csv` | Schema of the `sales_catalogs` tables, how lots join to sales, and how price, currency and transaction type are encoded |
| `MuseumofModernArt/collection`, `metmuseum/openaccess` | Artist fields available for joins (Wikidata, ULAN IDs) |
| `marcusrprojects/What-Makes-Art-Valuable` | Christie's and Sotheby's endpoints, and how fields are parsed and cleaned |
| `bradgwest/paap` | Christie's scraping pipeline (the repo is archived) |
| `ventositwaitang/Sotheby-s-and-Christie-s` | Sotheby's scraping method |
| `jasonshi10/art_auction_valuation`, `ahmedhosny/theGreenCanvas` | Feature engineering, including image features |
| `gisturiz/BaT-Auction-Crawler`, `KaledDahleh/bring-a-trailer-tracker` | Bring a Trailer results scraping |
| `philmorefkoung/Webscrapped-Watch-Dataset` | Watch field parsing (reference number, material) |

DeepWiki also runs a free MCP server, `https://mcp.deepwiki.com/mcp`, so Claude can query these repos directly once it is added as a connector in an environment where it isn't blocked.

---

## 7. Open items to resolve by hand (blocked in this sandbox)

- [ ] Identify the reference paper JSTOR 20108831 (title and authors).
- [ ] Read the terms of service and robots.txt of Phillips, Christie's, Sotheby's and Bring a Trailer.
- [ ] Confirm the licences of AppraiSet, the Figshare dataset and the Kaggle sets.
- [ ] Check the price and licence of the AWS Data Exchange watch-lots listing.
- [ ] Check whether Sotheby's prices require a login.
- [ ] Ask a Columbia librarian about Artnet and askART access.
