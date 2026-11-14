# Playwright Tests — CP (Comboios de Portugal) Train Booking

[![Playwright Tests](https://github.com/nicknorm007/playwright-tests/actions/workflows/playwright.yml/badge.svg)](https://github.com/nicknorm007/playwright-tests/actions/workflows/playwright.yml)

End-to-end test automation suite built with **Playwright and TypeScript** against the
[CP — Comboios de Portugal](https://www.cp.pt/passageiros/en/) (Portuguese national rail)
ticket-booking site.

This is an independent project, built and maintained by
[Nick Norman](https://github.com/nicknorm007) (Senior Software Engineer in Test). It was
written as a working evaluation of a live booking flow: automate the core purchase path,
structure the suite the way a production framework would be structured, and document the
testability problems found in the site along the way (see
[Testability observations](#testability-observations)).

## What is tested

| Test | Coverage |
| --- | --- |
| Verify buy tickets page title | The Buy Tickets page loads with the expected title. |
| Enter sample train times and destinations and verify cancel preserves form data | Data-driven booking flow: enter origin/destination and travel dates, submit, then cancel — and verify the form retains all four entered values (origin, destination, departure date, return date). |

Current data set: **Lagos → Porto–Campanhã**, departing +3 days out, returning +5 days out.
Dates are generated relative to today at runtime, so the suite never goes stale from
hard-coded dates.

## Design and patterns

- **Page-object modules** — locators and actions for each page live together in
  `tests/pages/` (`home.ts`, `buyTickets.ts`), kept as typed, immutable (`as const`)
  objects. Specs describe behaviour; selectors live in exactly one place.
- **Data-driven cases** — destinations are declared as data
  (`{ from, to, start_in_days, end_in_days }`) and the spec generates a test per row, so
  adding a route is a one-line change.
- **Relative-date utility** — `tests/common/dateUtils.ts` produces future travel dates in
  the exact format the site expects (e.g. `20 October, 2023`), offset by a configurable
  number of days.
- **Assertions on state, not just navigation** — the booking test verifies that cancelling
  after submit preserves the user's entered data, which is the behaviour a traveller
  actually depends on.

## Tech stack

- [Playwright Test](https://playwright.dev) (`@playwright/test` ^1.39)
- TypeScript
- Chromium (Desktop Chrome project)
- GitHub Actions for CI

## Getting started

**Prerequisites:** Node.js 18 or later (20 LTS recommended) and npm.

```bash
git clone https://github.com/nicknorm007/playwright-tests.git
cd playwright-tests
npm ci                        # install dependencies from the lockfile
npx playwright install chromium
```

**Run the tests:**

```bash
npx playwright test                 # headless run
npx playwright test --headed        # watch the browser drive the booking flow
npx playwright test --ui            # Playwright UI mode (debug / time-travel)
npx playwright show-report          # open the HTML report from the last run
```

Sample output:

```text
Running 2 tests using 1 worker

  ✓  1 [chromium] › tests/tickets/buyTickets.spec.ts › Verify buy tickets page title
  ✓  2 [chromium] › tests/tickets/buyTickets.spec.ts › Enter sample train times and
        destinations and verify cancel preserves form data

  2 passed
```

## Project structure

```text
playwright-tests/
├── .github/workflows/
│   └── playwright.yml      # CI: install, run tests, publish HTML report
├── tests/
│   ├── common/
│   │   └── dateUtils.ts    # relative future-date generation in the site's format
│   ├── pages/
│   │   ├── home.ts         # base URL and home page constants
│   │   └── buyTickets.ts   # Buy Tickets page locators and actions (page object)
│   └── tickets/
│       └── buyTickets.spec.ts
├── playwright.config.ts    # Chromium / Desktop Chrome project
├── package.json
└── package-lock.json
```

## Continuous integration

The GitHub Actions workflow in `.github/workflows/playwright.yml`:

- runs on every **push** and **pull request** to `main`, on a **weekly schedule**, and on
  demand via **Run workflow** (`workflow_dispatch`);
- installs dependencies with `npm ci`, installs the Chromium browser, and runs the suite
  on Ubuntu;
- uploads the **Playwright HTML report** as a build artifact (kept for 30 days) whether
  the run passes or fails, so failures can be triaged from the report.

> **Note:** these tests exercise a live third-party production website. If CP changes the
> page structure or form fields, the selectors in `tests/pages/buyTickets.ts` will need a
> refresh — the weekly scheduled run is there to surface that kind of breakage early.

## Testability observations

Notes recorded while automating the site — the kind of feedback a QE would hand back to
the development team:

- Many controls lack `id` attributes and proper CSS selectors, which forces brittle,
  attribute-based locators.
- Some submit controls are `<input>` elements rather than semantic `<button>` elements.
- Date fields are unforgiving of freehand entry; the expected format has to match exactly.

## Possible next steps

- Add data rows for additional routes and passenger scenarios.
- Add Firefox and WebKit projects alongside Chromium.
- Cover the search-results step of the booking flow, beyond form submission.
- Introduce Playwright fixtures to replace the module-level page objects.
