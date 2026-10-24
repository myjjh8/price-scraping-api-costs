# price scraping api: what it really costs per request, how the credit math works, and how to pick a plan without burning money

A price scraping API is a managed HTTP endpoint that fetches product pages from e-commerce sites and returns prices as structured JSON, so you don't have to run headless browsers, rotate proxies, or fight CAPTCHAs yourself. Most of these services advertise a monthly price plus a pool of "credits" — and that second number is where buyers get burned, because the credit count on a pricing page is almost never the number of requests you actually get.

This article walks through what these APIs cost, how per-request credit pricing works, and what ScraperAPI (one of the oldest players in this space) currently charges across all its plans — including the hidden limits that only show up in the billing docs.

> **Quick picks**
> - Testing the waters: the free plan gives you 1,000 credits with 5 concurrent connections, no credit card needed. Hobby at **$49/month** is the realistic starting point for anything production-adjacent.
> - Tracking Amazon prices specifically: Hobby's 100,000 credits covers **20,000 Amazon product pages** per month (5 credits each). That's enough for daily monitoring of roughly 650 ASINs.
> - Need geotargeting outside US & EU? That gate opens at Business ($299/month), not at the entry tier.
>
> 👉 [Check ScraperAPI's full plan list and current prices](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What a price scraping API actually does

You send a product URL, the service sends back price data. Everything between those two points — proxy rotation, browser rendering for JavaScript-loaded prices, CAPTCHA solving, retries — is handled on their side.

The reason this category exists is that DIY price scraping breaks in predictable ways. Prices render in the browser, not in the initial HTML. Stores show different prices per country. A Shopify store and Amazon use completely different page structures. And most large retailers put anti-bot walls in front of the product page, so a plain HTTP fetch from your own server dies after a few dozen requests.

A managed API bundles all of that into one GET request. The trade-off is that you pay per request, and — this is the part most comparison articles gloss over — different sites cost wildly different amounts per request.

## ScraperAPI plans and prices: the full list

Here is the complete plan lineup currently shown on ScraperAPI's pricing page. Annual billing saves 10% across the board (figures below show monthly price and the annual-billing per-month equivalent).

| Plan | Monthly | Annual (per mo) | API Credits | Concurrency | Buy |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Sign up free, no card needed](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44 | 100,000 | 20 | [Get Hobby at $49/mo](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134 | 1,000,000 | 50 | [See Startup plan details](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269 | 3,000,000 | 100 | [Check Business plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427 | 5,000,000 | 200 | [View Scaling plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877 | 10,500,000 | 300 | [View Professional plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777 | 21,500,000 | 500 | [View Advanced plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales about Enterprise](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Every paid plan includes the same feature set: JavaScript rendering, premium proxies, JSON auto-parsing, CAPTCHA and anti-bot handling, custom sessions and headers, automatic retries, and unlimited bandwidth. What actually separates the tiers is credits, concurrent threads, geotargeting depth, and analytics retention — Hobby and Startup are limited to US & EU geotargeting with a 30-day analytics dashboard, while Business and above get country-level targeting worldwide and unlimited data retention.

New accounts also get a 7-day trial with 5,000 credits and no credit card required, which is the sane way to verify your target sites work before paying anything.

## The credit math most reviews skip

This is the section to read before choosing any plan on that table, because the credit count is a budget, not a request count.

ScraperAPI charges per request based on the target domain:

| Target type | Credits per request |
| --- | --- |
| Standard websites | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |
| Cloudflare / Datadome / PerimeterX bypass | +10 when bypass triggers |

On top of that, request parameters add cost: `render=true` (JS rendering) adds 10 credits, `screenshot=true` adds 10, `premium=true` adds 10, and `ultra_premium=true` adds 30. Combinations stack non-intuitively — `premium` plus `render` costs 25 credits total, and `ultra_premium` plus `render` costs 75. Parameters like `country_code`, `session_number`, and `wait_for_selector` are free.

What does that mean in practice? Run the numbers on the Hobby plan's 100,000 credits at $49/month:

- Standard HTML pages: 100,000 requests (~$0.49 per 1,000)
- Amazon product pages: 20,000 requests (~$2.45 per 1,000)
- Google SERP queries: 4,000 requests (~$12.25 per 1,000)
- JS-rendered pages: 10,000 requests (~$4.90 per 1,000)
- Ultra-premium + JS rendering: about 1,333 requests (~$36.75 per 1,000)

Same plan, a 75x spread in effective per-request cost depending on what you scrape. This is why the starting price comparison tables you see everywhere ("ScraperAPI $49 vs ScrapingAnt $19") are the first line of the spreadsheet, not the conclusion. If your target is Amazon product pages, a $19 plan from a competitor and a $49 Hobby plan might cost you nearly the same per thousand requests once you account for each vendor's own credit scheme.

Two billing details worth knowing, both straight from the docs:

- You're only charged for successful requests — HTTP 200 and 404 responses, plus requests you cancel yourself before the 70-second timeout. Failed attempts don't eat your credits.
- Every response includes a `sa-credit-cost` header showing exactly what that request consumed, so you can audit your burn rate in real time.

Before committing to a plan, you can check the exact cost of any target URL in advance through the Domain Multiplier in the dashboard, or by hitting the account's URL cost endpoint with your API key and the URL plus parameters you plan to use. There's also a `max_cost` setting to cap spend per single scrape. Running your top 20 target URLs through the cost checker first is the single most useful five minutes you'll spend in this whole process.

👉 [Test your target URLs with the 7-day free trial](https://www.scraperapi.com/?fp_ref=coupons)

## Getting actual price data: the structured data endpoints

Raw HTML is fine if you enjoy writing CSS selectors, but for price monitoring you want JSON with typed fields. ScraperAPI ships pre-built structured data endpoints that return parsed fields directly — no parsing step on your side.

The e-commerce ones matter for this use case:

- **Amazon Product API** returns title, price, rating, review count, ASIN, availability, images, description, and seller info, across 21 regional marketplaces. Also available: an Amazon Search API (result listings with prices and sponsored flags) and an Offers API (third-party seller offers with condition and shipping).
- **Walmart scrapers** cover product, search, category, and review pages.
- **eBay endpoints** cover product and search pages.
- **Google Shopping API** returns price, merchant, rating, and availability per listing.

Cost-wise, structured endpoints bill the same as the core API on that domain — Amazon endpoints cost 5 credits per request, Google endpoints 25, Walmart 1. They're available on every plan including Hobby and Startup, so the entry tier isn't crippled here.

A minimal price-tracking loop then looks like this: register for an API key, call the Amazon Product endpoint with an ASIN and a `country_code`, store the returned price with a timestamp, and run the job on your own schedule. None of the scheduling logic lives inside ScraperAPI — that part stays in your cron job or queue, which is normal for this category (dedicated "scheduler built in" is rare even among pricier competitors).

Independent benchmark data from April 2026 (Scrapeway's testing) puts ScraperAPI's success rates at 98% on Amazon with a 6.5s average response, 93% on Walmart, and 99% on Etsy. For the structured endpoints specifically, the vendor claims 99.99% success on well-supported targets, and the Amazon product endpoint in particular has a reputation for holding up at volume.

👉 [Try the Amazon Product endpoint free for 7 days](https://www.scraperapi.com/solutions/ecommerce-data-collection/amazon-scraper/?fp_ref=coupons)

## How ScraperAPI compares on price

If your decision hinges on price, here are the verified entry points for the main alternatives as of mid-2026:

| Service | Entry price | Notes |
| --- | --- | --- |
| ScrapingAnt | $19/mo (100K credits) | JS rendering costs 10 credits (datacenter) or 125 (residential); failed requests cost 0; one independent benchmark recorded 68% success on a hard target |
| Oxylabs | $30/user/mo | Cheapest per-request rate for Amazon pages in one comparison (~1/3 to 1/6 of some rivals) |
| Apify | $29/mo usage-based | Free plan available; per-result Actor pricing |
| Scrapingdog | $40/mo | Claims lowest price per 1K calls at scale ($0.063) |
| ScrapingBee | $49/mo | Same entry tier as ScraperAPI |
| ZenRows | $69/mo | Higher entry point |
| Bright Data | $499/user/mo | Enterprise-scale, 437 pre-built scrapers |

Fair reading of that table: ScraperAPI is not the cheapest door price, and that's worth stating plainly. Where it earns the $49 is the combination of a long track record, structured endpoints that cover the exact e-commerce domains price scrapers care about, and billing that only charges successful requests. If your volume is tiny and you're scraping easy, unprotected storefronts, a $19 plan from ScrapingAnt is a legitimate way to spend less — just check its success rate on your actual targets, because cheap retries on a 68% success rate can erase the price gap fast. If you're pushing millions of Amazon lookups monthly, Oxylabs or Bright Data's per-request economics may beat everyone's.

## The limits nobody puts in the comparison table

Three constraints from the billing docs that affect real decisions:

1. **Credits don't roll over.** Unused credits in a billing period are gone at renewal. If your monitoring volume is spiky — say a big audit once a quarter — a smaller plan with bursty overage handling may fit better than a big flat plan.
2. **Running out behaves differently by tier.** On Hobby through Professional, credits exhausted means your account pauses until renewal unless you opt into auto-upgrade, which bumps you to the next plan (usually at a better per-credit rate). From Scaling up, you can enable pay-as-you-go to keep scraping past your limit at a fixed per-credit rate, capped at a spending limit you set.
3. **Concurrency is a hard wall.** Exceeding your plan's thread count returns a 429 and the request is rejected, not queued. There's no extra charge for going over — there's just no request. If you're scraping 500 URLs in parallel bursts, the concurrency column matters as much as the credits column.

The free plan also has real constraints worth restating: 1,000 credits, 5 concurrent connections, and no geotargeting. Good for validating that your parser handles the response format; not good for anything beyond that.

## FAQ

**How many Amazon prices can I track on the cheapest paid plan?**
Hobby gives 100,000 credits monthly; each Amazon request costs 5 credits, so 20,000 product page pulls. If you poll each ASIN once a day, that's roughly 650 ASINs tracked daily. On annual billing ($44/month effective) the same math applies to the credit allotment.

**Does the API return the sale price and the original price?**
The Amazon Product endpoint returns the current price among 18+ fields per product. If you need explicit list-price-vs-sale-price splitting with discount percentages as first-class fields, dedicated price-only APIs do that out of the box — but a structured Amazon pull gives you the live price your repricer or dashboard needs.

**Do I get charged when a request fails?**
No. Billing covers successful responses (200 and 404) and requests you cancel before the 70-second window closes. Each response's `sa-credit-cost` header tells you what was deducted.

**Is there a free option to test before paying?**
Two: a 7-day trial with 5,000 credits (no credit card), and a standing free plan with 1,000 credits and 5 concurrent connections for basic testing.

**Can I scrape sites protected by Cloudflare or DataDome?**
Yes, and bypass attempts that succeed add 10 credits per request on top of the base domain cost. The Domain Multiplier in the dashboard shows the exact cost before you run anything at scale.

## Which plan makes sense for which situation

A few closing decision rules, based on the verified numbers above:

- Just validating feasibility: start with the free 1,000 credits, confirm your target URLs return the fields you need, then decide.
- Daily price monitoring on a few hundred Amazon ASINs or Walmart SKUs: Hobby at $49 ($44 on annual) is the sweet spot, and the 7-day 5,000-credit trial lets you confirm your success rates first.
- Tracking 2,000+ ASINs daily, or you need pricing from non-US/EU storefronts: Business at $299 unlocks global geotargeting.
- Serious volume with unpredictable spikes: Scaling at $475 is the first tier with pay-as-you-go overage, which matters more than raw credits if your scraping load isn't flat.

One last practical tip before you buy: whatever plan you land on, run your real target list through the URL cost endpoint first, then check whether annual billing's 10% discount makes sense for your cash flow — it's the only discount currently on the table, and it applies across every tier.

👉 [Compare all ScraperAPI plans and start with the free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
