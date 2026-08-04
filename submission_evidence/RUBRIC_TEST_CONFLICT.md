# Udacity Rubric and Starter-Test Conflict

This note records a contradiction in course version `3.2.31`. The project tests
remain untouched (`git diff -- tests` is empty).

## Conflicting requirements

The live project rubric requires every moderation flag to be a `bool` with a
sensible default of `False`. The reviewer also requires `is_flagged` on each
modality subclass and asks for both flagged and unflagged behavior checks.

The supplied file `tests/test_moderation_result.py` instead contains four tests
named `test_all_fields_are_required`. They require construction with omitted
flags to raise `ValidationError` for text, image, video, and audio results.

Those requirements cannot both hold: an omitted field cannot simultaneously
default to `False` and be rejected as missing.

There is also a course-file mismatch:

- The rubric and reviewer feedback refer to `tests/test_moderation_results.py`.
- The project instructions and starter repository contain
  `tests/test_moderation_result.py`.

## Current implementation and verification

The production models follow the rubric and reviewer feedback:

- all moderation flags default to `False`;
- `ModerationResult` contains only the shared `rationale` field;
- each modality subclass implements its own `is_flagged` property;
- `is_flagged` is `False` when every flag is false and `True` when any flag is
  true.

The untouched full suite produces:

```text
4 failed, 46 passed
```

All four failures are the mutually incompatible `test_all_fields_are_required`
assertions. The other 46 official tests pass, including the live Gemini
connectivity check.

## Clarification requested

Please provide a corrected starter test file aligned with the live rubric, or
confirm in writing whether flag fields should default to `False` or be required.
No caller-specific validation, pytest hook, test alteration, or hidden bypass
has been added to make the contradictory assertions appear green.

Course references:

- Rubric: https://learn.udacity.com/nd608?version=3.2.31&partKey=cd13331&lessonKey=69f48c22-5636-42a8-97a4-2899439f02e0&project=rubric
- Instructions: https://learn.udacity.com/nd608?version=3.2.31&partKey=cd13331&lessonKey=69f48c22-5636-42a8-97a4-2899439f02e0&conceptKey=217538fb-a7f7-4b9e-84f9-57327b1f6e44
- Reviewer feedback: https://learn.udacity.com/nd608?version=3.2.31&partKey=cd13331&lessonKey=69f48c22-5636-42a8-97a4-2899439f02e0&project=submit
