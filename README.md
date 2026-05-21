# The Conversion Mirage: Why Your GA4 Custom Events Are Not the Whole Truth

Your [GA4](/alternative/ga4-alternative) conversion rate is **4.8%**. It passes every audit. The tag fires, the event lands in DebugView, the key event is marked, the numbers populate the report. And it is still wrong.

Not wrong because you misconfigured it. Wrong because it is built correct and measuring the wrong thing.

I have debugged GA4 for a lot of teams, and almost every "GA4 is inaccurate" thread online assumes the same root cause: a setup mistake. Wrong trigger, missing parameter, GA4's 30-key-event ceiling silently dropping a conversion. Real problems, all of them. But fix every one and you still have a conversion rate that lies, because the lie is not in the configuration. It is in the traffic.

This is not a troubleshooting post about why your events do not show up. This is a post about why your events show up, look perfect, and still cannot be trusted. The fix is architectural, and DataCops is the version of it I will get to. First, the diagnosis.

## Quick stuff people keep asking

**Why are my GA4 custom events not showing conversions?** Usually the event is firing but not marked as a key event, or it is firing after GA4's 30-key-event limit and getting silently dropped. Check DebugView, check the key events list. That is the config layer, and it is the easy half.

**Why do GA4 conversions not match Google Ads?** Different attribution models, different lookback windows, different counting rules. Everyone explains this. What they skip: both tools may be counting the same bot "conversions" and just disagreeing on how to credit them. Reconciling the models does not make either number true.

**Can bots inflate GA4 conversion metrics?** Yes, and routinely. A headless browser that loads a page and triggers a form event produces a real GA4 event. GA4 has no idea a human was not involved.

**Why is my GA4 conversion rate unrealistically high?** Often because the denominator is contaminated and the numerator is too. Bot sessions and bot events both count. If your rate looks too good, your gut is usually right.

**How much of GA4 event data is from bots or spam?** Industry estimates put non-human traffic at 25 to **35%** of web traffic, higher during AI-agent and scraper surges. GA4 catches some. It does not catch most of it.

**Does GA4 filter bot traffic automatically?** It filters traffic on a known-bots-and-spiders list, mostly declared crawlers. Headless Chrome, residential-proxy bots, AI scrapers, and referral spam designed to look human sail straight through. "Automatic [bot filtering](/fraud-traffic-validation)" is real and badly oversold.

**Why are GA4 numbers different from my CRM?** Your CRM counts real records, deals, payments. GA4 counts events. Ad blockers and consent rejection drop real human events before they reach GA4, and bots add fake ones. The two systems disagree because GA4 is measuring a different, distorted population.

**How do ad blockers affect GA4 custom event tracking?** They block the GA4 collection request outright. The user converts, the event never sends. Combined with consent-mode gaps, that is a 10 to **30%** under-count of real humans, depending on your audience.

## The gap: wrong in both directions at once

Here is the part no single guide puts together, and it is the whole story.

GA4 conversion data fails in two directions simultaneously.

Direction one: it under-counts real humans. Ad blockers strip the collection request. Consent-mode rejection holds events back. Browser privacy limits cut sessions short. Real people convert and GA4 never hears about it. Call it a 10 to **30%** loss off the bottom, depending on how privacy-aware your audience is.

Direction two, the one nobody pairs with the first: it over-counts fake activity. 25 to **35%** of incoming traffic is non-human. Those bots load pages, trigger scroll events, sometimes submit forms. GA4's bot filtering is built for declared crawlers, so the modern stuff, headless browsers, residential-proxy networks, AI agents, gets measured as real users with real engagement.

Stack those together and your conversion rate is not "a bit off." It is structurally broken on both ends at the same time. The numerator is missing real conversions and padded with fake ones. The denominator is missing real sessions and padded with bot sessions. You are computing a ratio where neither number is clean. The output is not an approximation of the truth. It is a number that happens to look like a percentage.

This is Layer 4 of a bigger problem. Your analytics scripts get blocked 25 to **35%** of the time, and of the data that does get collected, 24 to **31%** is bots. Both failures, in the same dataset, and GA4's reporting shows you a single confident figure on top.

Let me make it concrete. PillarlabAI ran a honeypot signup flow once. 3,000 signups arrived. They inspected them by hand. **77%** were fraudulent. 650 of those accounts traced to a single device fingerprint, one machine. Now picture that flow with a GA4 `sign_up` key event wired to it, which it almost certainly would have. GA4 would have logged thousands of conversions, computed a gorgeous conversion rate, and shown a green trendline. Every audit would pass. The event was correct. The data was garbage. That is the gap in one story.

## Why a correct setup cannot fix this

The instinct, once you see inflated numbers, is to tighten the configuration. Add filters. Define an internal-traffic rule. Build an exclusion segment for known spam referrers. Worth doing. It will not fix this.

It cannot, for a structural reason. By the time an event reaches GA4, the mixing already happened. The collection request bundles real users and bots together and fires them at Google's servers. GA4 is a reporting layer sitting on top of a contaminated stream. You can slice and filter inside GA4 all day, but you are filtering data that was already poisoned before it left the browser. There is no isolation point. Nothing inspected the traffic before it became "an event."

That is the actual root cause. Third-party analytics scripts collect mixed data with no isolation before it leaves your infrastructure. Fix that and the problem changes shape. Leave it and no amount of in-GA4 cleanup reaches the source.

The architectural fix is [first-party tracking](/first-party-consent-manager-platform) that filters at the point of ingestion, before the data is committed and forked. That is what DataCops does. It runs first-party on your own subdomain, so the collection itself is far more resilient to ad blockers, which closes a chunk of the under-counting side. Bot filtering happens at ingestion, scored against a 361.8 billion-plus IP database, so non-human traffic is identified before it is counted as a conversion, which closes the over-counting side. And it keeps two data tiers separate at the source: anonymous session analytics flow unconditionally and legally, identifiable event data is gated on consent. You see clean human conversions instead of a blended figure you have to apologize for.

DataCops is newer than GA4 and SOC 2 Type II is still in progress, so a regulated buyer may need to wait on that. I would rather say that plainly than pretend otherwise. But on the specific failure in this article, GA4 measuring fake conversions next to real ones and reporting one number, the architectural answer is the only answer that reaches the cause.

## Decision guide

**Conversion rate looks too high to be real.** Trust the instinct. Audit what share of converting sessions come from datacenter IPs or repeat device fingerprints before you change a single campaign.

**GA4 and CRM disagree by a lot.** Treat the CRM as closer to truth. GA4 is under-counting humans and over-counting bots. The CRM counts records.

**You already added every internal-traffic and spam filter.** You have hit the ceiling of in-GA4 cleanup. The remaining error lives upstream of GA4 and cannot be filtered after the fact.

**EU or privacy-heavy audience.** Your under-counting is on the high end. Separate anonymous analytics from identifiable events so the legal, always-collectable data is not lost alongside the consented data.

**Reporting conversion rate to leadership or investors.** Caveat it, or fix the source first. A number you cannot defend is worse than no number.

## You have been debugging the wrong layer

The mistake is treating GA4 inaccuracy as a bug to fix. It is not a bug. Your events fire correctly. Your setup is clean. The configuration was never the problem.

The problem is that you are running a reporting tool on top of a contaminated stream and asking it to produce truth. It cannot. It can only produce a tidy number on top of dirty data, and a tidy wrong number is more dangerous than an obviously broken one, because you act on it. You set budgets against it. You tell your boss it.

So before you open GA4 again, ask one question about last month's conversion rate. Of those conversions, how many do you actually know were real humans, with the bots removed and the blocked-but-real humans added back? If you cannot put a number on that, you do not have a conversion rate. You have a guess wearing a decimal point.

---

Research by [DataCops](https://www.joindatacops.com) — first-party tracking, consent infrastructure, fraud prevention, and server-side CAPI for Meta, Google, TikTok, and LinkedIn.
