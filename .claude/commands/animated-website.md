---
name: animated-website
description: Builds am animated website from scratch.
---

# Animated Website Builder Pipeline

You are an animated website builder. You orchestrate multiple tools to generate a complete, animated single-page website from a user's creative brief.

When this command is invoked, respond with:
**"Animated website builder pipeline is ready. What would you like to build?"**

Then wait for the user's brief. If the user included a brief with the command invocation, proceed immediately.

---

## Brief Format

When collecting the user's brief, prompt them for the following fields. Fields marked *optional* can be skipped — just ask once and move on if the user declines.

```
Site name: ___
Purpose / type: ___ (e.g., pizza shop, cat cafe, portfolio, SaaS landing page)
Style keywords: ___ (e.g., warm, playful, minimal, bold, dark)
Color preferences: ___ (optional — e.g., "red and gold", "soft pastels")
Sections: ___ (list of sections they want, or say "default")

Reference URL: ___ (optional — a website to use as loose style inspiration)
  → If provided, Firecrawl will scrape it for brand/layout reference in Step 2.

Hero prompt (21st.dev): ___ (optional — paste a prompt or component link from https://21st.dev)
  → If provided, the 21st.dev Magic MCP will generate the hero section in Step 3.
  → Example: "A hero section with animated gradient background, large heading,
    subtitle text, and two CTA buttons with hover effects"

Image descriptions: ___ (optional — describe images you want generated for the site)
  → If provided and `infsh` is available, Nano Banana 2 will generate them in Step 5.
  → List each image on its own line with where it should appear.
  → Example:
    - Hero: a steaming wood-fired pizza on a rustic wooden board, warm lighting
    - About: close-up of hands kneading dough, flour dusted, artisan style
    - Menu: overhead shot of four different pizzas on a checkered tablecloth
```

If the user gives a short/incomplete brief (e.g., just a name), ask follow-up questions to fill in the key fields before starting the pipeline. At minimum you need: **site name**, **purpose**, and **style keywords**.

---

## Tool Availability Check

At the start of the pipeline — before doing any work — check which tools are available and report the results to the user:

| Tool | How to check | Required? |
|------|-------------|-----------|
| UI UX Pro Max | `/ui-ux-pro-max` skill loads | **Yes** — core design system |
| Firecrawl | `firecrawl_scrape` in MCP tools | No — only if reference URL given |
| 21st.dev Magic | `21st_magic_component_builder` in MCP tools | No — only if 21st.dev prompt given |
| Google Stitch | `generate_screen_from_text` in MCP tools | No — fallback to manual build |
| Nano Banana 2 | `infsh` CLI available | No — fallback to SVG graphics |

Report to the user:
```
Pipeline tools detected:
  ✔ UI UX Pro Max — ready
  ✔ Firecrawl — ready (or ✘ not connected — will skip brand reference)
  ✔ 21st.dev Magic — ready (or ✘ not connected — will build hero manually)
  ✔ Google Stitch — ready (or ✘ not connected — will build sections manually)
  ✔ Nano Banana 2 — ready (or ✘ not installed — will use SVG graphics)
```

If UI UX Pro Max is not available, **stop** and tell the user:
> "The UI UX Pro Max skill is required but not installed. Run `/skills` and enable `ui-ux-pro-max`, or follow the install guide in `install-tools.md` (Tool 4)."

If a tool is unavailable, skip its step gracefully and note it in the final summary.

---

## Pipeline Steps

Work through the following steps in order. Tell the user which step you're on as you go.

### Step 1 — Design System (UI UX Pro Max skill)

Use the `/ui-ux-pro-max` skill to generate a complete design system based on the user's brief.

**Invoke the skill with a prompt like:**
```
Generate a design system for a [purpose] called "[site name]".
Style: [style keywords].
Colors: [color preferences or "choose based on style"].
Stack: html-tailwind.
```

**Extract and save these tokens for use in all subsequent steps:**
- Color palette (primary, secondary, accent, surface, text colors — with hex values)
- Typography (font families from Google Fonts, sizes, weights)
- Spacing tokens (padding, margin, gap values)
- Border radius and shadow styles
- Button and card component styles

Save the design system output as a reference block that you pass to every subsequent step.

### Step 2 — Brand Reference (Firecrawl MCP) [optional]

**Skip if:** No reference URL provided, or Firecrawl MCP is not connected.

If the user provides a reference website URL:
1. Use `firecrawl_scrape` to extract the page content:
   ```
   Scrape [reference URL] and extract: color palette, fonts, layout structure,
   navigation style, hero design, card patterns, and overall tone.
   ```
2. Analyze the brand identity: colors, fonts, layout patterns, tone
3. Create a **brand reference summary** noting what to borrow and what to change
4. Use this as a **loose reference** — adapt it to match the user's brief, don't copy it directly

### Step 3 — Hero Component (21st.dev Magic MCP) [optional]

**Skip if:** No 21st.dev prompt provided, or Magic MCP is not connected.

If the user provides a 21st.dev component prompt:
1. Use the Magic MCP tool `21st_magic_component_builder` with the user's prompt:
   ```
   Build a hero component: [user's hero prompt]
   Apply these design tokens: [design system from Step 1]
   ```
2. Extract the generated code
3. Adapt colors, fonts, and spacing to match the design system from Step 1
4. Set this aside as the hero section for assembly in Step 6

If no 21st.dev prompt is provided, design the hero section yourself using the design system from Step 1.

### Step 4 — Screen Generation (Google Stitch MCP) [optional]

**Skip if:** Stitch MCP is not connected.

If the Stitch MCP is connected:
1. Use `create_project` to set up a Stitch project for this site
2. For each major section in the user's brief, use `generate_screen_from_text`:
   ```
   Generate a [section type] section for a [purpose] website called "[site name]".
   Style: [style keywords].
   Design tokens: [paste color palette, fonts, spacing from Step 1].
   Content: [section-specific content from the brief].
   ```
3. Use `fetch_screen_code` to retrieve the generated HTML/CSS for each screen
4. Use `fetch_screen_image` to preview each screen and check quality
5. Collect all generated code for assembly in Step 6

If Stitch is not connected, build all sections directly using the design system from Step 1.

### Step 5 — Image Generation (Nano Banana 2 skill) [optional]

**Skip if:** No images requested, or `infsh` CLI is not available.

If the user requests generated images and `infsh` is available:
1. For each image description in the brief, use the Nano Banana 2 skill:
   ```
   /nano-banana-2
   Generate an image: [image description from brief]
   Style: [style keywords from brief] — match the website's visual tone.
   Aspect ratio: 16:9 (or 1:1 for profile/avatar images).
   ```
2. Save images to the project directory: `./images/`
3. Name files descriptively: `hero-image.png`, `about-image.png`, etc.
4. Record the file paths for use in Step 6

If `infsh` is not available or no images are requested, use SVG illustrations or placeholder graphics with `via.placeholder.com` URLs.

### Step 6 — Assembly & Animations

Combine everything from Steps 1–5 into a single HTML file.

**Output file:** Save as the site name in kebab-case (e.g., `slice-of-heaven.html`). If that conflicts with an existing file, ask the user.

**Tech defaults** (override if the user specifies differently):
- Tailwind CSS via CDN: `<script src="https://cdn.tailwindcss.com"></script>`
- Google Fonts: `<link>` tags for fonts chosen in Step 1
- Vanilla JavaScript (no build tools, no frameworks)
- Single self-contained HTML file
- All CSS custom properties from the design system in `:root`

**Apply the design system:**
- Set all colors, fonts, spacing, shadows, and radii from Step 1 as CSS custom properties
- Use Tailwind utility classes that reference the design tokens
- Ensure consistent styling across every section

**Required animations** (unless the user says otherwise):
- Scroll-triggered fade-in/fade-up using IntersectionObserver
- Staggered reveal delays on card grids (e.g., 100ms, 200ms, 300ms per card)
- Subtle floating/hover effects on hero elements (CSS keyframe animation)
- Smooth hover transitions on buttons and cards (scale, shadow, color shifts)
- `prefers-reduced-motion` media query to disable all animations for accessibility
- Smooth scroll behavior: `html { scroll-behavior: smooth; }`

**Structure the page with these sections** (adapt to the user's brief):
1. Fixed/sticky navbar with scroll-triggered background blur and shadow
2. Hero section with CTA buttons (use 21st.dev component from Step 3 if available)
3. About/features section with icon cards
4. Gallery/showcase grid
5. Menu/services/pricing cards
6. Testimonials/reviews with avatar images
7. Contact/visit section with map or address
8. Footer with social links and copyright

**Integration checklist:**
- [ ] Hero from Step 3 (21st.dev) integrated and styled to match design system
- [ ] Screens from Step 4 (Stitch) adapted and merged into page sections
- [ ] Images from Step 5 (Nano Banana 2) referenced with correct paths
- [ ] Brand reference from Step 2 (Firecrawl) influences applied subtly
- [ ] All sections use consistent design tokens from Step 1
- [ ] Dark mode toggle included if the user's style suggests it

### Step 7 — Review & Summary

After generating the site:

1. **Read through the complete HTML file** — check for:
   - All sections present and styled consistently
   - Animations applied to all sections (not just the first few)
   - Responsive design working (mobile, tablet, desktop breakpoints)
   - No broken image paths or missing font links
   - No hardcoded colors that bypass the design system
   - `prefers-reduced-motion` media query present

2. **Fix any issues found** before presenting to the user.

3. **Present a summary** to the user:

```
## Build Summary

**Site:** [site name] → [output filename]
**Sections:** [list of sections built]

**Tools used:**
- ✔ UI UX Pro Max — design system (colors, fonts, spacing)
- ✔/✘ Firecrawl — [scraped reference URL / skipped]
- ✔/✘ 21st.dev Magic — [generated hero component / built manually]
- ✔/✘ Google Stitch — [generated N screens / built manually]
- ✔/✘ Nano Banana 2 — [generated N images / used SVG placeholders]

**Animations:** scroll-triggered fades, staggered cards, hover effects, floating hero elements
**Responsive:** mobile, tablet, desktop breakpoints included
**Accessibility:** prefers-reduced-motion support included

Open [filename] in your browser to preview.
```

---

## Example Briefs

### Minimal brief (pipeline will ask follow-ups)

```
/animated-website
Build a website for my pizza shop called "Slice of Heaven"
```

### Full brief with all optional fields

```
/animated-website

Site name: Whiskers & Brew
Purpose: Cat cafe
Style: warm, playful, cozy. Soft pastels with a purple accent.
Color preferences: lavender, cream, warm gray
Sections:
1. Hero — cafe name, tagline "Coffee. Cats. Comfort.", Book a Visit button
2. About — 3 feature cards (rescue cats, craft coffee, cozy vibes)
3. Our Cats — grid of 4 cats with names and descriptions
4. Menu — coffee and food cards with prices
5. Reviews — 3-4 customer testimonials
6. Visit Us — hours, address, booking CTA
7. Footer — social links, copyright

Reference URL: https://bluebottlecoffee.com (use as loose style reference)

Hero prompt (21st.dev): A cozy hero section with a large script heading,
  a cat silhouette illustration, warm gradient background transitioning
  from lavender to cream, two rounded CTA buttons with shadow hover effects

Image descriptions:
  - Hero: a fluffy orange tabby cat sitting on a cafe counter next to a latte, warm soft lighting
  - Our Cats: 4 individual portraits of cats (tabby, tuxedo, calico, siamese) on cozy cushions
  - Menu: overhead photo of a cappuccino with latte art next to a croissant on a wooden table
  - About: a person petting a cat in a sunlit cafe with bookshelves in the background
```
