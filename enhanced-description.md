Overview
View Playwright test results from a local file or GitHub repository

Playwright Test Results Badge is a lightweight Chrome extension that helps you quickly see your Playwright test status directly from your browser toolbar.

Instead of opening CI dashboards or scrolling through logs, you get an instant visual summary of your test results that stays visible and automatically updated.

This extension is ideal for:

• 👨‍💻 Developers
• 🧪 QA engineers
• 🎓 Students learning testing or CI/CD
• 📄 Anyone using Playwright with automated tests

Why install this extension?

• ✅ Instantly know if your tests passed or failed
• ⏱️ Saves time by avoiding CI dashboards and logs
• 🔄 Automatically refreshes every 1 minute
• 🧑‍🎓 Beginner-friendly setup (great for students)
• 🔒 Privacy-first: no accounts, no tracking, no servers

What the extension does

Live badge on the Chrome toolbar:

• 🟩 Green 42 / 0 means all tests passed
• 🟥 Red 41 / 1 means one or more tests failed
• ⬜ Gray ? means no tests detected (crash, syntax error, or empty summary file)

This lets you see test health at a glance, even while browsing other sites.

Detailed popup view:

• 📊 Total number of tests
• ✅ Passed tests
• ❌ Failed tests
• ⚠️ Flaky tests
• 🕒 Last updated timestamp
• 🔄 Manual refresh button

Everything is shown in a simple, easy-to-read format.

Automatic refresh:

• ⏲️ The extension checks your test summary file every 1 minute in the background
• 🔄 You can also refresh manually at any time

Quick Setup Guide (3 files needed)

Step 1: Create the Summary Reporter

Create a file called "summary-reporter.js" in your project root with this exact code:

const fs = require('fs');

class SummaryReporter {
  onBegin(config, suite) {
    this.rootSuite = suite;
  }

  onEnd(result) {
    const summary = {
      schemaVersion: 1,
      passed: 0,
      failed: 0,
      flaky: 0,
      total: 0,
      startTime: new Date().toISOString(),
      isSummary: true
    };

    if (this.rootSuite) {
      for (const test of this.rootSuite.allTests()) {
        const status = test.outcome();
        if (status === 'expected') summary.passed++;
        if (status === 'unexpected') summary.failed++;
        if (status === 'flaky') summary.flaky++;
      }
    }

    summary.total = summary.passed + summary.failed + summary.flaky;
    fs.writeFileSync('test-summary.json', JSON.stringify(summary, null, 2));
    console.log('✅ Test summary created: test-summary.json');
  }
}

module.exports = SummaryReporter;

Step 2: Configure Playwright

Update your "playwright.config.ts" to include the reporter. Add this line to your reporters array:

reporters: [
  ['./summary-reporter.js'],  // Add this line
  ['html'],                   // Keep your existing reporters
  ['list']                    // Keep your existing reporters
],

Step 3: Create GitHub Actions Workflow

Create ".github/workflows/playwright.yml" with this exact content:

name: Playwright Tests
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

permissions:
  contents: write

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: lts/*
    - name: Install dependencies
      run: npm ci
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    - name: Run Playwright tests
      run: npx playwright test
    - name: Update test results
      if: always()
      run: |
        git config user.name "Actions Bot"
        git config user.email "actions@github.com"
        git add test-summary.json
        git diff --staged --quiet || git commit -m "chore: update test results [skip ci]"
        git push
    - uses: actions/upload-artifact@v4
      if: ${{ !cancelled() }}
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30

Step 4: Test Your Setup

1. Commit and push the three files above
2. Check GitHub Actions - your workflow should run automatically
3. Find the test-summary.json file in your repository
4. Click "Raw" to get the direct URL
5. Copy the URL and paste it into the Chrome extension

That's it! Your badge will now update automatically every time you push code.

How to use it (detailed workflow)

Step 1 — Generate a test summary file

Instead of publishing the full Playwright test report, generate a small and safe summary JSON file.

This summary file contains only test counts and a timestamp:
• passed - Number of successful tests
• failed - Number of failed tests  
• flaky - Number of flaky tests
• total - Total number of tests
• startTime - When tests were last run
• isSummary - Marker for the extension (always true)

No stack traces, no file paths, no emails, and no sensitive data are included.

Step 2 — Configure Playwright to create the summary

The custom reporter above automatically generates the summary file after every test run. The extension looks for the "isSummary: true" marker to confirm it's reading the right file format.

Step 3 — Add a CI workflow (recommended)

The GitHub Actions workflow above will:
• Run Playwright tests on every push or pull request
• Generate the summary JSON file using your custom reporter
• Commit and push only the summary file to the repository
• Use "[skip ci]" to prevent infinite workflow loops

Step 4 — Copy the raw summary file URL

After the CI workflow runs:
• Open the "test-summary.json" file in your repository
• Click "Raw" to get the direct file URL
• Copy this URL

GitHub UI links are also supported and automatically converted to raw links.

Step 5 — Connect the extension

• 🧩 Click the extension icon in Chrome
• 📋 Paste the summary JSON URL
• 💾 Click Save
• ⚡ The badge updates immediately and refreshes every minute

Important CI/CD notes:

• 📄 The extension only displays what exists in the summary file
• ⚠️ If tests crash or the summary file is empty, the badge shows a gray ?
• 🔁 The workflow generates the summary file even if tests fail (using "if: always()")

Works with CI tools such as:

• ⚙️ GitHub Actions (template provided above)
• ⚙️ GitLab CI
• ⚙️ Jenkins
• ⚙️ CircleCI
• ⚙️ Any CI system that can generate and host a public JSON file

Privacy and security:

• 🔐 No accounts
• 🚫 No tracking
• 📉 No analytics
• 🌍 No external servers

The extension fetches the summary JSON file directly in your browser.

Permissions explained:

• 🗂️ storage — saves your configured summary file URL
• ⏰ alarms — auto-refresh polling
• 📁 file access (optional) — required only for local file usage

No other permissions are requested.

Beginner friendly:

Perfect for:
• 🧑‍🎓 High school students learning testing
• 🆕 Beginners learning Playwright
• 🚀 First-time CI/CD users
• 👨‍🏫 Teachers demonstrating test automation

Typical use cases:

• 👨‍💻 Check test status while coding
• 🔍 Monitor CI results during pull request reviews
• 🧑‍🏫 Teach Playwright testing in class
• 📘 Learn how test reporting works
• 👀 Keep test health visible during development

Troubleshooting:

Badge shows gray ?
- Check if "test-summary.json" exists in your repository
- Verify the JSON contains "isSummary: true"
- Make sure the URL points to the raw file

Tests not updating?
- Confirm GitHub Actions workflow is running
- Check that the workflow has "contents: write" permission
- Verify the reporter is configured in "playwright.config.ts"

Need help?
The extension works with any JSON file containing the required fields. You can even create the summary manually for testing purposes.