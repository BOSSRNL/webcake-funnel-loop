# Mistake Registry — NEVER AGAIN list

Source: a full production audit of a live product funnel plus accumulated Webcake/POS platform traps. Employee 3 FAILS any output that repeats an item here. Employee 1 verifies every item BEFORE handoff; Employee 3 re-verifies with evidence.

## 🔴 HARD FAILS (shipped bugs that corrupted money/data)

| ID | Mistake | Rule going forward | How to verify |
|---|---|---|---|
| H1 | Confirmation popup used `{{total_price}}` — the PRE-combo figure. A bundle charged its discounted combo price, but the popup could show double that. | The popup recap must reflect the CHARGED amount. When value-combos are active, never surface raw `{{total_price}}` without verifying what it resolves to. **Mandatory: one live test order per launch; screenshot the popup; compare to POS `total_price_after_sub_discount`.** | Test order + POS field compare |
| H2 | `fb_conversion_value` hardcoded to the single-pack price on the child form — every browser Purchase reported that one value regardless of which bundle was bought. ROAS fed wrong data. | NEVER hardcode a conversion value on a multi-price form. Leave browser value empty/dynamic; Pancake server CAPI carries the true value; Meta dedups. If a value must be set, it must switch with the selected bundle. | Read the form element specials before publish |
| H3 | SEO `keywords` (and `document_colors`) left over from the cloned source page (the donor product's keywords on the new product's page). | After cloning ANY page: sweep ALL settings — title, description, keywords, favicon, thumbnail, document_colors, canonical — and rewrite for the current product. | Dump settings, grep for the donor product's name |
| H4 | Duplicate browser-side manual Purchase rule created in Events Manager alongside the form's Purchase → double counting risk. | One Purchase source per action. Check `ads_pixel_event_read` for existing rules before adding any; duplicates stay INACTIVE. Verify browser+server dedup to a single series after launch. | Pixel event read + 7-day stats WEB vs SERVER |
| H5 | Two new funnels launched Purchase-optimized campaigns while the only Purchases their brand-new pixels had ever received were launch test orders (already removed in POS). The campaigns spent almost nothing for a week. | A pixel with no real purchases does not get Purchase optimization: recommend ViewContent (or add an InitiateCheckout event) and switch once real purchases arrive. Keep test orders to the one H1 requires, record the id, and cancel it in POS after E3's check with the Boss's OK (`status: 6` + `status_name: "canceled"`, never `note`). Neither Test Events nor a POS cancel removes the Purchase from Meta. | The launch report states the pixel's real purchase count and the optimization event recommended; POS shows the test order canceled |

## 🟡 QUALITY FAILS

| ID | Mistake | Rule |
|---|---|---|
| M1 | Auto-events (e.g. SubscribedButtonClick) silently stopped firing after edits. | After any order-section edit, re-verify the tracked events still fire on the live page. |
| M2 | Favicon left empty. | Favicon is part of Phase B definition-of-done. |
| M3 | Sequence middle messages (2–4) had no hook → a fraction of the clicks message 1 got. | Every sequence message needs a reason-to-tap: promo, proof, demo, or urgency + CTA link. |
| M4 | m.me entry flow got dozens of sends and zero clicks. | The Messenger entry flow must itself contain a CTA worth tapping, not a dead-end greeting. |
| M5 | Four flows all auto-named "Sequence Message 6" — unmaintainable. | Name every flow `{PRODUCT} - {purpose}` at creation. |
| M6 | Combo weights inverted (the 3-pack weighed less than the single pack) → courier weight brackets wrong. | Variation weight = per-unit weight × quantity. Check all tiers ascending. |
| M7 | Stock hundreds of units below zero, selling only via `is_sell_negative_variation`. | Flag negative stock to the Boss at intake; it's a business decision, not silently OK. |
| M8 | Page record `custom_domain: null` while the domain worked — fragile binding. | After domain attach, confirm the binding survives a republish. |
| M9 | No email match key on any pixel event (EMQ capped ~7.8). | If the Boss approves an email field, add it; otherwise note the EMQ ceiling. |

## 🟢 POLISH FAILS (small, but E3 still bounces them)

| ID | Mistake | Rule |
|---|---|---|
| L1 | Typo shipped in a testimonial ("Ethusiast"). | Spellcheck every text-block. Proper nouns included. |
| L2 | Two byte-identical html-boxes stacked (duplicate hero banner). | Sweep for duplicate elements before publish. |
| L3 | Empty leftover `popTable` element in the popup. | Delete dead elements — no invisible cruft. |
| L4 | Donor page's palette entries left in `document_colors`. | Clean the palette to the locked design system. |
| L5 | Platform defaults left in a foreign language (VN livestream strings, VN tags). | Localize or note as intentionally unused. |

## ⚙️ PLATFORM TRAPS (Webcake/POS behavior — not bugs, but they bite)

| ID | Trap | Rule |
|---|---|---|
| P1 | `patch_page` saves source only — the live page does NOT update. | Always `publish_page` (dry_run then real) after edits; re-render preview after. |
| P2 | `add_section` validates at 960px regardless of page `width_section` and auto-moves elements. | Prefer patching full sections at the page's real width; re-check positions after any add_section. |
| P3 | `patch_page` silently drops oversized payloads. | Patch in chunks; verify the element actually changed after each patch. |
| P4 | POS value-combos, not the page, decide the charged price; Webcake re-derives radio options from combos on publish. | Prices live in POS. Read `total_price_after_sub_discount` for truth. Never edit page prices without checking combos first. |
| P5 | Buttons placed inside a form element submit the form. | Decorative buttons (tabs, image switchers) live OUTSIDE the form. |
| P6 | Inner divs at 150–200% width + hardcoded coords in click JS caused repeated alignment failures (selection panels). | Keep inner containers at 100%; never hardcode pixel coords in custom JS. |
| P7 | No-domain previews expire ~10 min after last publish. | Re-publish before re-checking a stale preview. |
| P8 | "What's in the box" style tab buttons only swap imagery — they don't set the bundle radio. | Either wire them to the radio too, or keep them visually distinct from the real selector so buyers aren't misled. |
| P9 | Global page JS lives in `settings.extra_script` (HTML-escaped string). `patch_page` only targets elements by id, so it can't reach `settings`; `update_page` needs the ENTIRE source (often 200KB+ for a full sales page) — too large for one call. | Have the Boss paste global JS into Webcake Settings → extra script, or attach it to an element (e.g. `onclick` in an `html-box`). The element route is fragile — editing that box in the editor can wipe it. Build the escaping with a script and assert the round-trip, never by hand. |
| P10 | `patch_page` with `dry_run:true` DID persist the change on a live page; the follow-up real commit then failed with `page_changed_externally` pointing at that same write. | Never assume a dry run was inert — re-read the page state after it. |
| P11 | `publish_page` sends `custom_domain:""` when the API reports no attached domain → no permanent public URL. | Don't publish blind. Confirm the domain with the Boss first, or let the Boss publish from the editor. |
