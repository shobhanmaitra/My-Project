# FestPass Jenkins assignment: evidence

Student: Aaditya Upadhyay (reg no 00457) | class code 001 | challenge code 00457
Job: `festpass-ci`, Pipeline script from SCM, `git://127.0.0.1:9418/festpass-java`, branch `*/main`, `Jenkinsfile`, agent `student-laptop`.

## Builds

| # | Commit | Cause | Result | Tests |
|---|--------|-------|--------|-------|
| 1 | `bf2f7c5` Configure my Jenkins assignment | Started by user (one bootstrap Build Now) | FAILURE | 28 run, 3 failed |
| 2 | `d06d957` Personalize my help introduction | Started by an SCM change | FAILURE | 28 run, 3 failed |
| 3 | `9547d0f` Free the seat when a booking is cancelled | Started by an SCM change | SUCCESS | 28 run, 0 failed |

Builds 2 and 3 were never started by hand. The checked-out commit in each Jenkins log matches `git log`.

## Failing assertion (builds 1 and 2)

`cancellationFreesSeat`: `SOLD OUT. Even the developer needs a ticket.`
Two other tests failed for the same reason: `cancellationsSurviveReload` ("Cancellation was lost") and `cliWorkflowPersistsAcrossCommands` ("List does not reflect persisted cancellation").

## Product stages

On both red builds, Build JAR, Smoke test JAR and Save artifact were skipped, and no JAR was published. Only `submission-evidence.csv` was archived.
On build 3 all stages ran, the smoke test passed (seven real-JAR commands), and `festpass.jar` was archived.

## What was wrong and what I changed

Business rule: cancelling a booking must give the seat back. `BookingService.cancel()` checked that the student had a booking but never removed them from the event, so the seat stayed taken, a second student got "SOLD OUT", and the cancellation vanished after a reload. I added one line, `event.students.remove(studentId);`, after that check. Tests, helpers and pipeline stages are untouched. The only Jenkinsfile change is the `pollSCM('* * * * *')` trigger.

## JAR output (build 3, from its download folder)

```
$ java -jar festpass.jar help
FestPass - campus event registration (personalized by Aaditya Upadhyay)
Usage: java -jar build/festpass.jar [--data FILE] COMMAND
  add-event ID "TITLE" CAPACITY
  list
  book EVENT_ID STUDENT_ID
  cancel EVENT_ID STUDENT_ID
  roster EVENT_ID
  summary
  help
Default data file: data/festpass.tsv
IDs are case-sensitive. Use synthetic student IDs, not personal data.
Exit codes: 0 success, 2 invalid request, 3 storage failure.
```

## Files

- `evidence/build-1-red/`, `evidence/build-2-red/`, `evidence/build-3-green/`: `submission-evidence.csv` for each build
- `jar-download/`: `festpass.jar`, `build-info.txt`, `smoke-test.txt`
- repository root: source code with full history (`src/`, `ci/`, `Jenkinsfile`, `submission.properties`)
