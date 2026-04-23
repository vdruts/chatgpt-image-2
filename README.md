```
██╗███╗   ███╗ █████╗  ██████╗ ███████╗      ██████╗ 
██║████╗ ████║██╔══██╗██╔════╝ ██╔════╝      ╚════██╗
██║██╔████╔██║███████║██║  ███╗█████╗  █████╗ █████╔╝
██║██║╚██╔╝██║██╔══██║██║   ██║██╔══╝  ╚════╝██╔═══╝ 
██║██║ ╚═╝ ██║██║  ██║╚██████╔╝███████╗      ███████╗
╚═╝╚═╝     ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝      ╚══════╝

╔═╗╦ ╦╔═╗╔╦╗╔═╗╔═╗╔╦╗
║  ╠═╣╠═╣ ║ ║ ╦╠═╝ ║ 
╚═╝╩ ╩╩ ╩ ╩ ╚═╝╩   ╩ 
```

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill for OpenAI's `gpt-image-2`. Opinionated for the three jobs it actually wins on: **editorial typography, multi-image composition, and dense infographics.**

One tool. Routes text prompts through `/images/generations` and multi-reference prompts through `/images/edits` automatically. Ships with battle-tested recipes, a prompting playbook, and a complete failure-mode cheatsheet. MIT. Fork-friendly.

---

## See it work in 30 seconds

```bash
node tools/generate.js \
  --prompt 'Minimal poster on cream background, single bold serif word: SHIPPED. Black ink, generous whitespace.' \
  --size 1024x1024 --quality low \
  --output ./shipped.png
```

Legible typography, correct spelling, clean composition, first try. That's the sell.

---

## What this skill is for

Three jobs `gpt-image-2` genuinely nails:

1. **Legible typography inside the image.** Posters, magazine spreads, infographics with real headlines and body copy. Text rendering accuracy sits above 95% across Latin, CJK, and Arabic scripts.
2. **True multi-reference composition.** The Edits endpoint accepts up to 16 reference images and combines them into one scene. Pass `product + lifestyle + brand asset`, get one image back.
3. **Structured layouts.** Grids, hierarchy, spacing, "do exactly this" prompts. `gpt-image-2` follows spatial instructions rather than hallucinating a vibe.

---

## Install

```bash
git clone https://github.com/vdruts/chatgpt-image-2 ~/.claude/skills/chatgpt-image-2
cd ~/.claude/skills/chatgpt-image-2
npm install
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart your session and `chatgpt-image-2` is available.

## Configure

Two one-time things.

**1. API key.** Set `OPENAI_API_KEY` in your environment, or drop it into `~/.claude/.env`. The tool checks `process.env`, then `./.env`, then `~/.claude/.env`, then `<skill-dir>/.env`.

**2. Organization verification.** OpenAI gates `gpt-image-2` behind ID-based org verification. Go to [platform.openai.com/settings/organization/general](https://platform.openai.com/settings/organization/general) and click Verify Organization. The verification itself takes about two minutes. Propagation to the API can take another fifteen to thirty. Until it clears you will get `403 Your organization must be verified`.

## Use from Claude Code

Once installed, describe what you want in natural language. The skill's triggers route requests involving "infographic", "meta ad", "viral LinkedIn image", "combine these images", or explicit mentions of "ChatGPT Images" / "gpt-image-2".

Example prompts that route here:

- *"Make me an infographic on the 5 stages of customer awareness."*
- *"Combine these 3 product photos into one lifestyle shot."*
- *"Create a viral LinkedIn image for a post on prompt engineering being dead."*

## Use from the CLI

### Text to image

```bash
node tools/generate.js \
  --prompt 'Editorial magazine cover. Bold serif headline reading "PRECISION" in black ink on a cream background. Geometric red and black accents, Bauhaus-inspired. No extra text.' \
  --size 1024x1536 \
  --quality high \
  --output ./cover.png
```

### Multi-image composition

Pass `--reference-image` multiple times. The tool switches to the Edits endpoint and composes:

```bash
node tools/generate.js \
  --prompt 'A premium gift basket on a white studio background. Image 1: body lotion, placed at back-left. Image 2: candle, placed center. Image 3: soap, placed front-right. Add a cream ribbon and a handwritten "Relax and Unwind" label. Preserve product labels exactly.' \
  --reference-image ./body-lotion.png \
  --reference-image ./candle.png \
  --reference-image ./soap.png \
  --output ./giftbasket.png
```

### All options

```
--prompt "<text>"               Required. Image description.
--reference-image <path>        Repeatable (up to 16). Adds a reference. Triggers Edits endpoint.
--output <path>                 Output path. Default: ./output.png
--size <size>                   Any WxH where max edge ≤ 3840, both edges are
                                multiples of 16, and long:short ratio ≤ 3:1.
                                Common: 1024x1024, 1024x1536, 1536x1024,
                                2048x2048, 3840x2160 (4K wide), 2160x3840
                                (4K tall), 3840x3840 (4K square), 3840x1280
                                (3:1 ultrawide), auto. Default: auto
--quality <quality>             low | medium | high | auto. Default: high
--model <model>                 gpt-image-2 | gpt-image-1.5 | gpt-image-1 | gpt-image-1-mini.
                                Default: gpt-image-2
--moderation <level>            auto | low. Default: auto
--n <count>                     Number of images to generate (1-4). Default: 1
```

---

## Prompting playbook

Distilled from OpenAI's cookbook guide, fal.ai's prompting notes, and a stack of real tests. Treat these as defaults, not dogma.

### 1. Structure beats length

Write your prompt in this order. Long rambling paragraphs underperform short labeled segments.

```
Scene / background  ->  Subject  ->  Key details  ->  Typography  ->  Constraints
```

Front-load what matters. `gpt-image-2` processes language sequentially. **The first ten words of your prompt carry the most visual weight.** If the headline is the point, say "Editorial poster with the headline 'X'" before describing the background.

### 2. Text rendering rules

This is where `gpt-image-2` earns its premium. Use the rules or you will lose them.

- **Always quote the exact text.** `'a poster reading "SHIPPED"'`. Unquoted text gets paraphrased.
- **Specify placement and typography.** Font style (serif / sans / mono), size (large / medium / small), weight, color, and position. "Bold serif, centered, 40% of frame height, black on cream."
- **Demand verbatim rendering.** Add `no extra text`, `no duplicate letters`, `no hallucinated words`. Models love to sprinkle filler.
- **Spell out tricky words letter by letter** for brand names or non-dictionary terms: `the word "ACME" (A-C-M-E)`.
- **Use `--quality high`** for small text, dense information panels, and multi-font layouts. `medium` blurs fine type.
- **Put every piece of text in its own quoted phrase** if you have multiple copy blocks. "Headline 'X', subhead 'Y', CTA 'Z'." Don't run them together.

### 3. Multi-reference labeling pattern

When using the Edits endpoint with 2+ references, label every image by role and reference the labels in your instruction:

```
Image 1: base scene to preserve
Image 2: jacket to composite onto the subject in Image 1
Image 3: boots to composite onto the subject in Image 1
```

Then: *"Apply Image 2 and Image 3 to the subject in Image 1. Keep Image 1's lighting, background, and camera angle unchanged."*

Without labels, the model guesses which reference is canonical. With labels, it follows instructions.

### 4. Preserve-and-change language

For surgical edits, state both sides explicitly. The model drifts otherwise.

- **Change only:** `"Replace the jacket with the one from Image 2."`
- **Keep:** `"Keep the face, hair, pose, background, lighting, and color temperature unchanged."`

Repeat the preserve list on every iteration. Drift compounds across edits.

### 5. Lighting and materials

Lighting separates flat images from cinematic ones. Be specific.

- "Fluorescent ceiling light mixed with neon signage glow"
- "Dramatic orange-red gradient backlight, subject in silhouette"
- "Golden hour natural light, low angle, warm shadows"

Materials: name textures, surfaces, reflectivity. "Matte cream paper, no gloss." "Brushed aluminum with fingerprint smudges." Generic prompts produce generic images.

### 6. Quality levers (use sparingly)

Add these ONLY when the base prompt underdelivers. Stacking them turns output mushy:

- `photorealism` is the single highest-leverage word for lifelike output. Drop it into any prompt aiming for a real-photo look and the realism jumps exponentially. Skip it for illustration, poster, or editorial-graphic work.
- `film grain` / `35mm` for analog warmth
- `macro detail` / `shallow depth of field` for product shots
- `candid, unposed` to defeat the default polished-studio look
- `hand-drawn, pencil texture` for editorial illustration

### 7. Aspect ratio and resolution (up to 4K)

`gpt-image-2` accepts any `WxH` that satisfies three constraints:

- **Max edge ≤ 3840px** (true 4K)
- **Both edges are multiples of 16**
- **Long-to-short ratio ≤ 3:1**

That opens up a much wider design space than "1024 or 1536." The ratio changes the reading experience as much as the content does — test at least two ratios for hero images. Defaults by use case:

| Ratio | Use case | Standard | 4K / hi-res |
|---|---|---|---|
| 1:1 | Instagram post, avatar, square poster | `1024x1024` | `3840x3840` |
| 2:3 | LinkedIn portrait, magazine cover | `1024x1536` | `2048x3072` |
| 3:2 | Editorial landscape, hero banner | `1536x1024` | `3072x2048` |
| 16:9 | YouTube thumbnail, widescreen hero | — | `3840x2160` |
| 9:16 | Reels, TikTok, Stories, vertical mobile | — | `2160x3840` |
| 4:3 | Classic photo landscape | `1024x768` | `2048x1536` |
| 3:4 | Classic photo portrait | `768x1024` | `1536x2048` |
| 2:1 | Wide banner, ticket-style layout | `1536x768` | `3072x1536` |
| 3:1 | Max-allowed ultrawide (hero strip, OG header) | — | `3840x1280` |

Notes:
- **`1920x1080` does NOT work** — 1080 is not a multiple of 16. Use `3840x2160` for exact 16:9 at 4K, or `1920x1088` for a close-to-16:9 frame at 2K.
- **`auto`** lets the model choose a shape that matches your prompt. Useful early. Lock a specific size once the composition is committed.
- **Cost scales with `size × quality`.** 4K at `--quality high` is the most expensive combination. The standard workflow: iterate at `1024` on `--quality low`, commit the composition, then re-render the winning prompt at 4K on `--quality high` for the final deliverable.
- **High-DPI screens and print love 4K.** 3840px on the long edge maps cleanly to retina displays and gives you headroom for poster print at 300 DPI up to ~13 inches.

---

## Recipes

`recipes/` contains battle-tested prompt skeletons for the three highest-leverage cases. Read the relevant one before composing your own. They encode aspect ratios, composition rules, and the specific gotchas for each use case.

- [`recipes/infographic.md`](recipes/infographic.md) - stats, frameworks, educational visuals
- [`recipes/meta-ad.md`](recipes/meta-ad.md) - FB / IG ad creative with hook overlay + context
- [`recipes/viral-linkedin.md`](recipes/viral-linkedin.md) - high-contrast typographic scroll-stops

---

## Failure modes and fixes

From OpenAI's own cookbook plus lived experience:

| Symptom | Fix |
|---|---|
| Text illegible or wrong characters | `--quality high`, quote the text, spell out tricky words letter-by-letter |
| Identity drifts across edits | Restate the preserve list every iteration |
| Unwanted creative reinterpretation | Add "no new elements", "preserve layout and perspective", lock specifics |
| Overpolished / stock-photo feel | Use "candid, unposed, natural light" instead of "studio, cinematic" |
| Duplicate / extra words appear | Add `no extra text`, `no duplicate letters`, `single headline only` |
| Compositing looks pasted-on | Name the lighting and shadows explicitly. "Match lighting from Image 1." |
| Slow generation (>60s) | Drop to `--quality medium`, or reduce reference image count |

---

## Costs

Per-call pricing is a function of size x quality x input image tokens (for multi-reference composition). Check [OpenAI pricing](https://platform.openai.com/docs/pricing) for current numbers. Two rules of thumb:

- Text-to-image at `--quality low` is cheap enough to iterate freely.
- Multi-reference edits at `--quality high` stack token costs fast. Every reference is processed at high fidelity. Budget accordingly.

---

## Limitations

- No transparent backgrounds on `gpt-image-2`. Omit `background: transparent` from prompts.
- Complex prompts can take up to two minutes. Plan for it.
- Org verification required (one-time, about two minutes plus propagation).
- Subject to OpenAI rate limits. Your tier determines throughput.

---

## When NOT to use this skill

- **Multi-slide carousel decks** -> use a carousel-specific tool. `gpt-image-2` generates one image at a time and cannot hold layout continuity across slides.
- **Realtime / streaming generation** -> not supported by `gpt-image-2`.
- **Transparent PNGs** -> not supported. Composite externally or use a different model.

---

## License

MIT. See [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome. Particularly interested in:

- Additional recipes (book covers, ebook mockups, app store screenshots)
- Aesthetic profiles for common brand systems
- Cost reporting / token accounting helpers
- Extended preserve-list patterns for multi-step editing workflows

---

## References

- [OpenAI Cookbook - GPT Image Models Prompting Guide](https://developers.openai.com/cookbook/examples/multimodal/image-gen-models-prompting-guide)
- [OpenAI Image Generation API Docs](https://developers.openai.com/api/docs/guides/image-generation)
- [fal.ai - GPT Image 2 Prompting Guide](https://fal.ai/learn/tools/prompting-gpt-image-2)
