Here is the complete, unified GitHub `README.md` file layout using all the sections we refined, topped with your finalized 340-character scientific summary.

---

```markdown
# 📡 5G Rooftop Coverage & Viewshed Analyzer

A procedural GIS and propagation engine simulating 5G C-Band rooftop coverage via 3GPP UMa path loss. It automates metric UTM warping, maps true line-of-sight viewsheds, and applies vector calculus to calculate effective Earth curvature, Fresnel zone clearance, and multi-condition topographic wave interactions across a 2 km district grid.

---

## 🚀 Key Features

* **Automated CRS Reprojection:** Converts geographic coordinates ($\text{WGS84}$) and raw DEM data into local UTM zone coordinates for accurate, metric-based spatial analysis.
* **Deterministic Viewshed Analysis:** Simulates true Line-of-Sight ($\text{LoS}$) between a $35\text{-meter}$ rooftop cell site and standard User Equipment ($\text{UE}$) within a $2\text{-kilometer}$ radius.
* **5G C-Band Link Budget Integration:** Built to accommodate standard 3GPP Urban Macro ($\text{UMa}$) parameters, computing true $\text{EIRP}$ based on transmitter power, cable loss, and antenna gain.
* **Custom Clutter Adjustments:** Supports adjustable terrain shadowing penalties ($\text{dB}$) to fine-tune signal degradation in dense urban environments.

---

## ⚙️ Module Breakdown & Technical Architecture

### 1. Configuration & Physics Parameters
Initializes environmental paths, site locations, and physical link parameters. The engine calculates the **Total Equivalent Isotropically Radiated Power ($\text{EIRP}$)** using the following equation:

$$\text{EIRP}_{\text{total}} = P_{\text{tx}}\text{ (dBm)} + G_{\text{ant}}\text{ (dBi)} - L_{\text{cable}}\text{ (dB)}$$

* **Base Frequency ($f$):** $3500.0\text{ MHz}$ (C-Band 5G).
* **Antenna Coordinates:** Centered near Jeddah ($21.483473^\circ\text{N}$, $39.244606^\circ\text{E}$).
* **Height Matrix:** Transmitting antenna ($h_{\text{bs}} = 35.0\text{ m}$), Receiving $\text{UE}$ ($h_{\text{ut}} = 2.0\text{ m}$).

### 🌍 2. Automatic UTM Zone Calculation
To perform accurate metric-based distance calculations, the engine dynamically determines the local **Universal Transverse Mercator ($\text{UTM}$)** zone and corresponding **EPSG code**:

$$\text{Zone} = \left\lfloor \frac{\text{lon} + 180}{6} \right\rfloor + 1$$

$$\text{EPSG} = \begin{cases} 32600 + \text{Zone}, & \text{if lat} \ge 0 \text{ (North)} \\ 32700 + \text{Zone}, & \text{if lat} < 0 \text{ (South)} \end{cases}$$

It also implements clean scoping and resets dataset pointers to `None` to guarantee exception-safe garbage collection when handling large raster arrays.

### 🗺️ 3. Spatial Alignment & Bounded Warping
Handles the coordinate transformations required to align raw geospatial datasets with the calculated local $\text{UTM}$ framework.
* **Coordinate Transformation:** Uses GDAL/OSR to transform geographic coordinates ($\text{EPSG:4326}$) into precise metric grid coordinates.
* **Bounded Warping (`gdal.Warp`):** Establishes an analysis bounding box ($\text{bbox}$) using a safety buffer ($\text{margin} = d_{\text{max}} + 500\text{ m}$) around the tower. It reprojects and crops the raw DEM in a single step using bilinear resampling to save CPU/memory resources.

### 👁️ 4. Viewshed Computation & Array Ingestion
Executes the visibility analysis and prepares the resulting spatial datasets for downstream RF propagation modeling.
* **Line-of-Sight Calculation (`gdal.ViewshedGenerate`):** Simulates visibility across the grid using a standard earth curvature refraction coefficient of $0.13$.
* **Array Realignment:** Reads the rasters into NumPy arrays, masks out `NoData` tags, and calculates the exact pixel coordinate offsets ($\Delta x, \Delta y$) between the elevation grid and the viewshed output to guarantee faultless pixel-to-pixel registration.

### 📉 5. Propagation Decay Math (3GPP UMa Vectorization)
Bypasses slow iterative loops by utilizing highly optimized, fully vectorized NumPy operations to calculate 5G signal decay across the entire coverage grid based on the **3GPP Urban Macro ($\text{UMa}$)** model.

* **Distance Mapping:** Computes 2D surface distance ($d_{\text{2D}}$) and true 3D distance ($d_{\text{3D}}$), pinning minimum thresholds to $10\text{ m}$ to prevent mathematical singularities.
* **Dual-State Path Loss ($PL$):** Splits equations based on the viewshed state ($255$ for $\text{LoS}$, otherwise $\text{NLoS}$):
  * **$\text{LoS}$ Path Loss:** Dependent on the breakpoint distance ($d_{\text{BP}} = 4 \cdot h_{\text{bs}}^{\prime} \cdot h_{\text{ut}}^{\prime} \cdot f / c$).
    $$PL_{\text{LoS}} = \begin{cases} 32.4 + 21\log_{10}(d_{\text{3D}}) + 20\log_{10}(f_{\text{GHz}}), & d_{\text{2D}} \le d_{\text{BP}} \\ 32.4 + 40\log_{10}(d_{\text{3D}}) + 20\log_{10}(f_{\text{GHz}}) - 9.5\log_{10}(d_{\text{BP}}^2 + (h_{\text{bs}} - h_{\text{ut}})^2), & d_{\text{2D}} > d_{\text{BP}} \end{cases}$$
  * **$\text{NLoS}$ Path Loss:** Factors in structural obstructions alongside custom terrain shadow penalties.
    $$PL_{\text{NLoS}} = 13.54 + 39.08\log_{10}(d_{\text{2D}}) + 20\log_{10}(f_{\text{GHz}}) - 0.6(h_{\text{ut}} - 1.5)$$
* **$\text{RSRP}$ Mapping:** Computes the **Reference Signal Received Power** ($\text{RSRP} = \text{EIRP} - PL$), clipping values to an industry receiver window of $[-120, -50]\text{ dBm}$ and applying a strict $\text{NaN}$ mask outside the $2\text{ km}$ maximum analysis radius.

### 🖼️ 6. District Footprint Profile Visualization
Renders a multi-layered spatial map using Contextily (`cx`) and Matplotlib, saving the final plot to `telecom_5g_rooftop_rsrp.png`.
* **Layer Stack:** Overlays an online OpenStreetMap basemap ($\alpha=0.7$), the background terrain elevation map ($\alpha=0.15$), and the vectorized 5G $\text{RSRP}$ coverage map ($\alpha=0.75$).
* **Features:** Plots a high-contrast delta symbol (`^`) at the transmitter coordinates, maps a precise dashed circle at the $2\text{ km}$ boundary line, and reformats raw decibel ratings into network performance brackets (**-120 dBm Outage** to **-80 dBm Excellent**).

### 📐 7. Point-to-Point Link Geometry & Fresnel Clearance
Profiles a singular ray path between the transmitter ($\text{TX}$) and a target User Equipment ($\text{UE}$). It analyzes terrain clearance along the path and exports the cross-section diagram to `telecom_5g_rooftop_fresnel.png`.
* **Dynamic Path Sampling:** Linearly interpolates $N=200$ sample points along the metric path between the tower and target positions. It utilizes zero-fault index clipping to safely extract underlying terrain elevation from the raw DEM matrix.
* **Effective Earth Curvature ($4/3$ Earth Model):** Corrects standard DEM values using the industry-standard $4/3$ atmospheric refraction model ($R_e \approx 8494.67\text{ km}$). This adjustment ensures that long-distance line-of-sight paths realistically account for the earth's bulge.
* **Fresnel Zone Geometry Mapping:** Computes the boundary radius of the first Fresnel zone ($F_1$) along every step of the signal path using the equation:
  $$F_1 = 17.32 \times \sqrt{\frac{d_1 \cdot d_2}{f_{\text{GHz}} \cdot D}}$$
* **Clearance Threshold Assessment:** Plots both the **100% Fresnel Zone boundary** and the critical **60% Rayleigh clearance line** ($\text{Ray}_{\text{LoS}} - 0.6 \cdot F_1$). This visual validation makes it easy to confirm whether trees, buildings, or rising terrain will physically block the C-band signal vector.

### 🗺️ 8. Visualization: Ray-Terrain Vector Wave-Interaction Map
This final processing block utilizes advanced surface derivatives to model how emitted RF vectors physically interact with the surrounding topography, exporting the structural analysis to `telecom_5g_rooftop_shadows.png`.

* **In-Memory Surface Derivative Extraction:** Employs `gdal.DEMProcessing` inside temporary virtual memory paths (`/vsimem/`) to eliminate physical disk I/O bottlenecks. It derives continuous slope ($\theta_{\text{slope}}$) and aspect ($\phi_{\text{aspect}}$) raster matrices from the aligned DEM while enforcing edge-pixel interpolation strategies.
* **Incident Wave Vector Calculus:** Computes the exact 3D interaction profile between arriving ray fronts and localized terrain faces. It derives the horizontal path azimuth ($\alpha_{\text{ray}}$) using an optimized modulo wrap and resolves the true structural incidence angle ($\psi_{\text{incidence}}$) across dimension-aligned submatrices via:
  $$\alpha_{\text{ray}} = \operatorname{atan2}(X_{\text{sub}} - x_{\text{obs}}, Y_{\text{sub}} - y_{\text{obs}}) \pmod{2\pi}$$
  $$\sin(\psi_{\text{incidence}}) = \sin(\theta_{\text{slope}}) \cdot \cos(\phi_{\text{aspect}} - \alpha_{\text{ray}})$$
* **Categorical Wave Interaction Zoning:** Leverages multi-condition NumPy selection arrays to categorize pixels within the $2\text{ km}$ maximum analysis radius into three critical RF interaction classes:
  1. **Forward Scattering ($1$):** Shallow grazing incidence ($0^\circ < \psi_{\text{incidence}} \le 15^\circ$) where signals bounce forward along the ground.
  2. **Direct Reflection ($2$):** High incident impact ($\psi_{\text{incidence}} > 15^\circ$) generating strong backscatter and signal reversals.
  3. **Topographic Shadow Blind Spot ($3$):** Blocked geometry ($\psi_{\text{incidence}} \le 0^\circ$ or falling completely within a non-line-of-sight viewshed block).
* **Geospatial Overlay Output:** Employs resource context managers (`with` blocks) for automated garbage collection and uses a custom categorical discrete color map (`ListedColormap`) to superimpose these interaction behaviors over an OpenStreetMap basemap.

---

## 📈 Script Outputs

1. **`dem_utm_aligned.tif`**: The reprojected and clipped DEM grid, aligned to the local UTM zone.
2. **`tower_viewshed.tif`**: A high-fidelity raster output where pixels represent line-of-sight visibility states.
3. **`telecom_5g_rooftop_rsrp.png`**: The complete visual district signal heatmap layered over urban maps.
4. **`telecom_5g_rooftop_fresnel.png`**: The point-to-point path profile detailing Earth curvature corrections and Fresnel clearance.
5. **`telecom_5g_rooftop_shadows.png`**: Categorized ray-terrain geometric wave reflections and diffraction bounds map.

---

## 🧰 Requirements & Dependencies
Ensure your Python environment has the geospatial and scientific packages installed:
```bash
pip install rasterio numpy shapely pyproj matplotlib contextily gdal

```

```

```
