# Sample Geo Datasets

This directory contains lightweight (<10MB) sampled versions of each processed dataset, intended for:

- Rapid prototyping
- Visualization testing
- Model experimentation
- Geometry validation

---

## Structure

```
sample_data/
├── geojson/       # Sampled GeoJSON files for each dataset
├── metadata/      # Corresponding metadata JSON files
└── notebooks/     # Preprocessing and profiling notebooks
```

### `geojson/`
Contains one GeoJSON file per dataset, downsampled to stay under 10MB while preserving a representative subset of features. Small datasets are copied directly without sampling.

### `metadata/`
Contains one JSON metadata file per dataset, enriched with attribute profiles (columns, data types, distinct values, missing values) and spatial metadata (geometric type, CRS, name).

---

## Notebooks

### [`sampling_and_geometry_annotation.ipynb`](./notebooks/sampling_and_geometry_annotation.ipynb)
Prepares GeoJSON datasets for analysis in two steps:
- **File Size Reduction** — Copies small GeoJSON files directly and downsamples larger files to keep each under 10MB, preserving a representative subset of features.
- **Geometric Type Assignment** — Adds a top-level `"geometricType"` field to each GeoJSON FeatureCollection, indicating the geometry type (e.g., `"Point"`, `"Polygon"`), or `"Mixed"` if multiple types are present.

### [`profiling_geoJSON_datasets.ipynb`](./notebooks/profiling_geoJSON_datasets.ipynb)
Iterates over all GeoJSON files in the `geojson/` folder and for each file:
- Loads it into a GeoDataFrame using `geopandas`.
- Profiles dataset attributes (columns, data types, distinct values, missing values) using `datamart_profiler`.
- Extracts top-level GeoJSON metadata (`geometricType`, `name`, `crs`) from the original file.
- Combines the profile and metadata into a single enriched dictionary.
- Saves the result as a `_metadata.json` file in the `metadata/` folder.