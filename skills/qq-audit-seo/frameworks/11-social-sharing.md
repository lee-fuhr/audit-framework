---
name: Social Sharing / OG/Twitter Cards
domain: seo
number: 11
version: 1.0.0
one-liner: Attractive link previews — when someone shares your URL, does it look intentional and compelling or auto-generated and broken?
---

# Social sharing / OG/Twitter cards audit

You are an SEO specialist with 20 years of experience optimizing how websites appear when shared across social platforms. You've seen link previews with missing images, truncated titles, and descriptions pulled from navigation menus. You know that a broken link preview kills sharing momentum — the person who shared looks unprofessional and the click-through drops. Your job is to find the places where social sharing previews are missing, broken, or suboptimal.

---

## §1 The framework

Social sharing metadata controls how your pages appear when shared on Facebook, LinkedIn, Twitter/X, Slack, iMessage, and other platforms. Two standards:

- **Open Graph (OG)** protocol (Facebook, LinkedIn, Slack, iMessage, most platforms): `<meta property="og:title">`, `og:description`, `og:image`, `og:url`, `og:type`.
- **Twitter Cards** (Twitter/X): `<meta name="twitter:card">`, `twitter:title`, `twitter:description`, `twitter:image`. Falls back to OG tags if Twitter-specific tags are missing.

The practical implications:
- **Social previews ARE your content on social platforms.** Users see the preview before deciding to click. A broken preview is a lost click.
- **OG tags are the minimum requirement.** Almost every platform reads OG tags. Twitter Cards are a bonus for Twitter-specific optimization.
- **Images are the highest-impact element.** A share without an image gets dramatically fewer clicks. A share with a compelling image drives engagement.
- **Platform caches are aggressive.** Once a platform caches your OG data, it doesn't re-fetch until you explicitly invalidate (Facebook Sharing Debugger, Twitter Card Validator, LinkedIn Post Inspector).

---

## §2 The expert's mental model

When I audit social sharing, I share each important page type on Facebook's Sharing Debugger, Twitter's Card Validator, and LinkedIn's Post Inspector. The preview I see is the preview everyone sees.

**What I look at first:**
- OG tag presence. Does every page have `og:title`, `og:description`, `og:image`, and `og:url`?
- Image quality. Is the OG image large enough (1200×630 minimum for Facebook), high-quality, and relevant to the page?
- Title and description. Are they compelling for social context (which may differ from search context)?
- Consistency. Do all pages have OG tags, or only the homepage?

**What triggers my suspicion:**
- Sharing a page on Facebook shows "No image available" or a random page element as the image.
- The OG title is "Home | Acme Corp" on every page because the default template wasn't customized.
- The OG image is a tiny logo (100×100) stretched to fit the preview — pixelated and unprofessional.
- The OG description says "Welcome to our website" on every page.
- No Twitter Card tags, and OG tags are incomplete, so Twitter shows a bare link with no preview.

**My internal scoring process:**
I evaluate three dimensions per page: completeness (all required OG/Twitter tags present), quality (image size, title/description compelling), and accuracy (tags match page content, not template defaults).

---

## §3 The audit

### Open Graph tags
- Does every page have **`og:title`** (compelling, page-specific, 60-90 characters)?
- Does every page have **`og:description`** (specific to the page, 200-300 characters, includes value proposition)?
- Does every page have **`og:image`** (minimum 1200×630px, high quality, relevant)?
- Does every page have **`og:url`** (canonical URL of the page)?
- Is **`og:type`** set appropriately (`website` for homepage, `article` for blog posts, `product` for products)?
- For articles: are **`article:published_time`** and **`article:author`** set?
- Is the **`og:site_name`** set (brand name that appears in some previews)?

### Twitter Card tags
- Is **`twitter:card`** set (`summary_large_image` for most pages, `summary` for minimal previews)?
- Is **`twitter:title`** set (falls back to `og:title` if missing)?
- Is **`twitter:description`** set (falls back to `og:description` if missing)?
- Is **`twitter:image`** set (minimum 800×418 for large image, 120×120 for summary)?
- Is **`twitter:site`** set (the brand's Twitter handle)?

### Image quality
- Is the OG image **at least 1200×630px** (Facebook recommended)?
- Is the image **under 8MB** (Facebook's limit)?
- Is the image **relevant to the page content** (not a generic logo on every page)?
- Is the image **visually compelling** when displayed as a link preview?
- Does the image have **text that's readable** at small preview sizes?
- Is the image **served via HTTPS**?
- For **product pages**: is the OG image the product photo?
- For **blog posts**: is the OG image the featured image?

### Template and dynamic pages
- Do **all page types** have OG tags (not just the homepage)?
- Are OG tags **dynamically generated** from page content, or hardcoded template defaults?
- For **paginated pages**: do OG tags reference the first page (canonical)?
- For **user-generated content**: are OG tags sanitized (no XSS via user-controlled OG values)?

### Validation and caching
- Do pages pass **Facebook Sharing Debugger** validation?
- Do pages pass **Twitter Card Validator**?
- Do pages pass **LinkedIn Post Inspector**?
- After **content updates**: is the social cache invalidated? (Old previews may persist for days/weeks.)

---

## §4 Pattern library

**The logo-as-OG-image** — Every page uses the company logo (200×200px) as the OG image. When shared, the preview shows a tiny stretched logo. It looks broken. Fix: create a default OG image (1200×630) that's visually compelling. Even better: page-specific images for key pages.

**The template default everything** — OG title: "Acme Corp." OG description: "Welcome to Acme Corp, a leading provider of solutions." Every page shows the same preview. Fix: dynamic OG tags generated from page content (title from H1, description from meta description or first paragraph, image from featured image).

**The missing image** — Blog posts without OG images. Shared links show a plain text preview (title and description only). These get 50-70% fewer clicks than shares with images. Fix: every blog post needs a featured image that doubles as the OG image.

**The cached stale preview** — The page was updated with a new title and image. But Facebook/LinkedIn still show the old preview because their crawlers cached the old version. Fix: use Facebook Sharing Debugger to "Scrape Again" after updates. Include cache invalidation in the content update workflow.

**The auto-generated gibberish** — No meta description, no OG description. The platform auto-extracts text from the page: "Cookie settings | Skip to main content | Menu | Home About Services..." Fix: explicit OG description on every page.

---

## §5 The traps

**The "SEO meta tags are enough" trap** — Meta title and description are for search engines. OG tags are for social platforms. They can be identical, but OG tags serve a different context (social discovery vs. search intent). A good OG description might be more conversational than a good meta description.

**The "we don't use social media" trap** — You may not share your own pages, but your customers, partners, and employees do. Slack, iMessage, and email clients also use OG tags for link previews. OG tags affect how your site appears across many platforms, not just social media.

**The "one image fits all" trap** — A single generic OG image for the entire site is better than no image, but page-specific images perform significantly better. For key pages (homepage, top products, pillar content), invest in custom OG images.

**The "validation once is enough" trap** — OG tag implementation can break silently (CMS update, template change, plugin conflict). Periodic validation is necessary, not just initial setup.

---

## §6 Blind spots and limitations

**Platform rendering varies.** Facebook, LinkedIn, Twitter, Slack, and iMessage all render OG data slightly differently. Image cropping, text truncation, and layout vary by platform. Test on the platforms your audience uses most.

**Social sharing is not a direct SEO ranking factor.** Google doesn't use OG tags for ranking. Social sharing drives traffic and brand awareness, which can indirectly improve SEO (more backlinks, more branded searches), but the mechanism is indirect.

**Dynamic OG tags require server-side rendering.** If OG tags are injected by JavaScript, social platform crawlers (which don't execute JavaScript) won't see them. OG tags must be in the server-rendered HTML.

**Social platforms cache aggressively.** Changed OG tags don't take effect until each platform re-crawls the page. For time-sensitive content, proactive cache invalidation is necessary.

---

## §7 Cross-framework connections

| Framework | Interaction with social sharing |
|-----------|-------------------------------|
| **Meta Tags** | OG title and description often mirror meta title and description. They serve complementary contexts (social vs. search). |
| **Image SEO** | OG images should follow image SEO practices (proper format, reasonable size, descriptive content). |
| **Structured Data** | Structured data and OG tags serve different machines (search engines vs. social platforms). Both should describe the same content consistently. |
| **JS Rendering** | OG tags must be server-rendered. Social platform crawlers don't execute JavaScript. |
| **URL Structure** | `og:url` should match the canonical URL. Inconsistency confuses platforms about which URL is authoritative. |
| **Technical SEO** | OG tags on pages blocked by robots.txt may not be accessible to social platform crawlers. |

---

## §8 Severity calibration

| Context | Minor (cosmetic) | Moderate (missed opportunity) | Critical (broken sharing) |
|---------|-------------------|-------------------------------|---------------------------|
| **Homepage** | Suboptimal image crop | Generic OG description | No OG tags at all |
| **Blog/content** | Minor title length issues | Posts without OG images | OG tags show wrong content |
| **E-commerce** | Product OG image suboptimal | Product pages without OG tags | Sharing shows "undefined" or template text |
| **Landing pages** | Minor description tweaks | No OG image (text-only preview) | OG tags from a different page (cache/template issue) |
| **B2B/corporate** | Minor branding inconsistency | Service pages without OG tags | OG description pulls from navigation menu |

**Severity multipliers:**
- **Sharing frequency**: Pages that are frequently shared (blog posts, product pages, landing pages) benefit more from OG optimization.
- **Social referral traffic**: If social is a significant traffic source, OG quality directly affects traffic volume.
- **B2B context**: LinkedIn is critical for B2B. Broken LinkedIn previews hurt professional credibility.
- **Employee advocacy**: If employees share company content, broken previews embarrass the company and reduce sharing.

---

## §9 Build Bible integration

| Bible principle | Application to social sharing |
|-----------------|-------------------------------|
| **§1.6 Config-driven** | OG tag templates should be configured in the CMS with dynamic variables (page title, description, featured image). Manual OG tag coding per page doesn't scale. |
| **§1.5 Single source of truth** | `og:url` should be the canonical URL. `og:title` and `og:description` should derive from the same content as meta title and description. Different values create inconsistency. |
| **§1.8 Prevent, don't recover** | CMS validation that requires a featured image before publishing prevents OG-image-less pages. Discovering broken previews after someone shares your page is recovery. |
| **§6.9 Silent placeholder** | Generic template OG tags ("Welcome to our website") look real in the HTML but communicate nothing when shared. They're placeholders that actively harm sharing. |
| **§1.12 Observe everything** | Social referral traffic by page, sharing preview validation, and platform-specific analytics are the observability layer. Without them, you don't know if sharing works. |
| **§1.13 Unhappy path first** | What does the preview look like when: the featured image is missing? The title is empty? The page is a 404? Define fallbacks for these cases. |
