---
description: Stage 4 of the website pipeline. Verify a built site against DESIGN-BLUEPRINT.md — computed-style diff for the mechanical claims, screenshot reads for the visual ones. Fixes drift, re-verifies, reports.
disable-model-invocation: true
---

# /design-verify

The build stage promises the site matches the blueprint. This proves it. Two passes: a
**computed-style diff** for everything the blueprint states as a value (fonts, hex, spacing,
tokens), and a **screenshot read** for everything only eyes catch (broken layout, contrast,
a section that didn't render). Then a bounded fix loop.

Requires `DESIGN-BLUEPRINT.md` in the project (the reference) and a running or startable
dev server. If the blueprint is missing, tell the user to run `/design-extraction` — there's
nothing to verify against.

The blueprint's **Implementation Checklist** is the spine of this skill. It was written to
be checked. Most of "does it match" is fact-checking that list, not taste.

## Step 1 — Ground

Read `DESIGN-BLUEPRINT.md`. Pull the checkable values:
- Typography: heading + body font families, the h1 scale
- Color System: every hex and its role
- Design System Variables: the token names and values
- Spacing: content max-width, section padding, the scale
- Page Structure: the section list — this is your screenshot target list
- Implementation Checklist: the pass/fail items

Then find the dev server: read the project's start command (package.json scripts, CLAUDE.md,
or the build's own notes). `curl -sf` the URL. Down → start it in the background, wait for
ready. Port busy → report which pid holds it before touching anything.

Marketing sites are usually one page, no login. If the site *does* have auth, read the login
component for real selectors — never invent credentials or routes (memory: don't invent
identifiers).

**Completion:** blueprint values extracted, dev server reachable.

## Step 2 — Run the harness

Reuse `scripts/design-verify.mjs` if the project has one. Else write it from the reference
below. Before running, tailor the `PROBES` selector list to this site's real structure —
read the built markup for the actual hero, heading, body, and CTA selectors. Commit the
script once it works so later runs reuse it.

It captures, per viewport (mobile / tablet / desktop):
- A full-page screenshot
- `getComputedStyle` for each probe selector (font, size, weight, color, background, padding, radius)
- Console errors + failed requests

Output lands in `/tmp/<repo>/design-verify-<timestamp>/`: PNGs, `computed.json`, `findings.json`.

**Completion:** harness run, screenshots + computed.json written for all three viewports.

## Step 3 — Pass A: computed-style diff (mechanical, no eyes)

Read `computed.json`. Compare each value against the blueprint. This pass is deterministic —
you're fact-checking, not judging. Build the measurement table:

| Property | Blueprint | Implementation | |
|---|---|---|---|
| H1 font | `Cabinet Grotesk` | `Cabinet Grotesk` | ✅ |
| Accent | `#2E5E3A` | `#3B82F6` | ❌ generic blue, not the blueprint accent |
| Section padding | `96px` | `64px` | ⚠️ under spec |
| Body line-height | `1.6` | `1.6` | ✅ |

Flag every mismatch. A wrong hex or a fallback font is `❌` — those are the blueprint's whole
point. Off-by-a-step spacing is `⚠️`. Also run the greppable checklist items here: banned
palette hexes present, em-dashes in copy, `100vh` instead of `100dvh`, imports that don't
resolve, missing focus ring / alt text / meta / 404.

**Completion:** measurement table built, every mechanical checklist item marked.

## Step 4 — Pass B: screenshot read (visual, eyes required)

Read every screenshot. This is the residue Pass A can't see. Hunt:
- Blank / empty sections, or a section from the Page Structure that didn't render at all
- Visible error text, broken layout, overlap, content bleeding off-screen
- Unreadable contrast (light-on-light, dark-on-dark) — a hex can pass Pass A and still fail here in context
- Zigzag past 3 consecutive image+text splits; bento with dead cells; hero that doesn't fit the viewport
- The generic tells the blueprint bans: three equal cards, centered-everything, the AI-gradient look
- Mobile: does the desktop layout actually reflow, or just shrink

Cross-check each screenshot against what its section was supposed to be per the blueprint.

**The taste line still holds.** With a prescriptive blueprint most of "does it match" became
fact-checkable in Pass A, and Pass B catches the visual failures. But the genuinely
subjective call — is this *good*, not just *correct* — is still Justin's. Report what you
see; don't claim design sign-off.

**Completion:** every screenshot read, visual failures listed with the screenshot path.

## Step 5 — Fix and re-verify

Fix what failed, working with the existing stack. **One to two targeted changes per pass,
never a rewrite** — and don't undo a passing item to fix a failing one. Max 2 fix loops.
Re-run the harness after each. Still failing after 2 → hand back to Justin with the failing
screenshot path and what you tried.

For visual polish beyond conformance (motion feel, micro-detail), that's `/impeccable polish`
or `/emil-design-eng`, not this skill. This skill verifies the contract; it doesn't redesign.

**Completion:** all `❌` resolved or handed back, `⚠️` resolved or noted as accepted.

## Step 6 — Report

Two tables and a path:
- **Conformance** (Pass A): the measurement table, blueprint vs implementation
- **Render** (Pass B): section × viewport → PASS/FAIL + one-line issue
- The screenshot dir

Say "verified against the blueprint" only for what you actually checked. Name what you
couldn't (a section that needs real content, an asset still a placeholder). If Justin is
remote, the screenshot dir + tables are a clean `/remote-verify` payload for the taste call.

## Reference script (tailor PROBES per site, then commit)

```js
// scripts/design-verify.mjs — run: node scripts/design-verify.mjs [url]
import { chromium } from 'playwright';
import fs from 'node:fs';
import path from 'node:path';

const BASE = process.argv[2] || 'http://localhost:5173';
const OUT = `/tmp/${path.basename(process.cwd())}/design-verify-${Date.now()}`;
const VIEWPORTS = [
  { name: 'mobile',  width: 390,  height: 844 },
  { name: 'tablet',  width: 768,  height: 1024 },
  { name: 'desktop', width: 1440, height: 900 },
];
// ponytail: probe a sensible fixed set; tailor selectors to the built markup before running.
const PROBES = ['h1', 'h2', 'p', 'a[class*="cta"], button', 'section', ':root'];
const STYLE_PROPS = ['fontFamily', 'fontSize', 'fontWeight', 'lineHeight', 'letterSpacing',
  'color', 'backgroundColor', 'paddingTop', 'paddingBottom', 'maxWidth', 'borderRadius'];

fs.mkdirSync(OUT, { recursive: true });
const browser = await chromium.launch();
const findings = [], computed = [];
for (const vp of VIEWPORTS) {
  const page = await browser.newPage({ viewport: { width: vp.width, height: vp.height } });
  const errors = [];
  page.on('console', m => { if (m.type() === 'error') errors.push(m.text()); });
  page.on('requestfailed', r => errors.push(`REQUEST FAILED ${r.url()}`));
  await page.goto(BASE, { waitUntil: 'networkidle' });
  await page.screenshot({ path: `${OUT}/${vp.name}.png`, fullPage: true });
  const styles = await page.evaluate((args) => {
    const [probes, props] = args;
    const out = {};
    for (const sel of probes) {
      const el = document.querySelector(sel);
      if (!el) { out[sel] = null; continue; }
      const cs = getComputedStyle(el);
      out[sel] = Object.fromEntries(props.map(p => [p, cs[p]]));
    }
    return out;
  }, [PROBES, STYLE_PROPS]);
  computed.push({ viewport: vp.name, styles });
  findings.push({ viewport: vp.name, shot: `${OUT}/${vp.name}.png`, consoleErrors: [...errors] });
  await page.close();
}
await browser.close();
fs.writeFileSync(`${OUT}/computed.json`, JSON.stringify(computed, null, 2));
fs.writeFileSync(`${OUT}/findings.json`, JSON.stringify(findings, null, 2));
console.log(OUT);
```

## Rules

- **Localhost/dev URLs only.** Never point this at a live third-party surface.
- **Blueprint is the reference, not your taste.** You check conformance to a written contract. Where the blueprint is silent, don't invent a rule and fail the site on it — note the gap instead.
- **Screenshots live in /tmp**, never the repo (self-cleaning, per the handoffs-in-tmp convention).
- **Console errors count as failures** even when the page looks fine.
- **Two viewports minimum reflow check.** A site that only shrinks on mobile fails Pass B.
- **You verify the contract. Justin judges taste.** Don't claim design sign-off — hand the tables over for the subjective call.
