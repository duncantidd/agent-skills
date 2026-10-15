# CSS Native — Modern Typography & Readability

## Typography Decision Matrix

| Goal | Tool | Why | 
| ----- | ----- | ----- | 
| Responsive font sizing | `clamp()` | Scales text smoothly between a minimum and maximum size based on the viewport, eliminating media queries. | 
| Headline line breaks | `text-wrap: balance` | Distributes words evenly across lines to avoid awkward single-word runovers. | 
| Paragraph text flow | `text-wrap: pretty` | Prevents orphans (single words on the last line) at the end of a block of text. | 
| Multi-line truncation | `line-clamp` | Truncates text to a specified line count with an ellipsis without fragile JS substring calculation. | 
| Safe word-wrapping | `overflow-wrap: break-word` | Prevents long URLs or unbroken tokens from breaking out of containers while keeping normal word breaks. | 
| Limiting line length | `ch` unit | Sets container width based on character count for optimal reading ergonomics (60-80 chars). | 
| Aligning icons to text | `em` or `ex` unit | Scales relative to the specific font's size or x-height, keeping them perfectly proportioned. | 
| Preventing FOIT (Invisible Text) | `font-display: swap` | Displays system fallback font immediately while custom web font finishes downloading. | 

## Fluid Typography

Avoid jumping between font sizes at arbitrary breakpoints. Use `clamp(MIN, VAL, MAX)` to create typography that fluidly adapts to the viewport.

```css
:root {
  /* Fluid Base: Scales between 16px and 20px */
  /* Formula: clamp(min-size, scaling-factor, max-size) */
  --text-base: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
  
  /* Fluid Heading: Scales between 2rem (32px) and 4rem (64px) */
  --text-h1: clamp(2rem, 1.2rem + 4vw, 4rem);
}

body {
  font-size: var(--text-base);
}

h1 {
  font-size: var(--text-h1);
}
```

*Tip: Use tools like [Utopia.fyi](https://utopia.fyi/) to generate mathematically consistent fluid typography scales.*

## Text Wrapping & Readability

Native CSS now solves classic typesetting problems that previously required JavaScript or manual `<br>` tags.

### Balance (For Headings)

Calculates the number of lines and distributes the text evenly. It is computationally expensive, so browsers limit it to blocks of text with fewer than 4-6 lines.

```css
h1, h2, h3, .hero-title {
  text-wrap: balance;
  max-inline-size: 25ch; /* Optional: constrain max width */
}
```

### Pretty (For Paragraphs)

Focuses on the end of the text block to ensure the final line doesn't end with a single, isolated word (an orphan). 

```css
p, .article-body {
  text-wrap: pretty;
}
```

## Ideal Line Length & Relative Units

Reading speed and comprehension drop if lines are too long or too short. Use the `ch` (character) unit to define optimal container widths.

```css
.article-container {
  /* 1ch = the width of the "0" (zero) character in the current font */
  max-inline-size: 65ch; /* 60-75ch is the sweet spot for reading */
  margin-inline: auto;   /* Center the reading column */
}
```

### Contextual Sizing (`em` and `ex`)

*   `rem`: Relative to the root `<html>` element. Use this for margins, padding, and font-sizes.
*   `em`: Relative to the *current* element's font size. Perfect for icons.
*   `ex`: Relative to the x-height of the current font.

```css
.inline-icon {
  inline-size: 1.2em;
  block-size: 1.2em;
  vertical-align: middle; /* or use flexbox */
}
```

## Variable Fonts

Instead of loading `Roboto-Regular.woff2`, `Roboto-Bold.woff2`, and `Roboto-Italic.woff2`, load a single Variable Font file to access an infinite spectrum of weights and styles.

```css
@font-face {
  font-family: 'Inter Variable';
  src: url('/fonts/Inter-Variable.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-style: normal;
}

body {
  font-family: 'Inter Variable', system-ui, sans-serif;
  font-weight: 400; /* Standard */
}

h1 {
  /* You are no longer restricted to 400, 500, 600, 700 */
  font-weight: 750; 
  
  /* Fine-tune specific axes if the font supports them (e.g., optical size, slant) */
  font-variation-settings: "opsz" 32, "slnt" -5;
}
```

---

## Modern System Font Stack

If you don't want to load external fonts, rely on the modern system font stack. It maps to San Francisco on Apple, Segoe UI on Windows, and Roboto on Android.

```css
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}
```

---

## Multi-Line Text Truncation (`line-clamp`)

Clamp multiline text to a maximum number of lines with an automatic ellipsis:

```css
.card-preview {
  /* Modern standard property with webkit fallback for universal compatibility */
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  overflow: hidden;
}
```

---

## Word Breaking & Hyphenation

Prevent awkward overflows from URLs or unbroken words without mangling ordinary text:

```css
/* Apply globally to root or body */
body {
  /* Breaks long strings like URLs ONLY if they would otherwise overflow */
  overflow-wrap: break-word;
}

/* For narrow reading columns or languages with long compound words (e.g. German) */
.narrow-column {
  hyphens: auto; /* Requires HTML lang="en" (or appropriate language tag) */
}
```

---

## Font Rendering & Faux-Font Prevention

Browsers attempt to "synthesize" bold and italic versions if they are not in the font file, producing distorted glyphs:

```css
body {
  /* Prevents artificial, distorted bold/italic synthesis */
  font-synthesis: none;
  /* Optimizes kerning and ligature rendering */
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}
```

### Font Loading Strategy (`font-display: swap`)

Always declare `font-display: swap` in `@font-face` to prevent Flash of Invisible Text (FOIT) and improve Largest Contentful Paint (LCP):

```css
@font-face {
  font-family: 'BrandSans';
  src: url('/fonts/brand-sans.woff2') format('woff2');
  font-display: swap; /* Falls back to system font instantly until loaded */
  font-weight: 400 700;
}
```

---

## Do Not

| BAD | GOOD | Why | 
| ----- | ----- | ----- | 
| `font-size: 16px` | `font-size: 1rem` | Hardcoded pixels override user browser preferences (e.g., visually impaired users who set their default size to 24px). | 
| `line-height: 24px` or `1.5em` | `line-height: 1.5` | Always use unitless `line-height`. Units cascade poorly and can cause text clipping in nested elements. | 
| `text-align: justify` | `text-align: start` (or left) | Justification creates uneven rivers of white space that hurt readability, especially for dyslexic readers. | 
| `text-align: left` / `right` | `text-align: start` / `end` | Logical properties ensure text alignment automatically respects Right-to-Left (RTL) languages. | 
| Wrapping headings in `<br>` | `text-wrap: balance` | Hardcoded `<br>` tags break unpredictably on mobile devices. | 
| `word-break: break-all` | `overflow-wrap: break-word` | `break-all` splits ordinary words anywhere mid-line; `overflow-wrap` only breaks when necessary. | 
| Omitted `font-display` | `font-display: swap` | Browsers hide text for up to 3 seconds during font download (FOIT), hurting UX and Core Web Vitals. | 
| Synthetic faux bolding | `font-synthesis: none` | Let real font weights render instead of muddy, browser-generated simulated bolding. | 
| JavaScript line truncation | `line-clamp: 3` | Native CSS clamping runs on compositor without layout shifts, DOM measuring, or string cutting bugs. | 