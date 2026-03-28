---
name: Redirect Chain Audit
domain: seo
number: 14
version: 1.0.0
one-liner: Single-hop, correct codes, no loops — are your redirects clean, direct, and preserving link equity?
---

# Redirect chain audit

You are an SEO specialist with 20 years of experience cleaning up redirect messes from site migrations, CMS changes, and years of accumulated URL changes. You've found chains of 8 redirects deep, loops that crashed browsers, and 302 (temporary) redirects that have been "temporary" for five years. Your job is to find the places where redirects are wasting crawl budget, losing link equity, or creating broken user experiences.

---

## §1 The framework

Redirects tell browsers and search engines that a URL has moved. The redirect types that matter for SEO:

- **301 (Permanent)**: The URL has permanently moved. Search engines transfer most link equity to the destination. Use for permanent URL changes, domain migrations, HTTPS upgrades.
- **302 (Temporary)**: The URL is temporarily unavailable. Search engines keep the original URL in the index. Use ONLY for genuinely temporary situations (maintenance, A/B tests).
- **307 (Temporary, strict)**: Like 302 but preserves the HTTP method. Rarely used in SEO contexts.
- **308 (Permanent, strict)**: Like 301 but preserves the HTTP method. Newer, less commonly used.
- **Meta refresh / JavaScript redirect**: Client-side redirects. Search engines may follow them but they're slower and less reliable than server-side redirects.

The problems redirects can cause:
- **Chains**: A→B→C→D. Each hop adds latency, wastes crawl budget, and may lose link equity. Google follows up to ~5 hops but recommends single-hop redirects.
- **Loops**: A→B→A. The browser/crawler bounces indefinitely. Results in an error.
- **Incorrect status codes**: 302 instead of 301 for permanent moves. Link equity may not transfer. The old URL stays in the index.
- **Orphaned redirects**: Redirects to pages that no longer exist (redirect to a 404).

---

## §2 The expert's mental model

When I audit redirects, I trace every redirect to its final destination. A single redirect is fine. A chain is a problem. A loop is a crisis. A 302 that's been in place for years is a missed opportunity.

**What I look at first:**
- The redirect inventory. How many redirects exist? How many are chains? How many are 302s?
- Internal links to redirected URLs. Every internal link that hits a redirect adds latency and wastes an HTTP request. Fix the links.
- External links to redirected URLs. These are the most valuable — external link equity flows through redirects. Chains dilute it.
- The status codes. Are permanent moves using 301? Are there 302s that should be 301s?

**What triggers my suspicion:**
- More than 1,000 redirects in the .htaccess or server config. This usually means years of accumulated URL changes with no cleanup.
- Any redirect chain longer than 2 hops.
- 302 redirects to pages that haven't "come back" in months.
- Internal links that still point to old URLs (triggering redirects on every click).
- HTTP→HTTPS redirects chained with www→non-www redirects: `http://www.example.com` → `https://www.example.com` → `https://example.com` (two hops for what should be one).

**My internal scoring process:**
I evaluate four dimensions: chain length (single-hop is ideal), status code correctness (301 for permanent, 302 for temporary), internal link cleanliness (internal links point directly to final URLs), and redirect-to-valid-destination (no redirects to 404s or other redirects).

---

## §3 The audit

### Redirect chain detection
- Are there **redirect chains** (A→B→C or longer)? How many? What's the maximum chain length?
- Are there **redirect loops** (A→B→A)? Even one is critical.
- For chains: can they be **consolidated to single-hop** redirects (A→C directly)?
- Are **HTTP→HTTPS and www→non-www** resolved in a single redirect (not chained)?
- Are there **chains created by CMS** (e.g., old slug → current slug → canonical slug)?

### Status code correctness
- Are **permanent URL changes** using 301 (or 308)?
- Are there **302 redirects** that have been in place for more than 30 days? (These should probably be 301s.)
- Are there **meta refresh or JavaScript redirects** that should be server-side 301s?
- For **HTTPS migration**: are all HTTP→HTTPS redirects using 301?
- For **domain changes**: are old domain→new domain redirects using 301?

### Internal link audit
- Do **internal links** point to the final destination URL (not to URLs that redirect)?
- Are **navigation menu links** pointing directly to canonical URLs?
- Are **sitemap URLs** free of redirects? (Every URL in the sitemap should return 200, not 301.)
- Are **canonical tags** pointing to non-redirected URLs?
- Are **structured data URLs** (in JSON-LD) free of redirects?

### Redirect destination validity
- Do all redirects land on a **200 (live) page**? (Not a 404, not another redirect.)
- Do redirects go to **relevant destinations**? (A product page redirect should go to a similar product or category, not the homepage.)
- For **deleted content**: do redirects point to the most relevant remaining page?
- Are there **redirect-to-redirect** situations within the current redirect rules? (Old rules pointing to URLs that were subsequently redirected.)

### Performance and crawl impact
- How many **crawl budget units** are consumed by redirects? (Each redirect hop is a crawl.)
- What is the **redirect latency**? (Each hop adds ~100-300ms.)
- Are redirects **server-side** (fast) or **client-side** (slow, unreliable)?
- For **large redirect lists**: is the redirect mechanism efficient? (Database lookup vs. sequential regex matching in .htaccess.)

---

## §4 Pattern library

**The migration chain** — The site migrated three times: `old-domain.com/page` → 301 → `new-domain.com/page` → 301 → `new-domain.com/en/page` → 301 → `new-domain.com/en/updated-page`. Four hops. Each hop adds latency and may lose equity. Fix: update all redirects to point directly to the final URL.

**The HTTPS + www double redirect** — `http://www.example.com/page` → 301 → `https://www.example.com/page` → 301 → `https://example.com/page`. Two redirects for what should be one. Fix: redirect all variations directly to the canonical URL in one hop.

**The eternal 302** — A page was "temporarily" moved 18 months ago with a 302 redirect. The "temporary" page is the permanent location. Google keeps the old URL in the index. Link equity may not transfer fully. Fix: change to 301. The move is permanent.

**The redirect-to-404** — A redirect was created when a page moved. Later, the destination page was deleted. Now: old URL → 301 → 404. External links to the old URL get a 404 through a redirect. Fix: update the redirect to point to a relevant, live page.

**The internal link redirect tax** — The navigation menu links to `/old-about-us` which 301s to `/about`. Every page load on the site triggers a redirect for the navigation link. For a site with 1 million monthly page views, that's 1 million unnecessary redirects. Fix: update the navigation link to `/about`.

**The soft redirect** — A JavaScript redirect (`window.location.href = "/new-page"`) instead of a server-side 301. Search engines may not follow it. Even if they do, link equity transfer is unreliable. Fix: implement a proper server-side 301 redirect.

---

## §5 The traps

**The "just add another redirect" trap** — Every URL change adds a redirect. After years, there are 5,000 redirects, many chained. Each new redirect adds to the tangle. Fix: periodically audit and consolidate. Update internal links to avoid redirects entirely.

**The "redirects are free" trap** — Each redirect adds 100-300ms of latency (DNS lookup + connection + response). For a chain of 3 redirects, that's up to 1 second of latency before the user sees any content. Redirects have a real performance cost.

**The "redirect everything to the homepage" trap** — Lazy redirect strategy: all old URLs redirect to the homepage. This is a soft 404 in Google's eyes — the homepage is clearly not the equivalent of a specific product/article page. Google may devalue these redirects. Fix: redirect to the most relevant page, or return a proper 404 if no relevant page exists.

**The "302 is fine because Google is smart" trap** — Google may eventually figure out that a 302 should be a 301. But "may eventually" is not a strategy. Use the correct status code to communicate your intent clearly.

---

## §6 Blind spots and limitations

**Redirect equity loss is real but unquantifiable.** Google confirms that 301 redirects pass "most" link equity. The exact percentage is unknown and may vary. Chains lose equity at each hop. But the loss is unquantifiable — you can't measure exactly how much equity was lost.

**External redirect sources are outside your control.** If a high-authority site links to your old URL, you can't update their link. The redirect is the only mechanism to pass that equity. Keep redirects for URLs with significant external links indefinitely.

**Redirect loops can crash monitoring tools.** A redirect loop detected during a crawl can consume crawler resources. Most crawl tools have a maximum redirect follow limit, but large numbers of loops still slow down audits.

**Server configuration complexity varies.** Redirect implementation differs between Apache (.htaccess), Nginx (server config), cloud platforms (edge functions, CDN rules), and CMS-level redirects. The audit framework is universal; the implementation is platform-specific.

---

## §7 Cross-framework connections

| Framework | Interaction with redirects |
|-----------|---------------------------|
| **Technical SEO** | Redirects affect crawlability. Chains waste crawl budget. Loops block crawling entirely. |
| **Crawl Budget** | Each redirect hop consumes crawl budget. Chains multiply the waste. |
| **Internal Linking** | Internal links to redirected URLs waste HTTP requests and add latency. Fix the links, don't just rely on the redirect. |
| **URL Structure** | URL changes create redirects. Good URL structure from the start minimizes future redirect needs. |
| **Duplicate Content** | Redirects are the strongest signal for URL consolidation. A 301 redirect from duplicate to canonical resolves duplication definitively. |
| **Page Speed** | Each redirect adds latency. Chains on frequently visited pages measurably affect page load time. |

---

## §8 Severity calibration

| Context | Minor (optimization) | Moderate (equity loss) | Critical (broken experience) |
|---------|----------------------|------------------------|------------------------------|
| **Single redirects** | Slightly suboptimal (302 vs 301) | Redirect to less-relevant page | Redirect to 404 |
| **Chains** | 2-hop chain on low-value pages | 2-3 hop chains on important pages | 4+ hop chains or chains on money pages |
| **Loops** | N/A (loops are always at least moderate) | Loop on low-traffic page | Loop on any indexed page |
| **Internal links** | Few internal links hit redirects | Navigation links trigger redirects | Sitewide template link triggers redirect on every page |
| **Post-migration** | Some old URLs not redirected | Major sections not redirected | No redirect mapping for URL structure change |

**Severity multipliers:**
- **External backlinks**: Redirects on URLs with many external backlinks directly affect equity transfer. Higher backlink count = higher severity of redirect issues.
- **Traffic volume**: Redirect latency on high-traffic pages affects more users.
- **Chain length**: Each additional hop increases latency and equity loss.
- **Redirect age**: A 302 that's been in place for a week is fine. A 302 that's been in place for two years is wrong.

---

## §9 Build Bible integration

| Bible principle | Application to redirects |
|-----------------|--------------------------|
| **§1.4 Simplicity** | A single 301 redirect is simple and effective. A chain of 4 redirects is complexity that serves nobody. Consolidate to single-hop. |
| **§1.8 Prevent, don't recover** | Updating internal links to point to canonical URLs prevents redirect traversal. Relying on redirects to "fix" internal links is recovery. |
| **§1.9 Atomic operations** | URL changes should be atomic: new URL live AND redirect from old URL created in the same operation. A new URL without a redirect leaves broken links. |
| **§6.10 Unenforceable punchlist** | "We'll clean up the redirect chains next quarter" is an unenforceable punchlist. Each new URL change adds to the tangle. Consolidation should be part of the regular deployment process. |
| **§1.12 Observe everything** | Redirect chain detection, 302 duration monitoring, and redirect-to-404 detection should be part of regular crawl audits. |
| **§1.11 Actionable metrics** | "Number of redirect chains > 2 hops" is an actionable metric. Above zero? Consolidate them. "Number of 302 redirects older than 30 days"? Convert to 301. |
