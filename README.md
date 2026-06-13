# Subject H3 Hexgrid Workflow

## Introduction

This workflow visualises the spatial distribution of a wildlife subject group from EarthRanger as an H3 hexagonal grid map. It answers the question: *where are my tracked animals spending their time?*

**What this workflow does:**
- Connects to **EarthRanger** and downloads GPS fixes for a chosen subject group over a selected time period
- Bins fixes into **H3 hexagonal grid cells** at a user-selectable resolution
- Classifies hexbin fix counts using **Natural Breaks** to reveal genuine density patterns
- Applies a **colour scheme** (viridis, plasma, blues, and more)
- Optionally **clips hexbins to a boundary polygon** from an ER spatial features group (e.g. park boundary)
- Optionally displays an **overlay layer** from an ER spatial features group (e.g. roads, fencelines, water sources) — handles mixed line and polygon geometry automatically
- Creates an **interactive hexgrid map** with tooltips showing the fix count for each hexbin
- Exports the classified hexbin layer as a **GeoPackage** for use in GIS software
- Shows key stats in a dashboard: total GPS locations, subjects tracked, active hexbins, and selected resolution

**Who should use this:**
- Conservation managers visualising habitat usage and ranging patterns for a wildlife species
- Field teams analysing where tracked animals or rangers are spending their time
- Researchers studying spatial distribution and movement density from GPS collar data
- Anyone needing a quick, clean coverage map directly from EarthRanger subject data

## Prerequisites

Before using this workflow, you need:

1. **Ecoscope Desktop** installed on your computer
   - If you haven't installed it yet, please follow the installation instructions for Ecoscope Desktop

2. **EarthRanger Data Source** configured in Ecoscope Desktop
   - You must have already set up a connection to your EarthRanger server
   - Your data source should be configured with proper authentication credentials
   - You'll need to know the name of your configured data source as it appears in Desktop

3. **Subject Group** set up in EarthRanger
   - You need at least one subject group configured in your EarthRanger system
   - You'll need the exact name of the subject group you want to analyse
   - Find it in EarthRanger under **Admin → Subject Groups**

## Installation

1. Open **Ecoscope Desktop**
2. Select the **Workflow Templates** tab
3. Click **+ Add Template**
4. Copy and paste this URL and press Enter:
   ```
   https://github.com/cllrssml/subject-hexgrid
   ```
5. Wait for the workflow template to be downloaded and initialised
6. The template will appear in your available template list

## Configuration Guide

Once you've added the workflow template, you'll need to configure it for your specific needs.

### Basic Configuration

#### 1. Workflow Details
Give your workflow run a name and description to help you identify it later.

- **Workflow Name** (required): A descriptive name for this run and the dashboard title
  - Example: `"Elephant Ranging — Q1 2026"`
- **Workflow Description** (optional): Additional context about this analysis
  - Example: `"Northern herd movement, dry season"`

#### 2. Data Source
Select your EarthRanger connection.

- **Data Source** (required): Choose from your configured EarthRanger data sources in the dropdown

#### 3. Time Range
Specify the period to analyse.

- **Since** (required): Start date and time — use the calendar picker
  - Example: `01/01/2026, 12:00 AM`
- **Until** (required): End date and time
  - Example: `03/31/2026, 11:59 PM`
- **Timezone** (optional): Select your local timezone
  - Example: `Africa/Nairobi (UTC+03:00)`

#### 4. Subject Group
The EarthRanger subject group to map.

- **Subject Group Name** (required): Exact name of the subject group in EarthRanger
  - Example: `"Elephants - Northern Herd"`
  - Find it in EarthRanger under **Admin → Subject Groups**
  - The name must match exactly as it appears in EarthRanger

#### 5. Boundary Group Name *(optional)*
Clip the hexbin map to a spatial boundary from EarthRanger.

- **Boundary Group Name** (optional): Name of an ER spatial features group to use as the clip boundary
  - Example: `"Park Boundary"` or `"Northern Zone"`
  - Find it in EarthRanger under **Admin → Map Layers → Feature Groups**
  - Leave **blank** to show hexbins across the full extent of the data without clipping

When a boundary is provided:
- Hexbins are clipped to the boundary polygon
- The boundary outline is drawn on the map as a white line
- The boundary group name is displayed on the dashboard

#### 6. Overlay Layer Group Name *(optional)*
Display an additional reference layer on the map.

- **Overlay Layer Group Name** (optional): Name of an ER spatial features group to display on top of the map
  - Example: `"Roads"`, `"Fencelines"`, `"Water Sources"`, `"Patrol Posts"`
  - Find it in EarthRanger under **Admin → Map Layers → Feature Groups**
  - Leave **blank** to add no overlay layer
  - Works with mixed geometry groups (e.g. a group containing both fencelines and boundary polygons)
  - Overlay features are drawn in orange

#### 7. Colour Scheme
Choose the colour palette for the hexbin density map.

- **Colour Scheme** (optional, default: `viridis`)
  - `viridis` — blue → yellow (colourblind-safe, recommended for most uses)
  - `plasma` — purple → yellow (high contrast)
  - `Blues` — light → dark blue (clean, professional)
  - `Greens` — light → dark green (natural feel)
  - `Oranges` — light → dark orange (warm tones)
  - `YlGn` — yellow → green
  - `YlOrRd` — yellow → orange → red (intuitive: warmer = more activity)
  - `RdYlGn` — red → yellow → green (traffic-light style)
  - `magma` — dark artistic palette (black → orange → white)
  - `inferno` — dark artistic palette (black → red → yellow)

#### 8. Hexagon Size (H3 Resolution)
Control how large each hexagon is on the map.

- **Hexagon Size** (optional, default: `7`, range: `4–11`)

  | Resolution | Approx. area per hex | Good for |
  |---|---|---|
  | 5 | ~252 km² | Very large landscapes, continental scale |
  | 6 | ~36 km² | Large ecosystems, big reserves |
  | 7 | ~5 km² | Medium to large reserves *(good default)* |
  | 8 | ~0.7 km² | Mid-size reserves, fine ranging patterns |
  | 9 | ~0.1 km² | Small reserves, high fix density |
  | 10 | ~0.015 km² | Sub-kilometre precision |
  | 11 | ~0.002 km² | Sub-hectare detail (dense data only) |

  Start with `7` and increase the number if your hexagons look too coarse.

#### 9. Number of Colour Bands
Controls how many density categories appear in the legend.

- **Number of Colour Bands** (optional, default: `5`, range: `2–10`)
  - Fix counts are automatically grouped using **Natural Breaks** — a method that finds natural gaps in your data so each band represents a meaningful range
  - `5` is a good default for most datasets
  - Use fewer bands (`3–4`) for small datasets or sparse data
  - Use more bands (`6–7`) if you have a wide spread of fix counts

### Advanced Configuration

#### Base Maps *(optional)*
Customise the base map layers displayed beneath the hexgrid.

- Default: Terrain (World Topo Map) at full opacity + Satellite at 50% opacity — a hybrid view that shows both landforms and imagery
- Available presets: Open Street Map, Roadmap, Satellite, Terrain, LandDx, USGS Hillshade
- Custom layers can be added with a tile URL

## Running the Workflow

1. **Configure** all required fields (Workflow Details, Data Source, Time Range, Subject Group)
2. **Review** optional fields — add a boundary and/or overlay if needed, adjust resolution and colour scheme
3. Click **Submit** — the workflow will appear in your **My Workflows** table
4. Click **Run** to start processing
5. Monitor progress; the workflow will show **Success** or **Failed** on completion

Processing time depends on the number of GPS fixes and the time range selected. Most runs complete in under a minute.

## Understanding Your Results

### Dashboard Layout

The workflow creates an interactive dashboard with 8 widgets:

| Row | Widgets |
|---|---|
| 1 | Total Locations · Subjects Tracked · Active Hexbins · Resolution |
| 2 | Subject Group · Boundary · Overlay |
| 3 | Hexgrid Map (full width) |

**Total Locations** — number of GPS fixes downloaded from EarthRanger for the selected group and time range

**Subjects Tracked** — number of unique subjects (animals/assets) that contributed at least one fix

**Active Hexbins** — number of hexagons containing at least one GPS fix

**Resolution** — the H3 resolution selected, shown with approximate hexagon area

**Subject Group / Boundary / Overlay** — the exact group names used, displayed for reference

### Hexgrid Map

- Each hexagon is coloured by fix count, from low (cool) to high (warm) based on your chosen colour scheme
- Hexagons with more fixes are brighter/warmer — indicating areas of higher activity
- Hover over any hexagon to see the exact fix count
- The boundary outline (if configured) is shown as a white line
- The overlay layer (if configured) is shown in orange — lines for paths/fencelines, polygons for areas

### Data Export

A **GeoPackage** (`.gpkg`) file containing the classified hexbin layer is saved to your results folder. You can open this in:
- **QGIS** (free) — drag and drop the `.gpkg` file
- **ArcGIS Pro** — use Add Data
- **Python** with `geopandas` — `gpd.read_file("hexgrid_hexbins.gpkg")`

## Common Use Cases

### Example 1: Basic ranging map for a species group
**Goal**: See where your elephants have been over the last quarter

**Configuration**:
- Time Range: last 3 months
- Subject Group: `"Elephants - Northern Herd"`
- Boundary: leave blank
- Overlay: leave blank
- Resolution: `7`
- Colour Bands: `5`

**Result**: Full extent hexgrid showing density of elephant fixes

---

### Example 2: Clipped map within a park boundary
**Goal**: Show only activity within the park boundary, with roads for reference

**Configuration**:
- Subject Group: `"Lions - All"`
- Boundary Group Name: `"Park Boundary"`
- Overlay Layer Group Name: `"Main Roads"`
- Resolution: `8`
- Colour Scheme: `YlOrRd`

**Result**: Hexbins clipped to the park outline, roads shown in orange, boundary outlined in white

---

### Example 3: High-resolution map for a small reserve
**Goal**: Fine-grained density map for a small, intensively monitored reserve

**Configuration**:
- Time Range: 1 month
- Subject Group: `"Rhinos - All"`
- Resolution: `9`
- Colour Bands: `4`
- Colour Scheme: `plasma`

**Result**: Small hexagons showing precise hotspots within the reserve

## Troubleshooting

### Workflow fails with "No data returned"
- Verify the **Subject Group Name** matches exactly as it appears in EarthRanger (case-sensitive)
- Check that subjects in the group have GPS fixes in the selected time range
- Try a wider time range to confirm data exists

### Boundary or overlay layer not showing
- Verify the **Boundary/Overlay Group Name** matches exactly as it appears in EarthRanger
- Confirm the spatial features group exists under **Admin → Map Layers → Feature Groups**
- Ensure the group contains at least one feature

### Map shows very few hexbins
- The subject group may have sparse data — try a lower resolution (smaller number) for larger hexagons
- Widen the time range to accumulate more fixes

### Hexbins look too coarse / too fine
- Increase the resolution number for smaller hexagons (more detail)
- Decrease the resolution number for larger hexagons (broader picture)
- Resolution `7` (~5 km²) is a good starting point for most African reserves

### Workflow runs but dashboard shows "Workflow not found"
- This is caused by a missing `layout.json` file — it should have been included automatically when installing via the GitHub URL
- If it persists, try removing and re-adding the template in Desktop
