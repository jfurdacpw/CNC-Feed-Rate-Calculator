# CNC Feed & Speed Calculator

A single-page calculator for router-bit feed rates and spindle speeds when cutting MDF and hardwood. It uses the standard chip-load relationship so you can go from "I have this bit and this RPM" to a safe feed rate, or back-solve for chip load or RPM.

**What it does:**

- **Inputs:** Tool diameter (mm or in), number of flutes, and any two of: chip load, spindle RPM, and feed rate. The third is calculated.
- **Results:** Feed rate, chip load, spindle RPM (with the formula shown), and a suggested Z plunge at 10% of the XY feed.
- **Units:** Toggle between metric (mm, mm/min, mm/tooth) and imperial (in, IPM, in/tooth). Preference is remembered.

Formula: **Feed = RPM x Flutes x Chip load** (with unit conversion for metric).

Designed to run on GitHub Pages and to embed cleanly in Notion.
