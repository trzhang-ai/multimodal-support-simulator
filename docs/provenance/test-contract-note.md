# Historical schema-contract note

This note preserves a test-contract discrepancy found in the supplied course version `3.2.31`. It explains the historical validation record; it is not an unresolved failure in the current portfolio branch.

## Original discrepancy

The intended moderation contract required every flag to be a `bool` with a safe default of `False`, plus an `is_flagged` property on each modality result. Four supplied assertions named `test_all_fields_are_required` instead required omitted flags to raise `ValidationError`.

Those contracts are mutually exclusive: an omitted field cannot both default to `False` and be rejected as missing. The historical untouched suite therefore produced:

```text
4 failed, 46 passed
```

All four failures came from those required-flag assertions. The remaining tests, including the live Gemini connectivity check, passed.

## Portfolio contract

The current branch treats `rationale` and the audio `transcription` as required content, while moderation flags default to `False`. The four stale tests now verify that contract directly:

- omitted flags resolve to `False`;
- `is_flagged` is `False` when all flags are false;
- existing positive cases continue to verify typed flag values.

Current CI runs the complete non-integration suite without a name-based test exclusion.

## Source references

- [Public upstream repository](https://github.com/udacity/cd13331-multimodal-public)
- [Historical rubric](https://learn.udacity.com/nd608?version=3.2.31&partKey=cd13331&lessonKey=69f48c22-5636-42a8-97a4-2899439f02e0&project=rubric)
- [Historical project instructions](https://learn.udacity.com/nd608?version=3.2.31&partKey=cd13331&lessonKey=69f48c22-5636-42a8-97a4-2899439f02e0&conceptKey=217538fb-a7f7-4b9e-84f9-57327b1f6e44)
