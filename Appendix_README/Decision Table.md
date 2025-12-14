## Decision Table Documentation

### Input Variables

All patient- and therapist-related variables are normalized and mapped into discrete categories to enable deterministic decision logic.

**Patient Variables**

| Input Variable   | Description                                                   | Type / Encoding                             |
| ---------------- | ------------------------------------------------------------- | ------------------------------------------- |
| therapy_setting  | Requested setting: individual or couple                       | Enum → Int (nbr_of_clients)                 |
| disease_category | Clinical focus: depression, anxiety, coupletherapy, addiction | Enum → Int (category 1–4)                   |
| weekday          | Desired appointment weekday (Mon–Fri)                         | Enum → Constant score (1)                   |
| daytime          | Preferred time of day: morning, afternoon                     | Enum → Int (daytimeDecision)                |
| gender           | Preferred therapist gender: female, male, non-binary          | Enum → Int (genderDecision 1–3)             |
| waiting_time     | Accepted waiting time: 1_week–4_weeks                         | Ordinal → Int (waitingTimeDecision −1 to 2) |

### Output Variable

**Therapist Variables**

| Output Variable | Description                        |
| --------------- | ---------------------------------- |
| Therapy_ID      | Selected therapist identifier/name |

---

### Derived Decision Layers

The DRD decomposes matching into smaller, focused decisions, each producing intermediate variables consumed by downstream logic.

| Decision ID / Name                              | Output Variable     | Depends On                                | Purpose                                                          |
| ----------------------------------------------- | ------------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| Decision_TherapySetting / Individual or Couple  | nbr_of_clients      | therapy_setting                           | Encode 1 vs. 2 clients.                                          |
| Decision_DiseaseCategory / Kind of Therapy      | category            | disease_category                          | Map therapy type to category 1–4.                                |
| Decision_Gender / Gender Therapist              | genderDecision      | gender                                    | Encode therapist gender preference.                              |
| Decision_WaitingTime / Waiting List             | waitingTimeDecision | waiting_time                              | Score waiting time (2, 1, 0, −1).                                |
| Decision_Weekday / Day                          | day                 | weekday                                   | Normalize weekday to constant score 1.                           |
| Decision_Daytime / Daytime                      | daytimeDecision     | daytime                                   | Normalize time preference (1).                                   |
| Decision_0i5oonf / Availability                 | availability_score  | day, daytimeDecision, waitingTimeDecision | Combine schedule and waiting-time into availability score (1–3). |
| Decision_TherapistMatches / Which Therapy-Match | Therapy_ID          | All above                                 | Final therapist selection via decision table.                    |

---

### Matching Logic

#### 1. Normalization & Encoding

- **Therapy setting**
  - `therapy_setting` → `nbr_of_clients`
    - individual → `1`
    - couple → `2`

- **Disease category**
  - `disease_category` → `category`
    - Encoded as integer values `1–4`

- **Gender**
  - `gender` → `genderDecision`
    - female → `1`
    - male → `2`
    - non-binary → `3`

- **Waiting time tolerance**
  - `waiting_time` → `waitingTimeDecision`
    - very short / immediate → `2`
    - short → `1`
    - neutral → `0`
    - long tolerated → `-1`

#### 2. Availability Scoring

- **Weekday preference**
  - `Decision_Weekday`
  - Normalizes calendar weekday preferences into a generic score: `day`

- **Daytime preference**
  - `Decision_Daytime`
  - Normalizes time-of-day preferences into `daytimeDecision`

- **Waiting time**
  - `Decision_WaitingTime`
  - Expresses tolerance or flexibility regarding waiting time

- **Combined availability**
  - `Decision_0i5oonf` *(Availability)*
  - Combines:
    - `day`
    - `daytimeDecision`
    - `waitingTimeDecision`
  - Output: `availability_score`
    - `3` → best availability match
    - `2` → good match
    - `1` → lowest configured positive match

#### 3. Therapist Selection (Main Decision Table)

- **Decision:** `Decision_TherapistMatches`
- **Purpose:** Select the most suitable therapist

- **Inputs**
  - `nbr_of_clients`
  - `category`
  - `genderDecision`
  - `availability_score`

- **Logic**
  - Each rule represents **one therapist**
  - Rules define allowed combinations of:
    - number of clients
    - disease category
    - gender compatibility
    - availability score

- **Output**
  - `Therapy_ID` (e.g. therapist identifier or name)

- **Example therapists**
  - Dr. Jordan
  - Dr. Müller
  - Dr. Ruth
  - Dr. Goth
  - Dr. Biesel
  - Dr. Jung

- **Rule behavior**
  - If an input combination matches a rule's filters → the corresponding `Therapy_ID` is returned

---

### Hit Policy Selection

**Chosen Hit Policy: `COLLECT (MAX)`**

**Rationale:**

- Multiple therapists may satisfy all hard constraints
- Each rule returns `(therapist_id, score)`
- `COLLECT (MAX)` selects the therapist with the **highest matching score**
- Deterministic, explainable, and scalable

**Tie-Break Strategy:**

1. Highest availability
2. Least recent assignment
3. Random fallback (optional)

---

### Why Rule-Based (Now)

- Full transparency for stakeholders
- Easy validation by domain experts
- Legally and ethically explainable
- Low data requirement

---
