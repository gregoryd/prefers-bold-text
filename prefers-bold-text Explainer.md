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

[Privacy and Security Considerations](#heading=)

[Draft Specification](#heading=)

[X.Y. Detecting the desire for bolder text: the prefers-bold-text feature](#heading=)

## 

## **Authors** {#authors}

* [Gregory Dardyk](mailto:gregoryd@google.com) 

## 

## **Introduction**

As digital accessibility becomes increasingly prominent, operating systems have introduced various user preferences to improve readability. One highly utilized accessibility feature across major mobile platforms (including [iOS](https://support.apple.com/en-bn/guide/iphone/iphd6804774e/ios) and [Android](https://support.google.com/accessibility/android/answer/11183305?hl=en#zippy=%2Cuse-bold-fonts)) is the **"Bold Text"** toggle. According to [accessibility statistics from Appt.org](https://appt.org/en/stats/bold-text), this feature is widely utilized, with approximately 13% of users turning on bold text on iOS and 5.2% of users turning it on for Android. When enabled, this setting increases the font weight of the system UI, significantly aiding users with low vision, astigmatism, or age-related visual decline (presbyopia).

Currently, web developers have no standardized way to detect this user preference. Consequently, users experience a jarring transition: their operating system UI is highly legible and bolded, but web content remains thin, delicate, and difficult to read. Browsers cannot simply apply the user's system-level font weight to all web pages automatically, because—if they did—many existing page layouts would break, causing content to overflow, become invisible, or lose interactivity.

This explainer proposes a new user preference media query, prefers-bold-text, allowing developers to proactively tailor their web typography to respect the user's system-level bold text setting without compromising their layout.

## **Use Cases**

1. **Adaptive Font Weights:** A user has enabled "Bold Text" in their OS accessibility settings. A news website detects this via @media (prefers-bold-text: bold) and increases its base body text weight from 400 to 700, ensuring the article is readable for that user.  
2. **Variable Font Optimization:** A site uses a variable font. When prefers-bold-text is active, the developer seamlessly shifts the weight and GRAD axes to provide a thicker, more legible text stroke without breaking the layout.  
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

## **Alternatives Considered**

* Using [**webkit-text-stroke-width**](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/-webkit-text-stroke-width)**:** webkit-text-stroke is a stylistic tool and is notoriously poor for general legibility, as it draws the stroke inside/outside the glyph paths in ways that often close up the counters (the empty spaces inside letters like 'e' and 'o'), reducing legibility rather than improving it.  
* Introducing **\<meta name="text-bold" content="bold"\>** \- this meta tag would allow websites to signal the UA that their layout supports switching to bold fonts (similar to [\<meta name="text-scale" content="scale"\>](https://github.com/w3c/csswg-drafts/issues/12380)). If this tag is present, UA will automatically increase the font weight for all fonts. While it is a step forward and may require less work from the website developers, it would not support the variable fonts and alternative fonts mentioned in the “Use Cases” section above.  We may consider adding the support for this meta tag in the future, in addition to the media query that is proposed in the current document.  
* Introducing **@media (font-weight-adjustment: 300\)** \- this media query would allow websites to get the recommended font weight adjustment, based on the OS settings. This follows the Android API approach mentioned above. We decided against it because supporting it would require more effort from the developers: from computing the font weight at runtime to mapping weight adjustment to variable font properties. If the need for more granularity arises in the future, we can adapt by supporting more values for prefers-bold-text.  
* User Agent Intervention to automatically embolden text to respect the user's preferences. The benefit of this approach is that all existing content could be improved immediately. The downsides of this approach are:  
  1. Existing fonts on the page may not have the available bolder font weights, requiring downloading different fonts, synthesizing bold, or using alternative fonts.  
  2. Compatibility: sites may expect specific font metrics. For example, a text editor may require specific text dimensions.  
  3. Detection of the user's preference is a prerequisite for an automatic intervention. Detection and opt-out controls would be required so that sites could fix cases where the automatic behavior is not desired. Detection would be needed to allow sites to adjust text used in images.  
  4. Interoperability: if the support of this setting is implemented in different ways in different  browsers, page authors will find it hard to achieve a consistent user experience for their website users.  
  5. Quality: bolding in mixed-language environments can degrade, rather than enhance, accessibility, as structurally dense scripts like Chinese lose legibility when their strokes become visually crowded.  
     

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