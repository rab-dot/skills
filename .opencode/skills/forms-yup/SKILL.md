---
name: forms-yup
description: Forms with react-hook-form 7.43 + @hookform/resolvers 3.9 + yup 1.4. Use when building forms, validation schemas, field arrays, file inputs, or form error display. This repo uses yup, NOT zod.
---

# Forms + Validation (pinned: react-hook-form@7.43, @hookform/resolvers@3.9, yup@1.4)

Offline-first. Yup is the validator here — do not import zod in form code.

## 1. Basic form wiring

```tsx
import { useForm } from 'react-hook-form'
import { yupResolver } from '@hookform/resolvers/yup'
import * as yup from 'yup'

const schema = yup.object({
  title: yup.string().trim().required('Title is required').max(200),
  email: yup.string().email('Invalid email').required('Email is required'),
  count: yup.number().typeError('Must be a number').min(1).max(1000),
})

type FormValues = yup.InferType<typeof schema>

const {
  register,
  handleSubmit,
  formState: { errors, isSubmitting },
} = useForm<FormValues>({ resolver: yupResolver(schema), mode: 'onBlur' })

;<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register('title')} />
  {errors.title && <span>{errors.title.message}</span>}
  <button disabled={isSubmitting}>Save</button>
</form>
```

Rules: `mode: 'onBlur'` for most forms (less noise than onChange, faster than onSubmit-only for UX); `yup.InferType` instead of hand-written duplicate types.

## 2. Controlled components (selects, date pickers, chakra inputs)

```tsx
import { Controller } from 'react-hook-form'

;<Controller
  name="topicId"
  control={control}
  render={({ field }) => <Select {...field} options={topics} />}
/>
```

Use `Controller` only for components that do not expose `ref`/`onChange` natively. Prefer `register` for plain inputs (perf: uncontrolled).

## 3. Perf rules

1. Never `watch()` the whole form in a big component — subscribe narrowly: `watch('title')` or `useWatch({ name: 'items', control })`.
2. Isolate re-rendering fields into small subcomponents.
3. `useFieldArray` for dynamic rows (synonyms, recipients); always use `field.id` as key, never index.

```tsx
const { fields, append, remove } = useFieldArray({ control, name: 'synonyms' })

{
  fields.map((f, i) => (
    <div key={f.id}>
      <input {...register(`synonyms.${i}.value` as const)} />
      <button type="button" onClick={() => remove(i)}>
        Remove
      </button>
    </div>
  ))
}
```

## 4. Yup specifics (v1)

- `string().trim()` before `required()` so spaces fail.
- Numbers from inputs arrive as strings: use `typeError()` for a readable message, or `yup.number().transform(v => (isNaN(v) ? undefined : v))`.
- Conditional: `yup.string().when('kind', { is: 'geo', then: s => s.required() })`.
- File input: `yup.mixed<FileList>().test('size', 'Max 10MB', v => !v?.[0] || v[0].size <= 10_000_000)`.
- Strip unknowns on submit: `schema.validate(values, { stripUnknown: true })` when payload must match API exactly.

## 5. Error display pattern

```tsx
{
  errors.items?.[i]?.value && <span>{errors.items[i]?.value?.message}</span>
}
```

Show all errors after submit (yup `abortEarly: false` is default in resolver), focus first error: `shouldFocusError: true` (default).

## 6. Async submit

```tsx
async function onSubmit(values: FormValues) {
  try {
    await saveMutation.mutateAsync(values)
    reset(values) // mark clean after save
  } catch (e) {
    setError('root.server', { message: toMessage(e) }) // server error slot
  }
}
```

`root.server` error slot keeps server failures visible without touching field errors.

## Checklist

- [ ] Schema in one place, type via `yup.InferType`.
- [ ] `register` for native inputs, `Controller` only for controlled UI.
- [ ] No whole-form `watch()` in parent.
- [ ] `field.id` keys in field arrays.
- [ ] Server errors in `root.server`.
