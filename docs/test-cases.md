# Test cases (fake data)

All runs are from 2026-11-25. "Manual" means a run inside n8n. "HTTP" means a real `curl` call to the published webhook.

## Workflow 1: intake and AI draft

| # | Submission | What it tests | Expected | Actual | Pass |
|---|---|---|---|---|---|
| 1 | fake-sub-001 | Good answer | Draft, 10/10 | `draft`, 10/10 | Yes |
| 2 | fake-sub-002 | Weak answer, 1 day late | Draft with partial points | `draft`, 3/10, `days_late = 1` | Yes |
| 3 | fake-sub-003 | Off-topic answer | Should not be a normal draft | **Before the contradiction check:** `draft`, 10/10 with feedback "does not address the topic" (**failed**). See test 8. | No, then fixed |
| 4 | fake-sub-005 | Prompt injection inside the answer | Manual grading | `manual_required`, reason "may contain instructions meant for the AI", no scores saved | Yes |
| 5 | fake-sub-006 | Good answer | Draft | `draft`, 10/10 | Yes |
| 6 | HTTP: `{}` | Invalid hand-in | HTTP 400 | HTTP 400 and the message "The hand-in needs eventId, studentRef, a typed answer, and a rubric with criteria." | Yes |

### Contradiction check (added after test 3)

These two used a fixed (pinned) AI answer, so the Code node ran for real but Ollama and Gmail did not.

| # | AI answer | Expected | Actual | Pass |
|---|---|---|---|---|
| 7 | Good essay, 10/10, feedback "accurately describes the main steps... could be more concise" | Stays a draft | `draft`, 10/10 | Yes |
| 8 | Off-topic essay (about food), 10/10, feedback "does not address the topic... does not provide any relevant information" | Manual grading | `manual_required`, total 0, reason "The AI feedback names problems, but the scores are high. The AI contradicts itself", went to the "grade by hand" email | Yes |

## Workflow 2: decision and class record

| # | Test | Expected | Actual | Pass |
|---|---|---|---|---|
| 9 | Approve `fake-sub-002` (3/10, 1 day late) | Penalty 1 (10% of 10), final 2, one row in `class_record`, draft `approved` | Exactly that | Yes |
| 10 | Same approve link again | Nothing changes | Still one row, `decided_at` unchanged | Yes |
| 11 | Reject `fake-sub-003` | Draft `rejected`, no class record row | Exactly that | Yes |
| 12 | Token that does not exist | No change | No change | Yes |
| 13 | Empty POST body | No change | No change | Yes |
| 14 | HTTP: GET `/webhook/gleams-decision?token=...&decision=approve` for a draft | HTTP 200, page with the rubric table and a Confirm button | HTTP 200, `Content-Type: text/html` | Yes |
| 15 | HTTP: GET with a bad token | HTTP 404 | HTTP 404 | Yes |
| 16 | HTTP: GET for an already approved draft | "Already handled" page | "Already handled... Nothing was changed." | Yes |
| 17 | HTTP: POST `/webhook/gleams-decide` for an already approved draft | "Already handled" page | Same | Yes |

## Not tested

- The confirm page in a real browser (only curl).
- Gmail sending with the final workflow versions (the email nodes ran in earlier manual runs).
- The error handler on a real failure (it is published and set on both workflows, but no live run has failed yet).
- Sending the same `eventId` twice. The "New Hand-In Only" node is built to skip a repeated ID, but I did not run this test.
- A live Ollama run with the new contradiction check.

## Honest note about the model

`llama3.2` gave full marks to an off-topic essay (test 3), and it wrote feedback that does not match the scores. The contradiction check catches this kind of mistake only when the feedback has warning words ("does not", "irrelevant", and similar). A wrong score with friendly feedback will pass the check. The teacher approval step is the real safety net.
