# Framework conventions for `test-automate`

Per-framework conventions so generated specs read like idiomatic code
for that framework, not a generic tutorial. This is a **starting
point**, not team-reviewed guidance — the first real pilot run should
have its generated code reviewed by a senior against how that project's
engineers actually write tests (page-object patterns, fixture setup,
naming), and this file updated to match. Don't treat it as settled
convention until that review happens.

Covers the two frameworks named in the Phase 4 plan. Add a new section
here, following the same shape, before `test-automate` supports another
framework.

## Playwright

- **File location/naming** — `tests/<module>.spec.ts` (or `.spec.js` in
  a JS project), one spec file per module/feature area. If the project
  already has a `tests/`, `e2e/`, or `playwright/` directory with an
  existing naming pattern, follow that instead of introducing a new one.
- **Structure** — group related cases with `test.describe('<module>', () => { ... })`;
  one `test(...)` per case. Shared setup goes in `test.beforeEach`, not
  repeated per test.
- **Assertions** — use Playwright's built-in `expect` (`expect(locator).toBeVisible()`,
  `.toHaveText(...)`, `.toBeChecked()`, etc.) over manual boolean checks.
  Prefer role/text/label locators (`page.getByRole(...)`, `page.getByText(...)`)
  over CSS selectors when the case description gives enough to identify
  the element that way.
- **Fixtures** — use Playwright's `test.extend` fixtures for shared
  setup (e.g. an authenticated page) rather than duplicating
  login/setup steps across multiple spec files.
- **Page objects** — if the project already uses a page-object pattern
  (a `pages/` or `page-objects/` directory), generate against that
  convention rather than inlining locators. If no such pattern exists
  yet, inline locators are fine for v1 scaffolding.

Example skeleton:

```ts
import { test, expect } from '@playwright/test';

// Source: docs/test-suite/test-suite_2026-08-05.md — Checkout > "user
// can apply a valid discount code"
test.describe('Checkout', () => {
  test('applies a valid discount code', async ({ page }) => {
    await page.goto('/cart');
    await page.getByLabel('Discount code').fill('SAVE10');
    await page.getByRole('button', { name: 'Apply' }).click();
    await expect(page.getByText('10% off applied')).toBeVisible();
  });
});
```

## WebdriverIO

- **File location/naming** — `test/specs/<module>.spec.js` (or `.ts`),
  one spec file per module/feature area. If the project already has a
  `test/specs/` or `specs/` directory with an existing naming pattern,
  follow that instead.
- **Structure** — group related cases with `describe('<module>', () => { ... })`;
  one `it(...)` per case. Shared setup goes in `before`/`beforeEach`,
  not repeated per test.
- **Assertions** — use WebdriverIO's built-in `expect` (`expect(el).toBeDisplayed()`,
  `.toHaveText(...)`, `.toBeExisting()`, etc.) over manual boolean checks.
  Prefer `$('...')`/`$$('...')` selectors consistent with whatever
  selector strategy (CSS, accessibility id, etc.) the project already
  uses elsewhere.
- **Hooks** — use `before`/`beforeEach` for shared setup (e.g.
  navigating to a starting page or logging in) rather than duplicating
  it in every `it`.
- **Page objects** — if the project already uses a page-object pattern
  (a `pageobjects/` directory is WebdriverIO's own convention), generate
  against that convention rather than inlining selectors. If no such
  pattern exists yet, inline selectors are fine for v1 scaffolding.

Example skeleton:

```js
// Source: docs/test-suite/test-suite_2026-08-05.md — Checkout > "user
// can apply a valid discount code"
describe('Checkout', () => {
  it('applies a valid discount code', async () => {
    await browser.url('/cart');
    await $('#discount-code').setValue('SAVE10');
    await $('button=Apply').click();
    await expect($('=10% off applied')).toBeDisplayed();
  });
});
```
