# Scraping Screenshot API: What One Screenshot Costs, Are Failed Captures Free, and Can You Get Full-Page and Element Shots From One Request?

Most dedicated screenshot endpoints price a capture at one credit per call. ScraperAPI doesn't sell screenshots that way. In its credit cost table, `screenshot=true` is listed at **10 credits per request**, and those credits come out of the same pool you use for ordinary page scrapes. The gap between those two pricing models matters less than three other things: whether failed captures are charged, whether loading images adds bandwidth credits on top, and whether the image comes back as base64 in the response body or as a link that expires.

Here's what the documentation and price tables actually say, plan by plan, plus a parameter-level comparison of four other screenshot endpoints so you know which questions to ask before you commit credits.

> **Quick view:** before anything else, check whether your monthly screenshot volume fits inside a ScraperAPI credit allowance, because screenshots draw from the same balance as your scraping. Hobby carries 100,000 credits and 20 concurrent threads, enough for roughly 10,000 screenshot requests a month at the listed 10 credits each. There's a 7-day trial with 5,000 credits, no card required.
> 👉 [Check ScraperAPI's current plans and trial](https://www.scraperapi.com/?fp_ref=coupons)

## ScraperAPI plans: credits, threads, and price

| Plan | Credits / month | Concurrent threads | Price | Start |
| --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | — † | [Hobby plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149/mo, or $134.10/mo billed annually ($1,609.20/yr) | [Startup plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | — † | [Business plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475/mo, or $427.50/mo billed annually ($5,130/yr) | [Scaling plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 (+250,000 bonus credits, limited offer) | 300 | $975/mo, or $877.50/mo billed annually ($10,530/yr) | [Professional plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | — † | [Advanced plan details](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom | [Talk to ScraperAPI about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

† Hobby, Business and Advanced publish their credit and concurrency figures in the plan table; their monthly prices weren't shown alongside those rows, so confirm the current figure on the pricing page before you compare tiers.

Below the paid tiers there's a free plan at 1,000 credits and 5 concurrent threads per month, and the trial gives you 5,000 credits for 7 days. Annual billing takes 10% off every self-serve plan, Enterprise excluded. Geotargeting is US and EU only on Hobby and Startup, country-level from Business upward.

Four billing behaviours are worth knowing because they change what "unlimited" means in practice:

- **Credits reset at renewal with no rollover.** An oversized plan doesn't bank unused capacity for a heavy month.
- **Pay-as-you-go starts at Scaling.** Hobby, Startup and Business either pause or auto-upgrade into the next tier.
- **Going over your thread limit returns a 429.** The request is turned away rather than queued, and there's no surcharge for it.
- **Every response carries a `sa-credit-cost` header.** Set `max_cost` per scrape to cap a single request.

## How the screenshot feature actually works in ScraperAPI

You add one parameter: `screenshot=true`. The docs state that this automatically enables JavaScript rendering so the full page content is loaded before the capture, and the resulting image is a **PNG**. The URL for it comes back in the `sa-screenshot` response header, not in the JSON body. Standard API requests and async requests are both supported; the docs mark proxy mode as coming soon.

That single parameter is the whole interface. There's no documented viewport width and height, no device scale factor, no visible-window versus full-page toggle, and nothing resembling a CSS selector for capturing one element. If your pipeline needs a mobile-width shot at 3× scale, or a crop of a price table, ScraperAPI's screenshot option as documented won't produce it. For monitoring a page's overall appearance or attaching a visual to an LLM prompt, the single flag is enough.

Two practical consequences follow from the header-based delivery. Your client has to read response headers, not just parse a body, and you should download the file immediately; the docs don't state how long that URL stays live.

### What a screenshot costs per plan

At the listed 10 credits per screenshot request, the per-unit cost depends entirely on which plan absorbs it. On Startup at $149 a month for 1,000,000 credits, a credit works out at about $0.000149, so a 10-credit screenshot lands near **$0.0015**. On Scaling at $475 for 5,000,000 credits, the same screenshot costs roughly **$0.00095**. Professional at $975 for 10,500,000 credits sits near **$0.00093**. Switching to annual billing drops each of those by the 10% discount.

One caution before you budget at that rate: the credit table prices `screenshot=true` at 10 credits and JS rendering at 10 credits as separate line items, and `screenshot=true` switches rendering on for you. Whether a screenshot request lands at 10 or 20 credits is the first thing to measure. Fire one request against a page you already scrape and read the `sa-credit-cost` header.

👉 [Start the 7-day trial and test the real credit cost](https://www.scraperapi.com/?fp_ref=coupons)

## Four other screenshot APIs, compared on what changes the bill

| Product | Output format | Cost basis | Full page and CSS element capture | Wait controls | How the image comes back |
| --- | --- | --- | --- | --- | --- |
| Scrapebit | PNG, JPEG, WebP; quality 0–100 for JPEG and WebP | 1 credit per screenshot | Both: `full_page` flag, `selector` for one element | `wait_for` selector, `delay` in ms, 30,000 ms default timeout | Download URL plus `expires_at` (sample showed 7 days) |
| Upscrape | JPEG only | 1 credit per successful call, failed calls free | Full page flag; no element selector | `wait_for_selector`, `wait_ms` capped at 10 s | Base64 inside the JSON body, no expiring link |
| Scrapfly | Format flags include `high_quality` and `print_media_format` | Requires JS rendering; `load_images` adds 3 credits per 100 kb of media | Both: `screenshots[name]=fullpage` or a CSS selector, up to 10 per scrape | Not documented in the screenshot reference | URL on Scrapfly's servers, needs `?key=` to download, retention from 1 week |
| ScrapeServ | JPEG by default; PNG or WebP via Accept header | Self-hosted, your own infrastructure | Section-by-section scrolling, 5 screenshots default and 10 max, `browser_dim` | `wait` in ms, 5,000 max, 1,000 recommended | `multipart/mixed` stream you parse and split |

Scrapebit is the only one of the four that exposes viewport presets with a device scale factor: desktop 1920×1080 at 1×, laptop 1366×768 at 1×, tablet 768×1024 at 2×, mobile 375×812 at 3×. It also takes `hide_selectors` for cookie banners and `click` for load-more buttons, both of which save a second request.

Upscrape runs for up to 120 seconds per call and returns useful metadata alongside the image: `final_url` after redirects, `rendered` to confirm a real browser drew the page, `status_code`, and a `warnings` array. Its 1440×900 JPEG sample came back at 60,650 bytes.

## Three costs and limits that don't show up in the headline price

**Bandwidth on rendered images.** Scrapfly's `load_images` flag is billed at 3 API credits per 100 kb of media downloaded, on top of whatever the scrape already costs. The docs say plainly that bandwidth usage is not heuristic, so a `cost_budget` cap won't contain it. Leaving the flag off keeps the price predictable, but Scrapfly's headless browsers don't render images or video by default, which is also why its shots can look different from a real browser.

**Silently skipped captures.** Scrapfly only accepts screenshots on GET requests. If its cache registers a HIT, the screenshots are ignored rather than regenerated. JavaScript-heavy pages can fail outright with `ERR::SCRAPE::UNABLE_TO_TAKE_SCREENSHOT`, and an invisible selector returns `ERR::SCRAPE::DOM_SELECTOR_INVISIBLE`.

**Format and wait ceilings.** Upscrape returns JPEG and nothing else, and its `wait_ms` ceiling is 10 seconds. A page that streams in content for 15 seconds will be captured before it finishes. ScrapeServ caps `wait` at 5,000 ms for the same reason.

**Expiring links versus inline payloads.** Scrapebit's sample response shows `created_at` and `expires_at` seven days apart. Scrapfly ties availability to your log retention policy, starting at one week, and the download URL needs your key appended. Upscrape avoids the problem entirely by putting base64 in `results[0].data.image`, which shifts the constraint to response body size.

## When self-hosting is worth the operations work

ScrapeServ is a Docker image that runs on port 5006, takes `POST /scrape`, and returns `multipart/mixed`: a JSON preamble with status and headers, the page data, then up to five screenshots by default. You can raise that to ten. Output defaults to JPEG and switches to PNG or WebP through the Accept header.

The defaults tell you what it expects: 4 GB memory limit per child process, 3 concurrent tasks, browser dimensions of 1280×2000 with a 2400×4000 ceiling, screenshot quality 85, and 1,000 ms wait. Each job gets a fresh Firefox context via Playwright, which the author describes as resource intensive but higher quality than many alternatives.

Security is handled at the container level: URL validation rejects loopback, local and non-HTTP targets, each site runs in a new browser context, and API keys are set as `SCRAPER_API_KEY`-prefixed environment variables passed as Bearer tokens. None of that protects you from feeding it a hostile URL you chose yourself. If you can't screen URLs before they hit the queue, a hosted service absorbs that risk for you. If you can, and your volume is high enough that per-shot pricing dominates your bill, the marginal cost of a screenshot drops to your own compute.

## Which option fits your pipeline

- **You already scrape with ScraperAPI and need occasional PNG evidence.** Add `screenshot=true` and budget 10 credits per request. One parameter, one header, no new vendor.
- **You need element-level crops or mobile viewport shots.** Scrapebit covers `selector`, `device_scale_factor` and presets; Scrapfly covers CSS selectors with named screenshots, up to 10 per scrape.
- **You want the image inside the JSON with no link to manage.** Upscrape, accepting JPEG-only and a 10-second wait ceiling.
- **You need images actually rendered in the shot.** Expect to pay Scrapfly's per-100 kb media surcharge, or self-host with ScrapeServ and pay in compute.
- **You need many shots per page.** Scrapfly at 10 per scrape or ScrapeServ at 10 per request; ScraperAPI's flag produces one.

## Two things to confirm before you pay

First, your monthly screenshot count times the real credits per request. Measure it with the `sa-credit-cost` header on a live call rather than trusting the table alone, then check the result against a plan's allowance, remembering that credits reset each month and don't roll over. ScraperAPI's 7-day refund policy gives you a clean exit if the measured cost doesn't match the plan.

Second, whether you need element-level capture, viewport presets or forced image rendering. ScraperAPI's screenshot interface is a single boolean, and no plan tier changes that. If your use case needs a crop of one table or a 390-pixel mobile render, budget for a second endpoint now instead of discovering it after you've committed credits.

👉 [Compare ScraperAPI plans and start with 5,000 free trial credits](https://www.scraperapi.com/?fp_ref=coupons)
