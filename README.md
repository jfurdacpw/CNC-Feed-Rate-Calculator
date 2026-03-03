# CNC Feed & Speed Calculator

Single-page calculator for router-bit feed rates and spindle speeds (MDF & hardwood). Use it on GitHub Pages and embed in Notion.

## Host on GitHub Pages

1. Push this repo to GitHub.
2. **Settings → Pages** → Source: **Deploy from a branch**.
3. Branch: `main` (or `master`), folder: **/ (root)**.
4. Save. The site will be at `https://<username>.github.io/CNC-Tool-Speeds-and-Feeds-Calculator/` (or your repo name).

## Embed in Notion

1. In Notion, type `/embed` or click **Embed** in the block menu.
2. Paste the full URL of your GitHub Pages site (e.g. `https://yourusername.github.io/CNC-Tool-Speeds-and-Feeds-Calculator/`).
3. Resize the embed block as needed. The page is responsive and works inside the iframe.

## Usage

- **Setup:** Tool diameter (in), flutes, material (MDF/Hardwood), tool max RPM, cut type, engagement.
- **Chip load:** Use Finish / Balanced / Productivity presets or enter a custom value (in/tooth). Ranges follow the doc (MDF vs hardwood, 1–3 flutes).
- **RPM:** Enter spindle RPM; hint updates by diameter (small/medium/large).
- **Results:** XY feed (IPM and mm/min), Z plunge (10% of XY), formula breakdown, and sanity checks (RPM vs max, chip load in range).

Formula: **Feed (IPM) = RPM × Flutes × Chip Load** · Metric: **mm/min = IPM × 25.4**.
