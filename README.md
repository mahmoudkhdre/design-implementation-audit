# Claude UX Audit Skill: Design-to-Implementation Audit

[العربية](README.ar.md)

A Claude Skill that guides a structured audit of a live website against its approved Figma designs. It covers desktop and mobile, with an extra module for bilingual Arabic/English (RTL/LTR) sites. Every finding is logged with evidence, a severity and an owner, and the same finding IDs are re-checked after fixes.

The skill came out of a real project, the bilingual corporate website for Al Wafaa Tires. The first audit there, of the Arabic site, was done by hand. Its repeatable steps were then written down as this skill, which was used for the audit of the live English site.

## Who it's for

- UI/UX and product designers who hand designs to a developer and need to check what was built.
- Front-end developers who want a self-review checklist before handover.
- Design QA reviewers working on bilingual or RTL websites.

## The problem it addresses

Comparing a live site with Figma by hand is slow, and it's easy to be inconsistent from page to page: some sections get measured and others get eyeballed, findings are written in different shapes, and a visual bug, a content mistake and a data problem end up mixed in one list. This skill fixes the order of work and the shape of each finding, so the review is repeatable and each issue reaches the person who can fix it.

## What it does

The skill is one instruction file, [`design-implementation-audit/SKILL.md`](design-implementation-audit/SKILL.md). It tells Claude to:

1. **Confirm scope first.** Live URL, Figma file, viewports (default 1440 desktop and 390 mobile), languages, design rules (fonts, grid, spacing), the official source for contact data and partner names, and whether the user is authorized to audit the site.
2. **Build a page-to-frame inventory.** Map each Figma frame to its live URL and confirm the table before auditing.
3. **Read expected values from Figma.** Typography, spacing, components, states and link destinations. Unclear values are marked "needs confirmation" rather than guessed.
4. **Capture the live site** at the same viewport widths: screenshots, computed styles, measured positions, console and network.
5. **Compare section by section** against a checklist: visual inspection by eye and by measurement, section spacing, icons, contact data, partner logos, links, search, filters and other functions, shared components, responsive behavior, bilingual/RTL, content and copy, forms end to end, 404 and empty states, real devices, performance, SEO basics, share previews, accessibility basics, motion and hover states, and a light passive security and tech-stack check.
6. **Log each finding** with an ID, classification, confidence, severity (P1/P2/P3), expected vs. observed, owner, proposed fix, verification step and status.
7. **Summarize** planned scope against completed coverage, and deliver a report and tracker.
8. **Re-verify after fixes** using the original finding IDs. A developer saying something is fixed doesn't count as verification.

## Inputs it needs

- The live (or staging) website URL.
- The Figma file, with the frames for each page and viewport.
- The project's design rules, if they aren't readable from Figma.
- Official contact details and partner/brand names to check against.
- Permission from the site owner for form test submissions and the passive security check.

## What it gives you

- A page-to-frame inventory and coverage ledger.
- Evidence-based findings, grouped by root cause, with severity and owner.
- An audit report and a findings tracker, with visual, content, data and technical issues kept apart.
- A re-verification record after fixes.

## What it does not do

- It doesn't fix the website, and it doesn't change the Figma file, production data or configuration.
- It isn't a penetration test. The security part is passive only: normal page loads and reading public responses. A professional security assessment is still recommended before launch.
- It doesn't guarantee that every issue is found. Coverage depends on the pages, viewports and tools available, and the report records what was actually reviewed.
- It doesn't replace design judgment. Findings marked "needs confirmation", design intent, severity calls and anything sent to a client or developer still need a person to decide.

## Why human review stays in the loop

The skill builds in checkpoints: confirm scope, confirm the page-to-frame inventory, approve form test submissions and the security check, and approve the report before it goes to anyone. Its integrity rules also forbid claiming a check that didn't happen. In practice, some checks will be blocked (a browser that times out, no real device available). The skill records those as incomplete instead of estimating them, and a reviewer should read the coverage ledger before trusting the totals.

## Prerequisites

The skill is written for a Claude environment that has:

- **Figma access** through a Figma connector (MCP), for reading frames, metadata and screenshots. The skill refers to tools such as `get_metadata`, `get_design_context` and `get_screenshot`.
- **A browser tool** that can open pages at a set width, take screenshots, run JavaScript in the page and read console and network output.
- Optionally, PageSpeed Insights or Lighthouse for the performance section.

**Tested setup:** the skill has been used in the Claude app with a Figma connector and a browser tool. Other setups, such as Claude Code or the API, have not been tested by the author.

## Installation

1. Download or clone this repository.
2. Add the `design-implementation-audit` folder, which contains `SKILL.md`, to your Claude environment as a custom skill. Where custom skills go depends on the product you use; see Anthropic's documentation on Agent Skills at docs.claude.com.
3. Make sure the Figma connector and a browser tool are available in the same session.

## Usage

Ask Claude for an audit and give it the essentials, for example:

> Audit https://example.com against the Figma file https://www.figma.com/design/FILE_KEY. Desktop 1440 and mobile 390, Arabic and English. Arabic is primary. Official phone: +20 1X XXXX XXXX.

Claude should first ask for anything missing from the scope list, then show you the page-to-frame inventory to confirm before it starts.

## Example finding format

This is the shape of a finding the skill produces (illustrative values):

| Field | Example |
|---|---|
| ID | `AUD-CONTACT-M-AR-004` |
| Category | Visual Consistency |
| Classification | Confirmed design deviation |
| Confidence | Confirmed |
| Severity | P2 |
| Expected | Form fields 48px high (Figma frame `Contact-mobile`, node id) |
| Observed | Fields render 40px high (measured with getBoundingClientRect at 390px) |
| Owner | Development |
| Proposed fix | Set input height to 48px per Figma |
| Verification | Re-measure field heights at 390px |
| Status | Open |

## Customizing

The defaults (viewports, grid, severity thresholds, phone formats) are starting points. Edit `SKILL.md` to match your own project rules. The skill already says that project-specific severity rules take precedence where they fit better.

## Origin

Developed by Mahmoud Khdre (UI/UX & Product Designer) from the Figma-to-live-site audit process on the Al Wafaa Tires corporate website. Client-specific data, audit evidence and findings are not included in this repository.

## Contributing

Contributions and additions to the skill are welcome. Open an issue to suggest an improvement or report a problem, or send a pull request with your changes.

## License

[MIT](LICENSE) © 2026 Mahmoud Khdre
