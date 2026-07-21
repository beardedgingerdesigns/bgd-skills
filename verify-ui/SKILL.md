---
name: verify-ui
description: Agent-side visual QA — verify your own UI work instead of asking Justin for screenshots. Drives Playwright against the repo's local dev server; logs in as each documented test role, screenshots key routes, captures console errors, then READS the screenshots and judges them. Use after any UI change before handing work back, when Justin says "verify your work", "check it yourself", "does it look right", or "/verify-ui", and in nightshift/background runs before closing a UI issue. Localhost/dev URLs only — never third-party dashboards.
bike-method-phase: 1  # Phase 1 — Training wheels. brandos only, run manually, until two clean weeks.
three-ms-attribution: |
  Adapted from The Three Ms of AI™ © 2026 Nate Herk.
---

# Verify UI

You are the render engine's eyes now. Before handing UI work back, prove it renders — don't make Justin screenshot it for you. (Origin: `retros/reflection-notes-2026-07-02` item #2 — ~60 screenshot relays across 5 repos in two months.)

## Contract — the repo must tell you (never guess)

Read the repo's CLAUDE.md local-dev section for four facts:

1. Dev start command + frontend URL
2. Test logins — role matrix with credentials
3. **Verify routes** — a small route set per role
4. Login flow type (password form vs magic link)

Any piece missing → **STOP** and print exactly what to add, e.g.:

```markdown
**Verify routes** (smoke set for /verify-ui):
| Role | Routes |
|---|---|
| dealer_admin | /portal/dashboard, /portal/brand |
```

Never invent credentials, routes, or selectors. (See memory: don't invent identifiers.)

## Procedure

1. **Health-check.** `curl -sf` the frontend URL. Down → start the dev command in the background and wait for ready. Port busy → report which pid holds it before killing anything.
2. **Script.** Reuse `scripts/verify-ui.mjs` if the repo has one. Else write it from the reference below, tailored to the app's real login form — read the login component for selectors, don't guess. Commit it once it works so later runs reuse it.
3. **Run.** For each role × route: log in, wait for network idle, full-page screenshot to `/tmp/<repo>/verify-ui-<timestamp>/`, collect console errors + failed requests into `findings.json`.
4. **Judge.** Read every screenshot with your own eyes. Hunt: blank/empty views, visible error text, broken layout, unreadable contrast (light-on-light, dark-on-dark), missing data where seed data should show. Cross-check against what your change was supposed to do.
5. **Fix and re-verify.** Max 2 fix loops. Still failing → hand back to Justin with the failing screenshot path and what you tried.
6. **Report.** One table: route × role → PASS/FAIL + one-line issue. Name the screenshot dir. Say "verified" only for what you actually looked at.

## Reference script (tailor per repo, then commit to that repo)

```js
// scripts/verify-ui.mjs — run: node scripts/verify-ui.mjs
import { chromium } from 'playwright';
import fs from 'node:fs';
import path from 'node:path';

const BASE = 'http://localhost:5173';                    // repo's frontend URL
const OUT = `/tmp/${path.basename(process.cwd())}/verify-ui-${Date.now()}`;
const MATRIX = [                                         // from CLAUDE.md dev logins + verify routes
  { role: 'dealer_admin', email: 'dealer@dealer.com', password: 'dealer',
    login: '/portal/login', routes: ['/portal/dashboard', '/portal/brand'] },
];

fs.mkdirSync(OUT, { recursive: true });
const browser = await chromium.launch();
const findings = [];
for (const m of MATRIX) {
  const page = await browser.newPage();
  const errors = [];
  page.on('console', msg => { if (msg.type() === 'error') errors.push(msg.text()); });
  page.on('requestfailed', r => errors.push(`REQUEST FAILED ${r.url()}`));
  await page.goto(BASE + m.login);
  // ponytail: selectors must match the real login form — read the component first
  await page.fill('input[type="email"]', m.email);
  await page.fill('input[type="password"]', m.password);
  await page.click('button[type="submit"]');
  await page.waitForLoadState('networkidle');
  for (const route of m.routes) {
    errors.length = 0;
    await page.goto(BASE + route, { waitUntil: 'networkidle' });
    const shot = `${OUT}/${m.role}--${route.replaceAll('/', '_')}.png`;
    await page.screenshot({ path: shot, fullPage: true });
    findings.push({ role: m.role, route, shot, consoleErrors: [...errors] });
  }
  await page.close();
}
await browser.close();
fs.writeFileSync(`${OUT}/findings.json`, JSON.stringify(findings, null, 2));
console.log(OUT);
```

## Rules

- **Localhost/dev URLs only.** Auth-walled third-party surfaces (Shopify admin, Cloudflare, Supabase dashboard, webmin) stay with Justin — that relay is by design.
- **Small route set.** 3-5 routes per role. This is smoke QA, not a crawl — image reads cost tokens.
- **Screenshots live in /tmp**, never the repo (self-cleaning, per the handoffs-in-tmp convention).
- **Console errors count as failures** even when the page looks fine.
- **Nightshift:** run this before closing any UI issue; paste the report table into the issue comment.
- **You judge "does it work / is it readable." Justin judges taste.** Don't claim design sign-off.
- **KPI** (retro Scoreboard): screenshot pastes/week in Justin's transcripts — baseline 30+, target <10 within two retros.
