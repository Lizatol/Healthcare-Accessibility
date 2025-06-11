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

### 2. **Population Centers via KDE**

* Uses Kernel Density Estimation to determine realistic population centers instead of geometric centroids.

### 3. **Healthcare Facility Classification**

* Categorized via multilingual tags into: Hospital, Clinic, Ambulatory, FAP, Pharmacy, etc.

### 4. **Routing Analysis**

* Computes real travel duration using OpenRouteService (ORS) API for walking or driving modes.

### 5. **Accessibility Index Formula**

```python
Accessibility Score =
  0.5 * Time Factor
+ 0.2 * Doctor Load Factor
+ 0.2 * Population Factor
+ 0.1 * Legal Urban/Rural Factor
```

Each factor is normalized between 0–1.

### 6. **Settlement Classification Function**

```python
def get_settlement_type(name):
    name_lower = str(name).lower()
    if any(term in name_lower for term in ['urban', 'city', 'город']):
        return 'urban'
    elif any(term in name_lower for term in ['village', 'деревня', 'сельское']):
        return 'rural'
    else:
        return 'unknown'
```

---

## Outputs

* `routes_df.csv`: all origin-destination pairs with duration and index
* `access_index`: Final accessibility score (0–100 scale)
* Maps: Interactive `folium` map, choropleth maps, scatter plots

---

## Requirements

```bash
pip install osmnx geopandas openrouteservice folium matplotlib seaborn tqdm
```

---

## Usage Example

```python
adm_gdf['center_density'] = adm_gdf.geometry.centroid
centers_gdf = gpd.GeoDataFrame(adm_gdf, geometry=adm_gdf['center_density'], crs=adm_gdf.crs)
med_map = create_interactive_map(adm_gdf, centers_gdf, med_gdf)
med_map
```

---

## License

MIT License

## Authors

Tolmacheva Elizaveta/ ITMO University, IDU
