# TDD and BDD Test Cases for REQ-LOGIN-01

## Requirement under test

> **Requirement ID: REQ-LOGIN-01**
>
> *"Users must be able to log in using their registered email and password. Unregistered users should see an error. After three failed attempts, the system should lock the account."*

**Application context:** Banking application login system.

---

## 1. TDD test objective

A developer-facing test objective that validates the business logic for account lockout:

```
Prevent login after three failed attempts by locking the user account.
```

Breaking this down the way a developer would when writing the test first:

| Element | Description |
| --- | --- |
| **Input** | A registered user submits an incorrect password three consecutive times. |
| **Expected outcome** | On the third failure the account state changes to `LOCKED` and any further login attempt is rejected — even if the correct password is later supplied. |
| **Business rule** | REQ-LOGIN-01: "After three failed attempts, the system should lock the account." |

*What would fail if the logic was missing?* Without a failed-attempt counter and a lock state, the system would let an attacker keep guessing passwords indefinitely. The TDD test asserts that the counter increments on each failure and that the lock is enforced on attempt four.

---

## 2. BDD scenarios

```gherkin
Feature: User login for the banking application
  As a registered customer
  I want to log in securely with my email and password
  So that I can access my account while unauthorized access is blocked

  Scenario: Successful login with registered credentials
    Given I have a registered account
    And I enter my correct email and password
    When I click the "Login" button
    Then I should be taken to the dashboard

  Scenario: Login with an unregistered email
    Given I do not have an account
    And I enter an unregistered email and any password
    When I click the "Login" button
    Then I should see an error message: "User not found"

  Scenario: Account lockout after three failed login attempts
    Given I have a registered account
    And I enter an incorrect password three times
    When I attempt to login again
    Then my account should be locked
    And I should see an error message: "Account locked. Please reset your password."
```

---

## 3. Edge case considerations

Risks and coverage gaps the team should also test:

1. **Blank password field** — What happens if a user enters a valid email but leaves the password blank? The form should reject the submission with a validation message rather than counting it as a failed attempt.
2. **Page refresh mid-attempts** — What happens if the login page is refreshed after two failed attempts? The failed-attempt counter must persist server-side so the count is not reset by reloading the page.
3. **Brute-force / bot abuse** — Can bots abuse the login form for brute-force attacks? Consider rate limiting, CAPTCHA, and ensuring the lockout is keyed to the account (and/or IP) rather than the browser session.

Additional gaps worth tracking:

4. **Case sensitivity & whitespace** — Are leading/trailing spaces or mixed-case emails normalized before lookup?
5. **Lockout duration & recovery** — Is the lock permanent until password reset, or time-based? The error message implies reset is required; the recovery flow needs its own coverage.
6. **Correct password on a locked account** — A locked account must stay locked even when the right password is entered.

---

## 4. Analyst contribution summary

These test cases translate the REQ-LOGIN-01 business rules into concrete, verifiable expectations that developers can implement test-first and testers can execute consistently. The TDD objective pins down the account-lockout logic at the unit level, while the BDD scenarios document the user-visible behavior for both valid and invalid login flows in plain language the whole team can review. Together with the edge-case list, they reduce ambiguity, surface security risks early, and give the team reusable documentation for building reliable, secure login functionality.
