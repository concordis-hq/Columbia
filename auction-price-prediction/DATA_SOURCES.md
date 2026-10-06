# Auction Price Prediction: Data Source Research

*Hedonic and ML models for watch, art and luxury-goods auction prices. APMA E4903 (Fall 2026) seminar project.*
*Research date: 2026-10-06. First pass done from search results; verified against the live sites the same day.*

**Confidence tags:**
- **✅ verified**: checked live on the primary source (page, file, robots.txt or terms of service).
- **[S]**: from secondary sources.
- **❓ unknown**: blocked by a bot wall or login, or not checked.

No CAPTCHAs were bypassed and no logins were used.

---

## TL;DR

1. **The course rule decides the data choice.** The APMA 4903 instructions require that *"Data used should also be world-readable (e.g., uploaded to Google drive so that the Colab or GitHub can be run reproducibly)."* ✅ So the key question is not "can we get it" but **"can we post it publicly?"**
2. **We cannot republish data from Christie's, Sotheby's, Bring a Trailer, EveryWatch or any commercial database.** Christie's and Sotheby's terms of service explicitly forbid scraping and reuse. ✅
3. **Shareable datasets that exist today:**
   - **Watches:** two CC0 Kaggle sets of *dealer asking prices*.
   - **Art:** a **CC BY 34.2k-sale art auction dataset with estimates** (figshare), the **CC0 Getty Provenance Index** (historical, 1M+ lots) and AppraiSet (CC BY).
4. **Best modern watch auction data:**
   - Scrape **Phillips, Antiquorum and Monaco Legend** lot pages. They are visible anonymously, rich in features, and their robots.txt files don't block lot pages.
   - Or use the **free ALT/FNDATA dataset on AWS** (600k watch lots, 1992 to today). It can't be redistributed, but derived results can be shared.
5. **Suggested project:** methods from the art literature, applied mainly to **watches**. A watch reference number pins down the item, and very little has been published on watches, so there is room for something new. Art serves as the comparison case. Details in §7.

---

## 1. Course requirements (APMA 4903, Fall 2026) ✅

Source: [4903-instructions.pdf](https://www.columbia.edu/~chw2/Courses/APMA4903/4903-instructions.pdf)

- Groups of 2–4 people.
- Meet the professor **2 weeks before** the talk with an outline and references.
- Send the materials **1 week before**.
- **Slide 2** lists the 1–3 primary references as standard citations, not URLs.
- **Each presenter needs a "technical nugget."** That is either a derivation, or a live code demo shown alongside its pseudocode. Talks are stopped for "slop" and "PowerPoint karaoke."
- **Slides, code and data must all be world-readable,** so that the work is reproducible.
- **Optional 4th credit point:** an individually written, *replicable* write-up on Medium, Substack, LinkedIn or arXiv. It must link to code and slides, and is due 5 pm one week after the last lecture.

**Reference paper:** JSTOR stable/20108831 is behind a JSTOR CAPTCHA and has no DOI in Crossref or OpenAlex. **We still need its title and authors.**

---

## 2. Is there research on predicting these prices? (art vs watches)

### Art: about 50 years of top-journal research

| Work | Contribution |
|---|---|
| Anderson (1974); Baumol (1986, *AER*) | Art as an investment, with low and volatile returns |
| Goetzmann (1993, *AER*) | Repeat-sales price indices |
| Mei & Moses (2002, *AER*) | Repeat-sales index; masterpieces underperform |
| Ashenfelter & Graddy (2003, *J. Econ. Lit.*) | The standard survey ("Auctions and the Price of Art") |
| Beggs & Graddy (2009, *AER*) | Anchoring effects |
| Renneboog & Spaenjers (2013, *Management Science*) | Hedonic index on 1.1M sales |
| Korteweg, Kräussl & Verwijmeren (2016, *RFS*) | Selection bias from unsold lots |
| **Aubry, Kräussl, Manso & Spaenjers (2023, *J. Finance*)** ([PDF](https://openaccess.city.ac.uk/id/eprint/35368/1/Biased%20Auctioneers%20JF.pdf)) | Neural network on images plus lot features, 1.2M paintings. Its valuations predict price-to-estimate ratios and which lots go unsold; auctioneer errors are persistent. **The ML benchmark to beat.** |

The art literature is rigorous but the problem is hard: every painting is unique, and most of the explained variation comes from the artist's identity. The best datasets (Blouin, Artnet) are proprietary.

### Watches: a thin literature

- **Ulmer, Schmid & Widenhorn (2024)**, *J. Investment Strategies*, [doi:10.21314/jois.2024.006](https://www.risk.net/journal-of-investment-strategies/7959635/). Hedonic model on more than 60k auction results from 1999–2020. The data source is not named in the abstract. ✅
- **Weisskopf & Masset (2025)**, "Time is Money: an Investment in Luxury Watches", SSRN 5119075. Not yet read.
- **Mayer (2022)**, Princeton senior thesis ([link](https://dataspace.princeton.edu/handle/88435/dsp01jw827f83x)). Repeat-sales index on 9,159 Rolex, AP and Patek auction results scraped from collectorsquare.com. ✅
- Everything else is industry reports: BCG (2023), Morgan Stanley with WatchCharts, Knight Frank, Bob's Watches.

### Takeaway

- **Rigour:** art is far ahead.
- **Modelling fit and novelty:** watches are ahead. A reference number pins down most hedonic characteristics, the same reference resells many times, and very little has been published.

---

## 3. Watches: where the data is

### 3a. Auction house websites (free to view, no APIs)

| House | What an anonymous visitor sees | Structure | robots.txt / terms |
|---|---|---|---|
| **Phillips** ✅ | Lot pages such as [Patek 5711/1P](https://www.phillips.com/detail/patek-philippe/178098). Fields: Manufacturer, Year, Reference No, Model Name, Material, Calibre, Dimensions, **Estimate**, **Sold For**. Box and papers appear in the "Accessories" text. The sale page ([CH080123](https://www.phillips.com/auction/CH080123)) lists every lot with reference, estimate and sold price. | Server-rendered React/Remix HTML, so the data is in the page. JSON-LD `offers.price` holds the **low estimate**, not the sold price. **Plain HTTP requests get a 403; a headless browser is needed.** | robots.txt disallows only `/search`, `/*/filter/` and `/bin/`. No website terms of service found. Conditions of Sale claim copyright on catalogue text and images. |
| **Antiquorum** ✅ | Price lists with **hammer and premium-inclusive** prices ([example](https://catalog.antiquorum.swiss/en/auctions/371/price-list)). Lot pages show estimate, sold price, **condition grade** (e.g. "AA"), brand, model, reference, year, diameter and provenance. | Plain server-rendered HTML; the easiest to collect. Records go back 50+ years. | robots.txt blocks only `/users/`. `Crawl-delay: 5`. |
| **Monaco Legend** ✅ | Estimate, premium-inclusive result, reference, year, case material, accessories ([example](https://www.monacolegendauctions.com/auction/exclusive-timepieces-41/lot-10)). | Server-rendered with schema.org JSON-LD. The sitemap lists about 22 sales (numbers 14–43). | Not checked. |
| **Christie's** ✅ | Price realised and estimate are visible; watch details are in free text. | JSON-LD plus a Next.js data stream. | **Terms forbid scraping:** "You will not … use any robot, spider, scripts … to data mine or scrape any of the content". |
| **Sotheby's** ✅ | Estimates are visible, but results show **"Log in to view results"**. | | **Terms forbid "screen scraping," "database scraping"**. `Crawl-delay: 15`. |
| **Heritage** | Blocked by a DataDome CAPTCHA. | | Heritage won about $1.8M from Christie's subsidiary Collectrium over scraping ([artnet](https://news.artnet.com/art-world/collectrium-heritage-data-theft-lawsuit-1612843)) [S]. **Avoid.** |

**Phillips volume ✅:** 101 watch sales since 2016, currently 10–13 a year. That is roughly 1.5–2.5k lots a year and about 15–20k lots in total.

### 3b. Aggregators and paid data

| Source | What it is | Cost | Redistribution |
|---|---|---|---|
| **ALT/FNDATA on AWS Data Exchange** ✅ ([listing](https://aws.amazon.com/marketplace/pp/prodview-jlpgxlg75fmae)) | "Historical Luxury Watch Auction Sales by Lot (Christie's, Sotheby's, Bonhams, Phillips)". Over 600k lots from 240+ houses, 1992 to today, 330k+ realised prices, daily updates. No data dictionary published. | **Free** on AWS. The provider also has a ["For Educators" academy](https://academy.altfndata.com/). | **No.** The subscriber "may not publish, disseminate, distribute … the Data". Derived data is allowed. |
| **EveryWatch** ✅ | Auction aggregator. | $179.88–10,000/yr. No API. | **No.** Terms ban "crawling for Content … Any automated use". |
| **WatchCharts API** | Model-level price indices. | About $5–8k/yr [S]. | ❓ The site is behind a Cloudflare challenge. |
| **Chrono24 / ChronoPulse** | Asking-price marketplace and a free index. | No public API found. | ❓ Behind a Cloudflare challenge. |
| **WatchBase** | Spec database (about 41.7k references). | Paid DataFeed [S]. | No [S]. |
| **eBay** ✅ | | The Finding API was shut down in Feb 2025; Marketplace Insights is restricted and covers only 90 days. | |

### 3c. Shareable watch datasets ✅ (asking prices, not auction prices)

| Kaggle dataset | Licence | Size | Columns |
|---|---|---|---|
| `vittoriohaardt/rolex-on-chrono24` | **CC0** | 87,117 × 12 | model, reference number, price, movement, case material, diameter, year, condition, **scope of delivery** (box/papers), location. Scraped 4 Dec 2022. |
| `yoerireumkens/timepiece-treasures-a-luxury-watches-dataset` | **CC0** | 163,598 × 10 | brand, model, reference, complication, case/bracelet material, dial, price. No source or date fields. |
| `beridzeg45/watch-prices-dataset` | "Other", but no licence text is given | 45,024 × 23 | Treat as **not shareable**. |

---

## 4. Art: where the data is

### 4a. Auction house websites

| House | What an anonymous visitor sees | Notes |
|---|---|---|
| **Christie's** ✅ | Price realised and estimate are visible ([example lot](https://www.christies.com/en/lot/lot-6110562)). The JSON-LD Product block contains the price realised. | **Terms forbid scraping and use in any "database … or compilation".** `window.chrComponents` (which old scrapers used) no longer exists. |
| **Sotheby's** ✅ | Estimates are visible; **results are hidden behind a login**. | Terms forbid scraping. `Crawl-delay: 15`. |
| **Phillips** ✅ | Lot and sale pages show estimate and sold price ([NY010225](https://www.phillips.com/auction/NY010225)). **Results PDFs** list lot number and premium-inclusive price ([example](https://www.dist.phillips.com/content/web/auction-results/auctionResultsFile_NY010225.pdf)). Lots missing from the PDF did not sell. | Needs a headless browser. No website terms of service found. |
| Bonhams, Heritage | ❓ Behind Cloudflare and DataDome bot walls. | |

### 4b. Commercial databases (none can be republished)

| Source | Notes |
|---|---|
| **Artnet Price Database** | 18M+ results; $32.50/day or $450–1,175/yr [S]. Whether Columbia Libraries has access is ❓: CLIO's bot check blocked automated lookup, so **check in a browser**. |
| **Artprice, MutualArt, askART, Invaluable, LiveAuctioneers, Barnebys, Artory** | Paid or manual-only access; no redistribution [S]. |
| **Blouin Art Sales Index** | Discontinued [S]. It was the source behind many papers and datasets. |
| **Artsy API** | Being retired; has no prices [S]. |

### 4c. Shareable and academic art datasets

| Dataset | Licence | What's in it |
|---|---|---|
| **"Buying a Work of Art or an Artist?"** (Springer Nature figshare, [DOI 10.6084/m9.figshare.24746268](https://springernature.figshare.com/articles/dataset/24746268)) ✅ | **CC BY 4.0** | **34,200 auction sales**, 590 living contemporary artists, 1996–2012, 23 countries. Columns: artist, nationality, birth year, artwork year, genre, auction house, `price_usd`, `real_price_usd`, **`estimate_min`/`estimate_max`**, `Ham_Prem` (hammer or premium flag), materials, height/width/depth, image link, plus ML-ready files. Caveat: the records originate from Blouin, but the publisher released them under CC BY. **The best modern, shareable art dataset found.** |
| **Getty Provenance Index: Sales Catalogs** ([GitHub](https://github.com/thegetty/provenance-index-csv)) ✅ | **CC0** | 1M+ lots and 22k sales, about 2.9 GB of CSV on S3. Coverage: 1650–1850 (Britain, France, Netherlands, Belgium, Germany, Scandinavia) and 1900–1945 (Germany, Austria, Switzerland). Columns: `transaction` (Sold / Bought In / Passed / Withdrawn), `price_amount_1..3` (hammer or high bid), `price_currency`, `est_price`, artist, dimensions, buyer and seller; tables join on `catalog_number`. Historical currencies. |
| **AppraiSet** (Mendeley, DOI 10.17632/2nfvz8g27c.1) ✅ | **CC BY 4.0** | 10k artworks, up to 22 variables. Field list ❓ because the Mendeley page returned a 502 error. |
| **MoMA** / **Met** open-access collections ✅ | **CC0** | Artist features to join on: nationality, gender, birth/death, Wikidata QID, ULAN ID. No prices. |
| Beggs & Graddy, openICPSR 113297 ✅ | Depositor's custom licence (text ❓) | Christie's and Sotheby's repeat sales, 1980–94, as Stata files. **Link to it, don't re-host it.** |
| Kaggle "the-price-of-art" ✅ | **Rules forbid reposting** | Estimates, price realised, provenance, literature. Benchmark use only. |
| Bocart & Hafner (JAE archive) ✅ | — | **No data deposited** (Artnet doesn't allow it). |
| GitHub scrapes ✅ (`marcusrprojects/What-Makes-Art-Valuable`, `jasonshi10/art_auction_valuation`, `bradgwest/paap`, `ventositwaitang/Sotheby-s-and-Christie-s`) | No licence, except ventositwaitang (MIT code) | Small (168 to 37.6k rows). The scraping methods are **out of date** for the current Christie's site and rely on endpoints that robots.txt disallows. Reference only. |

---

## 5. Other luxury goods (if we widen scope)

- **Bring a Trailer (classic cars)** ✅
  - 266,857 completed auctions, visible anonymously, with JSON embedded in the results page.
  - **But its terms forbid scraping and explicitly forbid using its data to train "any artificial intelligence or other algorithm … model".** Avoid.
  - Paid alternative: oldcarsdata API, free for 10 requests a month, then $49–699/mo. Its terms don't allow redistribution.
- **StockX 2019 Data Contest (sneakers)** ✅
  - 99,956 sales, still downloadable ([xlsx](https://s3.amazonaws.com/stockx-sneaker-analysis/wp-content/uploads/2019/02/StockX-Data-Contest-2019-3.xlsx)).
  - Released publicly for analysis, but the file carries no licence text.
- **Wine:**
  - Liv-ex and Wine-Searcher are trade-only or enterprise-priced [S].
  - The Ashenfelter Bordeaux set is a freely circulated teaching dataset, but tiny.
- **Whisky:** Rare Whisky 101 publishes index levels only [S].
- **Handbags:** sold by the same houses as watches; same constraints.

---

## 6. Using DeepWiki

[DeepWiki](https://deepwiki.com) answers questions about public GitHub repos. Its **MCP endpoint `https://mcp.deepwiki.com/mcp` works** ✅ (tools `read_wiki_structure`, `read_wiki_contents`, `ask_wiki_question`).

Of the 11 repos we care about, **only `MuseumofModernArt/collection` and `metmuseum/openaccess` are indexed.** The Getty repo and every auction-scraper repo return "Repository not found". To index a public repo, open `deepwiki.com/<owner>/<repo>` in a browser.

DeepWiki becomes more useful once **our own** project repo is on GitHub, for onboarding teammates. For the existing scrapers, the code was read directly (§4c).

---

## 7. Cross-reference and recommendation

| Candidate | Shareable | Modern | Hedonic features | Unsold flag | Size | Effort |
|---|---|---|---|---|---|---|
| **A. Phillips + Antiquorum + Monaco Legend watch scrape** | ◐ code plus facts-only table | ✅ | ✅✅ reference, year, material, grade, box/papers | ✅ | ~20–40k | Medium (headless browser for Phillips) |
| **B. ALT/FNDATA watches (AWS)** | ❌ raw data / ✅ derived results | ✅ 1992–today | ✅ (dictionary unseen) | ? | 600k | Low |
| **C. figshare contemporary art (CC BY)** | ✅ | ◐ 1996–2012 | ✅ with estimates | ❌ (sold lots only) | 34.2k | **Very low** |
| **D. Getty Provenance Index (CC0) + MoMA/Wikidata** | ✅ | ❌ historical | ✅ | ✅ | 1M+ | Low–medium |
| **E. Kaggle Chrono24 watches (CC0)** | ✅ | ✅ 2022 | ✅ | ❌ asking prices | 87k / 164k | Very low |
| F. Christie's / Sotheby's scrape | ❌ terms forbid it | ✅ | ✅ | ✅ | large | High |

### Recommended plan

1. **Watches are the main topic.**
   - Collect Phillips, Antiquorum and Monaco Legend lot pages (Candidate A), politely and following robots.txt.
   - Publish the code plus a **facts-only table**: house, sale, lot, date, brand, reference, year, material, estimates, price, sold flag, URL. Leave out descriptions and images.
   - **Ask the professor** whether this, or "code public, raw data private", satisfies the world-readable rule.
   - Use the **CC0 Chrono24 set (E)** as a fully shareable fallback and to compare asking prices with auction prices.
2. **Subscribe to ALT/FNDATA (B) for free** and use it for model training and validation if its fields hold up. Email them about the educators programme and about sharing permission.
3. **Art is the comparison case.**
   - Use the **figshare CC BY set (C)**: fully shareable, already has estimates, and is ML-ready.
   - Optionally add a historical chapter on the **Getty data (D)**.
   - **Do not scrape Christie's or Sotheby's.**
4. **Model and nugget ideas:**
   - Hedonic log-price regression with time dummies, which also gives a price index. The derivation can be the nugget.
   - A Heckman-style correction for unsold lots (Korteweg et al.).
   - Gradient boosting or a neural net, compared against **the auction house's own estimate as the baseline** ("can we beat the auctioneer?", as in Aubry et al.).
   - Data hygiene: convert premium-inclusive prices to hammer prices, convert to USD, and deflate with CPI.

---

## 8. Open items

- [ ] **Reference paper:** title and authors for JSTOR 20108831 (it's behind a CAPTCHA).
- [ ] Ask the professor whether a facts-only table or private raw data satisfies "world-readable data".
- [ ] Subscribe to ALT/FNDATA on AWS (free) and inspect its fields; email about the educators programme and redistribution.
- [ ] Find a Phillips website terms-of-service page (none found), and check Antiquorum's and Monaco Legend's terms.
- [ ] Check in a browser whether CLIO lists Artnet, askART or Artprice.
- [ ] Get the AppraiSet field list (Mendeley was returning errors).
- [ ] Read Weisskopf & Masset (2025) and the Ulmer et al. working paper to find their watch data sources.
- [ ] WatchCharts and Chrono24 terms (blocked by Cloudflare).
