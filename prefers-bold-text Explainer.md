# **Explainer: prefers-bold-text Media Query**

[Explainer: prefers-bold-text Media Query](#heading=)

[Authors](#authors)

[Introduction](#heading=)

[Use Cases](#heading=)

[Existing Implementations and Behavior](#heading=)

[Web](#web)

[Mobile OS](#mobile-os)

[Proposed Solution](#heading=)

[Syntax](#heading=)

[Example Usage](#heading=)

[Alternatives Considered](#heading=)

[Concerns with Automatic Bolding (User Agent Intervention)](#concerns-with-automatic-bolding-%28user-agent-intervention%29)

[Comparison with Text Size Scaling](#comparison-with-text-size-scaling)

[Preventing "Double-Bolding" When Combining Both Tiers](#preventing-"double-bolding"-when-combining-both-tiers)

[Developer Adoption and Ecosystem Drivers](#developer-adoption-and-ecosystem-drivers)

[Privacy and Security Considerations](#heading=)

[Draft Specification](#heading=)

[X.Y. Detecting the desire for bolder text: the prefers-bold-text feature](#heading=)

## 

## **Authors** {#authors}

* [Gregory Dardyk](mailto:gregoryd@google.com)

## 

## **Introduction**

As digital accessibility becomes increasingly prominent, operating systems have introduced various user preferences to improve readability. One highly utilized accessibility feature across major mobile platforms (including [iOS](https://support.apple.com/en-bn/guide/iphone/iphd6804774e/ios) and [Android](https://support.google.com/accessibility/android/answer/11183305?hl=en#zippy=,use-bold-fonts)) is the **"Bold Text"** toggle. According to [accessibility statistics from Appt.org](https://appt.org/en/stats/bold-text), this feature is widely utilized, with approximately 13% of users turning on bold text on iOS and 5.2% of users turning it on for Android. When enabled, this setting increases the font weight of the system UI, significantly aiding users with low vision, astigmatism, or age-related visual decline (presbyopia).

Currently, web developers have no standardized way to detect this user preference. Consequently, users experience a jarring transition: their operating system UI is highly legible and bolded, but web content remains thin, delicate, and difficult to read. Browsers cannot simply apply the user's system-level font weight to all web pages automatically, because—if they did—many existing page layouts would break, causing content to overflow, become invisible, or lose interactivity.

This explainer proposes a new user preference media query, prefers-bold-text, allowing developers to proactively tailor their web typography to respect the user's system-level bold text setting without compromising their layout.

## **Use Cases**

1. **Adaptive Font Weights:** A user has enabled "Bold Text" in their OS accessibility settings. A news website detects this via @media (prefers-bold-text: bold) and increases its base body text weight from 400 to 700, ensuring the article is readable for that user.
**Variable Font Optimization:** A site uses a variable font. When prefers-bold-text is active, the developer seamlessly shifts the weight and/or the grade (GRAD) axes to provide a thicker, more legible text stroke. Unlike altering weight, modifying grade does not affect the text's overall width or spacing, preventing unwanted changes to line breaks or page layout.  
3. **Alternative Font Families:** A website relies on a very thin, stylized display font for headings (e.g., font-weight: 200). Because simply bolding this specific font makes it look muddy, the developer uses the media query to swap it out for a robust, highly legible sans-serif alternative.

## **Existing Implementations and Behavior**

### **Web** {#web}

Some User Agents already attempt to respect the OS-level bold text preference, but their reach is limited without developer intervention. For example, Safari on iOS automatically increases the font weight of web text when the system "Bold Text" setting is enabled ([evidence](https://stackoverflow.com/questions/74100048/bold-text-on-webpages-when-bold-text-ios-settings-enabled))—but **only** if the webpage is using standard system fonts (like font-family: system-ui).

If a website relies on custom web fonts, this automatic system-level bolding is bypassed. Because a vast majority of websites use custom fonts for branding, users are left with an inconsistent experience where only parts of the web respect their accessibility needs. The prefers-bold-text media query is necessary to allow developers to bridge this gap and properly support the feature across all font stacks.

### **Mobile OS** {#mobile-os}

Major mobile ecosystems provide native programming interfaces that allow applications to query user preferences for bold typography:

* iOS uses the [LegibilityWeight enum](https://developer.apple.com/documentation/swiftui/legibilityweight) in SwiftUI to distinguish between regular and bold weights.
* Android offers the [Configuration.fontWeightAdjustment](https://developer.android.com/reference/android/content/res/Configuration#fontWeightAdjustment) property, which provides an integer value to be applied to the default font weight.

## **Proposed Solution**

We propose adding **prefers-bold-text** to the Media Queries Level 5 specification, under [User Preference Media Features](https://www.w3.org/TR/mediaqueries-5/#mf-user-preferences).

### **Syntax**

The prefers-bold-text media feature would accept the following values:

* **no-preference**: Indicates that the user has made no preference known to the system. This evaluates as false in the boolean context.
* **bold**: Indicates that the user has expressed a preference for bolder text to improve readability.

### **Example Usage**

**Example 1: Basic Font Weight Adjustment**

body {  
  font-family: system-ui, sans-serif;  
  font-weight: 400; /\* Standard reading weight \*/  
}

h1, h2, h3 {  
  font-weight: 700;  
}

@media (prefers-bold-text: bold) {  
  body {  
    font-weight: 600; /\* Increased base legibility \*/  
  }  
    
  h1, h2, h3 {  
    font-weight: 900; /\* Scale up headings to maintain visual hierarchy \*/  
  }  
}

**Example 2: Variable Fonts**

body {  
  font-family: "Roboto Flex", sans-serif;  
  font-variation-settings: 'wght' 400, 'GRAD' 0;  
}

@media (prefers-bold-text: bold) {  
  body {  
    /\* Utilizing the Grade axis to thicken text without reflowing the layout \*/  
    font-variation-settings: 'wght' 400, 'GRAD' 150;
  }  
}

### Authoring Guidance: Relative Weight Scaling & Preserving Hierarchy
OS "Bold Text" settings represent a **relative increase** in stroke weight rather than setting all text on a page to a single fixed weight (`700`). To preserve visual hierarchy between body copy, medium UI labels, and bold headings or `<strong>` emphasis, authors should shift weights up proportionally, following the model described in [CSS Fonts Level 4 `bolder`](https://drafts.csswg.org/css-fonts-4/#relative-weights).
For **Variable fonts (`GRAD` axis)** authors should increase `'GRAD'` (e.g., `+100` to `+150`), which thickens strokes across all weights while keeping relative weight hierarchy and character widths unchanged.

## **Alternatives Considered**

* Using [**webkit-text-stroke-width**](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/-webkit-text-stroke-width)**:** webkit-text-stroke is a stylistic tool and is notoriously poor for general legibility, as it draws the stroke inside/outside the glyph paths in ways that often close up the counters (the empty spaces inside letters like 'e' and 'o'), reducing legibility rather than improving it.
* Introducing **@media (font-weight-adjustment: 300)** - this media query would allow websites to get the recommended font weight adjustment, based on the OS settings. This follows the Android API approach mentioned above. We decided against it because supporting it would require more effort from the developers: from computing the font weight at runtime to mapping weight adjustment to variable font properties. If the need for more granularity arises in the future, we can adapt by supporting more values for prefers-bold-text.

## **Concerns with Automatic Bolding (User Agent Intervention)** {#concerns-with-automatic-bolding-(user-agent-intervention)}

Automatic bolding, where the browser unilaterally increases font weights across web pages, is another alternative approach. While it would immediately apply to legacy or unmaintained websites without developer action, it introduces major layout, typographic, and interoperability challenges:

* **Layout Integrity (Why Automatic Bolding Must Be Opt-In)**: Unilateral browser bolding alters character advance widths and font metrics, risking layout reflow, text clipping, and visual breakage across three key categories of content:

  1. **Legacy websites**: Pages built with fixed-width containers or tight pixel boundaries cannot safely absorb unexpected increases in text width.
  2. **Commercial websites and advertisements**: High-traffic sites invest heavily in calibrating their responsive layouts and optimizing visual hierarchies. Online advertisements are especially sensitive to font and layout shifts, as they operate within strictly constrained slot dimensions where minor metric changes cause text truncation, awkward line wraps, or clipped calls-to-action (CTAs).
  3. **Web applications**: Productivity tools, such as text editors and design apps, depend on exact font metrics and deliberate weight hierarchies. Overriding weights automatically risks corrupting cursor positioning and line wrapping.
* **Technical and Typographic Limitations**: Even when a site opts into automatic bolding, browser-driven weight adjustments face inherent rendering limits:

  * **Font weight availability**: Target bolder weights do not exist across all font families. This may force the browser to either substitute other fonts (causing metric mismatches) or fall back to synthetic bolding.
  * **Legibility degradation from synthetic bolding**: Algorithmic synthetic bolding thickens glyph outlines artificially, often closing up internal glyph counters and distorting letterforms. This is particularly harmful in dense, mixed-language, or non-Latin scripts (such as CJK ideographs), where multi-stroke characters become visually crowded and harder to read, directly contradicting the accessibility goal.
* **Cross-Browser Interoperability**: Automated bolding, if implemented, would heavily rely on engine-specific heuristics, e.g. to decide which elements to embolden. Because rendering pipelines and heuristics differ across user agents, developers cannot guarantee a consistent, predictable visual experience across browsers.
* **Necessity of Explicit Preference Detection**:

  * **Developer overrides**: Even alongside automatic browser heuristics, developers still require `@media (prefers-bold-text)` to refine and override specific components where automatic adjustments produce poor results.
  * **Non-DOM and rendered assets**: Browser-level DOM bolding cannot reach text rendered inside `<canvas>` elements, WebGL scenes, SVG diagrams, or bitmap images. Explicit preference detection is essential so developers can re-render canvas text or swap graphic assets to match the rest of the page.


## **Comparison with Text Size Scaling** {#comparison-with-text-size-scaling}

The CSS Working Group previously addressed a similar challenge for OS-level text size scaling by standardizing a two-tier model: a granular CSS primitive (`env(preferred-text-scale)` and `text-size-adjust`) for full author control, paired with a declarative HTML opt-in (`<meta name="text-scale" content="scale">`) for browser-managed scaling.

A key takeaway from that rollout was **adoption order**: owners of high-traffic websites, who invest heavily in calibrating layouts, typography and ad placements, adopted the granular `env(preferred-text-scale)` primitive first so they could scale selectively without breaking their designs. We propose the same sequencing for bold text: **ship detailed developer control first (`@media (prefers-bold-text)`), and layer opt-in automation second.**

* **Step 1 — Granular Developer Control**: Like `env(preferred-text-scale)`, `@media (prefers-bold-text)` ships first as the foundational standard, giving developers direct CSS control over font weights, variable font `GRAD` axes, font families, and non-DOM assets (`<canvas>`, SVG) without risking layout breakage.
* **Step 2 — Declarative Opt-In (`<meta name="text-bold">` vs. `<meta name="text-scale">`)**: A declarative `<meta name="text-bold" content="bold">` tag (or CSS property like `font-bold-adjust: auto`) can follow in a future revision as a low-effort opt-in for content-heavy sites with simpler layouts, once developers already have the media query to refine and override edge cases.

**Co-occurrence with Text Scaling and WCAG 2.2 Reflow ([1.4.4](https://www.w3.org/TR/WCAG22/#resize-text), [1.4.10](https://www.w3.org/TR/WCAG22/#reflow), [1.4.12](https://www.w3.org/TR/WCAG22/#text-spacing))**: Users who enable Bold Text often also enable OS Large Text. `@media (prefers-bold-text)` composes cleanly with `<meta name="text-scale" content="scale">` and `env(preferred-text-scale)`. While increasing `font-weight` expands proportional text width by less than 10%, authors should test `prefers-bold-text: bold` alongside 200% text scaling to ensure layouts continue to meet WCAG requirements.

## **Preventing "Double-Bolding" When Combining Both Tiers** {#preventing-"double-bolding"-when-combining-both-tiers}

If an author uses `@media (prefers-bold-text: bold)` to bump body text from `400` to `600`, a future auto-bolding mechanism must not add additional weight on top of that `600`. We define the forward-compatibility rules now:

1. **Opt-in isolation**: Because future standardized auto-bolding must be opt-in to avoid breaking web layouts, pages using `@media (prefers-bold-text)` will never be auto-bolded unless they explicitly add the opt-in.
2. **Automatic suppression on media-query-styled elements (Auto Dark Mode precedent)**: Just as Blink's Auto Dark Mode (`ForceDark`) disables algorithmic color inversion on elements styled with `color-scheme: dark` / `@media (prefers-color-scheme: dark)`, a User Agent implementing auto-bolding **must not** apply automatic weight increases to any element whose `font-weight`, `font-variation-settings`, or `font-family` is declared inside an active `@media (prefers-bold-text: bold)` block.
3. **Subtree opt-out (`font-bold-adjust: auto | none`)**: Mirroring `forced-color-adjust: none`, a future auto-bolding spec can provide `font-bold-adjust: none` so authors who opt in globally can still disable auto-bolding on specific subtrees (such as ads, code editors, or CJK text).


## **Developer Adoption and Ecosystem Drivers** {#developer-adoption-and-ecosystem-drivers}

While WCAG is intentionally OS-agnostic and does not mandate OS-level bold text support, four strong industry drivers incentivize web developer adoption:

1. **Statutory Accessibility Compliance (Section 508 \& EN 301 549)**: Unlike WCAG, **U.S. Section 508 (§ 503.2)** and **European Standard EN 301 549 ([V4.1.1, Clause 9.7, "User preferences"](https://www.etsi.org/deliver/etsi_en/301500_301599/301549/04.01.01_60/en_301549v040101p.pdf))** explicitly require software to respect platform-level settings for font size, font type/legibility, color, and contrast. Under these requirements, once browsers expose an OS preference via a standard CSS API (as happened with `prefers-reduced-motion` and `prefers-contrast`), honoring it becomes an actionable compliance expectation for public-sector and enterprise web applications.
2. **Parity Across Hybrid Apps, WebViews, and App Store Labels**: Modern mobile apps frequently mix native views (SwiftUI, Jetpack Compose) with embedded web surfaces (`WKWebView`, Android `WebView`, and PWAs). When OS Bold Text is enabled, thin WebViews inside an otherwise bold native app look broken to users. Furthermore, Apple's **App Store Accessibility Nutrition Labels** explicitly highlight whether an app supports **Bold Text**; exposing `prefers-bold-text` allows hybrid and WebView-based apps to achieve full platform parity and qualify for these badges.
3. **Proven Demand for `prefers-\*` Media Features**: Given the mainstream adoption of OS Bold Text (13% on iOS, 5.2% on Android), aligning with this preference reduces visual fatigue and improves retention - the same product incentives that drove widespread voluntary adoption of `@media (prefers-color-scheme)`.
4. **Minimal Implementation Cost via Design Tokens**:

   * **Root-level tokens**: As shown in *Example 1*, design systems that manage typography via CSS Custom Properties (`--font-weight-body`, `--font-weight-heading`) can support `prefers-bold-text` in fewer than 10 lines of `:root` CSS.
   * **Orthogonal to other `prefers-\*` queries**: Unlike color and contrast media queries, which interact to create a complex multi-palette testing matrix, font weight and `GRAD` adjustments are completely independent of color scheme, motion, or transparency, allowing standalone implementation and testing.
   * **Framework multiplier**: Once component libraries (Material Design, Tailwind, Bootstrap, CMS themes) update their default token scales, downstream sites inherit support automatically.

## **Privacy and Security Considerations**

Like all user preference media queries ([prefers-color-scheme](http://prefers-color-scheme), [prefers-reduced-motion](http://prefers-reduced-motion)), exposing this setting adds a minor bit of entropy to the user's fingerprinting surface. However, because system-level bold text is a mainstream legibility preference used by a vast demographic (including roughly 13% of iOS users), its identifying entropy is low. Also, it’s important to note that since this is such a popular preference, it does not uniquely identify a clinical disability.

To summarize, the accessibility benefits heavily outweigh the minimal fingerprinting risk. Furthermore, as per [the Privacy Considerations section in the Media Queries Level 5 specification](https://www.w3.org/TR/mediaqueries-5/#privacy), User Agents may choose to obscure this preference (e.g., returning no-preference) if the user is in a strict anti-tracking mode (like Tor Browser or Safari's Advanced Tracking and Fingerprinting Protection).

## **Draft Specification**

### **X.Y. Detecting the desire for bolder text: the prefers-bold-text feature**

**Name:** prefers-bold-text

**For:** @media

**Value:** no-preference | bold

**Type:** discrete

The prefers-bold-text media feature is used to detect if the user has requested the system to display text with a bolder font weight to improve legibility.

**no-preference** Indicates that the user has made no preference known to the system. This keyword value evaluates as false in the boolean context.

**bold** Indicates that the user has notified the system that they prefer an interface that uses bolder text for improved legibility and readability.
