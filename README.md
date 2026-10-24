# amazon product api python: Get Amazon Product Data Without PA-API Headaches — Code, Costs, and Which ScraperAPI Plan Actually Fits

Most people searching for an Amazon product API in Python want the same thing: give it an ASIN, get back price, rating, reviews, and product details as JSON. There are two ways to do that. The official route is Amazon's Product Advertising API (PA-API), which requires an Associates account and access keys, and whose most popular Python wrappers have quietly stopped being maintained. The other route is a scraping API that returns structured data from an ASIN directly — no seller or affiliate account needed. ScraperAPI is one of the main options in that second category, and this article walks through how its Amazon Product API works in Python, what it returns, what it costs in real request terms, and which of its plans makes sense before you commit.

If you just want the short version: the free plan gives you 1,000 API credits a month plus 5,000 credits for your first 7 days, and Amazon requests cost 5 credits each — so you can pull roughly 200 Amazon products a month for free, or about 1,000 during the trial week. Hobby at $49/month (or $44.10/month billed annually) gets you ~20,000 Amazon product requests monthly. That's enough for most price-tracking scripts, niche research projects, and small dashboards.

> **Quick picks**
> - Just experimenting? Start on the free plan — 1,000 credits/month, no credit card required.
> - Running a script that checks a few hundred products daily? Hobby ($49/mo) covers ~20,000 Amazon requests.
> - Every plan includes the same Amazon Product API endpoint; the tiers only differ in credits, concurrency, and geotargeting.
>
> 👉 [Claim the 7-day trial with 5,000 free credits](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Two routes to Amazon product data in Python

The fastest answer: if you don't have (and don't want to maintain) an Amazon Associates account, use a structured scraping API. If you're an affiliate who already qualifies for PA-API, the official API is still worth using — but know what you're signing up for.

The official PA-API route in Python looks like this: you install a wrapper such as **python-amazon-paapi** (requires Python 3.9+, has an async client option) or the older **python-amazon-simple-product-api**, then authenticate with an access key, a secret key, and an associate tag tied to your Amazon Associates account. The wrappers are genuinely nice to work with — you get `product.title`, `product.price_and_currency` and similar attribute access.

The catch is upstream. The last PyPI release of python-amazon-simple-product-api dates to February 2017. python-amazon-product-api's last release was March 2014. These are wrappers around an official API whose terms and quotas you have to satisfy yourself, and PA-API access itself is tied to having an Associates account in good standing. If you're building a price monitor, a market-research tool, or a dataset that pulls from thousands of ASINs, that dependency chain gets uncomfortable.

The ScraperAPI route skips all of that. You make a plain `requests` call with an API key and an ASIN, and the service fetches the Amazon page — handling proxies, CAPTCHAs, and JavaScript rendering behind the scenes — then returns the page already parsed into JSON. You never touch HTML parsing, and you never deal with Amazon's API quota system. The tradeoff is monetary: scraping costs credits, and Amazon is priced at a premium over generic pages.

## Which plan covers how many Amazon requests?

Here's the part most comparison posts skip: ScraperAPI prices domains differently, and that completely changes what your credits buy. A standard page costs 1 credit. **Amazon costs 5 credits per request.** (Google and Bing cost 25; LinkedIn costs 30.) Sites behind Cloudflare, Datadome, or PerimeterX add 10 credits per request when the bypass kicks in — that surcharge generally won't hit the Amazon endpoint, but it matters if you're scraping other stores alongside Amazon.

Converting credits into Amazon requests:

- **Free plan** — 1,000 credits/month → ~200 Amazon product pulls
- **7-day trial** — 5,000 credits → ~1,000 Amazon product pulls
- **Hobby** — 100,000 credits → ~20,000 Amazon pulls/month
- **Startup** — 1,000,000 credits → ~200,000 Amazon pulls/month
- **Business** — 3,000,000 credits → ~600,000 Amazon pulls/month

So the real question when choosing a tier is: how many ASINs do you check, how often? A script that refreshes 500 products once a day burns ~75,000 credits a month — Hobby won't cover it with headroom, Startup will with room to spare. A hourly refresh on 500 products needs closer to 1.8M credits, which puts you at the bottom of Business.

Full plan table (all tiers currently shown on the pricing page):

| Plan | Monthly price | Annual (per month, 10% off) | Credits/mo | Concurrent threads | Buy |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Sign up free](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Every paid tier includes JS rendering, premium residential and mobile IPs, CAPTCHA handling, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. The lower tiers have two practical limitations worth knowing: Hobby and Startup only support US & EU geotargeting (country-level targeting starts at Business), and their analytics dashboards only show the last 30 days of data.

## What the Product API endpoint actually returns

The endpoint is a single GET request. In Python:

python
import requests

payload = {
    'api_key': 'YOUR_API_KEY',
    'asin': 'B06X3WKY59',
    'tld': 'us',
}
r = requests.get(
    'https://api.scraperapi.com/structured/amazon/product',
    params=payload,
)
data = r.json()
print(data['product_information']['brand'])
print(data['average_rating'], data['total_reviews'])


Three parameters cover most needs: `api_key` and `asin` are required, `tld` defaults to `us` (amazon.com). `output_format` switches between JSON (default) and CSV. There's also an async variant at `async.scraperapi.com` for batch jobs where you fire off many ASINs and collect results later.

The response is a full structured dump of the product page. The documentation's example — a children's plush chair, ASIN B06X3WKY59 — shows what comes back: dimensions, materials, weight, model number, recommended age, best-sellers rank ("#9,307 in Home & Kitchen"), `average_rating` of 4.7, `total_reviews` of 5,665, `total_answered_questions`, list price, shipping price, availability, all product images, feature bullets, the seller name and ID, whether it's fulfilled by Amazon, and the release date. Variants come through in `customization_options`, each with its own URL, price, and image — so a color/size matrix is one request, not twenty.

Two fields deserve a mention because they're decision-useful and rarely covered in tutorials:

- **`is_coupon_exists`** tells you whether the product page currently has a coupon. If you're building a deal tracker, this single boolean saves you a second request.
- The endpoint also returns **all publicly available reviews** for the ASIN, per the official docs — useful for sentiment work without chaining a separate reviews endpoint.

One quirk that trips people up: **ASINs are marketplace-specific.** B06X3WKY59 on amazon.com may not resolve on amazon.de. And if you want to scrape one marketplace from another country's IP — the docs' example is pulling amazon.com from Canada to get Canadian shipping information — you must pass **both** `tld` and `country`/`country_code`. Passing only one gives you the wrong result. There's also ZIP code targeting if you need delivery-window precision at the postal-code level, which matters for Buy Box research.

In total the endpoint supports around 20 Amazon marketplaces: com, co.uk, ca, de, es, fr, ie, it, co.jp, co.za, in, cn, com.sg, com.mx, ae, com.br, nl, com.au, com.tr, sa, se, and pl.

## The cost math nobody does: annual vs monthly, and what "5 credits" means at scale

Annual billing takes 10% off every tier. On Hobby that's $529.20 for the year instead of $588 — about $59 saved. On Business it's $3,229.20 vs $3,588, saving $358.80. The discount is real but modest; don't let it push you into a higher tier you don't need. The credit-to-request conversion in the section above matters far more for your wallet.

The other thing to model is what happens when you run out of credits mid-month, because the behavior differs by plan and it's easy to get surprised:

- By default, scraping pauses until your credits renew.
- On Hobby through Professional, you can opt into **auto-upgrade**, which bumps you to the next tier when credits run out — usually at a better per-credit price, but it does mean a bigger bill than you planned.
- Scaling and above get **pay-as-you-go**, letting you continue at a fixed per-credit rate up to a monthly spending cap you set yourself. That cap is a limit, not a pre-charge.
- Exceeding your concurrent-thread limit returns a 429 and the request is refused — no queueing, and no extra charge.

That last point matters for Python scripts: if you fire 50 concurrent requests on Hobby (20 threads), you'll get 429s back rather than billing surprises. Build a simple retry-with-backoff around those and you're fine.

If you're unsure whether the data quality works for your use case, the 7-day trial with 5,000 credits is the right test — that's roughly 1,000 Amazon product pulls, enough to run a real script against a real ASIN list. And if the service disappoints, there's a 7-day no-questions-asked refund policy on paid plans, handled through support. Canceling is self-serve from the dashboard with no cancellation fee. 👉 [Start the trial and test it against your own ASIN list](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## When the official PA-API is still the better call

To be fair to the other side: if you already run an Amazon Associates site and qualify for PA-API, the official route costs you nothing per request within your quota, and a maintained wrapper like python-amazon-paapi gets you going in an afternoon. It requires Python 3.9+, your access key, secret key, and associate tag — the same credential set the older wrappers used.

Where PA-API falls short is data scope and flexibility. It's built for affiliates, so its data model centers on what affiliates need (prices, images, links) and doesn't hand you the full page picture: no guaranteed coverage of every public review, no seller/buy-box context the way the scraping route returns it, and no ability to target by zip code. For comparison-shopping engines, price-history trackers, or seller analytics, the scraping route returns a materially richer payload for 5 credits a pop.

A note on other scraping-API competitors: the category has several players (Oxylabs, Bright Data, and newer entrants), and pricing models vary widely — some charge per successful request in a similar credit fashion. ScraperAPI's own 2026 comparison content positions its Business plan at $299/month against Oxylabs' web scraper API at similar spend, emphasizing that it only charges for successful requests. Whichever direction you lean, the number to compare is price per *Amazon* request, since Amazon carries a premium multiplier on most platforms.

## FAQ

**Does the Amazon Product API work with plain requests, or do I need an SDK?**
Plain `requests` is all you need — the endpoint is a single GET with query parameters. No SDK, no auth signature dance, no XML. For high-volume jobs, the async endpoint accepts a POST with JSON and lets you collect results as they complete.

**How many Amazon marketplaces does it cover?**
About 20, from amazon.com and amazon.de to amazon.com.br, amazon.ae, and amazon.co.jp. Geotargeting below the TLD level (country-level IP targeting) starts with the Business plan; Hobby and Startup are US & EU only.

**What happens to my unused credits?**
They don't roll over — credits refresh with each billing cycle. If you consistently hit the ceiling before renewal, auto-upgrade (on monthly plans from Hobby up) or pay-as-you-go (Scaling and above) are the sanctioned ways to keep going, and a custom plan is available through support if neither fits.

**Is there a refund if it doesn't work for my project?**
Yes — 7 days, no questions asked, arranged through support. Combined with the free trial, you can test the Product API end-to-end on your real data before any money is at real risk.

**Can I get reviews and product data in one request?**
Yes. The Product API endpoint returns all publicly available reviews alongside the structured product data for the ASIN, so a review-sentiment pipeline needs one credit spend per product instead of two.

## Before you pick a plan

Match the tier to your actual request math rather than to the feature list, since every tier shares the same endpoint:

- Running a personal price-tracker for a few dozen products → the **free plan** is genuinely sufficient (~200 Amazon pulls/month).
- Daily refreshes on hundreds of ASINs, or a small SaaS dashboard → **Hobby** at $49/month (~20,000 Amazon requests), or Startup if you're also pulling search results or Google data, which burn credits 5x faster.
- Production systems with country-level targeting or hourly refresh cycles → **Business** and up; that's where geotargeting opens up and per-credit economics improve.

One last check before checkout: confirm your ASIN list spans the marketplaces you actually care about, since ASINs are TLD-specific and a US-only script silently returns errors on amazon.de. Then 👉 [open the plans page and start with the free 5,000-credit trial](https://www.scraperapi.com/pricing/?fp_ref=coupons) — if your script runs clean against real data within the week, upgrading is an easy call, and if it doesn't, the refund window has you covered.
