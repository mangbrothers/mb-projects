# Training Content Spec

**Mangiapane Brothers Lawn and Snow LLC. Version 1, written Tuesday 20 October 2026 by DI.**

This file is the only shared piece between DI and Fall + Winter Planning. Fall + Winter Planning writes module files to this spec. DI renders them on the site, grades the quizzes and runs the gates. If a rule here and a rule in a module disagree, this file wins.

Copies: Drive `Mangiapane Brothers/05 Hiring & Crew/Training/00 Spec/training-content-spec.md` and repo `mb-projects/training/SPEC.md`. They are the same file.

---

## 1. One file per module

- Every module is one Markdown file: `<module-id>.md`.
- The file starts with a front matter block, then slides, then the quiz (if any).
- Write plain text. No HTML. No pay numbers anywhere (they live only in the offer).
- A service rule you do not have yet goes in as `[[TODO: what is missing]]`. DI shows these to admins and hides the slide from trainees until it is filled.

## 2. Front matter

```
---
id: crew-snow
title: Snow
roles: [member, manager]
category: Snow
season: snow
order: 40
version: 1
completion: quiz
status: ready
---
```

| Field | Rule |
|---|---|
| `id` | Must be one of the ids in section 8. Lowercase, hyphens. Never change it once published. |
| `title` | What the trainee sees. Short. |
| `roles` | Any of `walker`, `member`, `manager`, `all`. These are the system's existing training roles. Crew modules list both `member` and `manager`. |
| `category` | One of: Safety, Lawn, Snow, Fall cleanup, Lights, Equipment, Customers, Company policy, Onboarding, Management. |
| `season` | `always`, `fall`, `lights`, `snow`, `lawn`. Unlock dates for each season are a setting (section 6), not written in the file. |
| `order` | Sort order on the Training tab. Lower first. Use steps of 10. |
| `version` | Whole number. Raise it by 1 on any change to slides or quiz after it is published. |
| `completion` | `quiz` (done when the quiz is passed) or `view` (done when the last slide is reached). Trainees cannot tick a module done by hand. |
| `status` | `draft` or `ready`. DI imports only `ready`. |

## 3. Slides

Slides are separated by a line containing only `---slide---`. Each slide:

```
---slide---
## Clear the driveway to the back gate
- Driveway up to the back gate. Never the backyard. {reveal}
- Front porch and the walk to the porch. {reveal}
- The public sidewalk. {reveal}
![Driveway cleared to the gate](image:snow-driveway-scope)
video: https://www.youtube.com/watch?v=XXXX
note: Speaker or admin note. Trainees never see this.
```

| Part | Rule |
|---|---|
| Heading | One `##` line. Required. Under 60 characters. |
| Body | Bullets or short paragraphs. Five bullets at most. One idea per bullet. |
| Image slot | `![caption](image:<name>)`. `<name>` is lowercase with hyphens. The photo goes in `Training/Images/<module-id>/<name>.jpg`. If the photo is not there yet, the slide shows a grey box with the caption. Ask for photos; do not skip the slot. |
| Video | `video: <url>` on its own line. YouTube or Drive. One per slide. Basic repairs will use this most. |
| Step reveal | `{reveal}` at the end of a bullet. Bullets marked this way appear one tap at a time. |
| Highlight | `{highlight: <label>}` on its own line. Used by the systems deck to ring a button on a screenshot (for example `{highlight: Check off}`). Content modules rarely need it. |
| Note | `note:` line. Admin and manager eyes only. |

Keep a module to 8 to 15 slides. A trainee is on a phone.

## 4. Quiz

The quiz comes after the last slide, under a line containing only `---quiz---`.

```
---quiz---
### Q1
How much snow sends crews out once the storm stops?
- [ ] Any amount
- [x] 1 inch or more
- [ ] 3 inches or more
- [ ] 5 inches or more
explain: Crews go out at 1 inch or more once the storm has stopped. At 5 inches or more the operations manager may send crews mid-storm.
slide: 2
```

| Part | Rule |
|---|---|
| `### Qn` | Numbered in order. |
| Question | One line. |
| Options | 3 or 4. Exactly one `[x]`. No "all of the above" or "none of the above" (options are shuffled). |
| `explain:` | Required. Shown after the trainee answers, right or wrong. One or two sentences. |
| `slide:` | Optional. The slide number the answer is on. A wrong answer shows "Review slide N". |

Count: 5 to 8 questions for a gate quiz (walker job, crew safety, crew conduct). 3 to 5 for every other quiz. The quiz belongs to the module it sits in; there are no standalone quiz files.

## 5. Scoring and records (DI builds this)

Database nodes. The first four already exist on the site; the rest are new.

| Node | What it holds |
|---|---|
| `trainingLib/{id}` | The imported module: front matter fields, `kind: "module"`, slides, quiz. |
| `trainingRoles/{role}/{id}` | `true` when a whole role gets the module. Filled from `roles`. |
| `trainingAssign/{k}/{id}` | Single-person assignments (existing). |
| `trainingDone/{k}/{id}` | `{ at, version, via: "quiz" or "view", score }`. Written only by a pass or by reaching the last slide. |
| `trainingQuiz/{k}/{id}/{attemptId}` | Every attempt: `{ at, version, score, total, pct, passed, answers }`. Kept forever. |
| `trainingViewed/{k}/{id}` | `{ at, lastSlide }`. For `view` modules and the systems deck. |
| `trainingPractice/{k}` | The manager practice run (section 7). |
| `trainingSettings` | Pass mark, retake rule, season dates, walker unlock action (section 6). |

- Pass: score as a percent is at or above `trainingSettings/passMark`.
- Fail: the attempt is stored, the trainee sees which questions were wrong with each `explain:` line, and the retake rule applies.
- A new `version` does not erase a pass. The office sees "Passed v1, current v2" and can reassign it if the change matters.
- Questions and options shuffle on every attempt.

## 6. Settings (Russel's open decisions, built as settings)

| Setting | Path | Default until Russel decides |
|---|---|---|
| Pass mark | `trainingSettings/passMark` | 80 |
| Retake rule | `trainingSettings/retake` = `{ mode: "unlimited" or "limited", max, waitMinutes }` | unlimited, wait 10 minutes after a fail |
| Walker unlock | `trainingSettings/walkerUnlock` = `"auto"` or `"review"` | review |
| Season dates | `trainingSettings/seasons/{season}` = `{ from: "YYYY-MM-DD" }` | always: open. fall, lights, snow: 2026-10-20. lawn: 2027-03-15 |

Snow opens on or before 1 November in every case.

## 7. Gate flags

The system computes these per person. Content files may reference them by name in a `note:` line; they never set them.

| Flag | True when |
|---|---|
| `offer_signed` | The agreement (offer) on his paperwork is signed. This is the existing paperwork check. |
| `quiz_passed:<module-id>` | `trainingDone/{k}/<module-id>` exists with `via: "quiz"`. |
| `viewed:<module-id>` | `trainingViewed/{k}/<module-id>` reached the last slide. |
| `practice_passed` | All five practice items in `trainingPractice/{k}` are true (below). |
| `can_schedule` | The role's gate (below) is met. Scheduling and shift assignment refuse anyone without it. |
| `can_manage` | `can_schedule` for crew, plus `viewed:systems-manager` and `practice_passed`. The crew manager role refuses anyone without it. |

Gates, stored at `trainingSettings/gates` so they can change without code:

- **walker** `can_schedule` = `offer_signed` and `quiz_passed:walker-job` and `viewed:systems-walker`. Then `walkerUnlock` decides: scheduled automatically, or an overview card for Russel.
- **crew** `can_schedule` = `offer_signed` and `quiz_passed:crew-safety` and `quiz_passed:crew-conduct` and `viewed:systems-crew`.
- **manager** `can_manage` = crew gate and `viewed:systems-manager` and `practice_passed`.

Practice run items, all on the training route under the training account:

| Item | Path | Proven by |
|---|---|---|
| Schedule | `trainingPractice/{k}/schedule` | He opened the practice day on his schedule. |
| Check-off | `trainingPractice/{k}/checkoff` | One house checked off on the training route. |
| Photo | `trainingPractice/{k}/photo` | That check-off has its photo saved. |
| Incident | `trainingPractice/{k}/incident` | One incident report filed on a training house. |
| End of day | `trainingPractice/{k}/eod` | One end-of-day form submitted for the training crew. |

Rules: the end-of-day form and incident report are the real ones, unchanged. Every practice entry carries `training: true`. Billing, the end-of-day review, office complaint flags and payroll skip anything with `training: true`. This is the reuse of the existing training route and system check, not a second setup.

## 8. Module ids (use these exactly)

| id | Title | roles | season | completion | Gate |
|---|---|---|---|---|---|
| `walker-job` | Hanging door hangers | walker | always | quiz | walker |
| `crew-safety` | Safety and protective gear | member, manager | always | quiz | crew |
| `crew-conduct` | Conduct on client properties | member, manager | always | quiz | crew |
| `handbook` | Crew handbook: conduct and attendance | member, manager | always | view | none |
| `crew-equipment` | Equipment: backpack blowers | member, manager | fall | quiz | none |
| `crew-fall-cleanup` | Fall cleanups | member, manager | fall | quiz | none |
| `crew-gutters` | Gutters, ladders and roofs | member, manager | fall | quiz | none |
| `crew-lights` | Holiday lights | member, manager | lights | quiz | none |
| `crew-snow` | Snow | member, manager | snow | quiz | none |
| `crew-snow-equipment` | Snow equipment | member, manager | snow | quiz | none (later) |
| `mgr-day` | Running the crew day and route | manager | always | quiz | none |
| `mgr-quality` | Quality checks | manager | always | quiz | none |
| `mgr-complaints` | Client handling and complaints | manager | always | quiz | none |
| `mgr-repairs` | Basic repairs | manager | always | view | none |
| `mgr-snow-calls` | Snow calls and measuring | manager | snow | quiz | none |
| `mgr-eod` | End-of-day equipment and reporting | manager | always | quiz | none |
| `systems-walker` | Using the site: door hangers | walker | always | view | walker |
| `systems-crew` | Using the site: crew | member, manager | always | view | crew |
| `systems-manager` | Using the site: crew managers | manager | always | view | manager |
| `systems-admin` | Using the site: office | admin only | always | view | none |

`systems-*` are written by DI, not Fall + Winter Planning. `crew-conduct` quiz questions may draw on the handbook as well, since the handbook has no quiz of its own.

## 9. File layout and handoff

```
Mangiapane Brothers/05 Hiring & Crew/Training/
  00 Spec/training-content-spec.md      this file
  Modules/Walker/walker-job.md
  Modules/Crew/crew-*.md, handbook.md
  Modules/Manager/mgr-*.md
  Modules/Systems/systems-*.md           DI only
  Images/<module-id>/<name>.jpg
```

- Fall + Winter Planning saves finished files straight into the right `Modules` folder with `status: ready`. Drafts stay `status: draft` in the same place.
- DI imports every `ready` file into `trainingLib`, assigns it by `roles`, and keeps a copy in the repo at `mb-projects/training/modules/`.
- To change a published module: edit the file, raise `version`, keep `status: ready`. DI re-imports.
- Crew fall packet order (first, for Crew 1 on 26 October): `crew-safety`, `crew-conduct`, `handbook`, `crew-fall-cleanup`, `crew-equipment`, `crew-gutters`, `crew-lights`, `crew-snow`.

## 10. What content must not do

- Repeat any pay number.
- Change, describe new fields for, or add steps to the end-of-day form or incident report.
- Explain how to use the site. That is the systems deck. Say "see Using the site" instead.
