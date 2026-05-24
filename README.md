# Interactive Web Map with Folium

Interactive web map developed in Python using the Folium library within Google Colab/Jupyter Notebook environments.

The project creates an interactive map with:

- satellite basemap;
- real-time geolocation;
- minimap;
- layer control;
- HTML export;
- browser visualization.

---

# Overview

This notebook demonstrates the creation of an interactive GIS environment using Folium and Leaflet plugins in Python.

The generated map can be exported as an `.html` file and opened locally in any web browser.

The project was developed for studies and applications involving:

- WebGIS;
- geospatial visualization;
- interactive cartography;
- environmental analysis;
- geoprocessing applications.

---

# Main Features

## Interactive Map Creation

```python
mapaint = folium.Map(
    location=[-28.75020792, -49.11252282],
    zoom_start=16,
    control_scale=True,
    tiles=None
)
```

Creates the main interactive map centered on the study area.

---

## Real-Time Geolocation

```python
folium.plugins.LocateControl(auto_start=True).add_to(mapaint)
```

Adds a real-time geolocation button to the map.

---

## Satellite Basemap

```python
folium.TileLayer(
    tiles='https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
    attr='Esri World Imagery'
).add_to(mapaint)
```

Adds the Esri World Imagery basemap.

---

## Layer Control

```python
folium.LayerControl().add_to(mapaint)
```

Allows enabling and disabling map layers.

---

## Minimap

```python
MiniMap().add_to(mapaint)
```

Adds an interactive minimap to the map interface.

---

## HTML Export

```python
mapaint.save("Interactive_map_Export.html")
```

Exports the final map as an interactive HTML file.

---

# Libraries Used

```python
folium
```

---

# Installation

This project uses a `requirements.txt` file to install the required libraries automatically.

Install the dependencies using:

```bash
pip install -r requirements.txt
```

The `requirements.txt` file contains:

```txt
folium
```

---

## Google Colab

```python
!pip install -r requirements.txt
```

---

# Project Structure

```text
Interactive_Map/
│
├── cod.ipynb
├── requirements.txt
├── Interactive_map_Export.html
└── README.md
```

---

# How to Run

## Google Colab

1. Upload the notebook to Google Colab;
2. Upload the `requirements.txt` file;
3. Install the dependencies;
4. Run the notebook cells sequentially;
5. The HTML map will be generated automatically.

---

## Jupyter Notebook

```bash
jupyter notebook
```

Open the notebook and run all cells.

---

# Generated Output

```text
Interactive_map_Export.html
```

The generated file can be:

- opened locally;
- hosted online;
- embedded into websites;
- used as a simple WebGIS application.

---

# Technologies

- Python
- Folium
- Leaflet.js
- Google Colab
- Jupyter Notebook

---

# Applications

- environmental monitoring;
- urban analysis;
- spatial visualization;
- geoprocessing education;
- GIS dashboards;
- WebGIS development.

---

# Author

Developed by Bruna Rocha.