---
title: Test Retries
---

{% note info %}
# {% fa fa-graduation-cap %} What you'll learn

- What are test retries?
- Why is test retries important?
- How to configure test retries
{% endnote %}

# Introduction

End-to-end (E2E) tests excel at testing complex systems. However, there are still behaviors that are hard to verify and make tests flaky (i.e., unreliable) and fail sometimes due to unpredictable conditions (eg., temporary outages in external dependencies, random network errors, etc.).  Some other common race conditions that could result in unreliable tests include:

- Animations
- API calls
- Test server / database availability
- Resource dependencies availability
- Network issues

With test retries, Cypress is able to retry failed tests to help reduce test flakiness and continuous integration (CI) build failures. By doing so, this will save your team valuable time and resources so you can focus on what matters most to you.

# How It Works

By default, tests will not retry when they fail. You will need to {% urlHash "enable test retries in your configuration" Configure-Test-Retries %} to use this feature.

When test retries is on, tests will automatically be retried up to X additional times based on your desired configuration. For example, if test retries is set to `2`, Cypress will retry tests up to 2 additional times (for a total of 3 attempts) before potentially being marked as a failed test.

When each test is run again, the following {% url "hooks" writing-and-organizing-tests#Hooks %} will be re-run also:

- `beforeEach`
- `afterEach`

{% note warning %}
However, failures in `before` and `after` hooks will not trigger a retry.
{% endnote %}

**The following is a detailed step-by-step example of how test retries works:**

Assuming we have set test retries to `2` (for a total of 3 attempts), the following is how the tests would run.

1. A test runs for the first time. If the {% fa fa-check-circle green %} test passes, Cypress will move forward with any remaining tests as usual.

2. If the {% fa fa-times red %} test fails, Cypress will tell you that the first attempt failed and will to run the test a second time.

{% img /img/guides/test-retries/attempt-2-start.png %}

3. If the {% fa fa-check-circle green %} test passes after the second attempt, Cypress will continue with any remaining tests.

4. If the {% fa fa-times red %} test fails a second time, Cypress will make the final third attempt to re-run the test.

{% img /img/guides/test-retries/attempt-3-start.png %}

5. If the {% fa fa-times red %} test fails a third time, Cypress will mark the test as failed and then move on to run any remaining tests.

{% img /img/guides/test-retries/attempt-3-fail.png %}

The following is a screen capture of what test retries looks like on the same failed test when run via {% url "`cypress run`" command-line#cypress-run %}.

{% img /img/guides/test-retries/cli-error-message.png %}

During {% url "`cypress open`" command-line#cypress-open %} you will be able to see the number of attempts made in the {% url "Command Log" test-runner#Command-Log %} and expand each attempt for review and debugging if desired.

{% video local /img/guides/test-retries/attempt-expand-collapse-time-travel.mp4 %}

# Configure Test Retries

## Global Configuration

Typically you will want to define different retry attempts for `cypress run` versus `cypress open`. You can configure this in your {% url "configuration file" command-line#cypress-open-config-file-lt-config-file-gt %} (`cypress.json` by default) by passing the `retries` option an object with the following options:

- `runMode` allows you to define the number of test retries when running `cypress run`
- `openMode` allows you to define the number of test retries when running `cypress open`

```jsx
{
  "retries": {
    // Configure retries for `cypress run`
    // Default is 0
    "runMode": 2,
    // Configure retries for `cypress open`
    // Default is 0
    "openMode": 0
  }
}
```

### Configure specific retries for all modes

If you want to change the number of attempts being made for all tests run for both `cypress run` and `cypress open`, you can configure this in your {% url "configuration file" command-line#cypress-open-config-file-lt-config-file-gt %} (`cypress.json` by default) by defining the `retries` property and setting the desired number of retries.

```jsx
{
  "retries": 1
}
```

## Custom Configurations

### Individual Test(s)

If you want a specific test to run a custom number of retries, you can set this by using the {% url "test's configuration" writing-and-organizing-tests#Test-Configuration %}.

```jsx
// Customize retries for an individual test
describe('User sign-up and login', () => {
  // `it` test block with no custom configuration
  it('should redirect unauthenticated user to sign-in page', () => {
    // ...
  })

  // `it` test block with custom configuration
  it('allows user to login', {
    retries: {
      runMode: 2,
      openMode: 1
    }
  }, () => {
    // ...
  })
})
```

### Test Suite(s)

If you want a suite of tests to re-run a custom number of times, you can do this by setting the suite's configuration.

```jsx
// Customizing retries for a suite of tests
describe('User bank accounts', {
  retries: {
    runMode: 2,
    openMode: 1,
  }
}, () => {
  // Individual tests will be run normally
  it('allows a user to view their transactions, () => {
    // ...
  }

  it('allows a user to edit their transactions, () => {
    // ...
  }
})
```

You can find more information about custom configurations here: {% url "Test Configuration" configuration#Test-Configuration %}

# Screenshots

When a test retries, Cypress will continue to take screenshots for each failed attempt or {% url "`cy.screenshot()`" screenshot %} and suffix each new screenshot with `(attempt n)`, corresponding to the current test attempt number.

With the following test code, you would see the below screenshot filenames when all 3 attempts fail:

```js
describe('User Login', () => {
  it('displays login errors', () => {
    cy.visit('/')
    cy.screenshot()
    // ...
  })
})
```

```js
// screenshot filename from cy.screenshot() on 1st attempt
'User Login -- displays login errors.png'
// screenshot filename on 1st failed attempt
'User Login -- displays login errors (failed).png'
// screenshot filename from cy.screenshot() on 2nd attempt
'User Login -- displays login errors (attempt 2).png'
// screenshot filename on 2nd failed attempt
'User Login -- displays login errors (failed) (attempt 2).png'
// screenshot filename from cy.screenshot() on 3rd attempt
'User Login -- displays login errors (attempt 3).png'
// screenshot filename on 3rd failed attempt
'User Login -- displays login errors (failed) (attempt 3).png'
```

# Dashboard

If you are using the {% url "Cypress Dashboard" dashboard %}, information related to test retries is not currently shown. Showing this information and other analytics related to test retries is on our {% url "product roadmap" https://cypress-io.productboard.com/roadmap/1238172-product-roadmap %}.

# Tips and Strategies

While test retries are great for helping to avoid false negatives from failing an entire test run, it is not a good replacement for writing good tests. Here are some tips and strategies to keep in mind in order to maximize the effectiveness of your tests with test retries:

- If you are noticing that you need to increase the number of retries, the cause is more likely due to how the tests are written and is worth spending the time to investigate.

# Frequently Asked Questions (FAQs)

## Will retried tests be counted as more than one test recording in my billing?

No. Tests recorded during `cypress run` with the `--record` flag will be counted the same with or without test retries.

We consider each time the `it()` function is called to be a single test for billing purposes. The test retrying will not count as extra test recordings in your billing.

You can always see how many tests you've recorded from your organization's Billing & Usage page within the {% url "Dashboard" https://on.cypress.io/dashboard %}.