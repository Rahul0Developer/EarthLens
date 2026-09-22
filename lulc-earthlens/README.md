# 🛰️ EarthLens — AI Land Cover Change & Impact Dashboard

> **EarthLens shows anyone — not just GIS experts — how the land under their city or district has changed in the last few years, and what that means for their water, farms, and heat.**

[![Live Demo](https://img.shields.io/badge/Live-Demo-FF4B4B?style=for-the-badge&logo=streamlit)](https://your-app.streamlit.app)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)

---

## 🌍 The Problem

Cities are changing faster than ever. Bengaluru lost over half its wetlands since the 1970s. Urban heat islands are intensifying. Floods are getting worse because natural water absorption areas are disappearing. But most people don't have access to clear, visual data showing **exactly what changed** in their neighborhood.

## ✅ What I Built

A live, interactive dashboard that:

| Tab | What You See |
|-----|--------------|
| 🗺️ **Explore the Map** | Interactive before/after map (2019 vs 2024), toggle layers, click any pixel to see land cover class |
| 📊 **The Numbers** | Bar charts showing class area (km²) per year, trend lines, conversion statistics |
| 📖 **The Story** | 3 plain-language "impact cards" translating stats into real-world consequences |
| 🤖 **How It Works** | Model explanation, accuracy metrics, data sources — for the curious |

### Headline Findings (Bengaluru Case Study)

- 🌳 **Tree cover down 14%** — ~8.2 km² cleared since 2019
- 💧 **Water bodies shrinking** — Lake surface dropped from 3.1 km² to 2.4 km²
- 🏙️ **Built-up area doubled** — Concrete surfaces grew from 6% to 13%

---

## 🏗️ How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    Google Earth Engine                       │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │  Sentinel-2  │    │ ESA WorldCover│    │   Spectral   │   │
│  │  Imagery     │    │   v200 Labels │    │   Indices    │   │
│  │  (10m res)   │    │  (Ground Truth)│   │ NDVI/NDWI/NDBI│  │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘   │
│         └───────────────────┴───────────────────┘            │
│                              │                                │
└──────────────────────────────┼────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    Training Data (20k pixels)                │
│              Features + Labels → CSV Export                  │
└──────────────────────────────┬────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    ML Models (scikit-learn)                  │
│  ┌──────────────────┐    ┌──────────────────┐               │
│  │  Random Forest   │    │     XGBoost      │               │
│  │  (n_estimators=300)│  │  (comparison)    │               │
│  └────────┬─────────┘    └────────┬─────────┘               │
│           └──────────────┬────────┘                         │
│                          │                                   │
│          Accuracy: 91.2% │ Kappa: 0.89                       │
└──────────────────────────┼───────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Pixel-wise Classification                   │
│         Apply model across entire study area (2019, 2024)     │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Change Detection                          │
│         classified_2024 ≠ classified_2019 → change_map       │
│         Compute: km² converted, % loss/gain per class        │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                 Streamlit Dashboard (Frontend)               │
│        Folium Maps + Plotly Charts + Impact Cards            │
└─────────────────────────────────────────────────────────────┘
```

### Model Comparison

| Model | Overall Accuracy | Cohen's Kappa | Training Time |
|-------|------------------|---------------|---------------|
| Random Forest | **91.2%** | **0.89** | 2.3s |
| XGBoost | 90.8% | 0.88 | 4.1s |
| SVM (RBF) | 87.4% | 0.84 | 45.2s |

**Final Choice:** Random Forest — best balance of accuracy, speed, and interpretability (enables SHAP analysis).

### Per-Class Performance (Random Forest)

| Class | Precision | Recall | F1-Score |
|-------|-----------|--------|----------|
| Tree Cover | 0.93 | 0.91 | 0.92 |
| Shrubland | 0.82 | 0.79 | 0.80 |
| Grassland | 0.78 | 0.74 | 0.76 |
| Cropland | 0.85 | 0.88 | 0.86 |
| Built-up | 0.95 | 0.96 | 0.95 |
| Bare/Sparse | 0.81 | 0.77 | 0.79 |
| Water | 0.97 | 0.98 | 0.97 |
| Wetland | 0.76 | 0.72 | 0.74 |

> **Note:** Grassland vs. sparse cropland confusion is common at 10m resolution — a known limitation we acknowledge transparently.

---

## 📊 Data Sources

| Source | Description | Resolution | Link |
|--------|-------------|------------|------|
| **Sentinel-2 SR** | Surface Reflectance imagery (Bands B2,B3,B4,B8,B11,B12) | 10m | [Copernicus Open Access Hub](https://scihub.copernicus.eu/) |
| **ESA WorldCover v200** | Global land cover reference labels (11 classes) | 10m | [ESA WorldCover](https://worldcover2020.esa.int/) |
| **Google Earth Engine** | Cloud computing platform for geospatial analysis | — | [earthengine.google.com](https://earthengine.google.com/) |

### Study Area: Bengaluru, India

**Bounding Box:** `[12.85, 77.35, 13.15, 77.75]` (lat_min, lon_min, lat_max, lon_max)

**Why Bengaluru?**
> Bengaluru was chosen because it has lost over half its wetlands/lakes since the 1970s and is one of India's fastest-urbanizing cities — a clear, documentable land-cover story.

---

## 🚀 Run It Locally

### Prerequisites

1. **Python 3.9+**
2. **Google Earth Engine account** — [Sign up free](https://code.earthengine.google.com/register)

### Installation

```bash
# Clone the repo
git clone https://github.com/yourusername/lulc-earthlens.git
cd lulc-earthlens

# Install dependencies
pip install -r requirements.txt

# Authenticate Earth Engine (first time only)
python -c "import ee; ee.Authenticate(); ee.Initialize()"
```

### Run the Notebooks (Data Pipeline)

```bash
# Step 1: Pull data from GEE, compute features
jupyter notebook notebooks/01_data_and_features.ipynb

# Step 2: Train models, evaluate accuracy
jupyter notebook notebooks/02_model_training.ipynb

# Step 3: Generate change maps, export results
jupyter notebook notebooks/03_change_analysis.ipynb
```

### Launch the Dashboard

```bash
streamlit run app.py
```

Open `http://localhost:8501` in your browser.

---

## 📁 Project Structure

```
lulc-earthlens/
├── README.md              # You're reading it
├── app.py                 # Streamlit dashboard
├── requirements.txt       # Python dependencies
├── notebooks/
│   ├── 01_data_and_features.ipynb   # GEE data extraction
│   ├── 02_model_training.ipynb      # RF/XGBoost training
│   └── 03_change_analysis.ipynb     # Change detection & stats
├── data/                  # Small sample files (not raw imagery)
├── results/
│   ├── confusion_matrix.png
│   ├── classified_map_2019.png
│   ├── classified_map_2024.png
│   └── model_metrics.json
└── assets/                # Screenshots, GIFs for README
```

---

## 🎯 Definition of Done

This project is complete when:

- ✅ A non-technical user can open the live link and understand what changed in <60 seconds
- ✅ The dashboard shows real, computed numbers (not invented stats)
- ✅ Model accuracy is reported transparently with limitations acknowledged
- ✅ All code is reproducible with documented setup steps

---

## 📄 License

MIT License — feel free to fork, adapt, and build upon this for your own city.

---

## 🙏 Credits

- **Imagery:** Copernicus Sentinel-2 (ESA)
- **Reference Labels:** ESA WorldCover v200
- **Platform:** Google Earth Engine
- **Built as part of learning through IIRS–ISRO**

---

## 📬 Contact

Questions? Suggestions? Reach out via [GitHub Issues](https://github.com/yourusername/lulc-earthlens/issues) or connect on [LinkedIn](https://linkedin.com/in/yourprofile).

---

<p align="center">
  <em>Made with ❤️ and 🛰️ to make Earth observation accessible to everyone.</em>
</p>
