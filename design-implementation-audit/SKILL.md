---
name: "design-implementation-audit"
description: "Audit a live website against its approved Figma designs (desktop + mobile, AR/EN), log evidence-based findings with severity and owner, deliver a report, and re-verify after fixes."
---

# Design-to-Implementation Audit

Use when the user asks to review/QA a live (or staging) website against its Figma designs, or to check whether a developer implemented the design correctly. Works for any site and any stack (WordPress, Shopify, Webflow, custom) because it inspects the rendered page. Has an extra module for bilingual Arabic/English (RTL/LTR) sites, and light checks for content, forms, performance, SEO, accessibility and security.

Review **every part** of every page in scope. "It works" is not enough, and "the code looks right" is not enough: each element must look as designed (checked by eye on screenshots AND by measurement), behave as designed, and lead where it should.

## 0. Scope (confirm before starting — ask only what is missing)
- Live URL (or staging) and the Figma file URL (including the prototype, for animations/hover in 4U).
- Confirm the user is authorized by the site owner to audit it (needed for form test submissions in 4N and the security check in 4L).
- Viewports. Default: desktop **1440**, mobile **390**. Tablet (1024×768) is responsive-only: check it doesn't break, don't audit it pixel-by-pixel unless asked.
- Languages (e.g. AR + EN) and which one is primary.
- Design rules to audit against (read from the Figma file or ask): fonts, type scale, grid (e.g. 12 col / 24 gutter / 80 margin / 1280 container desktop; 4 col / 16 margin mobile), colors, spacing scale, icon set.
- Source of truth for contact data: official phone numbers, WhatsApp, email, address, map location.
- Source of truth for partners/brands/clients: official names (AR + EN) and logo files.
- Where form submissions should arrive (email inbox / CRM / WhatsApp) and who can confirm they arrived.

## 0.1 Product data is placeholder by default
While a site is still being built, products are usually dummy/placeholder. **Treat all product data as placeholder unless the user says it is real.**
- Do not log product names, prices, specs, stock, images or counts as wrong. Note in the report: "Product data treated as placeholder."
- Still audit everything the products pass through: product card and detail templates, long-name wrapping, missing-image fallback, filters, sort, search, pagination, related products.
- Test logic against whatever data exists: searching the exact name of a placeholder product must find it; filtering by a brand that has placeholder products must return them; sorting must reorder them.
- If the user says the data is real, also check data accuracy and log it for the Data/Import owner.
- NOT placeholder, always checked: contact data (4D), partners/brands (4E), and the site's fixed copy (4M).

## 0.2 Audit Integrity & Evidence Gates
These rules apply to every audit phase and override any instruction that could lead to unsupported claims.

**A. Verify tool capability before committing to scope**
- Confirm access to the Figma file, required frames, live website, browser inspection, screenshots, and measurement capabilities before promising a complete audit.
- If a tool, permission, or source is unavailable, state the limitation and identify the affected checks.
- Never claim that a page was visually inspected, measured, interacted with, or tested unless that action actually occurred.
- Distinguish between verified observations, inferences, recommendations, and untested assumptions.

**B. Track actual coverage**
Maintain a coverage ledger throughout the audit:
- Pages and frames mapped, reviewed, and verified.
- Sections visually inspected and measured.
- Interactive elements actually tested.
- Languages, viewports, and devices covered.
- Checks blocked, skipped, or not applicable, with reasons.

The final report must distinguish planned scope from completed coverage. Never describe the entire site as audited when only a subset was reviewed.

**C. Evidence is mandatory for every finding**
Every finding must include:
- A unique ID.
- The specific Figma reference or approved requirement.
- Live-site evidence showing the observed behavior or appearance.
- A measurement, interaction result, or other verifiable observation where applicable.
- Confidence: Confirmed, Needs confirmation, or Inferred.

Do not invent Figma measurements, browser results, console errors, tool outputs, or screenshots. If evidence is insufficient, record the issue as needing confirmation rather than presenting it as a confirmed defect.

**D. Separate implementation defects from design questions**
Classify each discrepancy as one of:
- Confirmed deviation from the approved design.
- Confirmed functional or usability defect.
- Content or data discrepancy.
- Potentially intentional implementation difference requiring clarification.
- Design ambiguity or missing specification.

Do not automatically treat every pixel difference as a defect. Consider the approved responsive behavior, content variation, browser rendering, and documented exceptions. Ask for clarification when the intended behavior cannot be established reliably.

**E. Protect against duplicate findings**
Group repeated symptoms under one finding when they share a confirmed root cause, listing all affected pages and instances. Keep separate findings when the cause, fix, severity, or responsible owner differs. Do not merge distinct issues merely because they occur in the same component.

## 1. Build the page ↔ frame inventory
- List top-level frames in Figma (`get_metadata` on the UI page): desktop/mobile × AR/EN × page.
- Map each frame to a live URL. Output a table: `Page | Figma frame (node id) | Viewport | Language | Live URL`.
- Show the table to the user and confirm before auditing. The number of mapped frames is the planned scope; the coverage ledger (0.2B) records how many were actually reviewed.

## 2. Get the expected values from Figma
- For each frame, read sections and components (`get_design_context` / `get_metadata` / `get_screenshot`).
- Note what matters: font family/size/weight/line-height, field and button heights, container width, section order, shared components (header, breadcrumb, page header card, footer, floating CTAs), default states, icons and their style, alignment edges, link destinations, hover/pressed states and prototype interactions.
- **Spacing spec per section**: section padding (top/bottom/left/right) and every internal gap from auto-layout (`itemSpacing`, `padding*`) or by measuring between layers: title → subtitle, subtitle → content, grid row/column gap, inside cards (image → title → text → button), section → next section.
- Check whether Figma has designs for 404, empty states, loading states and form success/error; if not, note "not designed" (a missing specification per 0.2D) and judge them against the site's style.
- Export a Figma screenshot of every frame and section at the same width as the live viewport.
- Never guess a Figma value. If unclear, mark "needs confirmation".

## 3. Capture the live site
- Use the browser (built-in browser or Claude in Chrome) at each viewport and language, with the browser width set exactly to the audited viewport.
- Full-page screenshot per page, plus a close, zoomed crop of every section. Look at the crops yourself — don't rely on the DOM alone.
- Read computed styles (font-size, font-family, line-height, letter-spacing, word-spacing, text-align, height, padding, margin, gap, direction) and positions with `getBoundingClientRect()`.
- Read the console and network requests (needed for 4H, 4Q, 4L).
- Click and test every interactive element. Record what actually happened, and update the coverage ledger as you go.

## 4. Compare section by section — checklist
Severities named in this section are defaults; final severity follows section 10.

**A. Visual inspection (by eye + by measurement) — mandatory for every section**
Code can be "correct" while the page looks wrong. For every section:
1. *Side-by-side look.* Figma screenshot and live screenshot of the same section side by side at the same width (or overlaid at ~50%). Look for shifted elements, ragged edges, uneven gaps, cramped/loose text, broken rhythm.
2. *Measure what you see* with `getBoundingClientRect()` and log exact px.

Check:
- **Alignment of siblings**: checkboxes/radios start at the same x; labels, fields, buttons line up; card contents line up across a row; icons line up. Report the offset ("3rd checkbox is 12px right of the others").
- **Text block alignment**: paragraphs/headings/lists in a column share the same start edge (right in RTL, left in LTR); text-align matches Figma.
- **Text spacing**: line-height, paragraph gaps, word spacing. Flag huge/uneven word gaps (often `justify` on Arabic, double spaces, non-breaking spaces, wrong `word-spacing`/`letter-spacing`), tight/loose lines, orphans.
- **Equal sizes**: cards in a row, buttons in a group, inputs in a form.
- **Typography, colors, states, image ratios and cropping.**

**B. Section spacing vs. Figma (inside every section)**
Table per section: `Section | Spacing | Figma px | Live px | Diff`.
- Section padding matches and is consistent between similar sections across pages.
- Internal gaps (title → subtitle → content → CTA), grid row/column gaps, spacing inside cards — match Figma and are the same in every instance.
- Same kind of gap = same value everywhere.
- Repeat on mobile with the mobile frame's values.

**Tolerance** for 4A/4B (unless the project defines its own): ≤2px ok; 3–7px P3; ≥8px or inconsistent rhythm P2; content looks broken/hard to read P1. Before logging, rule out approved responsive behavior, content variation and browser rendering differences (0.2D).
Evidence: Figma crop + live crop with the problem marked + measured values.

Helper — misaligned siblings:
```js
(sel => { const side=document.dir==='rtl'?'right':'left';
  const els=[...document.querySelectorAll(sel)]; const xs=els.map(e=>Math.round(e.getBoundingClientRect()[side]));
  const cnt={}; xs.forEach(x=>cnt[x]=(cnt[x]||0)+1); const mode=+Object.keys(cnt).sort((a,b)=>cnt[b]-cnt[a])[0];
  return xs.map((x,i)=>({i,edge:x,off:x-mode})).filter(r=>Math.abs(r.off)>2);
})('SELECTOR')
```
Helper — vertical gaps between children of a section:
```js
(sel => { const kids=[...document.querySelector(sel).children].filter(e=>e.offsetHeight);
  return kids.slice(1).map((e,i)=>({from:kids[i].className||kids[i].tagName,to:e.className||e.tagName,gap:Math.round(e.getBoundingClientRect().top-kids[i].getBoundingClientRect().bottom)}));
})('SECTION_SELECTOR')
```

**C. Icons (every icon)**
Present where Figma has one, none extra; right meaning and same icon set; consistent style/size/stroke/color/spacing across siblings (e.g. every Finder tab has its icon); aligned with text; renders correctly (no broken SVG, empty box, wrong color, blur); RTL: directional icons flip, non-directional don't.

**D. Contact data — every occurrence**
Table: `Page | Language | Location | Displayed text | Link target | Correct?` (header, top bar, footer, contact page, cards, floating WhatsApp, product CTAs, popups; both languages and viewports).
Displayed number matches the official one digit by digit; `tel:`/`wa.me`/`mailto:` matches the displayed text; correct international format (`tel:+9665XXXXXXXX`, `wa.me/9665XXXXXXXX`); same everywhere, AR = EN; Arabic renders digits in the right order; email, address, map correct. Wrong phone/WhatsApp/email = **P1**. Without an official source to compare against, record as Needs confirmation.

**E. Partners / brands / clients — every logo**
Table: `Page | Language | Section | Position | Name shown (text/alt/title) | Logo shown | Expected name | Expected logo | Link target | Status`.
Name spelled exactly as official (AR + EN); each name with its own logo (not swapped/shifted), right `alt`/tooltip; links to its own page in the same language; image correct version, sharp, not stretched/cropped, transparent where designed, correct color treatment; consistent sizes and gaps; same set and order as Figma, nothing missing/duplicated, AR = EN; carousel shows all. Wrong name or swapped logo = **P1**; blurry/stretched/inconsistent = P2.

**F. Links and CTAs (every one)**
Table: `Source page | Language | Element | Expected destination | Actual URL | Same language? | Status`.
No 404/500 or dead `#`; goes to its own intended page; same language always (except the language switcher); switcher opens the equivalent page; logo → current-language home; external links correct, new tab as designed; anchors and breadcrumbs correct. Any failure is a finding even if it "opens something".

**G. Search (real queries, both languages)**
Exact placeholder product name → found; partial name, brand, size string (e.g. `315/80R22.5`), category; AR query on AR site and EN on EN, results in the same language; no-match → designed empty state; header + listing search; Enter and button; query persists; works with filters/sort; result cards match design and link correctly.

**H. Other functional behavior**
Filters (default state, each works, multi-select, reset), Finder/tabs, sort actually reorders (e.g. Newest-first), pagination keeps filters/sort, accordions/FAQ, sliders (controls, autoplay, titles on every slide), popups. Console: no JavaScript errors on any page. (Forms: 4N.)

**I. Shared components & layout**
Header, nav, breadcrumb, page header card, footer, floating WhatsApp/CTA on every page that has them in Figma; section order; legal pages styled; templates correct.

**J. Responsive behavior (mobile 390, tablet sanity check)**
Repeat 4A and 4B on mobile. Type scale, hero, nav drawer, filter/sort controls, grid columns, pagination, stacking order, touch targets, no horizontal scroll, partner logos readable. (Real devices: 4P.)

**K. Product IA and Bilingual/RTL**
Product attribute order, related products, specs table, labels and units. Real RTL semantics (`dir=rtl`), Arabic font and line-height, no unwanted justify gaps, mixed AR/EN strings in the right order, AR and EN have the same sections/components/links, no untranslated strings.

**L. Security and tech stack (light, passive, non-intrusive)**
Only with the owner's authorization. Surface hygiene check, **not a penetration test**: normal page loads and reading public responses only — no attack payloads, login attempts, brute force, aggressive scanners or load testing.
- **Tech stack**: CMS/platform, theme, page builder, main plugins/libraries and versions, server/CDN, exposed languages (e.g. PHP via `X-Powered-By`), from source, headers and asset paths. Report as "detected" with evidence.
- **HTTPS**: valid certificate, HTTP → HTTPS redirect, no mixed content.
- **Security headers**: HSTS, CSP, `X-Frame-Options`/`frame-ancestors`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`.
- **Information exposure**: public version numbers (generator tag, `?ver=`, `readme.html`), files that shouldn't be public (`/readme.html`, `/license.txt`, directory listing on uploads), user names exposed by a public endpoint (e.g. `/wp-json/wp/v2/users`) — one normal GET, then stop.
- **Known vulnerabilities**: for each detected component+version, search public advisories (WPScan, NVD/CVE, vendor changelog); report version, advisory, fixed version, source link. Never test the vulnerability. A version match alone is Inferred, not Confirmed exploitable.
- **Forms**: HTTPS, spam protection, no sensitive data in URLs. **External scripts**: list third-party scripts; flag unexpected ones.
Severity: outdated component with a public critical/high advisory, no HTTPS, exposed user list = P1; missing headers, exposed versions = P2; minor hygiene = P3. Owner: Development/Hosting. Always state: "Light passive security check; not a penetration test — a professional security assessment is recommended before launch."

**M. Content and copy (fixed site text, AR + EN)**
All fixed copy: headings, paragraphs, buttons, menus, footer, labels, placeholders, validation/error messages, legal pages, alt text, page titles. Product data is excluded (0.1).
- Spelling and grammar in Arabic (الهمزات، التاء المربوطة والهاء، الياء والألف المقصورة, spacing around punctuation) and English (spelling, capitalization consistency, grammar).
- Leftovers: Lorem ipsum, "Sample text", theme demo content, test pages, "Hello world" posts, default widget text.
- Copy matches Figma (or the approved content); same terms for the same thing everywhere (one name per category, one CTA wording).
- Copyright year current; company name spelled consistently.
- Arabic and English versions say the same thing (no missing sentences on one side).
Leftover demo/lorem content visible = P1; typos in headings/CTAs/nav = P2; minor typos in body = P3. Owner: Content.

**N. Forms end-to-end**
For every form (contact, quote request, newsletter, lead popup), in both languages:
- Fields, labels, placeholders, required markers as designed; field sizes per 4A.
- Validation: empty required fields, invalid email, invalid phone (accepts the local formats it should, e.g. 05XXXXXXXX and +9665XXXXXXXX); messages in the page's language, in the right place.
- Submit one test entry per form with the user's or site owner's OK, clearly marked (name "TEST – Audit", message "Audit test, please ignore"). Check: designed success message in the right language, form resets or redirects as designed, no duplicate submission on double-click.
- **Delivery**: ask the owner or the user to confirm the test entry arrived at the right inbox/CRM/WhatsApp with all fields correct; record who confirmed and when. Until confirmed, status is "Delivery unverified".
- Error state when submission fails (if testable without harm).
Form not delivering or no success feedback = **P1**.

**O. 404, empty, loading and error states**
- Visit a non-existent URL in each language: designed 404 page, in the right language, with header/footer and a way back (home link, search).
- Empty states: no search results, filter combination with no products, empty category — designed message, not blank space.
- Loading: skeletons/spinners where content loads; no flash of unstyled content or wrong font.
- Broken image fallback: a missing image shows a designed placeholder, not a broken-image icon or overflowing alt text.
Default theme 404 or blank empty states = P2.

**P. Real devices and browsers**
- Test the mobile pages on a real iPhone (Safari) and a real Android phone (Chrome), plus desktop Chrome, Safari or Edge, and Firefox. If no real device is reachable, use browser emulation, say so in the report, mark the real-device check incomplete in the coverage ledger, and give the user a short checklist to run on their own phone.
- On real devices check: Arabic font loads (iOS often falls back), text sizes, safe areas/notch, sticky header and floating WhatsApp not covering content, keyboard not hiding form fields or the submit button, input zoom on focus (inputs < 16px on iOS), tap targets, slider swipe, `tel:`/WhatsApp links open the right app.
- Log device/browser/OS version on every finding from this section.

**Q. Performance (as it affects the experience)**
- Run PageSpeed Insights / Lighthouse for the home page, a listing page and a product page, mobile and desktop, both languages. Record Performance score, LCP, CLS, INP/TBT.
- From network requests: oversized images (e.g. > 300 KB, or far larger than displayed size), images not WebP/AVIF, no lazy-loading below the fold, too many fonts/weights, render-blocking scripts, total page weight.
- Layout shift while loading (content jumping as images/fonts load) — check by eye too.
Targets: LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms. Mobile LCP > 4s or obvious layout jumps = P1; outside targets = P2. Owner: Development/Hosting. Scores vary per run: run twice and record the date.

**R. SEO basics**
Per page and language:
- Unique `<title>` and meta description in the page's language, not the theme default.
- One `<h1>` per page, logical heading order.
- `hreflang` linking each AR page to its EN equivalent and back (plus `x-default` if used); `lang` and `dir` correct.
- Canonical correct (not pointing to the other language or to staging).
- `sitemap.xml` exists and lists both languages; `robots.txt`/noindex doesn't block the site (common staging leftover: WordPress "Discourage search engines").
- Clean URLs per language; no staging/dev URLs in links.
- Images have meaningful `alt`.
Site blocked from indexing, or canonical/hreflang pointing to the wrong language = P1. Owner: Development/Content.

**S. Share preview and favicon**
- Open Graph and Twitter tags on home, main pages and product pages: `og:title`, `og:description`, `og:image` (≈ 1200×630, correct brand image, not a random theme image), `og:url`, in the page's language.
- Check how a link previews on WhatsApp and Facebook (a preview/debugger tool or pasting it in a chat): right title, image, description, language.
- Favicon and touch icons present, the client's logo, sharp on high-DPI and on a phone home screen; browser tab title correct.
Missing/wrong share image or default favicon = P2 (P1 if the business relies on WhatsApp sharing).

**T. Accessibility basics**
- Contrast: body text ≥ 4.5:1, large text and UI parts ≥ 3:1 (check text on images and colored sections).
- Images: meaningful `alt`; decorative images empty `alt`; logos carry the company name.
- Keyboard: Tab through header, menu, filters, forms and popups — visible focus ring, logical order, popups close with Esc, no keyboard trap.
- Forms: every field has a label (not placeholder only); errors shown near the field.
- Icon-only buttons have accessible names (`aria-label` for WhatsApp, search, close, menu).
- `lang="ar"` / `lang="en"` set correctly.
Unreadable contrast on key content or forms unusable by keyboard = P1; other gaps = P2/P3.

**U. Animations and hover vs. the prototype**
- Compare with the Figma prototype and component variants: hover, pressed, focus and active states on buttons, cards, links, menu items, logos (e.g. grayscale → color).
- Transitions and animations: type (fade/slide), direction (mirrored correctly in RTL), duration and easing close to the prototype; no janky or missing animations; no animation that hides content until it fires.
- Dropdowns, mega menus, drawers, accordions, popups open/close with the designed motion.
- Respect `prefers-reduced-motion` where animations are heavy.
Missing or wrong state = P2; timing differences = P3.

## 5. Log each finding (one row per finding)
Fields:
- `ID` — e.g. `AUD-PRODUCTS-M-AR-007` (page, viewport, language, number); `AUD-SEC-001`, `AUD-SEO-001`, `AUD-PERF-001` for site-wide items. IDs never change across rounds.
- `Page` / `Viewport` / `Language` / `Device-Browser` (for 4P) / `Section` / `Element` / all affected instances.
- `Category` — Visual Consistency, Alignment & Spacing, Section Spacing, Icons, Contact Data, Partners/Brands, Links, Search, Functional, Forms, States (404/empty/loading), Shared Components & Layout, Responsive, Real Devices, Product IA, Bilingual/RTL, Content & Copy, Performance, SEO, Share Preview, Accessibility, Motion & States, Security & Tech.
- `Classification` (0.2D) — Confirmed design deviation / Confirmed functional or usability defect / Content or data discrepancy / Possibly intentional — needs clarification / Design ambiguity or missing spec.
- `Confidence` (0.2C) — Confirmed / Needs confirmation / Inferred.
- `Severity` — P1 / P2 / P3 per section 10, or Needs confirmation when evidence is insufficient. Add the rationale for every P1.
- `Expected` — Figma value + node id/frame name, or approved requirement / official data / recommended setting / target value.
- `Observed` — live value + marked screenshot crop, measured px / DOM value, URL, response header, tool score, or device screenshot.
- `Owner` — Design, Development, Content, Data/Import, or Hosting. Keep visual, content, data and technical lists separate so each owner gets the right list.
- `Type` — Implementation gap vs. Content update.
- `Proposed fix` — concrete ("set section padding-top to 96px per Figma"; "swap logos 3 and 4"; "change tel: to +9665XXXXXXXX"; "convert hero image to WebP ≤ 200 KB"; "add hreflang pairs"; "untick 'Discourage search engines'"; "add aria-label='WhatsApp'"; "update plugin X to version Z").
- `Verification` — how to confirm it's fixed.
- `Status` — Open / Fixed / Partially fixed / Still open / Blocked / Not re-tested / Won't fix / Delivery unverified (forms).
Grouping follows 0.2E: one finding per confirmed root cause; separate findings when cause, fix, severity or owner differ.

## 6. Summarize
- Planned scope vs. completed coverage, from the coverage ledger: frames mapped / reviewed / verified, sections inspected and measured, links tested, contact occurrences checked, partner logos checked, search queries tested, forms tested (and delivery confirmed), languages, viewports and devices covered, checks blocked/skipped/not applicable with reasons.
- Findings: totals, by severity, category, classification and confidence.
- Tech stack detected, security summary, performance scores (with date), SEO status — or "incomplete" with the reason.
- The P1 list on its own, each with evidence and rationale.
- Open questions blocking fixes (missing copy, unconfirmed phone number, missing partner list, form delivery not yet confirmed, design ambiguities, no real device available) in a separate list.

## 7. Deliverables
- Audit report (doc): scope, method, coverage (planned vs. completed), notes ("product data treated as placeholder", "not a penetration test", devices real or emulated), summary numbers, findings by category, P1 section, technical section (stack, security, performance, SEO), clarification questions, appendix of evidence (Figma crop + marked live crop side by side) named by finding ID.
- Findings tracker (sheet) with the fields above, filterable by owner, severity, classification and confidence.
- Separate tabs: coverage ledger, section spacing, links, contact data, partners, search tests, forms, devices, performance, SEO, security.
- If mobile has many findings, a separate Mobile Implementation To-Do.
- Ask the user's approval before sending anything to the client or developer.

## 8. Re-verification after fixes
- Re-check every finding at the same viewport/language/device, by eye and by measurement; re-run the spacing, links, contact, partners, search, forms (one new test submission + delivery check), performance, SEO and security tables (fixes often break other things).
- Capture an "after" screenshot for each fixed finding, same crop as the original.
- Report: fixed / partially fixed / still open / blocked / not re-tested, with the remaining list for the next round.

## 8.1 Re-verification Integrity
After fixes, use the original finding IDs and re-check each issue against its original expected behavior and evidence.

For each finding, record:
- Original status and evidence.
- Fix claimed or reported by the developer.
- Actual post-fix observation.
- Re-test conditions: URL, viewport, language, and device/browser where relevant.
- Final status: Fixed, Partially fixed, Still open, Blocked, or Not re-tested.

A developer's statement that a fix is complete is not proof of successful verification.

Only mark a finding Fixed after confirming the relevant visual or functional result. If direct verification is unavailable, use Not re-tested or Blocked and explain why.

For case-study use, capture Before and After evidence under comparable conditions. Never present a design mockup as a live-site After screenshot, or imply that an unverified change was successfully delivered.

## 9. Audit Execution Strategy
- Preserve the agreed scope and all mandatory checks within that scope.
- Execute in stages: scope and access, Figma inventory, baseline capture, visual and functional audit, technical checks when authorized, reporting, and post-fix verification.
- Prioritize checks that block core user journeys, produce incorrect business information, or prevent reliable comparison.
- Do not silently skip checks to save time. Record deferred or blocked work in the coverage ledger.
- Do not run optional performance, SEO, accessibility, security, or real-device checks unless the required tools and access are available; when these checks are in scope, report them as incomplete if they cannot be performed.
- Before expanding the agreed scope or submitting forms, obtain the required authorization.
- Never modify the website, Figma file, production data, or configuration during an audit unless the user explicitly authorizes a separate implementation task.

## 10. Severity Calibration
Assign severity based on user impact, business risk, and evidence—not merely the size of a visual difference.
- **P1**: A verified issue breaks a core user journey, displays materially incorrect business information, prevents a critical submission, or creates a comparably serious launch risk (e.g. search/filter not working, link opens wrong page/language, wrong contact data, wrong partner name or swapped logo, form not delivering, demo/lorem content visible, site blocked from indexing, critical security exposure).
- **P2**: A verified issue materially degrades usability, consistency, accessibility, or important functionality without meeting P1 criteria (e.g. ≥8px misalignment, spacing not matching Figma, broken text spacing, blurry/stretched logos, missing/inconsistent icons, typos in headings/CTAs, default 404, missing share image, missing hover states, missing security headers, performance outside targets).
- **P3**: A verified, low-impact discrepancy or polish issue (e.g. 3–7px offsets, minor typos, animation timing, minor hygiene).
- **Needs confirmation**: The evidence is insufficient to assign a reliable severity. This is a verification state, not a substitute priority.

Use the existing project-specific severity rules where they are more appropriate, and document the rationale for P1 findings. Do not assign severity to an unverified assumption as though it were a confirmed defect.

## Rules
- Section 0.2 overrides everything else: no claim without an action that actually happened, and no finding without evidence.
- Evidence for every finding: screenshot crop, measured px / DOM value, URL, header or tool result, plus the Figma reference or official source.
- Always look at the screenshots yourself; a page that passes the DOM checks can still look wrong.
- Report what you observed; label anything uncertain, inferred or emulated.
- Product data is placeholder unless the user says otherwise; contact data, partner names/logos and fixed copy are always real and always checked.
- Form test submissions only with approval, clearly marked as tests, one per form per round.
- Security checks are passive and authorized only; never exploit or attack.
- Don't change the Figma file, the site, production data or configuration during the audit.