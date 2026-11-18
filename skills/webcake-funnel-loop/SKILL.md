---
name: webcake-funnel-loop
description: Website development loop for Webcake product funnels — 3-AI production loop (Employee 1 Full Stack Dev → Employee 2 Creative/Code Analyzer → Employee 3 Team Leader QA) that builds/edits a complete funnel: Webcake landing page, POS Pancake value-combos, Botcake Messenger sequence, Meta Datasets & Pixel, and domain wiring. Use when the boss asks to build, redesign, or fix a product landing page, website, or funnel on Webcake, or says "website development" or "webcake funnel loop". Output ships only after Employee 3 passes it with zero open CRITICAL/HIGH findings.
---

# Website Development — Webcake Funnel Factory (3-Employee Loop)

You are running a **3-employee production loop**. The user is **the Boss**. Nothing is shown to the Boss as "done" until it survives the full loop with zero open CRITICAL/HIGH findings. The loop never stops on its own while corrections, changes, or unfinished tasks remain.

**READ FIRST, EVERY RUN:** [references/mistake-registry.md](references/mistake-registry.md) — hard rules from a full production audit of a live product funnel. Any output that repeats a registry mistake is an automatic FAIL at Employee 3, no matter how good it looks.

---

## 0. The Three Employees

Implement the employees as genuinely separate passes. Employee 2 and Employee 3 run as **subagents** (Agent tool) so they review with fresh eyes and no attachment to the work. Never let Employee 1 grade their own homework as E2/E3.

| Role | Who | Job | Exit condition |
|---|---|---|---|
| **Employee 1 — Full Stack Developer** | Main assistant (you) | Does ALL production work: design system, page build, sections, events/JS, POS combos, Botcake, pixel wiring. Includes a **self-check pass** before handing off — run the E1 self-checklist (§6) so the work is already clean at your own end. | Self-checklist 100% green |
| **Employee 2 — Creative / Code & Visual Analyzer** | Subagent (general-purpose, or `gan-evaluator` if screenshots needed; if `gan-evaluator` is not installed, `general-purpose` with `render_preview` screenshots) | Reviews E1's output for: creativity, visual polish, layout/lining/layering precision, copy quality, code correctness, event-chain correctness. Returns a numbered findings list → **back to E1**. Loops E1↔E2 until the exit rule below is met. Only then passes to E3. | Exit rule met |
| **Employee 3 — Team Leader (Final QA)** | Subagent, **strict mode** | Last touch. Very strict on designs, layout, functions, data accuracy. Verifies against the mistake registry + full funnel checklist (§7). Any CRITICAL/HIGH finding → back to **E1** for reprocess → E2 re-review → E3 again. Only when E3 passes does the output go to the Boss. | Exit rule met, registry clean |

**Boss revision loop:** When the Boss sends changes at any point, they enter at Employee 1 and traverse the SAME full loop (E1 → E2 → E3 → Boss). No shortcuts, even for "small" changes — small changes caused the worst production bugs on record.

**Loop mechanics:**
- Track cycles: announce `Cycle N: E2 returned X findings (C critical, H high, M medium, L low)` so the Boss sees progress.
- Findings must be numbered, specific, and actionable ("tstRole3 has typo 'Ethusiast'", not "check the testimonials").
- E1 must answer every finding with FIXED / justification. Unresolved CRITICAL/HIGH = loop continues.
- If E1↔E2 exceeds 5 cycles on the same finding, escalate the disagreement to the Boss with both positions — do not deadlock silently.
- Every finding carries a severity: **CRITICAL** (wrong money figure, broken order path, pixel/POS data corrupted, something live breaks), **HIGH** (a real bug, a registry repeat, a missing requirement, a claim without evidence, a visible layout defect or typo at either breakpoint), **MEDIUM** (polish a buyer would barely notice that no registry item covers, maintainability), **LOW** (nitpick). A finding that matches a registry item is at least HIGH, and so is an image placeholder unless the Boss explicitly ships without the image. When two levels fit, take the higher.
- **Exit rule:** zero CRITICAL/HIGH open AND every MEDIUM either FIXED or listed for the Boss as known minor with a reason. LOW is fixed when cheap, otherwise listed. E3 may re-grade a finding upward; nobody downgrades or deletes a finding to end the loop.
- If a named agent type is not installed, run `general-purpose` with the same brief. From E2's second cycle on, continue the same E2 agent (it re-reviews the whole new diff, not just its own list) instead of starting a new one. E3 always starts fresh and runs on a different model than E1 (for example `sonnet` when E1 runs on Opus); note the model in the evidence.

---

## 1. Intake — ask the Boss BEFORE building

Ask in ONE short batch (skip anything already known from memory/context):

1. **Product** — name, what it is, price points, bundle structure (how many pcs per tier).
2. **Facebook Page link** — REQUIRED. The clickable FB logo on the site and the m.me confirmation button both point here. Also confirm the Botcake page id for that FB page.
3. **POS shop** — confirm the shop_id and whether product/variations already exist in POS.
4. **Pixel / dataset** — which pixel id, which ad account.
5. **Domain** — existing Hostinger domain to attach, or none yet.
6. **Creatives** — product photos on hand? (See §5 protocol if none.)
7. **Design direction** — any preference, or E1 proposes one (never generic; pick a real direction per the design rules).

Also request ONE-TIME tool approval for the webcake-landing/botcake/pancake-pos MCP servers so the loop isn't interrupted per call.

---

## 2. Build Order (Employee 1's pipeline)

Build in this order — later stages depend on earlier IDs:

### Phase A — POS first (source of truth for prices)
1. Ensure product + variations exist (`pos_product list_products_variations` for real variation IDs — NEVER invent IDs).
2. Create/verify **value-combos**, one per bundle tier, pattern:
   - `1pc {PRODUCT}` — combo value = tier-1 price — Apply by: Variant — Quantity in combo (1)
   - `2pcs {PRODUCT}` — combo value = tier-2 price — Quantity in combo (2)
   - `3pcs {PRODUCT}` — combo value = tier-3 price — Quantity in combo (3)
   - `is_free_shipping` per the offer; activated; no end date unless Boss says so.
3. **Weights must scale with quantity** (per-unit weight × qty). Registry M6: never let 3pcs weigh less than 1pc.
4. Record: product_id, every variation_id, every combo id + value_combo.

### Phase B — Webcake page
1. Lock a design system first (palette from product's primary color, type scale, spacing grid, radius/card spec) — every element derives from these tokens.
2. Build sections (announce bar → nav → hero → video/proof → order section → benefits → how-to → testimonials → FAQ → footer, adapted to product).
3. Order section wiring (a proven architecture):
   - Hidden `radio` bound to POS variations — each option's variations[] uses the **real** variation_id + qty; price = qty × retail (POS combo overrides the charge).
   - Visual bundle rows drive the radio via `set_field_value` + `change_color` chain + `custom_js` that checks the real radio and dispatches input/change/click.
   - Hidden `variant` mirror input for the human-readable label.
   - Child form = the submitting form; parent uses `multiForm` → child.
   - PH phone validator `^(09|\+639|639)\d{9}$`, address widget country 63, Province/City/Barangay all required.
4. Confirmation popup: order recap + **"Get Updates through Messenger"** button → `https://m.me/{FB_PAGE_ID}?ref={REF}` (the Botcake ref that starts the product's sequence).
5. FB logo (and any social icons) clickable → the Boss's Facebook Page link.
6. Page settings: title, description, **keywords for THIS product**, favicon, thumbnail, correct currency/country, fonts.
7. `validate_page` → publish (`publish_page` dry_run then real — patch alone does NOT go live) → `render_preview` at 1200 AND 420 → E1 self-check vs the visual.

### Phase C — Botcake
1. Sequence for the product, default template (timing adjustable — recommend what fits the product, Boss approves):
   - +2 minutes → **LIMITED-TIME PROMO**
   - +5 → **WHY YOUR {product category} NEEDS CARE**
   - +10 → **WHY CHOOSE {PRODUCT}?**
   - +15 → **QUICK & EASY TO USE**
   - +20 → **ORDER YOURS TODAY**
   - Units (minutes vs hours) are a per-product decision — propose, don't assume. Send window 08:00–22:00 all days unless Boss overrides.
2. Every flow gets a REAL descriptive name — never leave autogenerated names like "Sequence Message 6" (registry M5).
3. Ref link (`?ref=...`) wired to the entry flow; entry flow subscribes the customer to the sequence.
4. Messages must be product-specific, benefit-led, with clickable CTA back to the page. Messages 2–4 need real hooks (registry M3: middle messages die without them).

### Phase D — Datasets & Pixel
Goal: ads manager data ≈ POS actual data. Non-negotiables:
1. **NEVER hardcode `fb_conversion_value`** on the form (registry H2 — the one-price-forever bug). Leave the browser value empty/dynamic and let Pancake server CAPI carry the true per-order value; Meta dedups browser+server by event.
2. No duplicate manual Purchase rules in Events Manager. If one exists for the same action, it stays/goes INACTIVE.
3. After launch, verify: Purchase browser vs server counts dedup to one series; EMQ has phone at ~100% on Purchase.
4. Confirm PageView / ViewContent / Purchase all firing on the live domain before handing to E3.
5. **A test order is a real Purchase to Meta** (registry H5). It fires from the browser and from Pancake's server, and Meta keeps it: Events Manager → Test Events only lets you watch events arrive, it does not exclude them, and cancelling the order in POS does not take the Purchase back. So place only the ONE test order H1 requires (another only if the order path changed), record its id, and after E3's check cancel it in POS with the Boss's OK: send `status: 6` together with `status_name: "canceled"`, and never pass `note` (it overwrites the whole order note).
6. **New pixel = no Purchase optimization yet** (registry H5). A pixel whose only Purchases are test orders has nothing to learn from, and Purchase-optimized campaigns on it may not deliver at all. Tell the Boss the pixel's real purchase count and recommend optimizing for ViewContent (or adding an InitiateCheckout event) until real purchases arrive.

### Phase E — Domain
At your DNS provider, add the A and CNAME records Webcake's custom-domain screen shows for your page (check there for the current values), TTL 300. Attach in Webcake, verify the live URL serves the page.

---

## 3. Design Standard (strict)

- Pick a named style direction; forbidden: template-looking output, uniform padding everywhere, unlayered flat cards, default fonts without reason.
- **Layering/lining/layout precision is a pass/fail axis**: aligned edges on the section grid, consistent radius scale, consistent card treatment, no element visually clipping or overlapping unintentionally at EITHER breakpoint (1200 desktop / 420 mobile — check both, always).
- Per-section background discipline; single accent color used semantically.
- No fabricated social proof beyond what the Boss approves.
- Motion must be intentional (reveal on the key heading, transition on bundle switch) — not decorated everywhere, not absent entirely.
- Copy: zero typos (registry L1), consistent capitalization, PH-market English/Taglish as fits the product.

---

## 4. What Employee 2 checks (analyzer pass)

Numbered findings across:
1. **Creative quality** — does it look studio-made? Hierarchy, rhythm, depth, intentional states?
2. **Visual defects** — misalignment, spacing drift, broken layering, off-grid elements, both breakpoints (use `render_preview` / screenshots).
3. **Copy** — typos, wrong-product leftovers, weak hooks, inconsistent prices between sections.
4. **Code/events** — every clickable does what it claims; radio bridge intact; custom_js targets exist; no orphan/duplicate elements.
5. **Data consistency** — page prices == combo values == popup recap == pixel values.

## 5. Creatives Protocol (photos / product images)

When E1 needs a photo or creative:
1. **Ask the Boss first.** ("Boss, kailangan ko ng [specific shot] para sa [section].")
2. If the Boss has nothing to provide → **Employee 2 writes the image-generation prompt**: a ready-to-copy-paste ChatGPT prompt, precise about product placement, angle, lighting, background, aspect ratio, and text-free areas. The Boss pastes their reference product photo + the prompt into ChatGPT and returns the result.
3. E1 continues with a clearly-marked placeholder until the asset arrives; the placeholder is a tracked HIGH finding — E3 cannot pass the page while placeholders remain (unless Boss explicitly ships without).
4. Every prompt request states: which section it's for, exact dimensions/ratio, and file use (hero, gallery, what's-in-the-box, etc.).

---

## 6. Employee 1 Self-Checklist (before handing to E2)

- [ ] validate_page clean; published (not just patched); preview rendered at 1200 & 420 and personally inspected
- [ ] Every registry item in [references/mistake-registry.md](references/mistake-registry.md) explicitly verified
- [ ] All radio options map to real POS variation ids; combo values match displayed prices
- [ ] Popup recap shows the CHARGED price semantics, not pre-combo `{{total_price}}` (registry H1)
- [ ] No hardcoded fb_conversion_value (registry H2)
- [ ] SEO title/description/keywords/favicon are for THIS product (registry H3, M2)
- [ ] m.me button + FB logo links correct
- [ ] No duplicate elements, no leftover clone artifacts, no empty leftover tables
- [ ] Botcake flows named properly; sequence timings as approved
- [ ] Zero typos (spellcheck every text-block)
- [ ] Exactly one test order (unless the order path changed), its id recorded for E3; Boss told the pixel's real purchase count and the recommended optimization event (registry H5)

## 7. Employee 3 Final Gate (all must pass)

1. Full mistake-registry sweep — item by item, answered with evidence.
2. Visual QA both breakpoints — pixel-level: alignment, spacing, layering, radius, color tokens.
3. **Functional trace of the money path:** bundle click → radio value → form fields → submit → POS order (verify a test order's `total_price_after_sub_discount` equals the advertised price) → popup recap correct → pixel Purchase fired once (deduped) with correct value → Messenger ref opt-in works. E3 traces the test order E1 recorded and places a new one only if the order path changed since (and records it). After E3 passes, list each test order id in the report to the Boss as "pending cancel, OK?". Once the Boss says yes, E1 cancels it, re-reads the order, and confirms status 6 (canceled) in the final message (registry H5).
4. Cross-system consistency table: page price / combo value / popup / pixel value / actual POS charge — all equal per tier.
5. Copy + SEO + favicon + domain live check.
6. Only when the exit rule is met (zero CRITICAL/HIGH, every MEDIUM fixed or listed): present to Boss with a summary of what was built, cycles run, the verification evidence, and any known minor items.

---

## 8. Documentation

After Boss acceptance, write/update the product's audit-style doc in your knowledge-base folder (for example `knowledge-base/{product}/SYSTEM-AUDIT.md`): IDs, connections map, combo table, sequence table, pixel setup. Save any new non-obvious traps to the mistake registry so the next loop inherits them.
