# Healthcare Accessibility Evaluation Method

## Overview

This method introduces a composite approach to evaluate healthcare accessibility across settlements using open geospatial data, travel time routing, and weighted scoring. It combines advantages of classical spatial accessibility models (e.g., PPR, 2SFCA) with dynamic routing and density-aware center estimation.

---

## Novelty Compared to Existing Methods

| No | Method                                          | Key Limitation                                | Our Improvement                                                    |
| -- | ----------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------ |
| 1  | Population-to-Provider Ratio (PPR)              | Ignores distance/travel time                  | Adds travel-time via routing and facility capacity normalization   |
| 2  | Shortest Path                                   | One-to-one mapping only                       | Aggregates multiple nearby facilities per center                   |
| 3  | Service Area (Isochrone)                        | Fixed threshold zones                         | Dynamic per-route time with weight factors                         |
| 4  | 2SFCA                                           | Assumes uniform population                    | Uses KDE to define true centers of population                      |
| 5  | Enhanced 2SFCA                                  | High complexity                               | Simplifies through travel-time weights, no kernel tuning needed    |
| 6  | MCLP                                            | Optimization-focused, requires scenario setup | Our method evaluates current accessibility, not optimal allocation |
| 7  | P-Median / P-Center                             | Needs integer optimization                    | Avoids model-fitting, uses empirical spatial data                  |
| 8  | MHV3SFCA                                        | Probabilistic & complex                       | Simpler score function with interpretable components               |
| 9  | Composite Healthcare Accessibility Index (CHCA) | May use non-spatial data, subjective weights  | Keeps weights explicit and modular, spatially driven               |

---

## Key Components of the Proposed Method

### 1. **Data Sources**

* **OSM**: Buildings and healthcare facilities
* **Administrative Boundaries**: Population attributes
* **ORS API**: Travel time by road

### 2. **Workflow**

#### Step 1: Load and Preprocess Administrative Boundaries

```python
adm_gdf = gpd.read_file("path_to_admin_boundary.geojson")
adm_gdf["center_density"] = adm_gdf.geometry.centroid
```

#### Step 2: Extract Residential Buildings

Use OSM or existing shapefile of residential buildings. KDE (Kernel Density Estimation) is then applied to identify true population centers.

#### Step 3: Calculate KDE-based Population Centers

```python
# Assuming buildings_gdf is loaded
centers_gdf = calculate_true_density_centers(adm_gdf, buildings_gdf)
```

#### Step 4: Extract and Classify Medical Facilities

```python
med_gdf = get_medical_facilities(adm_gdf)
```

This uses tags in multiple languages to categorize facilities (e.g., hospital, clinic).

#### Step 5: Compute Travel Routes

```python
routes_df = calculate_nearest_facilities(centers_gdf, med_gdf)
```

This will calculate travel time from population centers to facilities using OpenRouteService.

#### Step 6: Prepare Data and Compute Accessibility Index

```python
routes_df_prepared = prepare_data(routes_df)
routes_df_prepared['access_index'] = routes_df_prepared.apply(calculate_final_accessibility, axis=1)
```

This uses a weighted composite score:

```python
Accessibility Score =
  0.5 * Time Factor
+ 0.2 * Doctor Load Factor
+ 0.2 * Population Factor
+ 0.1 * Legal Urban/Rural Factor
```

#### Step 7: Visualize and Export Results

```python
# Interactive map
med_map = create_interactive_map(adm_gdf, centers_gdf, med_gdf)
med_map

# Static plots
create_healthcare_dashboard(routes_df_prepared)
```

---

## Outputs

* `routes_df.csv`: All origin-destination pairs with duration and accessibility index
* `access_index`: Composite accessibility score (0–100 scale)
* `interactive_map.html`: Viewable in browser
* `healthcare_dashboard.png`: Histograms and scatter plots

---

## Requirements

```bash
pip install osmnx geopandas openrouteservice folium matplotlib seaborn tqdm
```

---

## Usage Example

```python
adm_gdf = gpd.read_file("adm_boundary.geojson")
buildings_gdf = gpd.read_file("buildings.geojson")
centers_gdf = calculate_true_density_centers(adm_gdf, buildings_gdf)
med_gdf = get_medical_facilities(adm_gdf)
routes_df = calculate_nearest_facilities(centers_gdf, med_gdf)
routes_df_prepared = prepare_data(routes_df)
routes_df_prepared['access_index'] = routes_df_prepared.apply(calculate_final_accessibility, axis=1)
create_healthcare_dashboard(routes_df_prepared)
```

---

## License

MIT License

## Authors

Tolmacheva Elizaveta / ITMO University / IDU
