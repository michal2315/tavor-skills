# Scaffold: form

## Stack

TODO(team): form library (react-hook-form / formik / native), schema library (zod / yup), UI components.

## Checklist

- [ ] Validation schema defined once and reused for types - TODO(team): where schemas live; shared with backend?
- [ ] Field components from the design system - TODO(team): names (`<FormInput>`, `<FormSelect>`...)
- [ ] Field-level error messages shown, matching existing forms
- [ ] Submit button disabled + loading state while submitting; no double submit
- [ ] Server errors shown to the user (TODO(team): toast? inline?) - not swallowed
- [ ] On success: TODO(team): toast + redirect / reset / invalidate queries
- [ ] Edit mode pre-fills values; unsaved-changes warning if other forms have one
- [ ] Backend validates too - frontend validation is not security
- [ ] Test: valid submit, validation error shown, server error shown

## Team notes

TODO(team)
