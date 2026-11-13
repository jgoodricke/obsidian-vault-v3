## Technical Details

- Store an explicit clear of an optional single-value field as null. An omitted value must remain distinct from a clear, including for optional relationship lists.
- Request normalisation must not make whitespace-only input indistinguishable from a truly empty field. Code inspection found submission-filtering and optional-field validation risks; the runtime cause has not yet been confirmed by a regression test.
- Apply the same clearing and validation behaviour to create and update operations.
- Distinguish between fields that are omitted, explicitly cleared or supplied with a value. Hidden, disabled or unavailable controls must not accidentally clear existing values.
- Support clearing all optional single-value fields and selection controls, not just text inputs.
- Explicitly clearing optional relationship lists must remove the associated links without deleting the related reference records.
- Preserve valid values such as 0, false, No and Either where supported.
- Do not change the formatting of non-empty text values as part of this fix.
- Reject whitespace-only optional text rather than converting it to a valid clear.
- Ensure validation failures prevent all changes from being persisted.
- Preserve existing password and Company logo handling, including password-policy requirements for User creation and updates.
- Unsupported clears of required relationships must return validation errors rather than silently retaining the existing values and reporting success. Omitted values must remain unchanged.
- Preserve existing ownership, organisation scope, deployment restrictions, hierarchy and reassignment rules.
- Preserve existing business rules that explicitly reset related values.
- Do not change the meaning of an empty User Facility Assignment list or introduce new ownership-removal operations.