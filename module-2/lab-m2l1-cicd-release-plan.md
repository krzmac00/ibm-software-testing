# Lab M2L1 — Release plan with CI/CD workflows

> **System under release:** Online Course Management System
> **Pipeline:** Jenkins + GitHub Actions → build → test → staging → UAT → production
> **Features in this release:**
> 1. **Course Bookmarking** — let learners save courses to revisit later
> 2. **Weekly Engagement Email Digest** — a scheduled summary email of learner activity
>
> **My role:** Systems Professional. I do **not** modify the CI/CD config or write code. I **map, document, and coordinate** the business logic around the pipeline so the release is safe, visible, and aligned with product goals.

---

## Step-by-step analysis (walkthrough)

### Step 1 — Understand the pipeline stages

| Stage | What happens | Inputs → Outputs | Automated? | Human review still needed? |
| --- | --- | --- | --- | --- |
| Code commit | Developers push feature branches for Bookmarking and Email Digest to GitHub | Source code → triggered pipeline run | ✅ Trigger is automatic | Code review / PR approval before merge |
| Build | Source is compiled and dependencies resolved/checked | Source + dependency manifest → build artifact | ✅ Fully automated | Only if the build breaks |
| Automated unit test | Tests confirm core logic behaves as expected (e.g., a bookmark persists, a digest is composed) | Build artifact → pass/fail report + coverage | ✅ Fully automated | Review of failures and coverage gaps |
| Staging deploy | The build is deployed to a staging server that mirrors production for preview/testing | Artifact → running staging environment | ✅ Deploy is automated | QA spot checks; SP reviews logs |
| UAT review | Key business workflows are exercised manually before the production push | Staging build → UAT sign-off (go / no-go) | ❌ **Manual gate** | **Yes — this is the primary human checkpoint** |
| Production deploy | Pipeline pushes the approved build to the live environment and alerts teams | Approved build → live release + notifications | ✅ Automated once gated | Monitoring confirmation post-deploy |

**Where human review still matters:** automation handles *correctness of the code*, but it cannot judge *fitness for the business*. A digest email may be technically "passing" while sending at the wrong time, with the wrong tone, or to opted-out learners. The UAT gate and the staging review are where human judgment closes that gap.

### Step 2 — Each stage in systems-professional-friendly language

- **Code commit** — *"Developers have finished a piece of the feature and submitted it."* Who cares: SP tracks which features/commits are in flight. SP visibility: **light** (track scope, not internals).
- **Build** — *"The computer assembles all the pieces into something runnable and checks nothing is missing."* Who cares: DevOps/QA. SP visibility: **low** unless it fails repeatedly (signals instability).
- **Automated unit test** — *"Automatic checks confirm the basic behavior works before anyone looks at it."* Who cares: QA and SP (failure tracking). SP visibility: **medium** — failure trends predict release risk.
- **Staging deploy** — *"A safe preview copy of the app, with the new features, that looks like the real thing but isn't live to learners."* Who cares: QA, Product Owner, SP. SP visibility: **high** — this is where I verify business workflows end to end.
- **UAT review** — *"Real people walk through the actual learner journeys and decide if it's good enough to launch."* Who cares: Product Owner, QA, SP. SP visibility: **high** — I coordinate it and prepare the summary of what's new.
- **Production deploy** — *"The features go live for real learners, and everyone is told."* Who cares: all teams. SP visibility: **high** — I confirm release notes, monitoring, and KPIs are in place.

### Step 3 — Business / QA involvement

- **When to loop in Product Owners & QA:** QA engages from the **automated test** stage (reviewing failures) and intensively at **staging**. Product Owners engage at **UAT**, where they sign off on go/no-go.
- **Is a UAT environment needed?** **Yes.** Staging doubles as the UAT environment so business workflows — *bookmark a course, see it in "Saved Courses," receive a correctly formatted weekly digest* — are validated against a production-like setup before launch.
- **Are release notes required before production?** **Yes.** The product team explicitly requires visibility and release notes prior to launch. Release notes must describe both features, any opt-out/privacy considerations for the email digest, and the rollout/monitoring plan.

### Step 4 — Automation failure scenarios

- **If tests fail in staging:** the pipeline halts before UAT/production — failing builds are never promoted. The failure is logged and surfaced to the responsible team.
- **Who is alerted:** developers and DevOps via the CI/CD alert channel (Slack/email); QA and the SP are notified for tracking and to hold the release schedule.
- **The SP's role in response planning:** I do not fix the code, but I **assess business impact and coordinate** — confirm whether the failure blocks the release, communicate revised timing to the Product Owner, ensure the failure is triaged (not ignored), and re-open the UAT/release checklist once a fix passes. I also watch for **alert fatigue** so genuine failures aren't lost in noise.

---

## Task 1 — CI/CD stages in plain language

| **Pipeline stage** | **Description (in plain language)** | **Who needs to be informed?** |
| --- | --- | --- |
| Code commit | Developers push new Bookmarking and Email Digest code to GitHub | Systems professional tracks features and commits |
| Build | Code is compiled and dependencies are checked into a runnable artifact | DevOps / QA |
| Automated unit test | Tests check that core code behaves as expected (a bookmark saves, a digest is generated) | QA; SP for tracking failures |
| Staging deploy | App is deployed to a staging server for preview/testing against production-like data | QA, Product Owner, SP |
| UAT review | Key business workflows are manually reviewed before the production push | SP coordinates UAT; Product Owner signs off |
| Production deploy | CI/CD pipeline pushes the approved code to the live environment | All teams alerted via Slack/email |

---

## Task 2 — Systems professional's checkpoints

As the systems professional, I review the pipeline at three points. First, I watch the **automated test** results across runs — not to debug code, but to spot failure trends that signal the release is unstable and to keep failures from being silently ignored. Second, at the **staging deploy** I review deployment logs for failed API responses tied to business workflows (e.g., a bookmark that doesn't persist, or a digest job that errors), and I prepare a plain-language summary of *what's new* for the Product Owner ahead of UAT. Third, after **UAT passes** I confirm the release notes are complete and shared, verify that monitoring and KPI dashboards are in place for the two new features, and only then allow the production gate to open. My checkpoints exist to ensure that "technically passing" also means "ready for the business and visible to the people who own it."

---

## Task 3 — Three CI/CD risks and mitigation strategies

| **Risk** | **Why it matters** | **Mitigation (systems professional role)** |
| --- | --- | --- |
| Feature deploys before UAT approval | Learners may see an unfinished bookmark feature or receive a malformed/early digest email | Keep the production step **blocked until the UAT task is explicitly checked off**; SP owns that gate |
| Alert fatigue from test/pipeline failures | Repeated noisy failures train teams to ignore alerts, so a critical failure slips through | Work with DevOps to define **threshold alerts** (e.g., escalate only after N retries) and route genuine failures to the right owner |
| Metrics not tracked in production | No way to tell if learners actually use bookmarks or whether the digest is opened vs. flagged as spam | SP reviews **KPI dashboards pre-release** and requires monitoring for both features before the production gate opens |

*(Bonus risk specific to this release — the **Email Digest** sends to learners. If opt-out/consent state isn't validated in UAT, the release risks emailing users who opted out. Mitigation: add an explicit "consent/opt-out respected" item to the UAT checklist.)*

---

## Task 4 — Pre-production checklist (systems-professional-owned)

Final checklist I own before CI/CD is allowed to deploy to production:

- [ ] Confirm staging pass **+** manual QA spot checks (bookmark persists; digest renders correctly)
- [ ] Product Owner **UAT sign-off** recorded (go/no-go)
- [ ] **Release summary doc** complete and shared (both features, opt-out/privacy notes for the digest)
- [ ] **Alert thresholds** reviewed with DevOps (no alert fatigue; real failures escalate)
- [ ] **Monitoring dashboard** updated with new-feature KPIs (bookmark usage, digest open/spam rates)
- [ ] **Email-consent/opt-out** behavior verified for the Weekly Engagement Digest
- [ ] **Rollback / hotfix path** confirmed with DevOps in case post-deploy monitoring flags a problem

---

## Systems professional notes

### How release automation aligns with business goals
The CI/CD pipeline lets the Online Course Management System ship Bookmarking and the Email Digest **quickly and repeatably**, while the SP-owned gates ensure speed never outruns quality or business intent. Automation guarantees the code *works*; the human checkpoints (staging review, UAT sign-off, release notes, monitoring) guarantee the features are *worth shipping* and *visible to the people accountable for them*.

### Key takeaway
My value here is **coordination and visibility, not configuration**. By translating technical stages into business language, gating production on UAT, and insisting on release notes plus monitoring, I make an automated pipeline safe for a real learner-facing release — and I keep the feedback loop tight when something fails.
