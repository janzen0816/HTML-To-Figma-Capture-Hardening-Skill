---
name: html-to-figma-capture-hardening
description: Use before importing HTML/CSS into Figma when visual fidelity matters and the page may contain icons from SVG/img/sprites/CDNs/icon fonts, CSS-generated decorative graphics, or CSS-generated decorative backgrounds such as grids, dots, stripes, gradients, masks, and pseudo-elements. Requires classifying visual sources, creating a hardened temporary HTML copy, verifying it locally, then capturing that copy into Figma.
---

# HTML To Figma Capture Hardening

Use this skill before Figma HTML-to-design capture. It helps avoid missing icons, distorted CSS artwork, and lost or stretched background patterns by converting fragile browser-rendered visuals into explicit capture-safe nodes.

## Non-Negotiable

Do not capture the original HTML directly.

Create a temporary `*-figma-hardened.html` copy, harden it, verify it, and capture only that copy. Never edit the original unless the user explicitly asks.

The original source file must remain byte-for-byte intact. Do not inject the Figma capture script into the original. Do not inline icons in the original. Do not convert CSS gradients, pseudo-elements, masks, or backgrounds in the original. All capture-only changes belong in the hardened copy.

If you accidentally touch the original, stop immediately. Preserve the changed version as the hardened copy, then restore the original from the pre-change contents before continuing. Tell the user exactly what happened and which file is now the hardened copy.

## Mental Model

Figma capture is more reliable when important visuals exist as explicit DOM/SVG/image nodes. Browser-only effects may look correct on screen but fail during capture.

Treat these as high-risk until inspected:

- icon fonts and CDN icon libraries,
- SVG sprites and `<use>` references,
- CSS pseudo-elements,
- CSS gradient artwork,
- CSS border/box-shadow/mask/clip-path drawings,
- CSS background patterns such as grids, dots, stripes, and repeated gradients,
- runtime-generated visuals that must exist before capture starts.

## Required Workflow

### 1. Create A Hardened Copy

Copy the source HTML to a temporary file such as:

```text
original-name-figma-hardened.html
```

Before making any capture-only edits, confirm that subsequent edits target the hardened copy path, not the source path. Treat the source path as read-only unless the user explicitly asks to modify it.

Add the Figma capture script only to the hardened copy, after other hardening changes:

```html
<script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>
```

### 2. Inventory Visual Sources

Search the HTML and embedded CSS for visual sources:

```bash
grep -n '<svg\|<use\|<img\|<picture\|<i\|icon\|font-family\|::before\|::after\|content:\|background\|linear-gradient\|radial-gradient\|repeating-\|box-shadow\|border\|mask\|clip-path\|filter' source.html
grep -n '<link\|@import\|cdn\|fontawesome\|material-icons\|remixicon\|bootstrap-icons\|iconfont\|phosphor\|lucide\|heroicons\|ionicons' source.html
```

Classify findings into three groups:

1. Icon sources
2. CSS-generated decorative graphics
3. CSS-generated decorative backgrounds

Do not start replacing assets until this classification is clear.

## Group 1: Icon Sources

Do not assume one icon library. Phosphor, Font Awesome, Material Icons, Remix Icon, Bootstrap Icons, Iconfont, Lucide, Heroicons, Ionicons, local SVGs, and custom systems all need source-specific handling.

### Classify Icons

- **Inline SVG**: complete `<svg><path ...></svg>` in the HTML. Usually safe; verify `viewBox` and color behavior.
- **Image icon**: `<img src="...">`, `<picture>`, or CSS `url(...)`. Ensure the asset loads through the local server; inline SVG if remote or unreliable.
- **SVG sprite/use**: `<svg><use href="#id"></use></svg>` or `sprite.svg#id`. Expand to full inline SVG when important; Figma may not resolve `<use>`.
- **Icon font/text glyph**: `<i class="...">`, `<span class="material-icons">home</span>`, classes backed by font files. Replace with official SVG from the actual library.
- **Pseudo-element icon**: CSS like `.icon::before { content: "\e900"; font-family: ... }`. Convert to a real SVG/DOM node and disable the pseudo-element.
- **Pure CSS icon**: icon drawn with borders, gradients, box-shadows, masks, or pseudo-elements. Convert to inline SVG or explicit DOM shapes if fidelity matters.

### Harden Icons

- Keep complete inline SVG unless it depends on external references.
- Inline external SVG files when feasible.
- Keep PNG/JPG only when local browser verification and Figma capture are expected to preserve it.
- Resolve `<use>` sprites into real paths/shapes.
- For icon fonts, identify the actual library and use its official SVGs. Do not hand-draw approximate replacements unless the user accepts approximation.

Generic icon CSS:

```css
.figma-inline-icon {
  width: 1em;
  height: 1em;
  display: inline-block;
  color: currentColor;
  fill: currentColor;
  flex: 0 0 auto;
}
```

The visible icon should not depend solely on external CSS, web fonts, or `content:` glyphs.

## Group 2: CSS-Generated Decorative Graphics

This group covers visible standalone or local decorative artwork created by CSS rather than explicit nodes.

Examples:

- pseudo-element shapes: `::before`, `::after`,
- pixel blocks made from multiple gradients,
- icons made with borders or box-shadows,
- masks and clipped shapes,
- decorative corner marks, badges, bullets, separators, ornamental squares.

### Harden Decorative Graphics

If the CSS graphic is important to visual fidelity, replace it with explicit SVG/DOM in the hardened copy.

Example: a five-square marker originally made from `::before` and gradients:

```html
<div class="section-label">
  <svg class="figma-css-graphic" aria-hidden="true" viewBox="0 0 48 48" fill="currentColor" focusable="false">
    <rect x="0" y="0" width="16" height="16"/>
    <rect x="16" y="0" width="16" height="16"/>
    <rect x="16" y="16" width="16" height="16"/>
    <rect x="32" y="16" width="16" height="16"/>
    <rect x="32" y="32" width="16" height="16"/>
  </svg>
  <span>Section title</span>
</div>
```

Disable the old CSS-generated graphic so it does not duplicate:

```css
.section-label::before {
  content: none;
}

.figma-css-graphic {
  width: 48px;
  height: 48px;
  display: block;
  color: currentColor;
  flex: 0 0 auto;
}
```

The class names above are examples. Use names that fit the page, but keep the principle: visible CSS-only artwork should become a visible node.

## Group 3: CSS-Generated Decorative Backgrounds

This group covers area-wide background patterns created by CSS.

Examples:

- grid backgrounds,
- dot patterns,
- stripe patterns,
- repeated linear/radial gradients,
- decorative noise/texture gradients,
- layered background patterns.

These are not the same as local icon graphics. They are background surfaces behind content.

### Harden Decorative Backgrounds

If Figma capture misses, simplifies, or stretches the background pattern, replace it with an explicit layer, usually SVG or a raster image.

For geometric patterns such as grids, SVG lines are often best. Avoid a fixed viewBox that gets stretched into a different rendered aspect ratio; square cells can become rectangles.

Use the element's rendered width and height and the same step for x and y:

```html
<svg class="figma-real-bg-pattern"
  data-pattern="grid"
  data-step="24"
  data-stroke="rgba(31,43,224,0.10)"
  aria-hidden="true"
  fill="none"
  focusable="false"></svg>
```

```css
.figma-real-bg-pattern {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.figma-has-bg-pattern {
  position: relative;
  overflow: hidden;
}

.figma-has-bg-pattern > :not(.figma-real-bg-pattern) {
  position: relative;
  z-index: 1;
}
```

```html
<script>
(function () {
  function materializePattern(svg) {
    var rect = svg.getBoundingClientRect();
    var w = Math.ceil(rect.width);
    var h = Math.ceil(rect.height);
    var step = Number(svg.dataset.step || 24);
    var stroke = svg.dataset.stroke || 'rgba(31,43,224,0.10)';
    svg.setAttribute('viewBox', '0 0 ' + w + ' ' + h);
    svg.setAttribute('preserveAspectRatio', 'none');
    svg.setAttribute('stroke', stroke);
    svg.setAttribute('stroke-width', '1');
    svg.setAttribute('vector-effect', 'non-scaling-stroke');
    var lines = '';
    for (var x = 0; x <= w; x += step) lines += '<line x1="' + x + '" y1="0" x2="' + x + '" y2="' + h + '"/>';
    for (var y = 0; y <= h; y += step) lines += '<line x1="0" y1="' + y + '" x2="' + w + '" y2="' + y + '"/>';
    svg.innerHTML = lines;
  }
  function run() {
    document.querySelectorAll('.figma-real-bg-pattern').forEach(materializePattern);
    window.__FIGMA_HARDENED_READY__ = true;
  }
  if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', run);
  else run();
  window.addEventListener('load', run);
})();
</script>
```

Use a capture delay such as `figmadelay=1800` or longer so runtime-generated background layers exist before Figma captures.

For non-geometric textures, consider rasterizing the background as an image layer if SVG would be too complex.

## Verification Gate

Before capture, report what remains in the hardened copy:

```bash
grep -n '<use\|::before\|::after\|content:\|linear-gradient\|radial-gradient\|repeating-\|background-size\|font-family\|mask\|clip-path' hardened.html
grep -n '<svg\|<img\|<picture' hardened.html
```

Remaining matches are not automatically wrong. Explain which are safe and which are intentionally left because they are non-critical or already capture-safe.

Stop before Figma capture if:

- important icons still depend only on icon fonts, CDN CSS, `content:` glyphs, or unresolved `<use>`,
- important CSS-generated graphics still have no real node equivalent,
- important background patterns still rely only on CSS gradients/patterns,
- local browser inspection shows grid cells or geometric patterns are distorted,
- the hardened copy was not actually opened with the Figma capture URL.

## Local Visual Verification

Serve the hardened file over HTTP, not `file://`. Compare it to the source in a browser.

Check:

- icons match their original library/source,
- CSS-generated decorative graphics match,
- CSS-generated decorative backgrounds match,
- geometric patterns keep their intended proportions,
- no important text or layout shifted.

## Capture

Capture the hardened local URL into Figma, poll until complete, and stop temporary local servers afterward.

## Diagnosis If Problems Persist

If Figma still shows missing icons, wrong decorative graphics, or bad background patterns, check:

- Was the original file captured instead of the hardened copy?
- Was a visual source misclassified?
- Did the scan focus on one icon library and miss another?
- Were sprite `<use>` references left unresolved?
- Were approximate icons used instead of source/official SVGs?
- Did runtime-generated layers exist before Figma capture started?
- Did a fixed SVG viewBox stretch a geometric pattern into the wrong proportions?
- Was visual verification skipped?

## Privacy And Cleanup

- Explain that Figma capture sends rendered page data to Figma's service.
- Tell the user where the hardened temporary file is.
- Recommend deleting temporary capture files when no longer needed.
