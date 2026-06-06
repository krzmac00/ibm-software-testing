# Lab M2L2 — Data Migration Readiness Checklist

> **Migration:** Legacy on-premises CRM → modern cloud-based CRM
> **Data in scope:** Customer contact info, communication history, subscription data
> **Key differences:** New schemas, stricter validations, new naming conventions
> **My role:** Systems Professional — map fields, coordinate dry runs, validate data, and secure stakeholder sign-off.

---

## Purpose

This checklist confirms the team is ready to migrate customer data from the legacy CRM to the new cloud CRM **safely and accurately**. It guards against the four main risks in this migration: missing fields, misformatted data, outdated records, and the lack of a rollback plan. Every owner completes and signs their items before the migration is approved.

**Who completes it:** the Analyst, QA, Systems Architect, Developer/DevOps, and Business Lead each own the items assigned to them. The Systems Professional tracks completion and collects sign-off.

---

## Readiness checklist

### Planning

| **Item** | **Description** | **Owner** | **Complete (Y/N)** |
| --- | --- | --- | --- |
| 1 | Legacy data inventory completed (contacts, communication history, subscriptions) | Analyst | ☐ |
| 2 | Rollback plan defined and documented | Systems Architect | ☐ |
| 3 | Dry-run / staging environment configured to mirror the new CRM | DevOps | ☐ |
| 4 | Pre-migration documentation complete and shared | Analyst | ☐ |

### Mapping

| **Item** | **Description** | **Owner** | **Complete (Y/N)** |
| --- | --- | --- | --- |
| 5 | Field mapping sheet finalized between legacy and new schemas | Analyst | ☐ |
| 6 | Field mappings reviewed and approved | Systems Architect | ☐ |
| 7 | Naming-convention differences resolved in the mapping | Developer | ☐ |
| 8 | Transformation rules and exceptions documented | Developer | ☐ |

### Validation

| **Item** | **Description** | **Owner** | **Complete (Y/N)** |
| --- | --- | --- | --- |
| 9 | Sample data validated against new validation rules (format, required fields) | QA | ☐ |
| 10 | Record counts reconciled between source and target | QA | ☐ |
| 11 | Outdated / duplicate records identified and handled | Analyst | ☐ |
| 12 | Business stakeholder alignment and sign-off confirmed | Business Lead | ☐ |

---

## Sign-off

Each stakeholder signs to approve the checklist and authorize the migration.

| **Name** | **Role** | **Signature** | **Date** |
| --- | --- | --- | --- |
|  | Analyst |  |  |
|  | QA |  |  |
|  | Systems Architect |  |  |
|  | Developer / DevOps |  |  |
|  | Business Lead |  |  |
