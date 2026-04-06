---
name: render
description: Visual Identity & Asset Creation — Treat visual debt as design debt. Covers brand systems, AI image generation, photo direction, asset creation, and platform-specific specs for any business or creator.
icon: palette
category: universal_compound
version: 1.0.0
author: CutTheChexx
tags: [design, branding, visual-identity, assets, AI-generation, photo-direction, graphics]
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Render: Visual Identity & Asset Creation

Visual debt is real debt. Bad branding, inconsistent assets, and weak visual direction compound every quarter. This skill treats visual identity like a product problem—actionable frameworks, not design philosophy.

---

## 1. Visual Identity Fundamentals

### Logo Considerations
A logo isn't decoration. It's a compression of your entire brand into one mark.

**What makes a working logo:**
- Reads at favicon size (16x16px). If it doesn't work small, it's not a logo.
- Works in one color (black/white). Color is polish; one-color is structural.
- Distinct from competitors. Similar shapes mean similar market perception.
- Timeless (not trendy). Logo refresh every 5 years is maintenance, not change.

**Logo types by business:**
- Wordmark: Name is the symbol (Google, Netflix, Slack). Best for brands that are their founder or category.
- Icon + Wordmark: Symbol + name together (Apple + "Apple"). Strongest for recall.
- Icon-only: Symbol alone (Apple, Nike). Only viable if you have massive brand spend behind it.
- Abstract/Geometric: Circles, grids, letterforms (Spotify, Airbnb). Works for platforms that can own new aesthetics.

**Red flags:**
- Overly complex (detail disappears at small sizes).
- Trendy effects (gradients, glows). They age immediately.
- Generic symbols (lightbulbs, rockets, stars). They read as "we're a startup."

### Color Psychology & Application
Color is the fastest way to trigger emotion and recognition.

**Primary color selection:**
- One anchor color carries 80% of weight. It appears on logo, primary UI, hero assets.
- Choose based on competitor landscape. Blue is trust (too common). Red is urgency/energy. Green is growth/health. Orange is friendly/informal. Purple is premium/creative.
- Test the color in context: web, print, social media thumbnails, favicon.

**Secondary & accent colors:**
- One secondary supports the primary (typically complementary or analogous).
- 2-3 accent colors for warnings, highlights, CTAs. These should pop against primary.
- Define color ratios: Primary 60%, Secondary 30%, Accents 10%.

**Contrast & accessibility:**
- Text on background must meet WCAG AA (4.5:1 contrast for body text, 3:1 for large text).
- Test with accessibility checkers (WAVE, Stark, Color Oracle).
- Never rely on color alone to communicate (icon + color for status indicators).

### Typography Pairing
Two typefaces maximum: one for headlines, one for body. Three is chaotic.

**Pairing strategies:**
- Serif + Sans (classic: Georgia + Helvetica, Garamond + Open Sans).
- Sans + Sans (modern: Montserrat + Lato, Inter + Space Mono for code).
- Script + Sans (decorative: Playfair + Raleway, only for luxury/lifestyle).

**Font selection:**
- Headline: Distinctive, memorable, limited use (logo, headers, CTAs).
- Body: Highly legible, neutral, readable at 14-16px on screen and 10-12pt in print.
- Monospace: Code, technical documentation, data tables.

**Web typography:**
- Stick to system fonts + 1-2 web fonts (Google Fonts, Typekit). Each font adds 50-100ms load time.
- Font weight variety: Regular (400) + Bold (700) minimum. Italic for emphasis, not readability.
- Line height: 1.5 for body text (20px for 14px base), 1.2-1.3 for headlines.
- Letter spacing: Tight for headlines (-1px), normal for body, loose for all-caps display (0.1-0.2em).

### Visual Hierarchy
Make scanning effortless.

**Hierarchy tools:**
- Size: Biggest = most important.
- Weight: Bold draws eye before light.
- Color: Accent color pulls focus before neutral.
- White space: Isolated elements feel important.
- Position: Top-left to bottom-right (Western reading pattern).

**Anti-pattern:** Every element competing for attention. Starve secondary elements of visual weight.

---

## 2. AI Image Generation

AI generation is production infrastructure now, not an experimental tool.

### Prompt Engineering Fundamentals

**Structure of a winning prompt:**
```
[Subject] + [Style/Aesthetic] + [Medium] + [Lighting] + [Mood] + [Camera/Composition] + [Technical params]
```

**Example breakdown:**
- Subject: "Woman in tech founder headshot"
- Style: "Modern, approachable, confident"
- Medium: "Professional portrait photography"
- Lighting: "Soft directional light, warm backlight"
- Mood: "Inspiring, trustworthy, human"
- Composition: "Close-up, eye contact, shallow depth of field"
- Params: "8k, high quality, magazine cover"

**What works:**
- Specific references (Vogue photography, Wes Anderson color palette, Apple product photography).
- Constraints (symmetrical, minimal, flat lay, overhead shot).
- Negative prompts (exclude blurry, low quality, distorted, watermark).

**What fails:**
- Vague subject matter (just "a building").
- Contradictory styles (photorealistic + cartoon).
- Technical jargon without context (Anisotropic filtering, subsurface scattering — models don't know these).
- Describing what you don't want without describing what you do.

### Style Consistency Across Generations

**Establish visual rules:**
1. Color palette: Define 3-5 dominant colors. Lock them in each prompt.
2. Lighting: Consistent (soft/hard, warm/cool, time of day).
3. Composition: Framing style (wide shots, close-ups, flat lays).
4. Subject types: Photography style (editorial, commercial, lifestyle, candid).
5. Quality bar: Resolution, detail level, artistic direction.

**Workflow:**
1. Generate 8-10 variations with the same prompt.
2. Pick 2-3 winners that best fit brand direction.
3. Refine the prompt based on what worked.
4. Lock that prompt; use it as the baseline for future variations.

### Prompt Templates by Use Case

**Product Photography:**
```
[Product name], [color/material detail], photographed on [surface/environment],
product photography style, studio lighting, clean white background,
neutral shadows, minimal, 8k resolution, professional product shot
```

**Lifestyle/Usage Shots:**
```
[Person/demographic] using [product], in [environment], candid,
natural daylight, warm color palette, editorial photography,
storytelling, real moment, high quality, professional
```

**Abstract/Conceptual:**
```
Abstract representation of [concept], [style direction: minimalist/maximalist],
[color palette], [composition: symmetrical/asymmetrical],
[medium: 3D render/illustration/photography], [mood]
```

**Portraits/Headshots:**
```
Headshot of [demographic], [style: professional/approachable/editorial],
professional portrait photography, soft lighting, [background: blurred/minimal/scenery],
eye contact, magazine quality, 8k, high resolution
```

**Social Media Carousel:**
```
Square composition, [specific dimensions], typography integrated,
bold color [accent color], minimal, scannable in feed,
high contrast, readable at thumbnail size
```

### Iteration Workflow
1. **First pass:** Generate 10 images with baseline prompt.
2. **Evaluation:** Which 2-3 nail the brief? What's missing?
3. **Refinement:** Adjust lighting, composition, mood, style references.
4. **Second pass:** Generate 8 with refined prompt.
5. **Selection:** Pick winner + 2 alternates.
6. **Documentation:** Save prompt, settings, timestamps. This is your playbook.

---

## 3. Brand Asset Creation

### Social Media Templates

**Standard sizes (2026):**
- Feed post: 1080x1080px (square), 1080x1350px (portrait), 1200x628px (landscape).
- Stories: 1080x1920px (mobile vertical).
- Reels/TikTok: 1080x1920px (9:16).
- Headers: 1500x500px (LinkedIn), 1200x400px (Twitter/X).

**Template system:**
- Create master template in Figma/Canva with brand colors, fonts, spacing.
- Build variants: Quote posts, product posts, educational posts, promotional posts.
- Use grids (8px grid standard) to enforce alignment.
- Include bleed guides (20px safe zone for text).

**Content buckets:**
- Educational (40%): Tips, frameworks, how-tos.
- Entertaining (30%): Memes, behind-the-scenes, personality.
- Promotional (20%): Product, sales, announcements.
- Community (10%): Reposts, user-generated, conversations.

### Presentation Templates
- Deck size: 1920x1080px (16:9) standard.
- Slide masters: Title, Content, Quote, Case Study, Data (minimum 5 master layouts).
- Consistency: Same background texture, color, typography across all slides.
- Data visualization: Charts should use brand colors, clear labeling, no 3D effects.

**What works:**
- Minimal backgrounds (solid color, subtle texture, not a photo under every text).
- Breathing room (don't cram 10 bullet points on one slide).
- One idea per slide. One headline, supporting content.

**Anti-pattern:** Presentation template that's a cluttered sample with fake data.

### Email Headers & Templates
- Header dimensions: 600x200px (mobile-safe width is 480px).
- CTA button: 44x44px minimum (thumb-friendly), high contrast from background.
- Font stack: System fonts (Arial, Helvetica, Verdana) for email reliability.
- Responsive: Stack all elements vertically on mobile.

### Ad Creatives
- Static image ads: 1200x628px, 1080x1080px, 1024x1024px (platform variations).
- Video ads: 15s, 30s, 1-min versions. Captions required (80% watch muted).
- Carousel ads: 1200x628px per card, 3-5 card minimum.
- Text overlay: Keep to <20% of image. Test readability at thumbnail size.

### Business Cards
- Size: 3.5x2in (standard US), 90x50mm (metric).
- Resolution: 300 DPI minimum for print.
- Bleed: 0.125in on all sides.
- Safe zone: 0.1875in from edge (keep text/logo away from trim).
- Content: Name, title, phone, email, website. Optional: LinkedIn, social handles.

---

## 4. Photo Direction

### Shot Lists
A shot list is insurance against bad results.

**For a single shoot:**
- Hero shots: 5-8 main images (wide variety of setups).
- Detail shots: 5-8 close-ups (hands, products, textures).
- Lifestyle: 5-8 in-context shots (person using product, environment).
- Variations: Different angles, expressions, lighting setups of the same scene.
- Backup shots: Extra takes of hero shots in case the main one has an issue.

**Documentation:**
```
Shot 1: Wide shot, product on white table, natural light from left, person's hands visible
Shot 2: Close-up of product detail, 6-inch distance, shadow-free setup
Shot 3: Lifestyle: Person holding product, walking toward camera, golden hour light
...
```

### Mood Boards
A mood board is a visual brief. It compresses the entire shoot into a reference document.

**Structure:**
- Color palette (3-5 colors, actual color swatches).
- Lighting reference (time of day, quality: soft/hard, warm/cool).
- Style reference (5-8 images from photographers/brands whose aesthetic aligns).
- Composition examples (framing, perspective, depth of field).
- Subject reference (if hiring models: demographics, expressions, styling direction).
- Material/texture reference (surface types, fabrics, backgrounds).

**Format:** One page, 12x9 inches or digital equivalent. All images should be credited/sourced.

### Art Direction Briefs
The brief is the legal contract between creative and stakeholder.

**Essential components:**
1. Objective: What is this shoot selling/communicating?
2. Audience: Who are these images for?
3. Deliverables: How many images? Specs? Formats?
4. Timeline: Shoot date, delivery date, revision rounds.
5. Constraints: Budget, location, available time, talent, equipment.
6. Visual direction: Style, mood, references, do's/don'ts.
7. Usage rights: Can these images be licensed, sold, or reused?

### Photography Style Guides
A style guide locks the visual language for consistency.

**Cover:**
1. Photography type (editorial, product, lifestyle, documentary).
2. Primary aesthetic (clean/textured, bright/moody, candid/posed, wide-angle/close-up).
3. Lighting (natural/studio, warm/cool, hard/soft).
4. Color treatment (warm tones, cool tones, desaturated, saturated, specific color cast).
5. Composition rules (rule of thirds, symmetry, leading lines, depth).
6. What to include (props, people, contexts, materials).
7. What to avoid (generic backgrounds, heavy filters, clichéd poses, specific visual tropes).

**Example snippet:**
```
Product Photography: Flat-lay, overhead angle, product on textured surface (concrete, linen, natural wood).
Lighting: Soft directional light from top-left, slight shadow to define depth.
Color: Warm, earthy palette. Keep backgrounds neutral (white, grey, natural materials).
Avoid: Shadows that obscure detail, shiny surfaces that create glare, overcrowded compositions.
```

---

## 5. Graphic Design Principles (CARP)

### Contrast
Make differences obvious. Don't just tint things darker.

- Size contrast: Headline 48px, body 14px. That's contrast.
- Color contrast: Accent color in foreground against neutral background.
- Weight contrast: Bold headline, light body text.
- Style contrast: Sans serif headline, serif body (or vice versa).

**Red flag:** All elements are the same visual weight. Your eye doesn't know where to look.

### Alignment
Alignment is invisible but its absence is loud.

- Left align body text (not centered, not justified).
- Align all elements to a grid (4px, 8px, or 16px grid standard).
- Align baselines of text on the same line when side-by-side.
- Check vertical alignment too (not just horizontal).

**Tool:** Enable "snap to grid" in your design software. Design faster, look more professional.

### Repetition
Repetition = brand recognition.

- Same font throughout the piece.
- Same color palette (don't introduce new colors mid-design).
- Same icon style (if using icons, all icons should match).
- Same spacing increments (if you use 16px once, use 16px everywhere).

**Benefit:** Repeating visual elements make your brand feel intentional and established.

### Proximity
Proximity groups related information.

- Related items should be close together (within 8px).
- Unrelated items should be far apart (minimum 16px, typically 24-32px).
- Consistent spacing between similar elements (three social icons should have equal spacing).

**Example:** Product card with image, title, price, button. Image and title are 8px apart (related). Button is 16px below price (related but slightly separated). Card is 24px from next card (new item).

---

## 6. Design for Non-Designers

You don't need to be a designer to create professional-looking assets.

### Tools & Where They Work

**Canva:**
- When: Quick social posts, simple graphics, one-off designs.
- What: Templates, drag-drop elements, basic brand kit setup.
- Limitations: Limited customization, fonts, can look "Canva-like."

**Figma:**
- When: Building systems, multiple variations, team collaboration.
- What: Full design control, components (reusable elements), prototyping.
- Learning curve: Steeper but worth it. Free tier is generous.

**Adobe Creative Suite (Photoshop, Illustrator, InDesign):**
- When: Professional print work, complex editing, very specific requirements.
- What: Industry standard for designers.
- Cost: Subscription model.

**Simple rule:** Canva for templates, Figma for systems, Adobe for exceptions.

### Quick Wins That Look Professional

1. **Use whitespace:** Remove clutter. Empty space makes designs feel intentional.
2. **Stick to 2 fonts:** Chaos = 3+ fonts. Master = 2 fonts, 2-3 weights.
3. **Use a color palette:** Not every color you like. Pick 3-5 and commit.
4. **Align to grid:** Everything should snap to invisible grid lines.
5. **6px minimum padding:** Don't cram text against edges.
6. **8px increments:** Spacing should be 8px, 16px, 24px, 32px (not random).
7. **Consistent shadows:** Drop shadows or no shadows. Not a mix.
8. **Readable text:** 14px minimum for body. 44px minimum for mobile buttons.
9. **One clear focal point:** Eye should know where to go first.
10. **Test at 50% size:** If it looks bad at half-size, it'll be bad on mobile.

---

## 7. Asset Management

### File Naming Convention
Chaotic naming = chaotic workflow.

**System:**
```
[Brand]_[AssetType]_[Purpose]_[Date]_[Version].[ext]

Examples:
AcmeCorp_SocialPost_ProductLaunch_2026-04-05_v1.png
AcmeCorp_EmailHeader_NewsletterQ2_2026-04-05_v2.psd
AcmeCorp_Logo_Primary_v1.svg
AcmeCorp_BusinessCard_Standard_Print_300dpi_v1.ai
```

**Rules:**
- No spaces (use underscores).
- Date format: YYYY-MM-DD (sorts correctly).
- Version numbers: v1, v2, v3 (not "final," "final_FINAL," "for_real_this_time").
- File extensions: Match the asset type (.svg for logos, .ai for print files, .png for web).

### Folder Structure
```
Brand-Assets/
├── 01_Brand-Guidelines/
│   ├── Brand-Book.pdf
│   ├── Logo-Usage.md
│   └── Color-Palette.png
├── 02_Logos/
│   ├── Primary/
│   │   ├── Logo_Primary_Color_v1.svg
│   │   └── Logo_Primary_BW_v1.svg
│   ├── Secondary/
│   ├── Icon/
│   └── Favicons/
├── 03_Web-Assets/
│   ├── Hero-Images/
│   ├── Product-Photos/
│   ├── Illustrations/
│   └── Icons/
├── 04_Social-Media/
│   ├── Templates/
│   ├── Post-Archive/
│   └── Stories/
├── 05_Print/
│   ├── Business-Cards/
│   ├── Brochures/
│   └── Billboards/
├── 06_Templates/
│   ├── Presentation/
│   ├── Email/
│   └── Ads/
└── 07_Archive/
    └── [Deprecated assets]
```

**Rules:**
- Numbered folders (01_, 02_) sort automatically.
- Subfolders by format/purpose, not by date.
- Archive old versions; don't leave them in active folders.

### Version Control for Visual Assets
Git isn't great for large files, but you still need version history.

**Options:**
1. Figma/Adobe Cloud: Built-in version control, easy rollback.
2. Google Drive: Simple but lacks design-specific features.
3. Abstract/Plant: Git for designers, but overkill for most.
4. Local backups: Timestamped folders (2026-04-05_BACKUP) with release notes.

**Minimum:**
- Keep the current version accessible.
- Archive previous versions with dates.
- Document what changed in each version (CHANGELOG.md).

### Brand Asset Library
This is your production blueprint.

**What to include:**
- Logo files (all formats: .svg, .png, .pdf, .ai).
- Color swatches (RGB, Hex, CMYK values).
- Fonts (list + where to download).
- Typography rules (sizes, weights, line heights).
- Icon sets.
- Image libraries (product, lifestyle, team).
- Templates (social, email, presentation).
- Photography style guide.
- Usage guidelines (what's allowed, what's not).

**Format:** Living document (Google Doc or Notion), updated quarterly. Version number on the front page. Last updated date.

---

## 8. Platform-Specific Specs

Save these. Memorize them. Bookmark this section.

### Social Media Image Sizes

| Platform | Post Type | Dimensions | Format |
|----------|-----------|-----------|--------|
| Instagram | Feed Post | 1080x1080px | JPG, PNG |
| Instagram | Stories | 1080x1920px | JPG, PNG, Video |
| Instagram | Reels | 1080x1920px (9:16) | Video |
| Facebook | Feed Post | 1200x628px | JPG, PNG |
| Facebook | Stories | 1080x1920px | JPG, PNG, Video |
| TikTok | Video | 1080x1920px (9:16) | Video |
| LinkedIn | Post | 1200x627px | JPG, PNG |
| LinkedIn | Header | 1500x500px | JPG, PNG |
| Twitter/X | Post | 1024x512px (2:1) | JPG, PNG, GIF |
| Twitter/X | Header | 1500x500px | JPG, PNG |
| Pinterest | Pin | 1000x1500px (2:3) | JPG, PNG |
| YouTube | Thumbnail | 1280x720px | JPG, PNG |
| YouTube | Header | 2560x1440px | JPG, PNG |

### Web Image Specs
- Hero image: 1920x1080px (16:9), max 500KB.
- Card image: 400x300px, 100KB max.
- Favicon: 32x32px (PNG), plus 16x16px (ICO).
- Product image: 600x600px (minimum), lossy compression, <200KB.
- Thumbnail: 200x200px, <50KB.

**Compression:** Use TinyPNG, Squoosh, or ImageOptim. Remove metadata.

### Email Image Specs
- Header: 600x200px, max 100KB.
- Body image: 600x400px max, <100KB.
- CTA button: 44x44px minimum (touch-friendly).
- Fallback: Always include alt text. Assume images don't load.

### Print Image Specs
- Resolution: 300 DPI (dots per inch).
- Color mode: CMYK (not RGB).
- File format: PDF (easiest for printers).
- Bleed: 0.125in on all sides.
- Safe zone: 0.1875in from edge (keep critical content away from trim).

**File naming:** Brand_Item_300dpi_Print_CMYK_v1.pdf

---

## 9. Visual Content Strategy

### Photos vs Illustrations vs Icons

**Photography:**
- Shows reality (product, person, place).
- Higher production cost, slower iteration.
- More credible; people trust photos.
- Use when: Showcasing real products, building trust, lifestyle/emotional content.

**Illustrations:**
- Shows concept, not reality.
- Lower cost, faster to iterate.
- More flexible (can show impossible things).
- Use when: Explaining concepts, abstract ideas, variety in style, playful tone.

**Icons:**
- Visual shorthand (< 1 second to understand).
- Fastest production, most flexible.
- Use when: Navigation, status indicators, step numbers, categories.

**Decision matrix:**
- Trust/credibility required? → Photography.
- Concept or abstract idea? → Illustration.
- Quick visual reference? → Icon.
- Mixed tone? → Combination of all three.

### Visual Storytelling
A single image should communicate: subject, mood, context, action.

**Strong visual:**
- Clear subject (you know what you're looking at immediately).
- Mood is unmistakable (happy, serious, playful, urgent).
- Context is implicit (office, outdoors, home, retail).
- Action is visible or implied (standing, running, thinking, creating).

**Weak visual:**
- Ambiguous subject.
- Neutral mood (could be anything).
- Generic context (white background, everywhere and nowhere).
- Static/boring (person just standing).

**Test:** Show the image to someone for 2 seconds. Can they describe the subject, mood, context, and action?

### Consistency Across Channels
Same brand should feel the same on Instagram, email, website, print.

**Locked elements:**
1. Logo position and size.
2. Brand colors (use exact same hex codes).
3. Typography (same fonts, same weights).
4. Photography style (same lighting, same mood, same subjects).
5. Icon style (same art direction).
6. Spacing ratios (if you use 16px spacing on web, use proportional spacing in print).

**Audit checklist:**
- [ ] Logo consistent across all channels.
- [ ] Colors match exactly (use color picker to verify).
- [ ] Fonts are the same.
- [ ] Photography style is recognizable as "ours."
- [ ] Icon style is consistent.
- [ ] Spacing feels proportional (not cramped on one, spacious on another).

---

## 10. Anti-Patterns

### Stock Photo Clichés
Recognize them. Avoid them. Ban them from your brand.

**Clichés to ban:**
- Business people with headsets, wide smiles, pointing at laptops.
- Diverse group sitting around wooden table, looking at documents, all smiling.
- Woman holding coffee, looking out window pensively.
- Hands coming together for "teamwork."
- Abstract colored spheres.
- Geometric shapes floating in space.
- "Innovation" is literally a lightbulb.

**Alternative:** Real product, real person, real moment. Or hire a photographer. Both cost less than looking generic.

### Inconsistent Branding
The worst offense. Same brand, different visual language.

**Red flags:**
- Logo size/position changes every piece.
- Color palette shifts (sometimes blue, sometimes purple).
- Fonts vary (Comic Sans on Instagram, Georgia on email).
- Photography style is unrecognizable (professional photo on one piece, cell phone photo on another).

**Fix:** Brand guidelines. Enforce them. Update them annually.

### Low-Res Assets
Blurry logo? Pixelated image? Jaggy text? You've failed the bar.

**Standards:**
- Logo: Always vector (.svg) or high-res PNG (300 DPI).
- Website images: 72 DPI minimum, but 150 DPI is better.
- Print: Always 300 DPI, CMYK, PDF.
- Social: 150 DPI minimum (scale up to 200% on high-res screens).

**Rule:** When in doubt, go higher resolution. Downsampling looks bad; upsampling looks worse.

### Design by Committee
Too many cooks. Every opinion. No direction.

**Symptoms:**
- Presentation version 47 with conflicting feedback.
- Design changes daily based on who's in the room.
- No single vision; aesthetic is scattered.

**Fix:** Creative director calls the shots. Final decision maker. Everyone else inputs, director decides.

### Over-Designing
More isn't better. Restraint is sophistication.

**Anti-pattern:** Every element is fancy. Gradients, shadows, textures, animations. It's chaos.

**Real approach:**
- 80% of the impact comes from typography, color, and alignment.
- Effects (shadows, gradients) are 5% of impact but take 50% of the time.
- Minimize. Remove. Simplify. If it doesn't add value, delete it.

---

## Workflows & Checklists

### Visual Brand Audit
Run this quarterly.

```
[ ] Logo
  [ ] Renders correctly at 32px (favicon size)
  [ ] Works in one color
  [ ] Distinct from top 5 competitors
  [ ] Current and usable everywhere

[ ] Color Palette
  [ ] Primary color correct hex
  [ ] Secondary color accurate
  [ ] Contrast ratios meet WCAG AA (4.5:1 for text)
  [ ] Consistent across all materials

[ ] Typography
  [ ] Headline font is distinctive
  [ ] Body font is legible at 14px
  [ ] Font pairing makes sense
  [ ] All weights available

[ ] Photography
  [ ] Style is consistent
  [ ] Mood is recognizable
  [ ] At least 20 hero images in library
  [ ] Photography guide is updated

[ ] Icons
  [ ] Icon style is consistent
  [ ] All icons match stroke weight and style
  [ ] At least 50 icons in library

[ ] Templates
  [ ] Social media templates updated
  [ ] Email template tested in major clients
  [ ] Presentation template has at least 5 layouts
  [ ] All templates use brand colors and fonts

[ ] Guidelines
  [ ] Logo usage guide is current
  [ ] Color palette documented (RGB, Hex, CMYK)
  [ ] Typography guide exists
  [ ] Photography style guide exists
```

### Asset Creation Workflow
For any new asset (post, email, ad, presentation):

1. **Define:** What's the purpose? Who's the audience? What action do we want?
2. **Reference:** Pull mood board. Check style guide.
3. **Create:** Design in Figma/Canva. Use templates.
4. **Test:** Does it match brand? Is it readable at 50% size?
5. **Review:** Does it pass the 2-second test?
6. **Export:** Correct format, correct resolution, correct naming.
7. **Archive:** File goes to brand asset library with versioning.

---

## Platform-Agnostic Principles

These work for any business, any creator, any brand.

1. **Start with constraints.** Platform size, duration, medium. Work within them.
2. **Typography carries 80% of the weight.** Get fonts right, everything else is easier.
3. **Whitespace is your friend.** More breathing room = more professional.
4. **Color should be intentional.** Not every color you like. Palette > chaos.
5. **Consistency = recognition.** Same system applied everywhere.
6. **Test small.** If it fails at 50% size, it fails on mobile.
7. **Real over perfect.** Real product beats generic mockup. Every time.
8. **Version control.** Always know what came before.
9. **Measure against competitors.** Are we better than similar brands?
10. **Update annually.** Design trends shift. Refresh your playbook yearly.

---

## Emergency Quick-Starts

### "I need a social post in 30 minutes"
1. Open Canva. Search your industry + "social post."
2. Pick a template.
3. Replace the template text with yours.
4. Swap the image with a product/team photo.
5. Change template colors to your brand colors.
6. Download as JPG.
7. Post.

### "I need hero images but no budget"
1. AI generation (Midjourney, DALL-E). 2-3 hours, $20-50.
2. Shoot on phone. Good lighting, clear subject. 30 minutes.
3. Unsplash + light Photoshop edit. 1 hour.

### "My brand looks inconsistent"
1. Audit all touchpoints (web, social, email, print).
2. List every element that's different.
3. Create a 1-page brand snapshot: logo, 5 colors, 2 fonts, 5 images.
4. Enforce it everywhere for 90 days.
5. Measure recognition (do people recognize your brand consistently?).

---

## Tools You Actually Need

- **Design:** Figma (free tier), Canva (Pro $120/yr).
- **Image generation:** Midjourney ($10-180/mo), DALL-E (usage-based).
- **Photo editing:** Photoshop (subscription) or free: Photopea (browser).
- **Icon management:** Figma components, Noun Project (subscription).
- **Asset hosting:** Google Drive, Dropbox, Figma.
- **Color tools:** Coolors.co, Adobe Color, Accessible Colors.
- **Contrast checker:** WebAIM, Stark (Figma plugin).

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
