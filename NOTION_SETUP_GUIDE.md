# CNC Feedrate & Tool Speed Calculator — Notion Setup Guide

Yes, you can have the calculator **reference a table** in Notion. Notion doesn’t support Excel-style VLOOKUP in formulas, but you can get the same result using **Relations** and **Rollups** between databases. This guide shows two approaches and the exact formulas to use.

---

## Option A: Two databases (calculator + reference table)

Best when you want to maintain chip-load and RPM defaults in one place and reuse them in many setups.

### 1. Reference database: “Chip Load & RPM Reference”

Create a database with one row per **Material + Flutes** combo. Other properties can be added later (e.g. cut type).

| Property name       | Type    | Purpose |
|--------------------|---------|--------|
| **Name**           | Title   | e.g. "MDF 1-flute", "Hardwood 2-flute" |
| **Material**       | Select  | MDF, Hardwood |
| **Flutes**         | Number  | 1, 2, or 3 |
| **Chip Load Min**  | Number  | in/tooth (e.g. 0.010) |
| **Chip Load Max**  | Number  | in/tooth (e.g. 0.016) |
| **Chip Load Mid**  | Number  | Optional: (Min+Max)/2 for “middle” default |

**Example rows:**

| Name           | Material | Flutes | Chip Load Min | Chip Load Max | Chip Load Mid |
|----------------|----------|--------|---------------|---------------|---------------|
| MDF 1-flute    | MDF      | 1      | 0.010         | 0.016         | 0.013         |
| MDF 2-flute    | MDF      | 2      | 0.006         | 0.012         | 0.009         |
| MDF 3-flute    | MDF      | 3      | 0.004         | 0.008         | 0.006         |
| Hardwood 1-flute | Hardwood | 1    | 0.008         | 0.014         | 0.011         |
| Hardwood 2-flute | Hardwood | 2    | 0.005         | 0.010         | 0.0075        |
| Hardwood 3-flute | Hardwood | 3    | 0.004         | 0.007         | 0.0055        |

You can add another table or block for **RPM defaults by diameter** (e.g. small 1/4" → 18k–22k) and type those into the calculator by hand, or add a second relation if you want to pull RPM by diameter.

---

### 2. Calculator database: “CNC Setup Calculator”

Each row = one setup (one tool + material + cut type + engagement).

| Property name           | Type     | Purpose |
|-------------------------|----------|--------|
| **Name**                | Title    | e.g. "1/2\" comp MDF profile" |
| **Tool type**           | Text or Select | e.g. Compression, V-bit, straight |
| **Diameter (in)**       | Number   | D in inches (e.g. 0.5 for 1/2") |
| **Flutes**              | Number   | Z (1, 2, or 3) |
| **Material**            | Select   | MDF, Hardwood |
| **Cut type**            | Select   | Profile, Pocket, V-groove, Drilling |
| **Engagement**          | Select   | Light, Heavy |
| **Tool max RPM**        | Number   | From tool body/packaging |
| **Chip Load Reference** | Relation | Relation to “Chip Load & RPM Reference” |
| **Chip load (in/tooth)**| Number   | Your chosen chip load (can be between min/max from reference) |
| **Fixed RPM**           | Number   | Spindle RPM you’ll use |
| **Entry method**        | Select   | Plunge, Ramp, Helix |

**Relation setup**

- In “CNC Setup Calculator”, add a relation **Chip Load Reference** → to database “Chip Load & RPM Reference”.
- When you link a row, you’ll pick the matching Material + Flutes row (e.g. “MDF 2-flute”).

**Rollups** (so the calculator “references the table”)

In “CNC Setup Calculator”, add:

- **Chip Load Min (from ref)**  
  - Rollup from **Chip Load Reference**  
  - Property: **Chip Load Min**  
  - Calculate: **First value** (or Max/Min if you prefer)

- **Chip Load Max (from ref)**  
  - Rollup from **Chip Load Reference**  
  - Property: **Chip Load Max**  
  - Calculate: **First value**

Now each setup row can show the recommended range from the reference table. You type **Chip load (in/tooth)** within that range (or use Chip Load Mid from a rollup if you add it).

---

### 3. Formula properties in “CNC Setup Calculator”

Add these as **Formula** type properties.

**XY Feed (IPM)**

``` 
prop("Fixed RPM") * prop("Flutes") * prop("Chip load (in/tooth)")
```

**XY Feed (mm/min)** (metric)

``` 
prop("XY Feed (IPM)") * 25.4
```

**Z plunge (IPM)** — 10% of XY feed (conservative)

``` 
prop("XY Feed (IPM)") * 0.1
```

**Z plunge (mm/min)**

``` 
prop("Z plunge (IPM)") * 25.4
```

**Sanity: RPM under max?**

``` 
if(prop("Fixed RPM") <= prop("Tool max RPM"), "✅ OK", "⚠️ Exceeds max RPM")
```

**Sanity: Chip in range?** (assumes you have rollups Chip Load Min/Max)

``` 
if(and(prop("Chip load (in/tooth)") >= prop("Chip Load Min (from ref)"), prop("Chip load (in/tooth)") <= prop("Chip Load Max (from ref)")), "✅ In range", "⚠️ Check range")
```

Optional: **Feed display** (for the template line “= _ × _ × _ = _ IPM”)

Use a text formula if you want a single cell to show the breakdown; Notion formulas can’t concatenate numbers with “×” in a pretty way in one step, so the cleanest is to keep **XY Feed (IPM)** as the main formula and add a **Note** or **Template** text block in the page that you fill manually, or use:

``` 
format(prop("Fixed RPM")) + " × " + format(prop("Flutes")) + " × " + format(prop("Chip load (in/tooth)")) + " = " + format(round(prop("XY Feed (IPM)"))) + " IPM"
```

(Use **Formula** type and **format()** so numbers display nicely.)

---

## Option B: Single database with embedded reference

If you prefer one database and no relations:

- Add **Chip Load Min** and **Chip Load Max** as Number properties.
- Use a **Formula** to set them by Material + Flutes with nested `if()`:

``` 
# Chip Load Min (formula)
if(prop("Material") == "MDF", 
  if(prop("Flutes") == 1, 0.01, if(prop("Flutes") == 2, 0.006, 0.004)),
  if(prop("Flutes") == 1, 0.008, if(prop("Flutes") == 2, 0.005, 0.004))
)
```

``` 
# Chip Load Max (formula)
if(prop("Material") == "MDF", 
  if(prop("Flutes") == 1, 0.016, if(prop("Flutes") == 2, 0.012, 0.008)),
  if(prop("Flutes") == 1, 0.014, if(prop("Flutes") == 2, 0.01, 0.007))
)
```

- You still type **Chip load (in/tooth)** and **Fixed RPM** and use the same **XY Feed** and **Z plunge** formulas as above. The “reference table” is encoded in these two formulas instead of a separate database.

---

## Copy/paste setup template (for page body)

You can put this in the **page** of each calculator row (open the row as a page) so every setup follows the same checklist.

``` 
**Tool:** [link to or type tool name]
**Dia:** [Diameter (in)] in
**Flutes:** [Flutes]
**Material:** [Material]
**Cut type:** [Cut type]
**Engagement:** [Engagement]
**Max RPM:** [Tool max RPM]

**Chosen chip load:** [Chip load (in/tooth)] in/tooth
**Fixed RPM:** [Fixed RPM]

**XY feed:** RPM × flutes × chip load  
= [Fixed RPM] × [Flutes] × [Chip load]  
= [XY Feed (IPM)] IPM ([XY Feed (mm/min)] mm/min)

**Fixed down feed (Z):** [Z plunge (IPM)] IPM ([Z plunge (mm/min)] mm/min)
**Entry method:** [Entry method]

---
Sanity: [RPM under max?] | [Chip in range?]
```

Replace the bracketed labels with the actual property values (or use Notion’s “/” to insert database property values in the page).

---

## Worked examples in the calculator

| Scenario | Tool | Dia | Flutes | Material | Chip load | RPM | XY Feed (IPM) | Z plunge |
|----------|------|-----|--------|----------|-----------|-----|----------------|----------|
| A        | 1/2" 2-flute compression | 0.5 | 2 | MDF | 0.010 | 18,000 | 360 | 36 |
| B        | 1/2" 2-flute | 0.5 | 2 | Hardwood | 0.007 | 18,000 | 252 | 25 |
| C        | 1" 2-flute V-bit | 1 | 2 | Hardwood | 0.006 | 18,000 | 216 | 20 |

With Option A, link each row to the correct “Chip Load & RPM Reference” row (e.g. “MDF 2-flute”, “Hardwood 2-flute”); the rollups will show the recommended range and the formulas will compute feed and Z plunge.

---

## Short answer

- **Yes**, the calculator can reference a table in Notion.
- **Preferred way:** Use a **Reference** database (e.g. chip load by Material + Flutes) and link it to the **Calculator** database with a **Relation**, then use **Rollups** to pull “Chip Load Min” and “Chip Load Max” into each setup row. Formulas then use your chosen chip load and fixed RPM to compute **Feed (IPM)**, **Feed (mm/min)**, and **Z plunge**.
- **Alternative:** One database with formula-based “Chip Load Min/Max” from Material + Flutes (Option B), no relation.

If you tell me whether you prefer Option A (two databases + relation) or Option B (single database with formulas), I can give you a step-by-step “click-by-click” for Notion or export-friendly CSV headers for the reference table.
