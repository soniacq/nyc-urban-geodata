# NYC Urban Datasets

A curated collection of preprocessing pipelines for multi-source NYC urban datasets, producing analysis-ready GeoDataFrames and GeoJSON files for spatial and temporal analysis.

---


## Documentation

The generation and preprocessing of each dataset is fully documented in individual Jupyter Notebooks, located in the [`notebooks/`](./notebooks/) folder. Each notebook walks through the data loading, cleaning, transformation, and export steps for its corresponding dataset.

| Dataset | Notebook |
|---|---|
| Bi-Annual Pedestrian Counts | [`pedestrian_counts.ipynb`](./notebooks/pedestrian_counts.ipynb) |
| NYC Bicycle Counts | [`bicycle_counts.ipynb`](./notebooks/bicycle_counts.ipynb) |
| Motor Vehicle Collisions — Crashes | [`vehicle_collisions_crashes.ipynb`](./notebooks/vehicle_collisions_crashes.ipynb) |
| NYC Air Pollution by Community District | [`air_pollution.ipynb`](./notebooks/air_pollution.ipynb) |
| Heat Vulnerability Index | [`heat_vulnerability_index.ipynb`](./notebooks/heat_vulnerability_index.ipynb) |
| Automated Traffic Volume Counts | [`NYC_automated_traffic_volume_counts.ipynb`](./notebooks/NYC_automated_traffic_volume_counts.ipynb) |
| NYPD Arrest Data | [`NYPD_arrests_data.ipynb`](./notebooks/NYPD_arrests_data.ipynb) |
| NYC Population | [`NYC_population.ipynb`](./notebooks/NYC_population.ipynb) |
| NYC Unemployment Rate | [`NYC_unemployment_rate.ipynb`](./notebooks/NYC_unemployment_rate.ipynb) |
| NYC Poverty Rate | [`NYC_poverty_rate.ipynb`](./notebooks/NYC_poverty_rate.ipynb) |
| NYC Housing Units | [`NYC_housing_units.ipynb`](./notebooks/NYC_housing_units.ipynb) |
| NYC Issued Licenses | [`NYC_Issued_Licenses.ipynb`](./notebooks/NYC_Issued_Licenses.ipynb) |


---

## Sample GeoJSON Datasets

To support rapid prototyping and testing, lightweight sampled versions (<10MB) of each dataset are provided in [`sample_data/geojson/`](./sample_data/geojson/). Each dataset has a corresponding metadata file in [`sample_data/metadata/`](./sample_data/metadata/).

For more details, see the [`sample_data/README.md`](./sample_data/README.md).

---

## Datasets

### 1. Bi-Annual Pedestrian Counts

Provides semi-annual counts of pedestrian volumes at key street and bridge locations across New York City to track long-term walking trends along commercial corridors and major pedestrian routes.

- Data collected twice yearly (May and September) at over 100 on-street locations, bridges, and the Hudson River Greenway.
- Used in transportation planning and the Mayor's Management Report.

**Preprocessing:** The original dataset was transformed from wide format (one column per month–time block) into long format, where each row represents a single pedestrian count observation for a specific location, month, and time block.

🔗 [Original Data](https://data.cityofnewyork.us/Transportation/Bi-Annual-Pedestrian-Counts/cqsj-cfgu/about_data)

---

### 2. NYC Bicycle Counts

Bicycle counts (trip observations) merged with fixed-location bicycle counters to attach coordinates. The merged dataset provides point locations (lon/lat + WKT), site descriptors, timestamps, and count measures suitable for mapping trends, evaluating corridors, and supporting planning studies.

**Preprocessing:** The `geom_wkt` column (Well-Known Text strings) contained missing values (NaN) that would cause parsing errors. These were handled using `on_invalid='ignore'`, setting invalid or empty geometries to `None`. The geometry column was then parsed using `GeoSeries.from_wkt()`, a vectorized method significantly faster than row-wise alternatives like `.apply(wkt.loads)` for large datasets.

🔗 [Dataset on HuggingFace](https://huggingface.co/datasets/oscur/NYC_bicycle_counts)

---

### 3. Motor Vehicle Collisions — Crashes

Contains detailed records of all police-reported motor vehicle collision events in NYC. Each row represents a single crash and includes date, time, location, contributing factors, injuries, and fatalities.

- Data sourced from MV-104AN police reports required for collisions with injuries, deaths, or significant property damage.
- Updated regularly for public safety analysis and transportation planning.

**Preprocessing:** The original dataset contained 2.2 million NYC crash records. Rows with missing or zero `LATITUDE`/`LONGITUDE` values were removed before converting to a GeoDataFrame using `points_from_xy()` with CRS `EPSG:4326`.

🔗 [Original Data](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data)

---

### 4. NYC Air Pollution by Community District

Provides average concentrations of multiple air pollutants (e.g., NO₂, PM2.5, O₃) for each NYC Community District, with spatial geometries included.

**Preprocessing:** Each row represents a location and time period, with pollutants pivoted into separate columns for easy analysis and mapping.

**Sources:**
- [Community District Boundaries](https://data.cityofnewyork.us/City-Government/Community-Districts/5crt-au7u/about_data)
- [Air Quality Data](https://data.cityofnewyork.us/Environment/Air-Quality/c3uy-2p5r/about_data)
- [NYC Community Air Survey (NYCCAS)](https://a816-dohbesp.nyc.gov/IndicatorPublic/data-features/NYCCAS/)

---

### 5. Heat Vulnerability Index

Combines environmental (surface temperature, % green space) and social indicators (% AC access, median income, % Black residents) to measure neighborhood heat vulnerability. HVI scores range from **1** (lowest risk) to **5** (highest risk), reporting relative heat mortality risk for each ZIP Code Tabulation Area (MODZCTA) in NYC.

**Preprocessing:** HVI dataset is first loaded from a CSV file and the ZCTA identifiers are standardized to five-digit strings to preserve leading zeros. ZCTA polygon geometries are then loaded from the U.S. Census TIGER/Line 2020 shapefile using GeoPandas. The HVI table is joined with the ZCTA geometries using the ZCTA code as the key, keeping only records present in both datasets. Finally, a new GeoDataFrame (geo_HVI) is created containing the HVI attributes and the corresponding polygon geometry, making the dataset ready for spatial visualization and analysis.

**Sources:**
- [Heat Vulnerability Index](https://data.cityofnewyork.us/Health/Heat-Vulnerability-Index-Rankings/4mhf-duep/about_data)
- [Census TIGER/Line 2020 shapefile](https://www2.census.gov/geo/tiger/TIGER2020/ZCTA520/tl_2020_us_zcta520.zip)

---

### 6. Automated Traffic Volume Counts

Contains traffic volume counts captured automatically by NYC DOT Automated Traffic Recorders (ATR) at bridge crossings and roadways. Counts do not cover the entire year and the number of days counted per location may vary year to year.

**Preprocessing:** Coordinates in `WktGeom` are stored in NYC's local coordinate system (`EPSG:2263`) and are reprojected to `EPSG:4326` (lat/lon).

🔗 [Dataset on HuggingFace](https://huggingface.co/datasets/oscur/automated-traffic-volume-counts)

---

### 7. NYPD Arrest Data

A complete record of every arrest made in NYC by the NYPD going back to 2006 through the end of the previous calendar year. Data is manually extracted every quarter and reviewed by the Office of Management Analysis and Planning. Each record includes the type of crime, location and time of enforcement, and suspect demographics.

**Preprocessing:** Rows with missing coordinates and invalid coordinates (0, 0) are removed.

🔗 [Original Data](https://data.cityofnewyork.us/Public-Safety/NYPD-Arrests-Data-Historic-/8h9b-rp9u/about_data)

---

### 8. NYC Population

Population counts by Sub-Borough Area from 2000 to 2023, sourced from the NYU Furman Center CoreData platform.

> **To access the data:** Visit the [Furman Center CoreData page](https://app.coredata.nyc/), select **Population**, and set the Region to **Sub-Borough Area**. This ensures population counts are available for each year individually rather than aggregated.

**Preprocessing:** Sub-Borough Area names were standardized by normalizing slash spacing and applying authoritative name corrections. Remaining entries were matched to official sub-borough names using high-threshold fuzzy string matching, enabling reliable merging with the official shapefile to attach geometries and borough identifiers.

🔗 [Furman Center CoreData](https://app.coredata.nyc/?mlb=false&ntii=pop_num&mlf=true&ntr=Sub-Borough%20Area&mz=11&vtl=https%3A%2F%2Fthefurmancenter.carto.com%2Fu%2Fnyufc%2Fapi%2Fv2%2Fviz%2F98d1f16e-95fd-4e52-a2b1-b7abaf634828%2Fviz.json&mln=true&mlp=false&mlat=40.715354&nty=2023&mb=roadmap&pf=%7B%7D&md=map&mlv=false&mlng=-74.005293&btl=Community%20District&atp=neighborhoods#)

---

### 9. NYC Unemployment Rate

Unemployment rates by Sub-Borough Area from 2000 to 2023, sourced from the NYU Furman Center CoreData platform.

> **To access the data:** Visit the [Furman Center CoreData page](https://app.coredata.nyc/), select **Unemployment Rate**, and set the Region to **Sub-Borough Area**. This ensures rates are available for each year individually rather than aggregated.

**Preprocessing:** Sub-Borough Area names were standardized by normalizing slash spacing and applying authoritative name corrections. Remaining entries were matched to official sub-borough names using high-threshold fuzzy string matching, enabling reliable merging with the official shapefile to attach geometries and borough identifiers.

🔗 [Furman Center CoreData](https://app.coredata.nyc/?mlb=false&ntii=pop_pov_pct&mlf=true&ntr=Sub-Borough%20Area&mz=9&vtl=https%3A%2F%2Fthefurmancenter.carto.com%2Fu%2Fnyufc%2Fapi%2Fv2%2Fviz%2F98d1f16e-95fd-4e52-a2b1-b7abaf634828%2Fviz.json&mln=true&mlp=false&mlat=40.659867&nty=2023&mb=roadmap&pf=%7B%7D&md=table&mlv=false&mlng=-74.786518&btl=Community%20District&atp=neighborhoods#)

---

### 10. NYC Poverty Rate

Poverty rates by Sub-Borough Area from 2000 to 2023, sourced from the NYU Furman Center CoreData platform.

> **To access the data:** Visit the [Furman Center CoreData page](https://app.coredata.nyc/), select **Poverty Rate**, and set the Region to **Sub-Borough Area**. This ensures rates are available for each year individually rather than aggregated.

**Preprocessing:** Sub-Borough Area names were standardized by normalizing slash spacing and applying authoritative name corrections. Remaining entries were matched to official sub-borough names using high-threshold fuzzy string matching, enabling reliable merging with the official shapefile to attach geometries and borough identifiers.

🔗 [Furman Center CoreData](https://app.coredata.nyc/?mlb=false&ntii=pop_pov_pct&mlf=true&ntr=Sub-Borough%20Area&mz=9&vtl=https%3A%2F%2Fthefurmancenter.carto.com%2Fu%2Fnyufc%2Fapi%2Fv2%2Fviz%2F98d1f16e-95fd-4e52-a2b1-b7abaf634828%2Fviz.json&mln=true&mlp=false&mlat=40.659867&nty=2023&mb=roadmap&pf=%7B%7D&md=table&mlv=false&mlng=-74.786518&btl=Community%20District&atp=neighborhoods#)

---

### 11. NYC Housing Units

Housing unit counts by Sub-Borough Area from 2005 to 2023, sourced from the NYU Furman Center CoreData platform.

> **To access the data:** Visit the [Furman Center CoreData page](https://app.coredata.nyc/), select **Housing Units**, and set the Region to **Sub-Borough Area**. This ensures counts are available for each year individually rather than aggregated.

**Preprocessing:** Sub-Borough Area names were standardized by normalizing slash spacing and applying authoritative name corrections. Remaining entries were matched to official sub-borough names using high-threshold fuzzy string matching, enabling reliable merging with the official shapefile to attach geometries and borough identifiers.

🔗 [Furman Center CoreData](https://app.coredata.nyc/?mlb=false&ntii=unit_num&mlf=true&ntr=Sub-Borough%20Area&mz=9&vtl=https%3A%2F%2Fthefurmancenter.carto.com%2Fu%2Fnyufc%2Fapi%2Fv2%2Fviz%2F98d1f16e-95fd-4e52-a2b1-b7abaf634828%2Fviz.json&mln=true&mlp=false&mlat=40.659867&nty=2023&mb=roadmap&pf=%7B%7D&md=table&mlv=false&mlng=-74.786518&btl=Community%20District&atp=neighborhoods#)

---

### 12. NYC Issued Licenses

Contains licenses issued by the NYC Department of Consumer and Worker Protection (DCWP), formerly the Department of Consumer Affairs (DCA).

**Preprocessing:** Rows with missing coordinates and invalid coordinates (0, 0) are removed.

🔗 [Original Data](https://data.cityofnewyork.us/Business/Issued-Licenses/w7w3-xahh/about_data)



---

**Note:** A good source of additional NYC data is [Furman Center Neighborhoods](https://www.furmancenter.org/neighborhoods/brooklyn/).