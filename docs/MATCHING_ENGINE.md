# Talent Group Matching Engine v0.1

## 1. Objective

Talent의 매칭은 1:1 데이팅 매칭이 아니라 **같은 활동에 신청한 사람들 중 3~5명을 하나의 좋은 그룹으로 묶는 것**이 목적이다.

기본 그룹 크기: **4명**

허용 범위: **3~5명**

MVP에서는 설명 가능한 Rule-based 방식만 사용한다. 사람의 매력, 외모, 연애 가능성, 인기도를 점수화하지 않는다.

---

## 2. Inputs

### User-side
- verification state
- account state
- profile minimum completeness
- GIVE
- GET
- INTEREST
- broad activity areas
- broad availability
- attendance / no-show / late-cancel trust events
- previous group encounters
- blocks

### Activity-side
- category
- area_code
- starts_at / ends_at
- min / target / max participants
- venue safety state
- applicant pool

### Explicitly excluded from group score in MVP
- profile photo attractiveness
- follower/like count
- income
- race/ethnicity/religion
- political preference
- sexual orientation
- health information
- gender balancing as a generic optimization target

> Gender should not be used to create a "better" group by default. If a future activity format requires a lawful, user-transparent participation rule, that must be treated as a separate product/policy decision rather than a hidden score feature.

---

## 3. Pipeline

```text
Applications
  ↓
Eligibility Filtering
  ↓
Hard Constraint Filtering
  ↓
Pairwise Scores
  ↓
Candidate Group Generation
  ↓
Group Scores
  ↓
Conflict Detection
  ↓
Group Selection
  ↓
Fallback / Waitlist
  ↓
Atomic Confirmation
  ↓
Notification
```

---

## 4. Eligibility Filtering

A user is eligible only when all are true:

1. account status = active
2. required phone/provider verification complete
3. minimum profile/onboarding complete
4. application status in `applied` or eligible `waitlisted`
5. no active participation restriction
6. activity is still matchable
7. user is not already confirmed into an overlapping activity
8. broad activity area is compatible with the activity area, unless the user explicitly applied to that activity and area confirmation is already implicit

Pseudo:

```python
def eligible(user, activity, application):
    return (
        user.status == "active"
        and user.verified
        and user.minimum_profile_complete
        and application.status in {"applied", "waitlisted"}
        and not user.participation_restricted
        and not overlaps_confirmed_group(user.id, activity.starts_at, activity.ends_at)
    )
```

---

## 5. Hard Constraints

Any candidate group fails if one is true:

- size < 3 or size > 5
- any pair has a block in either direction
- any member is suspended/restricted
- any member is already confirmed into an overlapping group
- activity/venue is invalid or cancelled
- a member is ineligible under safety/admin policy

Pair block check must be symmetric in matching even though DB blocks are directional:

```sql
EXISTS blocks(a,b) OR EXISTS blocks(b,a)
```

---

## 6. Pairwise Scores

Scores are normalized to `[0, 1]`.

### 6.1 Complementarity

Measures GIVE ↔ GET overlap in both directions.

```text
A.GIVE ∩ B.GET
B.GIVE ∩ A.GET
```

Suggested formula:

```python
forward = weighted_overlap(a.gives, b.gets)
reverse = weighted_overlap(b.gives, a.gets)
complementarity = min(1.0, 0.5 * forward + 0.5 * reverse)
```

A one-direction match still earns value, but two-way complementarity is strongest.

### 6.2 Interest similarity

Use Jaccard-like similarity but cap its influence so homogeneous groups do not dominate.

```python
interest = len(A & B) / max(1, len(A | B))
```

### 6.3 Time/location

Because activity applications are session-specific, time compatibility is primarily a hard eligibility gate. Remaining score can reflect declared availability/area consistency.

```python
time_location = 1.0
if declared_area_matches: +0
elif explicit_application_overrides_area_preference: 0.8
```

Do not infer home distance from private location.

### 6.4 Reliability

Derived from attendance ledger, not public rating.

Example v0.1:

```python
base = 0.75
base += 0.04 * min(attended_count, 5)
base -= 0.20 * recent_no_show_count
base -= 0.08 * recent_late_cancel_count
reliability = clamp(base, 0.0, 1.0)
```

New users should receive a neutral starting value, not a punitive low score.

### 6.5 Novelty

Reduce repeated pairings.

```python
if meetings_30d == 0: novelty = 1.0
elif meetings_30d == 1: novelty = 0.65
elif meetings_30d == 2: novelty = 0.30
else: novelty = 0.05
```

This is soft, not hard: users may legitimately meet again.

---

## 7. Pair Score v0.1

Suggested:

```text
PairScore =
0.35 × Complementarity
+ 0.25 × Interest
+ 0.15 × TimeLocation
+ 0.15 × Reliability
+ 0.10 × Novelty
```

Reliability can be computed per-user and averaged for pair scoring.

Weights are configuration, versioned as `match_version`.

---

## 8. Group-level Scores

Pair quality alone can produce poor groups, so each candidate group receives additional group features.

### 8.1 Pair cohesion

Average pair score across all unordered pairs.

### 8.2 Skill coverage diversity

Reward useful GIVE/GET coverage rather than demographic diversity.

Examples:
- at least one GIVE that satisfies another member's GET
- multiple distinct skill/interest nodes represented
- avoid all members having identical GIVE and no cross-help opportunity

### 8.3 Weakest-link penalty

Groups where one member has near-zero compatibility with everyone should be penalized.

```python
member_affinity[u] = avg(pair_score(u, v) for v in group if v != u)
weakest = min(member_affinity.values())
```

### 8.4 Repetition penalty

Penalty if most members recently met each other repeatedly.

### 8.5 Reliability floor

Do not automatically exclude a new user. Penalize only clearly risky recent behavior.

---

## 9. Group Score v0.1

```text
GroupScore =
0.50 × MeanPairScore
+ 0.20 × SkillCoverage
+ 0.15 × WeakestMemberAffinity
+ 0.10 × GroupNovelty
+ 0.05 × ReliabilityBalance
```

`ReliabilityBalance` should prevent one high-risk member from dominating but must not create an opaque social credit score.

---

## 10. Candidate Generation

For Closed Beta applicant pools, brute-force combinations are acceptable up to a configured threshold.

```python
from itertools import combinations

sizes = [4, 3, 5]  # target first
for size in sizes:
    for group in combinations(eligible_users, size):
        if hard_constraints_pass(group, activity):
            score(group)
```

For larger pools, move to heuristic generation:

1. seed with applicant with fewest compatible options
2. add member maximizing marginal group score
3. keep top-K beams
4. compare completed groups

This prevents combinatorial explosion.

---

## 11. Selection Across Multiple Groups

If one activity session can form multiple groups, group selection becomes a set-packing problem because one user can belong to only one group.

MVP heuristic:

```python
candidate_groups = sorted(candidate_groups, key=score, reverse=True)
selected = []
used_users = set()

for group in candidate_groups:
    if not any(u.id in used_users for u in group.members):
        selected.append(group)
        used_users.update(u.id for u in group.members)
```

Then perform a local improvement pass to see whether swapping groups increases:

1. number of confirmed users
2. total group quality
3. number of 4-person groups

Priority order:

```text
maximize confirmed participants
then maximize target-size (4) groups
then maximize total group quality
```

Do not sacrifice many users solely for a small score improvement.

---

## 12. Fallback Rules

### Exactly 2 eligible applicants

Do not form a 2-person group in MVP because it changes the intended product experience.

Actions:
- waitlist
- offer nearby future session/category alternatives
- notify before a clear cutoff

### Exactly 3 eligible applicants

Form a valid 3-person group if quality/safety hard constraints pass.

### Exactly 4

Default confirm if hard constraints pass.

### Exactly 5

A 5-person group is valid if quality passes. Prefer one 5-person group over confirming 4 and unnecessarily leaving 1 unmatched, unless there is a strong operational reason.

### 6 applicants

Prefer two groups of 3 over one 5 + one unmatched when quality is acceptable, because confirmed participation is the first optimization objective.

### 7 applicants

Prefer 4 + 3.

### 8 applicants

Prefer 4 + 4.

### Low GIVE↔GET complementarity

Do not fail solely for weak complementarity. Use interest/activity fit as fallback because activity itself provides a legitimate purpose.

### Low user density

Relax in this order:

1. novelty penalty
2. complementarity target
3. skill coverage target

Never relax:
- blocks
- suspension/restriction
- schedule conflicts
- group size minimum
- safety rules

---

## 13. Repeated Match Prevention

Repeated encounter is not forbidden; it can indicate real social value.

Policy:
- penalize repeated pairings in automatic matching
- never prevent a mutually connected pair from joining the same future activity if both independently choose it, unless product rules say otherwise
- track repeat meetings separately for Second Meeting Rate

Avoid gaming: Second Meeting Rate should require actual confirmed/check-in evidence, not merely chat activity.

---

## 14. Cold Start

Cold Start should be handled operationally before algorithmically.

### Product tactics
- one local market only
- three categories only
- limited time windows
- scheduled activity templates rather than infinite choices
- aggregate demand into fewer sessions

### Algorithm tactics
- neutral reliability for new users
- no penalty for missing history
- broad taxonomy mapping (e.g. smartphone photography and portrait photography may share parent concept)
- interest/activity fit can compensate for weak GIVE↔GET data

---

## 15. Matching Pseudocode

```python
def run_matching(activity_id):
    activity = load_activity_for_update(activity_id)
    assert activity.status in {"open", "matching"}

    applicants = load_active_applicants(activity_id)
    users = [u for u in applicants if eligible(u.user, activity, u)]

    pair_scores = {}
    for a, b in combinations(users, 2):
        if blocked_either_direction(a, b):
            continue
        pair_scores[(a.id, b.id)] = calculate_pair_score(a, b, activity)

    candidates = []
    for size in candidate_size_order(len(users)):
        for members in generate_candidate_groups(users, size):
            if not hard_constraints_pass(members, activity):
                continue
            score, diagnostics = calculate_group_score(members, pair_scores)
            candidates.append((score, members, diagnostics))

    selected = select_non_overlapping_groups(candidates)

    with db.transaction():
        revalidate_users_and_activity(selected, activity)
        persist_groups(selected, match_version="rule-v0.1")
        mark_applications_selected(selected)
        waitlist_unselected(users, selected)
        create_group_conversations(selected)
        write_match_diagnostics(selected)
        enqueue_outbox_events(selected)

    return selected
```

---

## 16. Explainability

`group_match_scores` is internal diagnostic data.

Allowed internal explanation:

```json
{
  "top_factors": [
    "photo GIVE matched another member's GET",
    "shared walking interest",
    "no recent repeated pairing"
  ]
}
```

Not allowed:
- "user attractiveness score"
- "dating desirability"
- hidden sensitive demographic scoring

Users do not need raw scores. Product UI may simply say why an activity fits them, not why another person was ranked above someone else.

---

## 17. Validation Metrics

Offline outcome should evaluate the engine:

- group formation rate
- applicant confirmation rate
- 3/4/5-person group distribution
- join → check-in rate
- satisfaction by match version
- mission helpfulness
- mutual connection rate
- second meeting rate
- report/safety-event rate

A higher mutual connection rate alone is not sufficient evidence of better matching because the product is not solely a dating product.

---

## 18. Versioning

Every group stores `match_version`.

Example:
- `rule-v0.1`
- `rule-v0.2`

Any weight/constraint change should be documented in CHANGELOG and compared against offline outcomes before rollout.
