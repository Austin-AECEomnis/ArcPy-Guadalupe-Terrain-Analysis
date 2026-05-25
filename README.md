# 🌊 ArcPy Guadalupe Corridor, 107 Structures Terrain Analysis
An automated ArcPy script that reproduces the full terrain
analysis workflow used to identify 107 high-risk structures
matching the Camp Mystic flood signature in the Guadalupe
River corridor, Kerr County, TX. The entire manual ArcGIS
Pro workflow, originally built across multiple sessions, runs
end to end in a single executable script.

## 📋 Overview

On July 4, 2025, the Guadalupe River at Hunt, TX rose
approximately 26 feet in 45 minutes, inundating structures
throughout the corridor. A terrain analysis conducted in
ArcGIS Pro identified 107 structures sharing the same
elevation, slope, and flow accumulation characteristics
as the Camp Mystic site, the most heavily impacted location
in the event.

This script automates that analysis from raw inputs to final
output. Given a DEM, a FEMA flood zone boundary, and a
structure point dataset, it derives terrain surfaces, extracts
values to structure locations, and applies the validated
two-pass filter that produces the 107-structure result.

## 🗂️ Repository Contents

| File | Description |
|---|---|
| guadalupe_107_structures_terrain_analysis.py | Full ArcPy automation script |
| README.md | Methodology, usage, and troubleshooting documentation |

## ⚙️ Requirements

| Requirement | Detail |
|---|---|
| Software | ArcGIS Pro 3.x |
| Python Environment | ArcGIS Pro default arcgispro-py3 conda environment |
| Extensions | Spatial Analyst (checked out automatically by script) |
| Input 1 | USGS 3DEP 1/3 arc-second DEM as GeoTIFF |
| Input 2 | FEMA 1% Annual Chance flood zone polygon in geodatabase |
| Input 3 | Structure points within flood zone in geodatabase |

## 🔁 Workflow Overview

The script executes seven sequential steps:

**Step 1, Buffer flood zone.** The FEMA flood zone polygon
is buffered by 3,000 feet on all sides. This creates a study
area mask with sufficient surrounding terrain context for
accurate raster analysis. The original manual workflow used
a Kerr County parcel selection as the clip boundary. The
buffered flood zone replicates that corridor extent without
requiring parcel data. Any polygon defining your study area
boundary can be substituted here.

**Step 2, Clip DEM.** The raw county-wide DEM is clipped to
the buffered flood zone using Extract by Mask. Running terrain
tools on the full county DEM is computationally prohibitive
at 1/3 arc-second resolution. Clipping first reduces
processing time from over an hour to a few minutes with no
loss of analytical accuracy.

**Step 3, Generate Slope.** Slope is derived from the clipped
DEM in degrees. Camp Mystic structures fall in the 0.18 to
3.36 degree range, the lowest slope values in the corridor,
confirming flat valley floor positioning.

**Step 4, Generate Flow Direction.** Flow Direction is derived
using the D8 method. Values range 1 to 255 representing eight
compass directions. At Camp Mystic, values range 16 to 128,
indicating water arriving from multiple directions simultaneously,
consistent with the S-curve river geometry at that location.

**Step 5, Generate Flow Accumulation.** Flow Accumulation is
derived from Flow Direction. Each cell value represents the
number of upstream cells draining through it. Camp Mystic
structures show values of 0 to 33 upstream cells, indicating
a low convergence zone despite sitting at the base of a
280,684-acre watershed.

**Step 6, Extract terrain values to structures.** Slope and
Flow Accumulation values are extracted to the 255 structure
points using Extract Multi Values to Points. Fields SLOPE_VAL
and FLOWACC_VAL are added to the attribute table.

**Step 7, Two-pass terrain signature filter.** Pass 1 applies
a slope ceiling of 3.3563 degrees, yielding approximately 114
structures. Pass 2 applies a flow accumulation ceiling of 33
upstream cells to those 114, yielding the final 107 structures.
The two-pass sequence mirrors the original manual selection
logic exactly.

## ✅ Validated Output

| Metric | Value |
|---|---|
| Pass 1 slope filter result | 114 structures |
| Pass 2 flow accumulation result | 107 structures |
| Location validation | Confirmed correct against original manual output |
| Runtime on clipped DEM | Under 10 minutes |

## 🗺️ A Note on Hillshade

Hillshade was generated during the original manual workflow
and served as the primary visualization layer in the published
ArcGIS portfolio. It was intentionally omitted from this
script because it contributed nothing to the analytical
selection logic. Every structure in the final 107 was
identified using Slope and Flow Accumulation exclusively.
Hillshade confirmed the flat valley floor visually but played
no role in the filter.

To add Hillshade as an optional visualization step, insert
the following after Step 2:

```python
from arcpy.sa import HillShade
hillshade_out = os.path.join(gdb, "Auto_Hillshade")
hillshade_raster = HillShade(dem_clipped, 315, 45, "", 1)
hillshade_raster.save(hillshade_out)
print(f"  Hillshade saved: {hillshade_out}")

## 🐛 Troubleshooting Notes

These issues were encountered during development and are
documented here for future users adapting this script to
their own data.

**County-wide DEM performance.** Running Slope, Flow Direction,
and Flow Accumulation on an unclipped county-wide DEM at 1/3
arc-second resolution exceeded one hour on Step 1 alone before
the run was cancelled. Always clip the DEM to your study area
before running any terrain tools. The buffer and clip steps
at the top of this script exist specifically to solve this
problem.

**Layer name mismatch.** The FEMA flood zone layer displayed
in the ArcGIS Pro Contents pane as Kerr_County_Floodzones but
was stored in the geodatabase as Kendall_FloodZones. ArcPy
references the geodatabase name, not the display name. Always
verify actual feature class names using arcpy.ListFeatureClasses()
before referencing layers in a script.

**ExtractMultiValuesToPoints field naming.** The original manual
workflow stored extracted values in fields named SLOPE_VAL and
FLOWACC_VAL. The script explicitly assigns these names during
extraction to ensure the downstream selection queries match.
If adapting this script, verify your field names match your
query strings exactly.

**207 of 255 points received values.** On an early test run,
48 structure points returned null values for terrain fields,
indicating those points fell outside the clipped DEM extent.
The buffer distance was increased to 3,000 feet to fully
capture all 255 structure locations within the raster extent.
If null values appear in your output, increase the buffer
distance.

## 🔗 Related Portfolio Products

This script is one component of a larger Guadalupe River
flood vulnerability analysis. The full portfolio includes:

- StoryMap narrative and findings: https://arcg.is/0bGXv02
- Experience Builder interactive application: https://experience.arcgis.com/experience/09c67703781c49ddbc0830655aba9473/
- Live monitoring Dashboard: https://www.arcgis.com/apps/dashboards/ac7607e8f4fa4185a97697b25cd6b181
- ArcGIS Online Web Map: https://aeceomnis.maps.arcgis.com/apps/mapviewer/index.html?webmap=8719efe8dae54c499e312fec5910b899
- USGS live gauge pipeline: https://github.com/Austin-AECEomnis/USGS-Guadalupe-LiveStage

## 👤 Author

Austin Addington Berlin
Founder, AECE Omnis LLC
AI-GIS Convergence Research
linkedin.com/in/austinberlin
github.com/Austin-AECEomnis
