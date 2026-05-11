# HTML To Figma Capture Hardening Skill

English | [简体中文](README.zh-CN.md)

A Codex Skill for improving the reliability and visual fidelity of HTML/CSS imports into Figma.

When you import a web page, poster, long-form layout, portfolio page, or static HTML file into Figma, you may run into missing icons, distorted CSS grid backgrounds, dropped pseudo-element decorations, unresolved SVG sprites, or icon fonts that turn into empty boxes. This skill guides Codex to create a hardened temporary HTML copy first, then capture that copy with Figma HTML-to-design.

The core idea is simple: do not capture fragile browser-only effects directly. Convert important visuals into explicit DOM, SVG, or image nodes that Figma can understand more reliably.

## Problems This Skill Helps Solve

- Icons from Phosphor, Font Awesome, Material Icons, Lucide, and similar libraries disappear after HTML import.
- `<svg><use></use></svg>` or external SVG sprites do not resolve in Figma.
- Icons provided by CDN CSS, icon fonts, or pseudo-element `content` become blank.
- CSS `::before` / `::after` decorations, corner marks, arrows, badges, and labels are missing.
- CSS `linear-gradient`, `radial-gradient`, or `repeating-linear-gradient` grid, dot, or stripe backgrounds are stretched or dropped.
- Posters, reports, exhibition boards, and long pages lose geometric patterns or background texture during capture.
- You need to protect the original HTML and avoid polluting source files with Figma-only edits.

Search keywords: HTML to Figma, Figma HTML import, Figma capture, missing icons, icon font, SVG sprite, CSS pseudo-elements, CSS gradient background, design import hardening.

## How It Works

Instead of capturing the original page directly, this skill follows a safer workflow:

1. Create a temporary copy, such as `example-figma-hardened.html`.
2. Scan the HTML and CSS for high-risk visual sources.
3. Convert important icons, CSS decorations, and background patterns into more stable explicit nodes.
4. Inject the Figma capture script only into the hardened copy.
5. Serve the hardened copy over local HTTP and visually verify it.
6. Capture the hardened copy into Figma and poll until the generated file is complete.

The original HTML should remain unchanged by default. Any capture-only changes belong in a temporary file such as `*-figma-hardened.html`.

## Usage

Install this skill in your Codex skills directory, then ask Codex something like:

```text
Use the html-to-figma-capture-hardening skill to import this HTML into a new Figma file.
```

For more explicit instructions:

```text
Follow this skill's rules: create a hardened HTML copy, fix icons and CSS backgrounds, then capture it into Figma.
```

If there are multiple HTML files in the folder, name the entry file:

```text
Use html-to-figma-capture-hardening to harden index.html and import it into Figma.
```

## Recommended Directory Structure

For publishing this as a Codex Skill, use the standard structure:

```text
html-to-figma-capture-hardening/
  SKILL.md
  README.md
  README.zh-CN.md
```

The skill instructions should live in `SKILL.md`. The README files are for humans: they explain the use case, workflow, and when someone should reach for this skill.

## Hardening Strategy

### 1. Icon Sources

The skill asks Codex to classify icon sources first instead of assuming a single icon system.

Common high-risk sources include:

- icon fonts, such as `<i class="ph ph-plant"></i>`;
- CDN icon CSS;
- SVG sprites and `<use>`;
- pseudo-element icons, such as `.icon::before { content: ... }`;
- pure CSS icons.

The usual hardening path is to use official SVGs, inline SVGs, or expand external SVG references into real nodes.

### 2. CSS-Generated Decorative Graphics

Many pages draw decorative marks with `::before`, `::after`, gradients, borders, box shadows, masks, or clip paths. These can look correct in the browser but fail during Figma capture.

This skill converts important decorative graphics into explicit SVG or DOM nodes, then disables the old pseudo-element version to avoid duplicates.

### 3. CSS-Generated Backgrounds

Grid, dot, stripe, and repeated-gradient backgrounds are especially fragile in Figma capture. This skill recommends converting them into SVG background layers, especially for geometric grids.

That helps avoid fixed `viewBox` stretching where square grid cells become rectangular after import.

## Pre-Capture Checks

Before importing into Figma, the skill asks Codex to inspect what risky patterns remain in the hardened copy:

```bash
grep -n '<use\|::before\|::after\|content:\|linear-gradient\|radial-gradient\|repeating-\|background-size\|font-family\|mask\|clip-path' hardened.html
grep -n '<svg\|<img\|<picture' hardened.html
```

Remaining matches are not automatically wrong. The important part is to explain why they are safe or why they do not affect the main visual fidelity.

If important icons still depend on icon fonts, important backgrounds still rely only on CSS gradients, or local preview shows distorted geometry, fix those issues before capture.

## Notes

- Do not capture the original HTML directly.
- Do not inject the Figma capture script into the original file.
- Do not damage the source page just to make a Figma import work.
- Local preview should be served over HTTP, not opened with `file://`.
- Runtime-generated SVGs or background layers may need enough `figmadelay` before capture.
- Figma capture sends rendered page data to Figma's service. Confirm that sensitive pages are allowed to be uploaded before capture.
- After capture, keep the hardened copy for reproducibility or delete it when no longer needed.

## Example Prompt

```text
Please use the html-to-figma-capture-hardening skill:
1. Do not modify the original HTML.
2. Create a *-figma-hardened.html copy.
3. Inspect icons, SVG sprites, pseudo-elements, CSS gradient backgrounds, and grids.
4. Convert key visuals into capture-safe SVG/DOM nodes.
5. Verify the page over local HTTP, then import it into a new Figma file.
6. Tell me the hardened file path and the Figma link.
```

## Why This Skill Exists

Figma HTML-to-design capture is useful for quickly turning web pages into editable design files, but it is not always reliable with browser-specific rendering tricks. This skill moves the repair work before capture: inventory the visual sources, harden the fragile ones, verify locally, then import.

It is especially useful for:

- importing AI-generated HTML into Figma;
- importing exhibition boards, portfolios, reports, and presentation-like pages;
- handing a web visual draft to designers for further editing;
- fixing the classic "looks fine in the browser, missing in Figma" problem;
- building a repeatable HTML-to-Figma workflow that does not pollute source files.

## License

If you open source this skill, add the license that fits your project. Common choices include MIT, Apache-2.0, or CC BY 4.0.
