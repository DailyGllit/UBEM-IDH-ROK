# National-Scale UBEM with Intrinsic Degree-Hour (IDH) Method in Republic of Korea [UBEM-IDH-ROK]

## Introduction
This repository contains Python scripts and visualized data for the **Urban Building Energy Modeling (UBEM)** project targeting the Republic of Korea. 

Utilizing the **Intrinsic Degree-Hour (IDH)** method, this project analyzes the energy performance of approximately **1.6 million buildings** nationwide. Unlike traditional degree-day methods that use fixed base temperatures (e.g., 18.3°C), the IDH approach identifies the specific **Optimal Base Temperature ($T_{b,opt}$)** for each individual building based on its monthly energy consumption patterns.

## Key Features

### 1. Minimal Data Requirement
* **Overcoming Data Scarcity:** Analyzes building energy performance using only two variables: **Monthly Energy Bill** and **Outdoor Temperature**.
* **Accessibility:** Eliminates the need for complex physical datasets (e.g., 3D geometry, envelope insulation details) which are often unavailable at the urban scale.

### 2. Physics-Informed Interpretability
* **Beyond Black-Box:** Unlike simple statistical correlations, this method derives meaningful physical parameters:
    * **$T_{b,opt}$:** The unique 'Balance Point Temperature' for each building.
    * **$U''$:** The overall heat loss coefficient (Insulation performance).
* **Behavioral Insight:** Captures actual occupant behavior (e.g., heating preferences) rather than relying on standard design assumptions.

### 3. Scalability to National Level
* **Massive Coverage:** Successfully processed **1.2+ million buildings** across South Korea by leveraging the computational efficiency of the algorithm.
* **Spatio-Temporal Integration:** Integrated local weather data from **146 stations** to reflect precise local climate conditions for every administrative district (SIGUNGU).

---

##  Methodology: The IDH Method

The core concept is to find the unique **Balance Point Temperature(IDH)** for each building where heating/cooling is not required. (Ahn et al., 2025)

### 1. Governing Equation
The monthly energy consumption ($E_m$) is modeled as:

$$E_m = U'' \times IDH(T_{b,opt}) + E_{base}$$

Where:
* **$T_{b,opt}$**: Optimal Base Temperature (Search Range: $10^\circ C \sim 30^\circ C$)
* **$U''$**: Total Energy Performance Coefficient (Slope) indicating insulation/efficiency.
* **$IDH$**: Intrinsic Degree-Hours calculated from hourly weather data.
* **$E_{base}$**: Base Load (Intercept), representing weather-independent energy usage.

### 2. Optimization Algorithm
The algorithm iterates through a range of potential base temperatures ($10^\circ C$ to $30^\circ C$) for each building to find the specific $T_b$ that maximizes the **Coefficient of Determination ($R^2$)**.

---

## Repository Structure

```bash
├── data/
│   ├── raw/                # Original Energy & Weather Data (Private)
│   └── processed/          # Cleaned datasets & IDH lookup tables (Private)
├── src/
│   ├── code_dataprep.ipynb      # Scripts for loading and merging data
│   ├── code_regression.ipynb   # Calculate IDH / regression &optimization
│   └── code_visualize.ipynb  # Review and visualize final data
├── results/
│   ├── UBEM_Result_Final.csv  # Final output (Private)
│   └── figures/            # Visualization plots
└── README.md
```

## Outputs & Results

The final analysis generates a CSV file containing IDH model parameters for each building.

### 1. Result File Structure (`UBEM_Result_Final.csv`)

| Column | Description | Data Type |
| :--- | :--- | :--- |
| **`PNU`** | Unique Building Identifier (19-digit code) | String |
| **`STN_ID`** | Weather Station ID (KMA) | Integer |
| **`Tb_opt`** | Optimal Base Temperature ($^\circ C$) | Float |
| **`U_coeff`** | Total Energy Performance Coefficient (Slope) | Float |
| **`E_base`** | Base Energy Load (Intercept) | Float |
| **`R2`** | Model Accuracy Score ($0.0 \sim 1.0$) | Float |
| **`Sample_N`** | Number of monthly data points used | Integer |

### 2. Result

**First Analysis** (Only R2>0)
```bash
>> Original Buildings : 1,233,505
>> Valid Buildings : 1,226,227 (R2 > 0)
============================================================
[IDH Modeling First Report]
============================================================
1. Success Rate : 99.4% (Valid Buildings / Every Building)
2. Model Accuracy Average : 0.5934 (R2 Score)
3. Highly Trusted Buildings : 497,684 (40.6% is R2 >= 0.7)
------------------------------------------------------------
4. The Most Frequent IDH : 10°C
5. Average IDH : 18.38°C
------------------------------------------------------------
6. Average Thermal Insulation Performance (U) : 1.32
============================================================
```

**Second Analysis** (Only 10<IDH<29 & 0<U<Top5%)
```bash
>> First Valid Buildings : 1,226,227
>> Second Valid Buildings : 845,952
============================================================
[IDH Modeling Second Report]
============================================================
1. Success Rate : 69.0% (2nd Valid Buildings / 1st Valid Buildings)
2. Refined Model Accuracy Average : 0.6254 (R2 Score)
3. Highly Trusted Buildings : 385,920 (45.6% is R2 >= 0.7)
------------------------------------------------------------
4. The Most Frequent IDH : 21°C
5. Average IDH : 18.50°C
------------------------------------------------------------
6. Average Thermal Insulation Performance (U) : 0.29
============================================================
```
* **Key Finding**: After refinement, the most frequent base temperature shifted to 21°C. This indicates that Korean buildings tend to start heating at higher outdoor temperatures compared to the conventional standard of 18.3°C (65°F).

  

## Limitations & Future Work
### Limitations
* **Building Usage Classification:** The current analysis applies a unified model across all building types (Residential, Commercial, Industrial, etc.) without distinction. This may include non-heated spaces (e.g., warehouses), leading to statistical outliers.
* **Spatial Granularity:** Due to the massive dataset size, the analysis focuses on administrative districts (SIGUNGU) rather than precise geospatial coordinates (Latitude/Longitude), limiting detailed spatial interpretation.

### Future Plans
* **Targeted Spatial Analysis:** Future research will narrow the spatial scope to specific cities (e.g., Seoul, Busan) to conduct high-resolution spatial analysis.
* **Multidimensional Integration:**
    * **Building Registers:** Integrating detailed building ledgers to classify energy patterns by usage (Apartments, Offices, Retail).
    * **Socio-Economic Factors:** Analyzing correlations with regional income levels, population density, and building age.
    * **Urban Context:** Considering micro-climate factors such as the Urban Heat Island (UHI) effect.
---

## Requirements

To run this project, you need the following environment:

* **Language:** Python 3.8+
* **Libraries:** pip install pandas numpy scipy scikit-learn geopandas tqdm requests matplotlib seaborn

## Data Sources

This project utilizes official public datasets from the South Korean government.
**Note:** Due to data privacy and redistribution policies, the raw datasets are not included in this repository.

### 1. Building Energy Consumption Data
* **Source:** **Architecture HUB (건축데이터 민간개방 시스템)**
* **URL:** [https://open.eais.go.kr/](https://open.eais.go.kr/)
* **Description:** Monthly electricity and gas consumption data for individual buildings, provided by the Ministry of Land, Infrastructure and Transport (MOLIT).

### 2. Meteorological Data
* **Source:** **Korea Meteorological Administration (KMA) Weather Data Service**
* **URL:** [https://data.kma.go.kr/](https://data.kma.go.kr/)
* **Description:** Hourly outdoor temperature data collected from 146 Synoptic Weather Observation Systems (ASOS) nationwide.

### 3. Geospatial Data (Administrative Divisions)
* **Source:** **Statistics Korea (KOSTAT) - SGIS**
* **Repository:** [southkorea/southkorea-maps](https://github.com/southkorea/southkorea-maps)
* **Description:** GeoJSON data for South Korean municipal boundaries (2013 version), originally provided by the Statistical Geographic Information Service (SGIS) and simplified by the open-source community.


## Data Privacy & License

### Code License
* This project is licensed under the MIT License.
* The methodology of the algorithm is based on **Ki Uhn Ahn, Kim, Deuk-Woo and Kim Hye-Gi. (2025). Intrinsic Degree Hour (IDH) Methodology for Data-Driven Urban Building Energy Modeling (UBEM). 한국건축친환경설비학회 논문집, 19(4), 180-190.**

### Data Disclaimer
* The raw building energy data and temperature data are confidential and were obtained for specific research purposes.
* Users wishing to replicate this study must obtain their own access permissions from the relevant data providers.
