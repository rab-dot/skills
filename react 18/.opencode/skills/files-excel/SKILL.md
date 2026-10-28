---
name: files-excel
description: File upload, download and Excel with react-dropzone 14.2, file-saver 2.0 and exceljs 4.3. Use when building drag-n-drop upload, file validation, client-side download, or Excel import/export.
---

# Files + Excel (pinned: react-dropzone@14.2, file-saver@2.0, exceljs@4.3)

Offline-first. Files validated client-side AND server-side; client checks are UX, not security.

## 1. Dropzone upload

```tsx
import { useDropzone } from 'react-dropzone'

const { getRootProps, getInputProps, isDragActive, fileRejections } =
  useDropzone({
    accept: {
      'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': [
        '.xlsx',
      ],
      'text/csv': ['.csv'],
    },
    maxSize: 10 * 1024 * 1024,
    maxFiles: 1,
    onDrop: (accepted) => void upload(accepted[0]),
  })

;<div {...getRootProps()}>
  <input {...getInputProps()} />
  {isDragActive ? 'Drop it' : 'Drag xlsx/csv or click'}
</div>
```

Always render `fileRejections` (size/type reasons) — silent reject is the top UX bug. Upload via axios with `onUploadProgress` (see api-realtime).

## 2. Download (file-saver)

```ts
import { saveAs } from 'file-saver'

const { data } = await api.get('/docs/export', { responseType: 'blob' })
saveAs(
  new Blob([data], {
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
  }),
  'docs.xlsx'
)
```

`responseType: 'blob'` mandatory; derive filename from `content-disposition` when present.

## 3. Excel export (exceljs client-side)

```ts
import ExcelJS from 'exceljs'

const wb = new ExcelJS.Workbook()
const ws = wb.addWorksheet('Docs')
ws.columns = [
  { header: 'Title', key: 'title', width: 40 },
  { header: 'Created', key: 'createdAt', width: 18 },
]
rows.forEach((r) => ws.addRow(r))
ws.getRow(1).font = { bold: true }
ws.autoFilter = { from: 'A1', to: 'B1' }

const buf = await wb.xlsx.writeBuffer()
saveAs(new Blob([buf]), 'docs.xlsx')
```

Cap client export at ~20k rows; beyond that request server-side file. `writeBuffer` + Blob (never base64 strings in memory).

## 4. Excel import (validate everything)

```ts
const wb = new ExcelJS.Workbook()
await wb.xlsx.load(await file.arrayBuffer())
const ws = wb.worksheets[0]
const rows = ws.getSheetValues() // 1-based, index 0 empty
```

Validate: header row matches expected names, required cells non-empty, dates via `isValid`, collect ALL row errors (row number + field) — never stop at first bad row. Reuse yup schemas (forms-yup) per row where possible.

## 5. Large files

> 50MB or >100k rows: chunked/resumable upload or server job + polling (SignalR `ImportProgress`), never a single blocking request. Show progress + cancel.

## Checklist

- [ ] accept/maxSize/maxFiles set; rejections rendered.
- [ ] Blob downloads with correct MIME + filename.
- [ ] Export capped; import reports all row errors.
- [ ] Progress + cancel for large transfers.
