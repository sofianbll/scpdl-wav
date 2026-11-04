# portfolio-site Skill Extraction Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extract site generation from `portfolio-polish` into a standalone `portfolio-site` skill with semantic colors, Google Fonts CDN, OG image generation (variant B), and title clipping fix.

**Architecture:** New skill at `~/.config/opencode/skills/portfolio/portfolio-site/` owns templates, scripts, color/font guides, and OG generation. `portfolio-polish` Step 7 delegates entirely to `portfolio-site`. Semantic color tokens (`primary`, `secondary`, `accent`) replace visual names (`peach`, `pink`, `purple`). Google Fonts via CDN `<link>` with system fallback CSS vars. OG image at 1200x630 via Playwright (`fill-og-image.py`).

**Tech Stack:** Python 3 (build-site-config.py, fill-site.py, fill-og-image.py), HTML + Tailwind v4 CDN + daisyUI v5 CDN (site-template-modern.html, site-template.html, og-image-template.html), Playwright via npx (OG screenshots), Bash (git, gh)

---

### Task 1: Create portfolio-site skill skeleton

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/SKILL.md`
- Create: directories `assets/`, `scripts/`, `references/`, `evals/`, `evals/fixtures/`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p ~/.config/opencode/skills/portfolio/portfolio-site/{assets,scripts,references,evals/fixtures}
```

- [ ] **Step 2: Write SKILL.md frontmatter and full body**

File: `~/.config/opencode/skills/portfolio/portfolio-site/SKILL.md`

```markdown
---
name: portfolio-site
description: Use when a local project repo needs a portfolio landing page or refreshed GitHub Pages /docs content — generating docs/index.html from project scan, configuring colors/fonts/hero stats/sections, producing an OG image for social previews, or validating generated site HTML. Also use when portfolio-polish reaches its site step.
---

# Portfolio Site

## Overview
Generates a GitHub Pages-compatible landing page (`docs/index.html`) and OpenGraph image (`docs/assets/og-image.png`) for portfolio projects. Owns the full site pipeline: project scan → config → customization → render → OG generation → verification.

## When to Use

```
User asks for a landing page
  ┌─ "portfolio site" / "project website"?
  │   └─ YES → Use this skill.
  └─ portfolio-polish reaches Step 7?
      └─ YES → This skill is the standard site generator.
```

Do NOT use for generic non-portfolio sites — `site-github` handles those.

## Pipeline

### Step 1 — Scan project
```bash
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/build-site-config.py <project-path> > /tmp/site-config.json
```

### Step 2 — Customize (GATE — mandatory)
Present the user with template choice, colors, hero layout, stats, sections, and fonts. See [references/config-schema.md](references/config-schema.md) for the full JSON schema.

**Template choice:** `modern` (recommended) or `simple`.

**Colors:** Must be semantic tokens (primary, secondary, accent, bg, surface, elevated, border, text, muted + light variants + radial). Never use visual names (peach, pink, purple). The script auto-detects defaults; the orchestrator overrides. See [references/site-color-guide.md](references/site-color-guide.md).

**Hero stats:** Strict rules.
- Screenshot hero → exactly 0 or 2 stats.
- Centered hero → exactly 0 or 3 stats.
Invalid counts → stop, ask user to adjust.

**Fonts:** Choose a preset or customize. See [references/font-guide.md](references/font-guide.md). Fonts load via Google Fonts CDN with system fallback CSS vars.

### Step 3 — Render site
```bash
mkdir -p <project-path>/docs
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-site.py \
  ~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template-modern.html \
  /tmp/site-config.json \
  <project-path>/docs/index.html
touch <project-path>/docs/.nojekyll
```

### Step 4 — Generate OG image
```bash
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-og-image.py \
  ~/.config/opencode/skills/portfolio/portfolio-site/assets/og-image-template.html \
  /tmp/site-config.json \
  /tmp/og-image.html
npx playwright screenshot /tmp/og-image.html <project-path>/docs/assets/og-image.png --width 1200 --height 630
```

### Step 5 — Verify
Checklist in [references/verification-checklist.md](references/verification-checklist.md).

## Templates

| Template | File | Use case |
|----------|------|----------|
| Modern (default) | `assets/site-template-modern.html` | Portfolio/public showcase |
| Simple | `assets/site-template.html` | Minimal/technical repos |

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/build-site-config.py` | Scan project, output JSON config |
| `scripts/fill-site.py` | Render HTML from template + config |
| `scripts/fill-og-image.py` | Render OG HTML from template + config |

## References

| File | Content |
|------|---------|
| `references/config-schema.md` | Full JSON config schema |
| `references/site-color-guide.md` | Color inference rules |
| `references/font-guide.md` | Font presets and Google Fonts CDN setup |
| `references/verification-checklist.md` | Post-generation checks |
```

- [ ] **Step 3: Commit skeleton**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/ && git -C ~/.config/opencode commit -m "feat(skills): scaffold portfolio-site skill"
```

---

### Task 2: Move and refactor templates — semantic colors + Google Fonts + title fix

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template-modern.html`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template.html`

- [ ] **Step 1: Copy modern template from portfolio-polish**

```bash
cp ~/.config/opencode/skills/portfolio/portfolio-polish/assets/site-template-modern.html \
   ~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template-modern.html
```

- [ ] **Step 2: Refactor colors to semantic tokens in modern template**

In `site-template-modern.html`, replace all visual token references with semantic ones.

Replacements in Tailwind config `colors` block:

```js
// OLD
colors: {
  bg: 'var(--bg)',
  surface: 'var(--surface)',
  elevated: 'var(--elevated)',
  border: 'var(--border)',
  text: 'var(--text)',
  muted: 'var(--muted)',
  peach: 'var(--peach)',
  pink: 'var(--pink)',
  purple: 'var(--purple)'
}

// NEW
colors: {
  bg: 'var(--bg)',
  surface: 'var(--surface)',
  elevated: 'var(--elevated)',
  border: 'var(--border)',
  text: 'var(--text)',
  muted: 'var(--muted)',
  primary: 'var(--primary)',
  secondary: 'var(--secondary)',
  accent: 'var(--accent)'
}
```

Replacements in CSS custom properties block:

```css
/* OLD */
--peach: #ffb347;
--pink: #ff6b6b;
--purple: #6c5ce7;
--peach-dim: rgba(255,179,71,.15);
--peach-dimmer: rgba(255,179,71,.08);
--peach-glow: 0 0 30px rgba(255,179,71,.2);

/* NEW */
--primary: #f77f00;
--secondary: #e36414;
--accent: #6366f1;
--primary-dim: rgba(247,127,0,.15);
--primary-dimmer: rgba(247,127,0,.08);
--primary-glow: 0 0 30px rgba(247,127,0,.2);
```

Replace all class occurrences:
- `text-peach` → `text-primary`
- `text-pink` → `text-secondary`
- `text-purple` → `text-accent`
- `bg-peach` → `bg-primary`
- `bg-pink` → `bg-secondary`
- `bg-peach/10` → `bg-primary/10`
- `bg-pink/10` → `bg-secondary/10`
- `bg-purple/10` → `bg-accent/10`
- `border-peach/30` → `border-primary/30`
- `from-peach` → `from-primary`
- `to-pink` → `to-secondary`
- `to-purple` → `to-accent`
- `--peach` → `--primary`
- `--pink` → `--secondary`
- `--purple` → `--accent`
- `--peach-dim` → `--primary-dim`
- `--peach-dimmer` → `--primary-dimmer`
- `--peach-glow` → `--primary-glow`
- `var(--peach-dimmer)` → `var(--primary-dimmer)`

- [ ] **Step 3: Add Google Fonts CDN links in modern template**

Insert after `<meta charset="UTF-8">`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
{{FONT_LINKS}}
```

Replace hardcoded body font CSS with CSS vars:

```css
/* OLD — body */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  ...
}

/* NEW */
body {
  font-family: var(--font-body);
  ...
}
```

```css
/* OLD — headline */
.headline {
  font-family: -apple-system, "SF Compact Display", "Helvetica Neue", sans-serif;
  font-stretch: condensed;
  letter-spacing: -0.03em;
}

/* NEW */
.headline {
  font-family: var(--font-display);
  letter-spacing: -0.03em;
}
```

```css
/* OLD — code */
code, pre {
  font-family: "JetBrains Mono", "SF Mono", Consolas, monospace;
}

/* NEW */
code, pre {
  font-family: var(--font-mono);
}
```

Insert font vars in the CSS custom properties block:

```css
--font-display: system-ui, -apple-system, sans-serif;
--font-body: system-ui, -apple-system, sans-serif;
--font-mono: "JetBrains Mono", "SF Mono", Consolas, monospace;
```

- [ ] **Step 4: Fix title clipping in modern template**

In the `.text-gradient` rule, add safe rendering properties (around the current `leading-[1.15]` area):

```css
.text-gradient {
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  display: inline-block;
  overflow: visible;
  padding-bottom: 0.08em;
  margin-bottom: -0.08em;
  padding-right: 0.03em;
  margin-right: -0.03em;
}
```

On headline elements using `.text-gradient`, ensure they have `leading-[1.15]`:

```html
<!-- OLD -->
<h1 class="... leading-[1] ...">...</h1>

<!-- NEW -->
<h1 class="... leading-[1.15] ...">...</h1>
```

Add `overflow-visible` to the headline container.

- [ ] **Step 5: Add OG image meta placeholders in modern template**

Replace the OG/Twitter image meta tags:

```html
<!-- OLD -->
<meta property="og:image" content="{{OG_IMAGE}}">
<meta name="twitter:image" content="{{TWITTER_IMAGE}}">

<!-- NEW -->
{{OG_IMAGE}}
{{TWITTER_IMAGE}}
```

Where these now resolve to full `<meta>` tags including `og:image:width` and `og:image:height`:

```html
<meta property="og:image" content="assets/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta name="twitter:image" content="assets/og-image.png">
```

- [ ] **Step 6: Apply same refactors to simple template**

Copy and apply the same semantic color + font + title clip refactors to `site-template.html`.

```bash
cp ~/.config/opencode/skills/portfolio/portfolio-polish/assets/site-template.html \
   ~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template.html
```

Apply the same edits as Steps 2-5.

- [ ] **Step 7: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/assets/ && git -C ~/.config/opencode commit -m "feat(portfolio-site): semantic colors, Google Fonts, title clipping fix in templates"
```

---

### Task 3: Move and refactor scripts — semantic colors + fonts

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/scripts/build-site-config.py`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-site.py`

- [ ] **Step 1: Copy build-site-config.py**

```bash
cp ~/.config/opencode/skills/portfolio/portfolio-polish/scripts/build-site-config.py \
   ~/.config/opencode/skills/portfolio/portfolio-site/scripts/build-site-config.py
```

- [ ] **Step 2: Refactor detect_colors() to semantic tokens**

In `build-site-config.py`, replace `detect_colors()` return value (lines 147-171):

```python
def detect_colors(project_path):
    """Return a dict of semantic color tokens. These are dark-mode placeholders.
    The orchestrator MUST override these with project-specific
    colors inferred from language/category/logo before calling fill-site.py.
    See references/site-color-guide.md for inference rules."""
    return {
        "bg": "#0a0908",
        "surface": "#131110",
        "elevated": "#1a1715",
        "border": "#2a2725",
        "text": "#f5f0eb",
        "muted": "rgba(245,240,235,0.58)",
        "primary": "#6366f1",
        "secondary": "#f59e0b",
        "accent": "#10b981",
        "light_bg": "#faf8f5",
        "light_surface": "#ffffff",
        "light_elevated": "#f0ede8",
        "light_border": "#e0dbd3",
        "light_text": "#1a1815",
        "light_muted": "rgba(26,24,21,0.55)",
        "radial_1": "rgba(99,102,241,0.08)",
        "radial_2": "rgba(245,158,11,0.05)",
    }
```

Replace `story["items"][i]` default colors from `"peach"` to `"primary"`:

```python
# Line ~303
"badge_color": "primary",
# Line ~307
"file_color": "primary",
```

- [ ] **Step 3: Add font detection to build-site-config.py**

Add `detect_fonts()` function after `detect_colors()`:

```python
def detect_fonts(project_path):
    """Return a dict of font tokens with Google Fonts CDN links and CSS vars.
    Default: system fonts only (no external CDN). Orchestrator overrides per project."""
    return {
        "display": "system-ui, -apple-system, sans-serif",
        "body": "system-ui, -apple-system, sans-serif",
        "mono": '"JetBrains Mono", "SF Mono", Consolas, monospace',
        "links": "",  # Google Fonts CDN <link> tags, empty = system fonts only
    }
```

Add `fonts` key to the main config dict (insert after `colors`):

```python
config = {
    "project_name": ...,
    "meta_description": ...,
    "colors": colors,
    "fonts": detect_fonts(project_path),
    ...
}
```

- [ ] **Step 4: Update OG image detection for new path**

In the meta tags section (lines 260-267), change to point at `assets/og-image.png`:

```python
# -- Meta tags --
og_image = ""
twitter_image = ""
favicon = ""
logo_path = proj / "docs" / "assets" / "logo.png"
og_path = proj / "docs" / "assets" / "og-image.png"
if logo_path.exists():
    favicon = '<link rel="icon" type="image/png" href="assets/logo.png">'
if og_path.exists():
    og_image = '<meta property="og:image" content="assets/og-image.png">\n<meta property="og:image:width" content="1200">\n<meta property="og:image:height" content="630">'
    twitter_image = '<meta name="twitter:image" content="assets/og-image.png">'
elif logo_path.exists():
    # Fallback to logo until OG is generated
    og_image = '<meta property="og:image" content="assets/logo.png">'
    twitter_image = '<meta name="twitter:image" content="assets/logo.png">'
```

- [ ] **Step 5: Copy fill-site.py**

```bash
cp ~/.config/opencode/skills/portfolio/portfolio-polish/scripts/fill-site.py \
   ~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-site.py
```

- [ ] **Step 6: Refactor fill-site.py placeholder map**

In `build_placeholder_map()`, replace the color key loop (lines 147-152):

```python
# OLD
for k in ["bg", "surface", "elevated", "border", "text", "muted",
          "peach", "pink", "purple",
          "light_bg", "light_surface", "light_elevated",
          "light_border", "light_text", "light_muted",
          "radial_1", "radial_2"]:

# NEW
for k in ["bg", "surface", "elevated", "border", "text", "muted",
          "primary", "secondary", "accent",
          "light_bg", "light_surface", "light_elevated",
          "light_border", "light_text", "light_muted",
          "radial_1", "radial_2"]:
```

Add font placeholders:

```python
# Fonts
fonts = config.get("fonts", {})
m["FONT_LINKS"] = fonts.get("links", "")
for k in ["display", "body", "mono"]:
    m[f"FONT_{k.upper()}"] = fonts.get(k, "")
```

In `build_placeholder_map()`, update nav letter fallback (line 170) — replace `from-peach to-pink` with `from-primary to-secondary`:

```python
m["LOGO_IMG"] = f'<span class="w-10 h-10 rounded-lg bg-gradient-to-br from-primary to-secondary flex items-center justify-center text-[{bg}] font-black text-base">{nav_letter}</span>'
```

Update version pill (line 173) — `text-peach border-peach/30 bg-peach` → `text-primary border-primary/30 bg-primary`:

```python
m["VERSION_PILL"] = f'<span class="flex items-center gap-1.5 text-xs font-mono text-primary border border-primary/30 px-3 py-1.5 rounded-full"><span class="w-1.5 h-1.5 rounded-full bg-primary pulse-dot"></span>{version}</span>' if version else ""
```

- [ ] **Step 7: Refactor renderers — default colors**

Search-and-replace across all renderer templates in `fill-site.py`:
- `color=color` defaults from `"peach"` to `"primary"` in `STAT_CARD_TMPL`, `FEATURE_CARD_TMPL`, `STORY_CARD_H`, `STORY_CARD_V`
- `{color}` references in icon/glass templates remain dynamic (they use the color from the config item)

```python
# In render_stats() — default color
color = s.get("color", "primary")

# In render_feature_cards() — default color
color = item.get("color", "primary")

# In render_story_cards() — default badge color
color = item.get("badge_color", "primary")
file_color = item.get("file_color", "primary")

# In render_quick_panels() — default command color
color = cmd.get("color", "primary")
```

Update `QUICK_CLI_PANEL` copy button class from `text-peach` to `text-primary`:

```python
QUICK_CLI_PANEL = """...
          <button class="ml-auto text-muted hover:text-primary transition-colors ...">
```

Update FOOTER_PILL_TMPL to use semantic:

```python
FOOTER_PILL_TMPL = '<span class="text-[10px] font-mono border border-border px-3 py-1 rounded-full">{tech}</span>'
```

- [ ] **Step 8: Fix badge path in fill-site.py**

Update `BADGES_SKILL_DIR` to resolve from the new skill location:

```python
# OLD
BADGES_SKILL_DIR = Path(__file__).resolve().parent.parent.parent / "readme-wizard" / "assets"

# NEW — resolve from portfolio-site's own directory to siblings
BADGES_SKILL_DIR = Path(__file__).resolve().parent.parent.parent / "readme-wizard" / "assets"
```

(The relative structure `portfolio-site/scripts/ -> ../../ -> readme-wizard/` stays the same since both are under `~/.config/opencode/skills/portfolio/`.)

- [ ] **Step 9: Add CSS font var injection in fill-site.py**

In the `fill()` function, after placeholder replacement but before removing leftover placeholders, inject font CSS vars into the `:root` block or `body` style if `{{FONT_DISPLAY}}`, `{{FONT_BODY}}`, `{{FONT_MONO}}` appear. These are already handled by the placeholder map in Step 6.

- [ ] **Step 10: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/scripts/ && git -C ~/.config/opencode commit -m "feat(portfolio-site): refactor scripts for semantic colors, fonts, OG image"
```

---

### Task 4: Create OG image pipeline (variant B)

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/assets/og-image-template.html`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-og-image.py`

- [ ] **Step 1: Write og-image-template.html**

Based on the validated variant B from `downloads/og-tests/b-full-bleed.html`.

File: `~/.config/opencode/skills/portfolio/portfolio-site/assets/og-image-template.html`

```html
<!doctype html>
<html>
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 1200px; height: 630px; overflow: hidden;
    background: {{COLOR_BG}};
    font-family: {{FONT_DISPLAY}};
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 24px; position: relative;
  }
  .bg-grid {
    position: absolute; inset: 0;
    background-image:
      linear-gradient(rgba(255,255,255,.02) 1px, transparent 1px),
      linear-gradient(90deg, rgba(255,255,255,.02) 1px, transparent 1px);
    background-size: 40px 40px;
    mask-image: radial-gradient(ellipse 60% 70% at 50% 45%, black 30%, transparent 70%);
  }
  .glow {
    position: absolute; width: 600px; height: 400px;
    top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: radial-gradient(ellipse, {{COLOR_RADIAL_1}} 0%, {{COLOR_RADIAL_2}} 40%, transparent 70%);
  }
  .logo { height: 88px; width: auto; z-index: 1; position: relative; }
  .name {
    z-index: 1; position: relative;
    font-size: 72px; font-weight: 900;
    letter-spacing: -0.04em; line-height: 1.15;
    display: inline-block; overflow: visible;
    padding-bottom: 0.08em; margin-bottom: -0.08em;
    padding-right: 0.03em; margin-right: -0.03em;
    background: linear-gradient(135deg, {{COLOR_PRIMARY}}, {{COLOR_SECONDARY}}, {{COLOR_ACCENT}});
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
  .tagline {
    z-index: 1; position: relative;
    font-size: 24px; color: {{COLOR_MUTED}};
    letter-spacing: 0.03em;
    font-family: {{FONT_BODY}};
  }
</style>
</head>
<body>
  <div class="bg-grid"></div>
  <div class="glow"></div>
  {{LOGO_IMG}}
  <div class="name">{{HEADLINE}}</div>
  <div class="tagline">{{TAGLINE}}</div>
</body>
</html>
```

- [ ] **Step 2: Write fill-og-image.py**

File: `~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-og-image.py`

```python
#!/usr/bin/env python3
"""
Fill the OG image template with project data from a JSON config.
Usage: python fill-og-image.py <og-template.html> <config.json> [output.html]

Reads og-image-template.html, replaces all {{PLACEHOLDER}} tokens, writes output.
The output HTML is then captured at 1200x630 via Playwright.
"""
import json
import sys
from pathlib import Path


def main():
    if len(sys.argv) < 3:
        print("Usage: fill-og-image.py <og-template.html> <config.json> [output.html]", file=sys.stderr)
        sys.exit(1)

    template_path = sys.argv[1]
    config_path = sys.argv[2]
    output_path = sys.argv[3] if len(sys.argv) > 3 else None

    template = Path(template_path).read_text(encoding="utf-8")
    config = json.loads(Path(config_path).read_text(encoding="utf-8"))

    colors = config.get("colors", {})
    hero = config.get("hero", {})
    fonts = config.get("fonts", {})
    nav = config.get("nav", {})

    # Build placeholder map
    placeholders = {}

    # Colors
    for k in ["bg", "primary", "secondary", "accent", "muted", "radial_1", "radial_2"]:
        placeholders[f"COLOR_{k.upper()}"] = colors.get(k, "")

    # Hero
    placeholders["HEADLINE"] = hero.get("headline", config.get("project_name", "Project"))
    placeholders["TAGLINE"] = hero.get("tagline", "")

    # Fonts
    placeholders["FONT_DISPLAY"] = fonts.get("display", "system-ui, -apple-system, sans-serif")
    placeholders["FONT_BODY"] = fonts.get("body", "system-ui, -apple-system, sans-serif")

    # Logo — absolute file:// path for Playwright local screenshot
    logo_src = nav.get("logo_src", "")
    project_path = config.get("_project_path", "")
    if logo_src and project_path:
        logo_abs = str(Path(project_path) / "docs" / logo_src)
        placeholders["LOGO_IMG"] = f'<img class="logo" src="file://{logo_abs}" alt="">'
    else:
        placeholders["LOGO_IMG"] = ""

    # Replace
    for key, value in placeholders.items():
        template = template.replace("{{" + key + "}}", str(value) if value is not None else "")

    if output_path:
        Path(output_path).write_text(template, encoding="utf-8")
        print(f"Written to {output_path}")
    else:
        print(template)


if __name__ == "__main__":
    main()
```

- [ ] **Step 3: Update build-site-config.py to embed _project_path**

In `build-site-config.py` main(), add project path to config (needed by fill-og-image for logo resolution):

```python
config = {
    ...
    "_project_path": str(proj),
}
```

- [ ] **Step 4: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/assets/og-image-template.html skills/portfolio/portfolio-site/scripts/fill-og-image.py skills/portfolio/portfolio-site/scripts/build-site-config.py && git -C ~/.config/opencode commit -m "feat(portfolio-site): add OG image pipeline (variant B)"
```

---

### Task 5: Create reference files

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/references/site-color-guide.md`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/references/font-guide.md`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/references/config-schema.md`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/references/verification-checklist.md`

- [ ] **Step 1: Copy and adapt site-color-guide.md**

```bash
cp ~/.config/opencode/skills/portfolio/portfolio-polish/references/site-color-guide.md \
   ~/.config/opencode/skills/portfolio/portfolio-site/references/site-color-guide.md
```

Edit to use semantic token names (`primary`/`secondary`/`accent`) instead of `peach`/`pink`/`purple`. Update palette tables accordingly.

- [ ] **Step 2: Write font-guide.md**

File: `~/.config/opencode/skills/portfolio/portfolio-site/references/font-guide.md`

```markdown
# Font Guide

## Presets

### system (default — no external CDN)
- Display: `system-ui, -apple-system, sans-serif`
- Body: `system-ui, -apple-system, sans-serif`
- Mono: `"JetBrains Mono", "SF Mono", Consolas, monospace`
- Links: _(empty)_

### technical
- Display: `"Space Grotesk", system-ui, sans-serif`
- Body: `"Inter", system-ui, sans-serif`
- Mono: `"JetBrains Mono", "SF Mono", Consolas, monospace`
- Links: `<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Space+Grotesk:wght@500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">`

### bold-music (recommended for audio/music/DJ tools)
- Display: `"Archivo Black", Impact, "Arial Narrow Bold", sans-serif`
- Body: `"DM Sans", system-ui, sans-serif`
- Mono: `"JetBrains Mono", "SF Mono", Consolas, monospace`
- Links: `<link href="https://fonts.googleapis.com/css2?family=Archivo+Black&family=DM+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">`

### creative
- Display: `"Clash Display", "DM Serif Display", Georgia, serif`
- Body: `"Satoshi", "DM Sans", system-ui, sans-serif`
- Mono: `"JetBrains Mono", monospace`
- Links: `<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=DM+Serif+Display&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">`

## GATE — Font Customization

Present the user with presets and let them choose. The `fonts.links` field is the raw `<link>` HTML; `fonts.display`, `fonts.body`, `fonts.mono` are CSS `font-family` values (with fallbacks).

## CSS Variables

The template injects:
```css
--font-display: (fonts.display);
--font-body: (fonts.body);
--font-mono: (fonts.mono);
```

## System Fallback

Every preset includes a system font fallback chain. If Google Fonts CDN is unavailable, the page degrades to system fonts.
```

- [ ] **Step 3: Write config-schema.md**

File: `~/.config/opencode/skills/portfolio/portfolio-site/references/config-schema.md`

```markdown
# Config Schema

Full JSON schema consumed by fill-site.py and fill-og-image.py.

## Top-level keys

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `project_name` | string | yes | Project display name |
| `meta_description` | string | yes | SEO description |
| `colors` | object | yes | Semantic color tokens |
| `fonts` | object | yes | Font tokens |
| `meta` | object | no | OG/Twitter/favicon meta tags |
| `nav` | object | yes | Nav bar config |
| `hero` | object | yes | Hero section config |
| `story` | object | no | Evolution timeline |
| `features` | object | no | Feature cards |
| `quick` | object | yes | Quick start panels |
| `demo` | object | no | Demo section |
| `faq` | object | no | FAQ items |
| `architecture` | object | no | Folder tree |
| `footer` | object | yes | Footer info |
| `_project_path` | string | no | Absolute project path (for OG logo) |
| `template_variant` | string | no | `modern` or `simple` |

## colors

```json
{
  "bg": "#0a0908",
  "surface": "#131110",
  "elevated": "#1a1715",
  "border": "#2a2725",
  "text": "#f5f0eb",
  "muted": "rgba(245,240,235,0.58)",
  "primary": "#f77f00",
  "secondary": "#e36414",
  "accent": "#6366f1",
  "light_bg": "#faf8f5",
  "light_surface": "#ffffff",
  "light_elevated": "#f0ede8",
  "light_border": "#e0dbd3",
  "light_text": "#1a1815",
  "light_muted": "rgba(26,24,21,0.55)",
  "radial_1": "rgba(247,127,0,0.10)",
  "radial_2": "rgba(227,100,20,0.06)"
}
```

## fonts

```json
{
  "display": "\"Archivo Black\", Impact, sans-serif",
  "body": "\"DM Sans\", system-ui, sans-serif",
  "mono": "\"JetBrains Mono\", \"SF Mono\", Consolas, monospace",
  "links": "<link href=\"https://fonts.googleapis.com/css2?family=...\" rel=\"stylesheet\">"
}
```

## hero

```json
{
  "variant": "centered",
  "eyebrow": "Python CLI Tool",
  "headline": "SCPDL-Wav",
  "tagline": "SoundCloud WAV — Playlist Downloader",
  "cta_primary_text": "Get Started",
  "cta_primary_url": "#quick-start",
  "cta_secondary_text": "View on GitHub",
  "cta_secondary_url": "https://github.com/Sofian-bll/scpdl-wav",
  "stats": [
    {"value": "4", "label": "Commands", "desc": "cli surface", "color": "primary", "icon": "bolt"},
    {"value": "2", "label": "Runtime Deps", "desc": "pip install", "color": "secondary", "icon": "database"},
    {"value": "1", "label": "Star", "desc": "on GitHub", "color": "accent", "icon": "star"}
  ],
  "screenshot": null
}
```

Stats MUST be 0 or 3 for centered hero, 0 or 2 for screenshot hero.

## quick

```json
{
  "variant": "cli",
  "title": "Quick Start",
  "intro": "<p class=\"text-muted\">Get up and running in under 60 seconds.</p>",
  "panels": [
    {
      "id": "setup",
      "label": "Setup",
      "commands": [
        {"text": "git clone https://github.com/...", "color": "accent", "comment": null},
        {"text": "cd scpdl-wav", "color": "accent", "comment": null},
        {"text": "pip install -r requirements.txt", "color": "primary", "comment": null}
      ]
    }
  ]
}
```

## features

```json
{
  "title": "Features",
  "intro": null,
  "items": [
    {
      "title": "Full Playlist Downloads",
      "desc": "Download complete SoundCloud playlists.",
      "color": "primary",
      "icon": "globe"
    }
  ]
}
```

## footer

```json
{
  "license": "MIT",
  "year": "2026",
  "author": "Sofian",
  "tech": ["python", "ffmpeg", "mutagen", "scdl"]
}
```
```

- [ ] **Step 4: Write verification-checklist.md**

File: `~/.config/opencode/skills/portfolio/portfolio-site/references/verification-checklist.md`

```markdown
# Verification Checklist

After generating `docs/index.html`:

- [ ] `docs/index.html` exists and is non-empty
- [ ] `docs/.nojekyll` exists
- [ ] Zero `{{PLACEHOLDER}}` tokens remain
- [ ] No `peach`, `pink`, `purple` token names remain in CSS/classes
- [ ] Colors match user's chosen palette
- [ ] Template matches user's choice (`modern` or `simple`)
- [ ] Sections match user's selection (no extra, no missing)
- [ ] Hero variant matches user's choice (exactly one of `hero-screenshot` or `hero-centered`)
- [ ] Hero stats obey layout rules (`screenshot` → 0 or 2, `centered` → 0 or 3)
- [ ] Stats are factual and consistent with page content
- [ ] Headline text is not visually clipped (verify with Playwright at 1920x1080)
- [ ] `{{FONT_LINKS}}` is replaced (empty string or Google Fonts CDN links)
- [ ] Font CSS vars (`--font-display`, `--font-body`, `--font-mono`) are set
- [ ] `og:image` points to `assets/og-image.png` with `og:image:width=1200` and `og:image:height=630`

After generating `docs/assets/og-image.png`:

- [ ] `docs/assets/og-image.png` exists
- [ ] Dimensions are exactly 1200x630
- [ ] Logo is visible and not clipped
- [ ] Title text is not clipped
- [ ] Tagline is legible
```

- [ ] **Step 5: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/references/ && git -C ~/.config/opencode commit -m "docs(portfolio-site): add color guide, font guide, config schema, verification checklist"
```

---

### Task 6: Create evals

**Files:**
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/evals/evals.json`
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/python-cli/` (minimal Python CLI fixture)
- Create: `~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/react-vite/` (minimal React web fixture)

- [ ] **Step 1: Write evals.json**

```json
{
  "skill_name": "portfolio-site",
  "evals": [
    {
      "id": 1,
      "name": "python-cli-centered-hero-with-3-stats",
      "prompt": "Use portfolio-site on evals/fixtures/python-cli. Choose modern template, centered hero with exactly 3 stats (Commands, Runtime Deps, Stars), Python-blue palette, Features and Quick Start sections only. Render docs/index.html and docs/.nojekyll. Do NOT deploy.",
      "expected_output": "Generates docs/index.html with modern template, centered hero, 3 stats, blue palette, no story/demo/faq/architecture sections, zero placeholders.",
      "files": ["docs/index.html", "docs/.nojekyll"],
      "assertions": [
        {"text": "docs/index.html exists", "check": "file_exists"},
        {"text": "docs/.nojekyll exists", "check": "file_exists"},
        {"text": "Zero {{PLACEHOLDER}} tokens", "check": "no_placeholders"},
        {"text": "Contains Python primary color", "check": "contains", "value": "#3776ab"},
        {"text": "Features section present", "check": "section_present", "value": "features"},
        {"text": "Quick Start section present", "check": "section_present", "value": "quick-cli"},
        {"text": "No FAQ section", "check": "section_absent", "value": "faq"},
        {"text": "No deploy commands executed", "check": "no_command", "value": "git push"}
      ]
    },
    {
      "id": 2,
      "name": "screenshot-hero-stat-guard",
      "prompt": "Use portfolio-site on evals/fixtures/react-vite. Choose screenshot hero layout and 3 stats. Do not render until stats are valid.",
      "expected_output": "Agent stops and asks user to reduce to 0 or 2 stats before rendering. Does not silently truncate.",
      "files": [],
      "assertions": [
        {"text": "Does not call fill-site.py before validating stats", "check": "step_order"},
        {"text": "Asks user to adjust stat count", "check": "user_interaction"}
      ]
    },
    {
      "id": 3,
      "name": "portfolio-polish-handoff",
      "prompt": "portfolio-polish has reached Phase 5 Step 7 for evals/fixtures/react-vite. Generate only the site artifact. Do not run audit, license, release, README, deploy, screenshot, or Bentofolio steps.",
      "expected_output": "Uses portfolio-site skill for site generation only. Produces docs/index.html, docs/.nojekyll, and OG image. Does not wrap other pipeline steps.",
      "files": ["docs/index.html", "docs/.nojekyll"],
      "assertions": [
        {"text": "Only site artifacts produced", "check": "files_count", "value": 2},
        {"text": "No audit/license/release/readme steps executed", "check": "no_command", "value": "portfolio-audit"},
        {"text": "No deploy/screenshot steps executed", "check": "no_command", "value": "gh api"}
      ]
    }
  ]
}
```

- [ ] **Step 2: Create python-cli fixture**

```bash
mkdir -p ~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/python-cli
```

Create minimal files:

`evals/fixtures/python-cli/README.md`:
```markdown
# py-tool

A Python CLI tool for data processing. Downloads and converts files.

## Quick Start

git clone https://github.com/test/py-tool
cd py-tool
pip install -r requirements.txt
python main.py
```

`evals/fixtures/python-cli/requirements.txt`:
```
requests>=2.28.0
pandas>=1.5.0
```

`evals/fixtures/python-cli/main.py`:
```python
print("hello")
```

`evals/fixtures/python-cli/LICENSE`:
```
MIT License

Copyright (c) 2026 Test

Permission is hereby granted...
```

```bash
cd ~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/python-cli && git init && git add -A && git commit -m "init" && git tag v1.0.0
```

- [ ] **Step 3: Create react-vite fixture**

```bash
mkdir -p ~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/react-vite/src
```

Create minimal files:

`package.json`:
```json
{"name":"react-app","dependencies":{"react":"^18.2.0","vite":"^5.0.0"}}
```

`README.md`:
```markdown
# react-app

A React web application built with Vite.

## Quick Start

npm install
npm run dev
```

```bash
cd ~/.config/opencode/skills/portfolio/portfolio-site/evals/fixtures/react-vite && git init && git add -A && git commit -m "init" && git tag v1.0.0
```

- [ ] **Step 4: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-site/evals/ && git -C ~/.config/opencode commit -m "test(portfolio-site): add evals and fixtures"
```

---

### Task 7: Update portfolio-polish to delegate Step 7

**Files:**
- Modify: `~/.config/opencode/skills/portfolio/portfolio-polish/SKILL.md` (lines 286-426, 521-522, 551)

- [ ] **Step 1: Replace Phase 5 Step 7 content with delegation**

Replace lines 286-426 with:

```markdown
### Step 7 — Site (Landing Page)

Delegates entirely to the `portfolio-site` skill.

#### CONTEXT_REQUIRED
Before running step 7, invoke the skill:
- `/portfolio-site` or `Skill("portfolio-site")`

The `portfolio-site` skill handles:
- Project scan → JSON config generation
- Template choice (modern / simple)
- Color customization with semantic tokens (primary, secondary, accent)
- Font customization with Google Fonts CDN presets
- Hero layout and hero stat selection (strict: centered → 0 or 3, screenshot → 0 or 2)
- Section selection
- Rendering `docs/index.html` and `docs/.nojekyll`
- OG image generation (`docs/assets/og-image.png`, 1200x630)
- Site verification checklist

#### Workflow

1. Run `portfolio-site` on the project. It will ask for template, colors, fonts, hero layout, stats, and sections.
2. Guide the user through choices as `portfolio-site` presents them.
3. After generation, verify:
   - `docs/index.html` exists
   - `docs/.nojekyll` exists
   - `docs/assets/og-image.png` exists (1200x630)
   - Zero placeholders
   - No `peach`/`pink`/`purple` in HTML
```

- [ ] **Step 2: Update Requirements section**

Replace line 551 (`build-site-config.py + fill-site.py (step 7)`) with:

```markdown
- `portfolio-site` (step 7)
```

- [ ] **Step 3: Update Common Mistakes**

Replace line 534:

```markdown
| **Using `site-github` in this pipeline** | Different tool, different output | Use `portfolio-site` skill |
```

- [ ] **Step 4: Update pipeline summary line 43-46**

Replace Step 7 description:

```markdown
7. site               portfolio-site            → docs/index.html + OG image
```

- [ ] **Step 5: Commit**

```bash
git -C ~/.config/opencode add skills/portfolio/portfolio-polish/SKILL.md && git -C ~/.config/opencode commit -m "refactor(portfolio-polish): delegate site step to portfolio-site skill"
```

---

### Task 8: Regenerate SCPDL-Wav site with new skill

**Files:**
- Modify: `docs/index.html`
- Create: `docs/assets/og-image.png`

- [ ] **Step 1: Run build-site-config.py from new skill**

```bash
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/build-site-config.py /Users/sofian/Developer/10-Personal/Soundcloud-Wav-Playlist > /tmp/scpdl-wav-site-config.json
```

- [ ] **Step 2: Customize config for SCPDL-Wav**

```bash
python3 -c "
import json
cfg = json.load(open('/tmp/scpdl-wav-site-config.json'))

# Override colors — SCPDL-Wav palette
cfg['colors']['primary'] = '#f77f00'
cfg['colors']['secondary'] = '#e36414'
cfg['colors']['accent'] = '#6366f1'
cfg['colors']['radial_1'] = 'rgba(247,127,0,0.10)'
cfg['colors']['radial_2'] = 'rgba(227,100,20,0.06)'

# Override fonts — bold-music preset
cfg['fonts'] = {
    'display': '\"Archivo Black\", Impact, \"Arial Narrow Bold\", sans-serif',
    'body': '\"DM Sans\", system-ui, -apple-system, sans-serif',
    'mono': '\"JetBrains Mono\", \"SF Mono\", Consolas, monospace',
    'links': '<link href=\"https://fonts.googleapis.com/css2?family=Archivo+Black&family=DM+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap\" rel=\"stylesheet\">',
}

# Hero
cfg['hero']['variant'] = 'centered'
cfg['hero']['eyebrow'] = 'Python CLI Tool'
cfg['hero']['headline'] = 'SCPDL-Wav'
cfg['hero']['tagline'] = 'SoundCloud WAV — Playlist Downloader'
cfg['hero']['stats'] = [
    {'value': '4', 'label': 'Commands', 'desc': 'cli surface', 'color': 'primary', 'icon': 'bolt'},
    {'value': '2', 'label': 'Runtime Deps', 'desc': 'pip install', 'color': 'secondary', 'icon': 'database'},
    {'value': '1', 'label': 'Star', 'desc': 'on GitHub', 'color': 'accent', 'icon': 'star'},
]

# Features
cfg['features'] = {
    'title': 'Features',
    'intro': None,
    'items': [
        {'title': 'Full Playlist Downloads', 'desc': 'Download complete SoundCloud playlists in one command.', 'color': 'primary', 'icon': 'globe'},
        {'title': 'Lossless WAV Output', 'desc': 'Automatic M4A/MP3/OPUS to WAV conversion via FFmpeg.', 'color': 'secondary', 'icon': 'bolt'},
        {'title': 'Metadata Preservation', 'desc': 'Title, artist, genre, track number, date, and cover art preserved.', 'color': 'accent', 'icon': 'tag'},
        {'title': 'Multi-Format Input', 'desc': 'Support for URLs, text file playlists, and dry-run mode.', 'color': 'primary', 'icon': 'code'},
    ],
}

# Quick Start with real SCPDL-Wav commands
cfg['quick']['panels'] = [
    {
        'id': 'setup',
        'label': 'Setup',
        'commands': [
            {'text': 'git clone https://github.com/Sofian-bll/scpdl-wav', 'color': 'accent', 'comment': None},
            {'text': 'cd scpdl-wav', 'color': 'accent', 'comment': None},
            {'text': 'python3 -m venv .venv && source .venv/bin/activate', 'color': 'secondary', 'comment': None},
            {'text': 'pip install -r requirements.txt', 'color': 'primary', 'comment': None},
            {'text': 'brew install ffmpeg', 'color': 'accent', 'comment': '# macOS only'},
        ],
    },
    {
        'id': 'run',
        'label': 'Run',
        'commands': [
            {'text': 'python scpdlwav.py \"https://soundcloud.com/...\"', 'color': 'primary', 'comment': '# Direct playlist URL'},
            {'text': 'python scpdlwav.py playlists.txt', 'color': 'accent', 'comment': '# Batch from text file'},
            {'text': 'python scpdlwav.py --dry-run \"https://...\"', 'color': 'secondary', 'comment': '# Preview only, no download'},
            {'text': 'python scpdlwav.py -v \"https://...\"', 'color': 'primary', 'comment': '# Verbose mode'},
        ],
    },
]

cfg['template_variant'] = 'modern'

json.dump(cfg, open('/tmp/scpdl-wav-site-config.json', 'w'), indent=2)
"
```

- [ ] **Step 3: Render site with fill-site.py**

```bash
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-site.py \
  ~/.config/opencode/skills/portfolio/portfolio-site/assets/site-template-modern.html \
  /tmp/scpdl-wav-site-config.json \
  /Users/sofian/Developer/10-Personal/Soundcloud-Wav-Playlist/docs/index.html
touch /Users/sofian/Developer/10-Personal/Soundcloud-Wav-Playlist/docs/.nojekyll
```

- [ ] **Step 4: Generate OG HTML**

```bash
python3 ~/.config/opencode/skills/portfolio/portfolio-site/scripts/fill-og-image.py \
  ~/.config/opencode/skills/portfolio/portfolio-site/assets/og-image-template.html \
  /tmp/scpdl-wav-site-config.json \
  /tmp/scpdl-wav-og.html
```

- [ ] **Step 5: Capture OG image with Playwright**

```bash
npx playwright screenshot /tmp/scpdl-wav-og.html docs/assets/og-image.png --width 1200 --height 630
```

Check that Playwright is available. If not, install first:

```bash
npx playwright install chromium 2>/dev/null || echo "Playwright may need manual install"
```

- [ ] **Step 6: Verify — run checks**

```bash
python3 -c "
from pathlib import Path
import re

# 1. Index exists
html = Path('docs/index.html').read_text()
assert len(html) > 100, 'index.html too small'

# 2. No placeholders
placeholders = re.findall(r'\{\{[A-Z0-9_]+\}\}', html)
assert not placeholders, f'Placeholders found: {placeholders}'

# 3. No peach/pink/purple
for bad in ['peach', 'pink', 'purple']:
    assert bad not in html.replace('--peach', ''), f'{bad} found in HTML'

# 4. Semantic colors present
assert '--primary' in html, 'Missing --primary'
assert '--secondary' in html, 'Missing --secondary'
assert '--accent' in html, 'Missing --accent'

# 5. OG meta correct
assert 'og:image' in html, 'Missing og:image'
assert 'assets/og-image.png' in html, 'og:image not pointing to OG'
assert 'og:image:width' in html, 'Missing og:image:width'
assert 'og:image:height' in html, 'Missing og:image:height'

# 6. Fonts
assert 'fonts.googleapis.com' in html, 'Missing Google Fonts CDN'
assert '--font-display' in html, 'Missing --font-display'
assert '--font-body' in html, 'Missing --font-body'
assert '--font-mono' in html, 'Missing --font-mono'

# 7. OG image exists and is 1200x630
import struct
og = Path('docs/assets/og-image.png')
assert og.exists(), 'OG image missing'
data = og.read_bytes()
w, h = struct.unpack('>II', data[16:24])
assert (w, h) == (1200, 630), f'OG image {w}x{h} != 1200x630'

# 8. .nojekyll
assert Path('docs/.nojekyll').exists(), '.nojekyll missing'

print('All verification checks passed')
"
```

- [ ] **Step 7: Commit project**

```bash
git add docs/index.html docs/.nojekyll docs/assets/og-image.png && git commit -m "docs: regenerate site with portfolio-site skill (semantic colors, Google Fonts, OG image)" && git push
```

---

### Task 9: Push skills repo

**Files:**
- Push all `~/.config/opencode` skill changes

- [ ] **Step 1: Push skills/dotfiles repo**

```bash
git -C ~/.config/opencode push
```
```

