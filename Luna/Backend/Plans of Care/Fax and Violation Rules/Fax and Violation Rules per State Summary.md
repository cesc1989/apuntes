# State-by-State POC Fax and Violation Rules Summary

Generated: 2025-11-05 17:22:16

## States Grouped by Resolution Mode

| Resolution Mode | States | Count |
|-----------------|--------|-------|
| `referral` | Georgia, Indiana, Michigan, Minnesota, Missouri, New York, Oklahoma, Pennsylvania, Texas, Virginia | 10 |
| `plan_of_care` | California, Delaware, Florida, Wisconsin | 4 |
| `functional_progress` | Connecticut, District of Columbia, Illinois, Kansas, Ohio, Tennessee | 6 |

**Total**: 20 states with Direct Access POC configuration

---

## Referral Mode States by Fax Trigger Strategy

States with `resolution_mode: "referral"` grouped by their fax trigger strategy:

| Trigger Strategy | States | Count | Configuration |
|-----------------|--------|-------|---------------|
| `chart_signature` | Virginia | 1 | POC faxed immediately when chart is signed |
| `days_elapsed` | Indiana (25d), Minnesota (75d), Missouri (20d), Oklahoma (20d), Pennsylvania (15d), Texas (15d) | 6 | POC faxed after N days from initial visit |
| `days_or_visits_elapsed` | Georgia (10d/4v), Michigan (15d/8v), New York (20d/8v) | 3 | POC faxed after N days OR M visits, whichever comes first |

**Key Insight**: Virginia is the **only** referral-mode state using `chart_signature` trigger strategy, meaning POCs are generated immediately when the chart is signed rather than waiting for time/visit thresholds.

---

## Detailed State Configuration

## California

**Resolution Mode**: `plan_of_care`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_or_visits_elapsed`
  - `days`: `35`
  - `visits`: `10`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 40
  - `visits_elapsed`: 10
- **Violation**:
  - `days_elapsed`: 45
  - `visits_elapsed`: 12

---

## Connecticut

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `chart_signature`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## Delaware

**Resolution Mode**: `plan_of_care`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `20`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## District Of Columbia

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `15`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 20
- **Violation**:
  - `days_elapsed`: 30

---

## Florida

**Resolution Mode**: `plan_of_care`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `20`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## Georgia

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_or_visits_elapsed`
  - `days`: `10`
  - `visits`: `4`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 15
  - `visits_elapsed`: 6
- **Violation**:
  - `days_elapsed`: 21
  - `visits_elapsed`: 8

---

## Illinois

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_or_visits_elapsed`
  - `days`: `5`
  - `visits`: `8`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 10
  - `visits_elapsed`: 8
- **Violation**:
  - `days_elapsed`: 15
  - `visits_elapsed`: 10

---

## Indiana

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `25`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 35
- **Violation**:
  - `days_elapsed`: 42

---

## Kansas

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `chart_signature`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 10
- **Violation**:
  - `days_elapsed`: 15
  - `visits_elapsed`: 10

---

## Michigan

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_or_visits_elapsed`
  - `days`: `15`
  - `visits`: `8`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 15
  - `visits_elapsed`: 8
- **Violation**:
  - `days_elapsed`: 21
  - `visits_elapsed`: 10

---

## Minnesota

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `75`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 80
- **Violation**:
  - `days_elapsed`: 90

---

## Missouri

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `20`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## New York

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_or_visits_elapsed`
  - `days`: `20`
  - `visits`: `8`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
  - `visits_elapsed`: 8
- **Violation**:
  - `days_elapsed`: 30
  - `visits_elapsed`: 10

---

## Ohio

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `chart_signature`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## Oklahoma

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `20`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 25
- **Violation**:
  - `days_elapsed`: 30

---

## Pennsylvania

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `15`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 21
- **Violation**:
  - `days_elapsed`: 30

---

## Tennessee

**Resolution Mode**: `functional_progress`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `5`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 15
- **Violation**:
  - `days_elapsed`: 30

---

## Texas

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `days_elapsed`
  - `days`: `15`

### Violation Rules
- **Pending**:
  - `days_elapsed`: 20
- **Violation**:
  - `days_elapsed`: 30

---

## Virginia

**Resolution Mode**: `referral`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `chart_signature`

---

## Wisconsin

**Resolution Mode**: `plan_of_care`

### Fax Rules
- **Anchor**: `initial_visit`
- **Trigger**:
  - `strategy`: `chart_signature`

---
