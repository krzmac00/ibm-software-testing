# Lab M1L2 — Auditing and annotating a login test plan

> **REQ-001** — "The system must allow valid users to log in using their email and password. Invalid login attempts must be rejected with a clear error message."

---

## Step 1 — Review of existing test cases

The sample plan contains five test cases (TC-001 … TC-005), all mapped to REQ-001. The table is reproduced below with an inline **🔎 Annotation** column capturing the systems professional's review notes (incomplete fields, missing scenarios, and edge cases).

| Test Case ID | Test Title | Objective | Preconditions | Test Steps | Expected Result | Priority | Review Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TC-001 | Valid login | Verify login succeeds for valid user credentials | User has valid account | 1. Enter valid email 2. Enter valid password 3. Click login | User is successfully logged in | High | ⚠️ **Expected result is vague.** "Successfully logged in" should be observable — e.g., *"User is redirected to the dashboard and a session is established."* No mention of the user *role* (student/admin/faculty), though all three are named in the scenario. |
| TC-002 | Invalid password | Verify login fails with incorrect password | User has valid account | 1. Enter valid email 2. Enter wrong password 3. Click login | Error: "Invalid username or password" | High | ✅ Solid negative case. ⚠️ Minor: message says *"username"* but REQ-001 logs in by **email** — wording is inconsistent with the requirement. Good that the message is generic (does not reveal whether the email exists). |
| TC-003 | Invalid email format | Verify login fails for improperly formatted email | None | 1. Enter malformed email 2. Enter any password 3. Click login | Error: "Invalid email format" | Medium | ⚠️ **Precondition "None" is incomplete** — should state the login page is loaded. Step is ambiguous: *"malformed email"* needs a concrete example (e.g., `user@@domain`, `userdomain.com`). |
| TC-004 | Blank fields | Verify login fails when email and/or password fields are blank | None | 1. Leave email and/or password blank 2. Click login | Error: "Email and password cannot be blank" | High | ⚠️ **One test bundles three scenarios** (email blank, password blank, both blank). Should be split or use a data table so each path has a deterministic expected result. Precondition incomplete. |
| TC-005 | Case sensitivity | Verify password field is case-sensitive | User has known password with case | 1. Enter valid email 2. Enter password in wrong case 3. Click login | Error: "Invalid username or password" | Medium | ✅ Good edge case. ⚠️ Does not also confirm the **email is case-*insensitive*** (the complementary rule), which is the more common real-world requirement. |

### General notes on test cases

- **Status / actual result / test data** columns are absent — there is no place to record execution outcomes.
- **Expected results** are mostly written as UI error strings but are not tied to *where* the message appears or to HTTP/security behavior.
- **Inconsistent error vocabulary:** "Invalid username or password" vs. an email-based system.

---

## Step 2 — Evaluate coverage (gap analysis against REQ-001)

REQ-001 has two halves: **(a)** valid users *can* log in, and **(b)** invalid attempts are *rejected with a clear error message*. Decomposing the requirement surfaces scenarios the draft does **not** cover:

| Aspect of REQ-001 | Covered by | Gap? |
| --- | --- | --- |
| Valid credentials succeed | TC-001 | ✅ Covered |
| Wrong password rejected | TC-002 | ✅ Covered |
| Malformed email rejected | TC-003 | ✅ Covered |
| Blank fields rejected | TC-004 | ✅ Covered (but bundled) |
| Password case sensitivity | TC-005 | ✅ Covered |
| **Unregistered / non-existent email** | — | ❌ **Missing** |
| **Account lockout after repeated failures** | — | ❌ **Missing** (security-critical) |
| **Expired / forced-reset password** | — | ❌ **Missing** |
| **SQL injection / malicious input** | — | ❌ **Missing** (edge case called out in instructions) |
| **Leading/trailing whitespace & email case-insensitivity** | — | ❌ **Missing** |

**Conclusion:** the draft validates the "happy path" plus basic negatives, but leaves **security and resilience gaps** (lockout, injection, expired credentials) that are essential for a system serving students, admins, and faculty.

---

## Step 3 — Suggested improvements (new test cases)

Four new test cases are added to close the highest-priority gaps identified above. TC-006 follows the example format from the instructions.

### TC-006 — Reject login with expired password

| **Field** | **Entry** |
| --- | --- |
| Test Case ID | TC-006 |
| Requirement ID | REQ-001 |
| Test title | Reject login with expired password |
| Objective | Verify that the system blocks users with expired passwords and displays the correct message |
| Preconditions | User account exists with an expired password; login page loaded |
| Test steps | 1. Enter email 2. Enter (correct but expired) password 3. Click login |
| Expected result | Login is blocked; system shows "Password expired. Please reset." and routes the user to the reset flow |
| Priority | Medium |
| Owner | QA Team |

### TC-007 — Account lockout after multiple failed attempts

| **Field** | **Entry** |
| --- | --- |
| Test Case ID | TC-007 |
| Requirement ID | REQ-001 |
| Test title | Lock account after repeated failed login attempts |
| Objective | Verify the system locks an account (or applies throttling) after a defined number of consecutive invalid attempts, to mitigate brute-force attacks |
| Preconditions | User has a valid account; lockout threshold configured (e.g., 5 attempts) |
| Test steps | 1. Enter valid email 2. Enter wrong password 3. Click login 4. Repeat until threshold is exceeded |
| Expected result | After the Nth failed attempt the account is locked; system shows "Account locked. Try again later or reset your password." and further attempts are rejected even with the correct password |
| Priority | High |
| Owner | QA Team |

### TC-008 — Reject login for unregistered email

| **Field** | **Entry** |
| --- | --- |
| Test Case ID | TC-008 |
| Requirement ID | REQ-001 |
| Test title | Reject login with non-existent account |
| Objective | Verify that an email not present in the system is rejected with a generic message that does not reveal whether the account exists (no user enumeration) |
| Preconditions | The email used is not registered; login page loaded |
| Test steps | 1. Enter an unregistered, well-formed email 2. Enter any password 3. Click login |
| Expected result | Error: "Invalid email or password" — identical to the wrong-password message so attackers cannot distinguish the two cases |
| Priority | High |
| Owner | QA Team |

### TC-009 — Reject SQL injection / malicious input

| **Field** | **Entry** |
| --- | --- |
| Test Case ID | TC-009 |
| Requirement ID | REQ-001 |
| Test title | Reject login with SQL injection attempt |
| Objective | Verify that malicious input in the email or password field is safely handled and does not bypass authentication or expose data |
| Preconditions | Login page loaded |
| Test steps | 1. Enter `' OR '1'='1` in the email field 2. Enter `' OR '1'='1` in the password field 3. Click login |
| Expected result | Login is rejected with the standard invalid-credentials error; no authentication bypass, no SQL error leaked to the UI, input is sanitized/parameterized |
| Priority | High |
| Owner | QA Team |

---

## Step 4 — Updated traceability matrix

The original matrix maps only TC-001 … TC-005. Rows for the new cases are appended so **every** scenario derived from REQ-001 is traceable to a test.

| **Requirement ID** | **Requirement description** | **Test case ID** | **Test case description** | **Status** |
| --- | --- | --- | --- | --- |
| REQ-001 | Login and error handling logic | TC-001 | Valid login | Not started |
| REQ-001 | Login and error handling logic | TC-002 | Invalid password | Not started |
| REQ-001 | Login and error handling logic | TC-003 | Invalid email format | Not started |
| REQ-001 | Login and error handling logic | TC-004 | Blank fields | Not started |
| REQ-001 | Login and error handling logic | TC-005 | Case sensitivity | Not started |
| REQ-001 | Login and error handling logic | TC-006 | Reject login with expired password | Not started |
| REQ-001 | Login and error handling logic | TC-007 | Account lockout after multiple failed attempts | Not started |
| REQ-001 | Login and error handling logic | TC-008 | Reject login for unregistered email | Not started |
| REQ-001 | Login and error handling logic | TC-009 | Reject SQL injection attempt | Not started |

> **Note on matrix quality:** every row currently reuses the same generic requirement description ("Login and error handling logic"). For stronger traceability, REQ-001 should be split into sub-requirements (REQ-001a *valid login*, REQ-001b *clear rejection of invalid attempts*) so the matrix shows which half of the requirement each test exercises.

---

## Systems professional notes

### What I improved
- Added an **annotation against each existing test case**, flagging vague expected results (TC-001), requirement-wording inconsistencies ("username" vs. email in TC-002/TC-005), incomplete preconditions (TC-003/TC-004), and a bundled multi-scenario case (TC-004).
- Strengthened coverage with **4 new test cases** (TC-006 expired password, TC-007 account lockout, TC-008 unregistered email, TC-009 SQL injection).
- Extended the **traceability matrix** so all nine test cases map back to REQ-001.

### Gaps I discovered
- The draft covered the happy path and basic negatives but **omitted security and resilience scenarios**: brute-force lockout, user-enumeration protection, expired credentials, and injection handling.
- The test plan **lacks execution-tracking fields** (test data, actual result, status, pass/fail) — it documents intent but cannot record outcomes.
- **Error-message vocabulary is inconsistent** and, in places, leaks information that could aid an attacker (distinguishing "wrong password" from "no such account").

### Top two recommendations for strengthening QA readiness
1. **Decompose REQ-001 into atomic, testable sub-requirements** and adopt a one-scenario-per-test-case rule (split TC-004). Atomic requirements + atomic tests make the traceability matrix meaningful and expose gaps automatically.
2. **Add a security & negative-path test suite as a release gate** — lockout/throttling, user-enumeration resistance, injection/sanitization, and expired/forced-reset flows — and standardize on a single generic error message ("Invalid email or password") to avoid information disclosure.
