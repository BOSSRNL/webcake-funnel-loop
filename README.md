# Webcake Funnel Loop

**A Claude Code skill that builds a complete e-commerce sales funnel on Webcake, then makes three AI "employees" prove the money path works before you see it.**

One request covers the whole funnel:

| Layer | Tool | What the skill does |
|---|---|---|
| Landing page | Webcake | Design system first, then the page, the order form, and the confirmation popup |
| Prices & orders | Pancake POS | Bundle tiers as value-combos; POS is the source of truth for what is charged |
| Messenger follow-up | Botcake | m.me entry flow plus a named, product-specific message sequence |
| Tracking | Meta Pixel + Conversions API | PageView / ViewContent / Purchase, browser and server, deduplicated |
| Domain | your DNS provider | Points the domain at Webcake and checks the live URL |

## The loop

```
You (the Boss) ask for a funnel or a change
        ↓
[Employee 1: Full Stack Developer]   builds it, publishes, renders at 1200 and 420, self-checks
        ↓          ↑ numbered findings, each with a severity
[Employee 2: Creative / Code Analyzer]  visual, copy, code, and data-consistency review
        ↓
[Employee 3: Team Leader]  strict final gate on a different model: traces the money path end to end
        ↓ zero CRITICAL/HIGH findings
You get it, with a per-tier consistency table as proof
```

Employee 3 does not pass a funnel until, for every bundle tier, these five numbers match: page price, POS combo value, popup recap, pixel value, and the actual POS charge on a real test order.

## Why it works

- **Built from real production bugs.** The mistake registry holds 30 traps found while auditing live funnels: a popup showing the pre-discount total, a conversion value hardcoded to one price, SEO keywords left over from a cloned page, Purchase-optimized ads launched on a pixel whose only purchases were test orders, and more. Repeating one is an automatic fail.
- **Prices live in POS, not on the page.** The skill reads what POS actually charged instead of trusting what the page shows.
- **Design is pass/fail.** Alignment, spacing, and layering are checked at both desktop and mobile widths; template-looking output bounces.
- **Safe with live systems.** One test order per launch, cancelled only with your OK, and the domain is confirmed with you before publishing.

## What's inside

```
skills/webcake-funnel-loop/
├── SKILL.md                     intake questions, build order, design standard, checklists, final gate
└── references/
    └── mistake-registry.md      30 never-again traps (money, quality, polish, platform behaviour)
```

## Requirements

- Claude Code
- MCP servers for **Webcake**, **Pancake POS**, and **Botcake**, plus a **Meta Ads** connector for pixel checks
- A Webcake account, a Pancake POS shop, a Botcake page, and a Meta pixel

The defaults target the Philippine market (PH mobile-number validation, country code 63, peso pricing, English or Taglish copy). Change them in `SKILL.md` for other markets.

## Install

1. Copy `skills/webcake-funnel-loop` into your Claude Code skills folder:
   - Windows: `C:\Users\<you>\.claude\skills\`
   - macOS / Linux: `~/.claude/skills/`
2. Restart Claude Code.
3. Ask Claude to build a Webcake funnel, or say **"webcake funnel loop"**.

## Related

For any other technical work (scripts, automations, APIs, web apps), see [claude-code-build-loop](https://github.com/BOSSRNL/claude-code-build-loop), the general-purpose version of the same loop.

## License

MIT. Use it, fork it, adapt it. A link back is appreciated.

---

Built by [Rannelle](https://github.com/BOSSRNL) · Builder of e-commerce funnels & AI automations 🇵🇭
