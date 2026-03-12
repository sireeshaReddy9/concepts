# CSS Fundamentals: A Technical Paper

---

## Table of Contents

1. [The Box Model](#1-the-box-model)
2. [Inline vs Block Elements](#2-inline-vs-block-elements)
3. [Positioning: Relative & Absolute](#3-positioning-relative--absolute)
4. [Common CSS Structural Classes](#4-common-css-structural-classes)
5. [Common CSS Styling Classes](#5-common-css-styling-classes)
6. [CSS Specificity](#6-css-specificity)
7. [CSS Responsive Queries](#7-css-responsive-queries)
8. [Flexbox & Grid](#8-flexbox--grid)
9. [Common Header Meta Tags](#9-common-header-meta-tags)
10. [CSS Variables](#10-css-variables)
11. [References](#references)

---

## 1. The Box Model

Every HTML element is treated as a rectangular box. The CSS Box Model defines how the size and spacing of that box is calculated. It consists of four layers:

```
+---------------------------+
|         Margin            |
|  +---------------------+  |
|  |      Border         |  |
|  |  +---------------+  |  |
|  |  |    Padding    |  |  |
|  |  |  +---------+  |  |  |
|  |  |  | Content |  |  |  |
|  |  |  +---------+  |  |  |
|  |  +---------------+  |  |
|  +---------------------+  |
+---------------------------+
```

- **Content** — The actual text or image inside the element
- **Padding** — Space between the content and the border
- **Border** — A line surrounding the padding
- **Margin** — Space outside the border, separating it from other elements

### box-sizing

By default, `width` and `height` only apply to the content area (`content-box`). Using `border-box` makes the width include padding and border, which is far more intuitive:

```css
* {
  box-sizing: border-box;
}

.card {
  width: 300px;    /* includes padding and border */
  padding: 20px;
  border: 2px solid #ccc;
}
```

---

## 2. Inline vs Block Elements

### Block Elements

Block elements take up the full width of their container and always start on a new line.

```html
<div>, <p>, <h1>–<h6>, <section>, <article>, <header>, <footer>, <ul>, <li>
```

```css
div {
  display: block;  /* default */
  width: 100%;
}
```

### Inline Elements

Inline elements only take up as much width as their content and do not start on a new line. You cannot set `width` or `height` on inline elements.

```html
<span>, <a>, <strong>, <em>, <img>, <button>, <label>, <input>
```

```css
span {
  display: inline;  /* default */
}
```

### Inline-Block

`inline-block` combines both — the element flows inline but respects `width`, `height`, `padding`, and `margin`.

```css
.badge {
  display: inline-block;
  width: 80px;
  padding: 4px 8px;
}
```

| Property        | Block | Inline | Inline-Block |
|-----------------|-------|--------|--------------|
| New line        | Yes   | No     | No           |
| Width / Height  | Yes   | No     | Yes          |
| Padding/Margin  | All sides | Left/Right only | All sides |

---

## 3. Positioning: Relative & Absolute

CSS `position` controls how an element is placed in the document flow.

### `position: static` (default)

The element follows the normal document flow. `top`, `left`, `right`, `bottom` have no effect.

### `position: relative`

The element stays in the normal flow but can be nudged using `top`, `left`, etc. It also becomes the **reference point** for any absolutely positioned children.

```css
.parent {
  position: relative;
  top: 10px;   /* moves down 10px from its original spot */
}
```

### `position: absolute`

The element is removed from the normal flow and positioned relative to its nearest `position: relative` (or `absolute`/`fixed`) ancestor. If none exists, it positions relative to the `<body>`.

```css
.parent {
  position: relative;
}

.tooltip {
  position: absolute;
  top: 0;
  right: 0;  /* sits in the top-right corner of .parent */
}
```

### `position: fixed`

Positioned relative to the browser viewport. Stays in place when scrolling.

```css
.navbar {
  position: fixed;
  top: 0;
  width: 100%;
}
```

### `position: sticky`

Acts like `relative` until a scroll threshold is met, then behaves like `fixed`.

```css
.sidebar {
  position: sticky;
  top: 20px;
}
```

---

## 4. Common CSS Structural Classes

Structural classes define layout and page scaffolding. These are conventions widely used across projects and frameworks.

```css
/* Page wrapper - centers and constrains content width */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
}

/* Rows and columns for grid-style layouts */
.row {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.col {
  flex: 1;
}

/* Utility wrappers */
.wrapper { width: 100%; }
.section  { padding: 60px 0; }
.hero     { min-height: 100vh; }

/* Spacing utilities */
.mt-1 { margin-top: 8px; }
.mt-2 { margin-top: 16px; }
.p-1  { padding: 8px; }
.p-2  { padding: 16px; }
```

---

## 5. Common CSS Styling Classes

Styling classes handle visual appearance — typography, color, alignment, and visibility.

```css
/* Typography */
.text-center  { text-align: center; }
.text-left    { text-align: left; }
.text-right   { text-align: right; }
.bold         { font-weight: bold; }
.italic       { font-style: italic; }
.uppercase    { text-transform: uppercase; }

/* Color utilities */
.text-primary   { color: #3490dc; }
.text-danger    { color: #e3342f; }
.bg-light       { background-color: #f8f9fa; }
.bg-dark        { background-color: #343a40; }

/* Visibility */
.hidden         { display: none; }
.invisible      { visibility: hidden; }
.show           { display: block; }

/* Borders & Shadows */
.rounded        { border-radius: 4px; }
.rounded-full   { border-radius: 9999px; }
.shadow         { box-shadow: 0 2px 4px rgba(0,0,0,0.1); }

/* Buttons */
.btn {
  display: inline-block;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background-color: #3490dc;
  color: white;
}
```

---

## 6. CSS Specificity

Specificity determines which CSS rule wins when multiple rules target the same element. It is calculated as a score with four components: `(inline, IDs, classes/attributes/pseudo-classes, elements/pseudo-elements)`.

| Selector                  | Specificity Score |
|---------------------------|-------------------|
| `*`                       | 0-0-0-0           |
| `p`                       | 0-0-0-1           |
| `.card`                   | 0-0-1-0           |
| `#header`                 | 0-1-0-0           |
| `style="..."`             | 1-0-0-0           |
| `!important`              | Overrides all     |

### Examples

```css
p { color: black; }               /* 0-0-0-1 */
.intro p { color: blue; }         /* 0-0-1-1 — wins over above */
#main p { color: red; }           /* 0-1-0-1 — wins over above */
p { color: green !important; }    /* overrides everything */
```

### Best Practices

- Avoid `!important` — it makes debugging very difficult
- Prefer classes over IDs for styling
- Keep specificity as low as possible for maintainability

---

## 7. CSS Responsive Queries

Media queries allow different styles to be applied based on the screen size or device properties.

### Syntax

```css
@media (condition) {
  /* styles applied when condition is true */
}
```

### Common Breakpoints

```css
/* Mobile first approach */
/* Base styles target mobile */

/* Tablet */
@media (min-width: 768px) {
  .container { padding: 0 32px; }
}

/* Desktop */
@media (min-width: 1024px) {
  .container { max-width: 1200px; }
}

/* Large screens */
@media (min-width: 1440px) {
  .hero { font-size: 2rem; }
}
```

### Other Query Types

```css
/* Dark mode */
@media (prefers-color-scheme: dark) {
  body { background: #121212; color: white; }
}

/* Print */
@media print {
  .navbar, .sidebar { display: none; }
}

/* Orientation */
@media (orientation: landscape) {
  .hero { min-height: 60vh; }
}
```

### Mobile-First vs Desktop-First

Mobile-first uses `min-width` (start small, scale up). Desktop-first uses `max-width` (start large, scale down). Mobile-first is generally the recommended approach today.

---

## 8. Flexbox & Grid

### Flexbox

Flexbox is a one-dimensional layout system — it arranges items along a single axis (row or column).

```css
.container {
  display: flex;
  flex-direction: row;       /* or column */
  justify-content: center;   /* main axis alignment */
  align-items: center;       /* cross axis alignment */
  flex-wrap: wrap;
  gap: 16px;
}

.item {
  flex: 1;                   /* grow to fill available space */
}
```

**Key properties:**

| Property          | Values                                      |
|-------------------|---------------------------------------------|
| `flex-direction`  | `row`, `column`, `row-reverse`              |
| `justify-content` | `flex-start`, `center`, `space-between`     |
| `align-items`     | `flex-start`, `center`, `stretch`           |
| `flex-wrap`       | `nowrap`, `wrap`                            |
| `flex`            | shorthand for `grow shrink basis`           |

### CSS Grid

Grid is a two-dimensional layout system — it handles both rows and columns simultaneously.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);   /* 3 equal columns */
  grid-template-rows: auto;
  gap: 24px;
}

/* Spanning columns */
.featured {
  grid-column: span 2;
}
```

**Named template areas:**

```css
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

### When to Use Which

- Use **Flexbox** for one-dimensional layouts: navbars, card rows, centering elements
- Use **Grid** for two-dimensional layouts: full page layouts, dashboards, image galleries

---

## 9. Common Header Meta Tags

Meta tags go inside the `<head>` element and provide metadata about the page to browsers, search engines, and social platforms.

```html
<!DOCTYPE html>
<html lang="en">
<head>

  <!-- Character encoding -->
  <meta charset="UTF-8">

  <!-- Responsive viewport -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Page description for SEO -->
  <meta name="description" content="A technical paper on CSS fundamentals.">

  <!-- Author -->
  <meta name="author" content="Your Name">

  <!-- Prevent search engine indexing -->
  <meta name="robots" content="noindex, nofollow">

  <!-- Open Graph (for social sharing previews) -->
  <meta property="og:title" content="CSS Fundamentals">
  <meta property="og:description" content="A deep dive into CSS concepts.">
  <meta property="og:image" content="https://example.com/preview.png">
  <meta property="og:url" content="https://example.com/css-paper">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="CSS Fundamentals">

  <!-- Favicon -->
  <link rel="icon" href="/favicon.ico" type="image/x-icon">

  <!-- Page title -->
  <title>CSS Fundamentals</title>

</head>
```

---

## References

- MDN Web Docs — *The Box Model*. https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model
- MDN Web Docs — *CSS Positioning*. https://developer.mozilla.org/en-US/docs/Web/CSS/position
- MDN Web Docs — *Specificity*. https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity
- MDN Web Docs — *Flexbox*. https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox
- MDN Web Docs — *CSS Grid Layout*. https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout
- MDN Web Docs — *Using Media Queries*. https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries
- MDN Web Docs — *CSS Custom Properties*. https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties
- W3Schools — *HTML Meta Tags*. https://www.w3schools.com/tags/tag_meta.asp
- CSS-Tricks — *A Complete Guide to Flexbox*. https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- CSS-Tricks — *A Complete Guide to Grid*. https://css-tricks.com/snippets/css/complete-guide-grid/
