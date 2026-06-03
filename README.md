📡 5G Rooftop Coverage & Terrain Analyzer

This project is a computer tool that simulates how a 5G C-Band signal spreads across an area from a rooftop tower. It automatically handles map projections, trims terrain data, maps out what the tower can physically "see," and calculates how hills and slopes bounce or block the signal over a 2 km neighborhood.

Key Features
Automatic Map Matching:
 Converts standard GPS coordinates into matching meter-based grid coordinates to keep distance measurements highly accurate.

Line-of-Sight Visibility:
Figures out exactly what parts of the ground have a direct, unblocked view of a 35-meter-tall rooftop antenna within a 2 km radius.

Real-World 5G Setup:
Built around standard telecom equations to track signal strength while accounting for radio power, antenna strength, and cable signal loss.
City Clutter Settings:
 Includes an adjustable penalty option to simulate how city buildings and dense trees further weaken the signal.

⚙️ How the Code Works Step-by-Step

1. Basic Settings & Physics Limits
Sets up where data files are stored, where the tower is built, and how the radio behaves. It calculates the total starting power leaving the antenna by adding the raw transmitter power to the antenna's boost, and subtracting any signal lost in the connecting cables.
Frequency: 3500 MHz (Standard mid-band 5G).
Tower Height: 35 meters tall.
User Height: 2 meters tall (simulating someone holding a phone).

🌍 2. Automatic Zone Finder
To measure distances in precise meters instead of degrees, the code automatically calculates which regional map grid zone the tower belongs to based on its GPS position. It also cleans up computer memory safely behind the scenes, so the script doesn't crash when opening massive files.

🗺️ 3. Map Trimming & Aligning
Takes large, raw elevation files and matches them up with local metric grid maps. It looks at the tower's location, adds a safety buffer of 500 meters past the 2 km signal range, and crops out a perfect square slice of the terrain map so the computer doesn't waste time processing unnecessary data.

👁️ 4. Blocked View Calculation
Runs a visibility check across the cropped map to find out which areas are hidden behind hills or structures. It factors in standard atmospheric refraction (how air bends radio waves slightly) and matches up the pixel grids of the terrain map and the visibility map perfectly so there are no alignment errors.

📉 5. Signal Decay Math
Calculates exactly how much the 5G signal drops in strength for every single pixel on the map. Instead of checking pixels one by one in a slow loop, it calculates everything at once using fast matrix math.
Unblocked Paths: If a pixel has a direct line-of-sight to the tower, it uses standard open-air formulas that adjust if the user gets past a specific distance threshold.
Blocked Paths: If a hill blocks the path, it applies a tougher formula that adds extra penalties for hidden terrain shadows.
Signal Mapping: Generates the final received power map, caps the numbers to realistic phone limits (-120 dBm for a dead zone up to -50 dBm for perfect signal), and cuts off everything outside the 2 km boundary.

🖼️ 6. Coverage Map Visualization
Creates a beautiful, easy-to-read map layout and saves it as `telecom_5g_rooftop_rsrp.png`. It layers three things together: a real OpenStreetMap background, a faint gray layout of the hills, and a bright color overlay showing signal strength from "Outage" to "Excellent." It also draws a clear dashed circle marking the 2 km boundary line.

📐 7. Side-View Profile & Wave Clearance
Cuts a straight line through the map from the tower to a single target user and creates a side-view cross-section plot saved as `telecom_5g_rooftop_fresnel.png`.
Earth Curve Adjustment: Adjusts the hill heights to account for the physical curve of the Earth over long distances, using industry standard atmospheric rules.
The Fresnel Zone: Draws an oval-shaped "bubble" around the signal path. Radio waves don't just travel on a laser line; they need space to breathe. 
Clearance Lines: The plot highlights the full wave bubble and a critical 60% boundary line. If the brown ground pushes up into these lines, engineers instantly know the terrain is blocking the signal.

🗺️ 8. How Waves Interact with Slopes
The final part of the script analyzes how the terrain's steepness and direction face the incoming radio waves, saving the results as `telecom_5g_rooftop_shadows.png`. It automatically sorts the neighborhood into three visual zones:
1. Forward Scattering: Areas where the signal hits a gentle slope and skims forward along the ground.
2. Direct Reflection: Steeper hills facing the tower directly, causing the signal to bounce right back toward the source.
3. Terrain Shadows: Blind spots that are completely blocked from the signal due to the shape of the land or being trapped on the back-slope of a ridge.

📈 Script Outputs

1. dem_utm_aligned.tiff: The trimmed and aligned elevation map file.

2. tower_viewshed.tiff: A black-and-white grid file separating unblocked and blocked views.

3. telecom_5g_rooftop_rsrp.png: The final colored 5G signal strength map over the city.

4. telecom_5g_rooftop_fresnel.png: The side-view graph showing Earth curve and wave clearance.

5. telecom_5g_rooftop_shadows.png: The map categorizing reflections, scatters, and blind spots.

6. temp_slope.tif: An intermediate terrain file calculating the angle/steepness of the hills. 

7.temp_aspect.tif: An intermediate terrain file tracking the compass direction each hill slope faces.
