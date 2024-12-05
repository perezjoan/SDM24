# README: Geographic Data Analysis 

## Introduction
This repository contains a series of Python scripts designed to retrieve, process, analyze, and visualize geographic data. The tools leverage OpenStreetMap (OSM) data to explore a specific area (Imredd), perform spatial analyses, and generate insights into urban structures and accessibility. The outputs include enriched geospatial datasets, clustering analyses, and comprehensive visualizations compiled into a professional PDF report.

## Main Objective
This project aims to detect and analyze points of interest (POI) within a defined geographic area, here Imredd in Nice. We use a Python script to identify and visualize these points by leveraging geographic data, APIs, or other information sources.

## Installation Steps
Follow these steps to run Python algorithms on Windows:
Download and install QGIS, tested with version 3.40.1.
Install the Anaconda distribution of Python: Anaconda3-2024.10-1-Windows-x86_64 - Python 3.12.
Create a specific environment IMREDD : 
1. Open Anaconda Prompt.

2. Create and Activate the IMREDD Environment
	
	          conda create -n IMREDD
	          conda activate IMREDD

3. Install Necessary Packages

		conda install jupyter
   		conda install -c conda-forge --strict-channel-priority osmnx
		conda install -c conda-forge momepy
		conda install -c conda-forge reportlab
		conda install -c conda-forge matplotlib  
		conda install -c conda-forge fiona
		conda install -c conda-forge numpy
		conda install -c conda-forge scikit-learn

5. Launch Jupyter Notebook

		jupyter notebook

6. In Jupyter Home, open and run the .ipynb scripts. Always remember to use this environment with 'conda activate IMREDD'

Activate the environment and run the associated Python scripts.

## Data Used
The protocol is designed for a global application, requiring only the coordinates of a bounding area for the area of interest. However, in this example, the choice was made to use a portion of the city of Nice.
Detailed Objectives

## Requirements
The toolkit requires the following Python libraries:
- `geopandas`
- `matplotlib`
- `matplotlib-scalebar`
- `contextily`
- `fiona`
- `numpy`
- `scikit-learn`
- `momepy`
- `osmnx`

Install dependencies using:
```bash
pip install geopandas matplotlib matplotlib-scalebar contextily fiona numpy scikit-learn momepy osmnx
```

## Usage

1. Execute the modules in order:
   - **Module 1**: Data retrieval and indicator calculation.
   - **Module 2**: Clustering analysis.
   - **Module 3**: Visualization and reporting.
2. Check the outputs:
   - Enriched GeoPackages (`SDM24_Imredd_filtered.gpkg` and `SDM24_Imredd_filtered_clusters.gpkg`).
   - PDF report: `geospatial_analysis_report.pdf`.

## Key Libraries and Tools

- **OSMnx**: For OSM data retrieval.
- **GeoPandas**: Geospatial data handling and processing.
- **Momepy**: Urban form analysis.
- **Scikit-learn**: Machine learning for clustering and modeling.
- **Matplotlib & Contextily**: Visualization and mapping.

---

## Module 1
This script retrieves, processes, and analyzes geographic data for a specific area (Imredd). The data is sourced from OpenStreetMap (OSM), and the final output is saved as a GeoPackage (`SDM24_Imredd_filtered.gpkg`). The steps involve retrieving building data, street network data, points of interest (POI), cleaning and filtering the data, calculating key indicators, and performing some basic spatial analyses. 
### Key steps:
#### 1. **Data Retrieval**
   - **Buildings:** OSM data related to buildings is downloaded for the specified bounding box using the OSMnx library. The data is cleaned to retain only valid polygons, and unnecessary columns are dropped.
   - **Streets:** The street network is retrieved from OSM using a graph-based approach, and the resulting graph is converted to a GeoDataFrame.
   - **Points of Interest (POI):** Points of interest (such as amenities, shops, and tourism-related features) are downloaded for the same region and cleaned to retain only point geometries.
#### 2. **Data Filtering and Preprocessing**
   - After downloading the raw data, the script filters and keeps only the relevant columns for each dataset (buildings, streets, POI).
   - For buildings, relevant columns include height, number of floors, and geometry.
   - For streets, essential attributes like the type of highway, speed limits, and accessibility are retained.
   - For POI, the columns of interest are amenities, names, types of shops, and cuisine.
#### 3. **Spatial Analysis and Indicator Calculation**
   - **Building Surface Area:** The script calculates the surface area of buildings in square meters using the geometry of the buildings, projected into a metric coordinate reference system (EPSG:2154).
   - **Convexity & Elongation:** Momepy's functions are used to calculate the convexity and elongation of each building's geometry. These indicators provide information about the building's shape.
   - **POI Count within Buffers:** For each building, the number of POIs within a 10-meter and 400-meter buffer zone is counted, which helps analyze the proximity of amenities to buildings.
   - **Soft Mobility Network Length:** The script calculates the total length of roads suitable for soft mobility (such as footways, cycleways, and bridleways) within a 400-meter radius around each building.
#### 4. **Data Enrichment**
   - The columns representing the building height (`H`) and number of floors (`FL`) are optionally filled based on available data. If one is missing, it is inferred from the other (height = number of floors * 3 meters).
   - Data is then saved into a GeoPackage, enriched with the calculated indicators: convexity, elongation, POI counts, surface area, and soft mobility length.
#### 5. **Modeling and Prediction (Optional)**
   - The script performs a basic predictive modeling task using decision trees. It trains a model to predict the number of floors (`FL`) based on other spatial features like convexity, elongation, and proximity to POIs or soft mobility paths.
   - The model's performance is evaluated using accuracy on a test dataset, and predictions are made for the entire filtered building dataset.
#### 6. **Final Output**
   - The final dataset, which contains both the original and enriched features, is saved into a new GeoPackage (`SDM24_Imredd_filtered.gpkg`) for further analysis or visualization.

### Output:
The script generates a GeoPackage containing several layers:
- `osm_all_buildings`: Raw building data from OSM.
- `osm_all_streets`: Raw street data from OSM.
- `osm_poi`: Raw POI data from OSM.
- `filtered_buildings`: Processed building data with calculated indicators (surface area, convexity, elongation, POI count).
- `filtered_streets`: Processed street data (retaining only relevant attributes).
- `filtered_poi`: Processed POI data (retaining only relevant attributes).
- `filtered_buildings_v2`: Final dataset with additional predicted values for the number of floors (`FLPRED`).

---

## Module 2
This script performs clustering analysis on building data using the KMeans algorithm and evaluates the clustering quality using the silhouette score. The analysis is based on a set of building features, such as convexity, elongation, proximity to points of interest (POI), surface area, and mobility accessibility. The process involves loading geospatial data from a GeoPackage, normalizing the data, applying KMeans clustering for various numbers of clusters, evaluating the results using silhouette scores, and finally assigning the optimal clusters to each building.
### Key steps:
1. **Loading Data**: 
   - The script loads the `filtered_buildings_v2` layer from a GeoPackage (`SDM24_Imredd_filtered.gpkg`), which contains building data with attributes like convexity, elongation, proximity to POI, surface area, and mobility access.
2. **Data Preparation**:
   - The selected variables (`C`, `E`, `P10`, `P400`, `S`, `M`, `FLPRED`) are extracted from the GeoDataFrame. The script checks if all the necessary columns are present and drops any rows with missing values.
   - The data is then normalized using `StandardScaler` to standardize the numerical features, ensuring they are on the same scale before applying clustering.
3. **KMeans Clustering**:
   - The script runs the KMeans algorithm on the normalized data for a range of cluster values (from 4 to 12). It calculates the silhouette scores for each clustering attempt, which helps assess the quality of the clustering results.
   - The silhouette score measures how similar an object is to its own cluster compared to other clusters, with higher values indicating better-defined clusters.
4. **Optimal Cluster Selection**:
   - The script identifies the optimal number of clusters based on the highest silhouette score. The KMeans algorithm is then rerun with this optimal cluster number.
5. **Assigning Clusters**:
   - The resulting cluster labels are added to the original GeoDataFrame as a new column, "cluster."
6. **Visualization**:
   - A plot is generated to visualize the silhouette scores for different cluster counts, helping to visually determine the optimal number of clusters.
7. **Exporting Results**:
   - The GeoDataFrame, now with assigned clusters, is saved to a new GeoPackage (`SDM24_Imredd_filtered_clusters.gpkg`) for further analysis or visualization.
### Output:
- A new GeoPackage (`SDM24_Imredd_filtered_clusters.gpkg`) containing the clustered buildings, with a new column "cluster" indicating the assigned cluster label for each building.
- A plot visualizing the silhouette scores for different numbers of clusters.

---

## Module 3
This Python script is a comprehensive geospatial analysis and visualization tool that performs various tasks, including loading GeoPackage files, generating maps, and analyzing clustered data. Below is an overview of the key features and functionalities:
### Key steps:
1. **GeoPackage File Handling:**
   - The script utilizes GeoPandas to load and process GeoPackage files.
   - Layers from the files (`SDM24_Imredd_filtered.gpkg` and `SDM24_Imredd_filtered_clusters.gpkg`) are loaded for analysis.
2. **Data Visualization:**
   - **Multi-Layer Map Visualization:** Generates a panel with three maps representing:
     - Roads
     - Points of Interest
     - Residential Buildings
   - Overlays these layers with a base map using the `contextily` library.
   - Includes legends, titles, and a clean layout.
   - **Composite Map:** Combines multiple layers (roads, points of interest, and residential buildings) on a single map. Enhancements include:
     - Scale bar (via `matplotlib_scalebar`).
     - North arrow.
     - Custom legend.
3. **Cluster Analysis:**
   - Loads and visualizes cluster data from GeoPackage files.
   - Defines unique cluster colors for clear representation.
   - Generates the following visualizations:
     - **Cluster Profile Chart:** Plots synthetic profile data for each cluster.
     - **Spider Chart:** Displays data across multiple categories like Convexity, Elongation, Surface Area, etc.
     - **Cluster Distribution Map:** Highlights building clusters on a map with a custom legend and contextual base map.
4. **PDF Report Generation:**
   - Compiles all the generated figures and analyses into a structured PDF report.
   - Pages include:
     1. Cover Page.
     2. Table of Contents.
     3. Layer Maps.
     4. Silhouette Scores Visualization (for cluster evaluation).
     5. Cluster Profile & Spider Charts.
     6. Cluster Distribution Map.
   - Outputs the report as `geospatial_analysis_report.pdf`.
5. **Customization:**
   - Cluster data is flexible and supports user-defined inputs for visualization.
   - Base maps and map projections can be tailored to specific requirements.
### Usage
1. Place the required GeoPackage files (`SDM24_Imredd_filtered.gpkg` and `SDM24_Imredd_filtered_clusters.gpkg`) in the working directory.
2. Run the script to generate visualizations and analyses.
3. Check the `geospatial_analysis_report.pdf` file for the compiled results.
### Visual Outputs
The script produces the following visualizations:
- Individual layer maps.
- Cluster analysis charts (profile and spider charts).
- Comprehensive cluster distribution map.

