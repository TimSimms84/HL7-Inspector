# HL7 Inspector

A local, single-file HL7 message viewer. Paste any HL7 message and visually inspect every segment and field — no server, no install, no data leaves your machine. Safe for PHI.

## Usage

1. Double-click `hl7-inspector.html` to open it in your browser.
2. Paste an HL7 message into the text box.
3. **Hover** over any field chip to see its ID and name (e.g. `PID-5: Patient Name`).
4. **Click** a field chip to open the detail panel, which shows:
   - Field ID and name
   - Raw value with a Copy button
   - Component breakdown (e.g. `PID-5.1: Family Name`, `PID-5.2: Given Name`)
   - Repeat tabs if the field has multiple values separated by `~`

## Pasting from SQL Server

HL7 messages stored in SQL use `\r` (carriage return) as the segment terminator. The VS Code MSSQL extension replaces those with spaces when copying a cell value. The tool handles this automatically — paste the single-line result directly and it will split on segment boundaries.

---

## Adding Custom Segments (Z-segments)

Custom Z-segments (e.g. `ZHH`, `ZPH`, `ZAL`) parse and display automatically without any changes — you'll see the segment badge and all field chips. To add **field names** and **component labels**, edit two sections near the top of the `<script>` block in `hl7-inspector.html`.

### Step 1 — Add field names

Find the `HL7_FIELDS` object and add an entry:

```js
ZHH: {
  1: "Customer Field",
  2: "Medication Covered",
},
```

Only define the fields you care about. Unknown fields still display — they just won't have a label.

### Step 2 — Add component names (optional)

If any of your fields contain `^`-separated components, add an entry to `HL7_COMPONENTS`:

```js
"ZHH-1": { 1: "System ID", 2: "System Name" },
```

The key format is always `"SEGMENT-fieldNumber"`. Skip this step entirely for simple scalar fields (Y/N, dates, plain text).

### Step 3 — Add a color category (optional)

Find the `SEG_CLASS` object and map your segment to one of the existing color categories:

```js
const SEG_CLASS = {
  // ...existing entries...
  ZHH: 'seg-rx',       // green — pharmacy-related
  ZPH: 'seg-rx',
  ZAL: 'seg-clinical', // red — allergy/clinical
};
```

Available categories:

| Class | Color | Intended for |
|---|---|---|
| `seg-MSH` | Purple | Message header |
| `seg-patient` | Blue | PID, PV1, NK1 |
| `seg-order` | Orange | ORC |
| `seg-rx` | Green | RXE, RXO, RXR, pharmacy |
| `seg-obs` | Teal | OBR, OBX, lab/results |
| `seg-clinical` | Red | AL1, DG1, diagnoses |
| `seg-misc` | Gray | Everything else (default) |

If you don't add an entry to `SEG_CLASS`, the segment will default to gray.
