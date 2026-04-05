# Formik — Engineering Reference

## Best Practices

### Form State Ownership
- Use Formik for forms with multiple fields, validation rules, submit lifecycle, or server-returned field errors.
- Keep form state local and ephemeral — Formik should not become a global state container.
- Always declare a complete `initialValues` object so fields stay controlled from the first render.

### Validation
- Validate at the form boundary with a dedicated `validate` function or a schema-based adapter.
- Keep validation deterministic and side-effect free.
- Prefer reusable schema helpers for repeated rules (email, password strength, numeric ranges).
- If the project uses `zod`, map schema failures into Formik field errors instead of duplicating rules in multiple places.

### Submission Flow
- Handle async submissions with explicit `try/catch/finally` and always reset the submitting state:
  ```ts
  const formik = useFormik({
    initialValues,
    onSubmit: async (values, helpers) => {
      try {
        await login(values);
      } catch (err) {
        helpers.setStatus('Login failed');
      } finally {
        helpers.setSubmitting(false);
      }
    },
  });
  ```
- Use `setFieldError` for per-field API errors and `setStatus` for form-level failures.
- Disable the submit action while `isSubmitting` is true to prevent double submits.

### Component Integration
- Prefer `useField()` or small field wrapper components for consistent labels, errors, and accessibility behavior.
- In Ionic React, adapt `IonInput` and similar components manually because they emit `onIonChange`, not the standard DOM `onChange`:
  ```tsx
  <IonInput
    value={values.email}
    onIonChange={(event) => setFieldValue('email', event.detail.value ?? '')}
    onIonBlur={() => setFieldTouched('email', true)}
  />
  ```
- Show validation errors only when the field has been touched or after submit.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Missing or partial `initialValues` | Uncontrolled/controlled input bugs | Declare the full value shape up front |
| Mirroring form state in `useState` | Duplicated sources of truth | Let Formik own form values |
| Swallowing submit errors | User gets no feedback | Use `setFieldError` / `setStatus` |
| Leaving submit enabled during async request | Double submit race conditions | Respect `isSubmitting` |
| Inline validation logic scattered across JSX | Hard to test and reuse | Centralize validation helpers |
| Forcing Formik onto trivial one-input forms | Unnecessary abstraction | Use plain React state when simpler |

---

## Review Criteria

- [ ] `initialValues` fully defines the form shape
- [ ] Validation is centralized and deterministic
- [ ] Submit handler uses explicit async error handling
- [ ] API field errors map to `setFieldError` or `setStatus`
- [ ] Inputs expose accessible labels and error text
- [ ] No duplicated form state in local `useState`
- [ ] Ionic components correctly adapt `onIonChange` / `onIonBlur`

---

## Trade-offs

**Formik vs plain React state**: Formik reduces boilerplate on medium/large forms with validation and submission flows. For very small forms, plain React state can be simpler.

**Formik vs newer form libraries**: Formik is stable and well understood, but some teams prefer hook-first alternatives for lower rerender cost. Choose based on existing team familiarity and complexity needs.

---

## Implementation Notes

- Use English field names and error keys consistently if the codebase already standardizes on that convention.
- Prefer small reusable wrappers for repeated form patterns (`TextField`, `PasswordField`, `SubmitButton`).
- Pair with `zod` or another runtime schema to keep UI validation and API contract validation aligned.
