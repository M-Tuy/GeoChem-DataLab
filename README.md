# GeoScreen_SE: Arsenic and Uranium Geochemical Screening

**Author:** Marius Tuyishime
**Status:** Pågående, preliminära resultat
**Data source:** SGU Markgeokemi, regional provtagning (layer: `moran_0063mm_hno3_icpms`)

---

## 1. Background

Arsenic (As) and uranium (U) occur naturally in bedrock and soils. Elevated concentrations can be environmentally relevant, depending on concentration, medium, land use, mobility, and exposure pathways.

This project builds a simple screening method to identify sample points with relatively high As and U concentrations in Swedish till soils, using SGU's regional soil geochemistry data. It flags candidate areas for further investigation, especially where As and U are both elevated at the same point. This is a first step.

Future versions will add sediment, surface soil, pH, land use, and bedrock geology, and move toward spatial interpolation and machine learning, shifting from identifying relative highs to predicting risk. Later stages will narrow the scope to a smaller region for more detailed, site-specific analysis.

---

## 2. Data and Method

**Data source:**
- Product: [markgeokemi-regional](https://www.sgu.se/produkter-och-tjanster/geologiska-data/geokemi--geologiska-data/markgeokemi/), regional provtagning (SGU)
- Layer: `moran_0063mm_hno3_icpms`
- Medium: Till (moraine), fine fraction <0.063 mm
- Method: HNO₃ leach / ICP-MS
- CRS: SWEREF99 TM (EPSG:3006)

**Cleaning steps:**
1. Dropped rows missing sample ID, coordinates, or geometry.
2. Confirmed CRS as EPSG:3006.
3. Filtered to rows with valid, positive values for As, Fe, Ca, Al, and U. Excluded SGU's "0 = not analyzed" placeholder and below-detection-limit values (stored as negative numbers, per SGU documentation).

**Output:** 27,981 analysis-ready sample points (original Swedish dataset: 28,471 points).

**Percentile-based thresholds:**
- 75th percentile → "elevated"
- 95th percentile → "high" / hotspot

The computed thresholds, in ppm:

| Element | 75th percentile | 95th percentile |
|---|---|---|
| As | 3.90 | 13.00 |
| U | 2.40 | 4.60 |

**Combined prioritization:** Priority 1 = both As and U in top 5%. Priority 2 = either As or U in top 5%. Background = neither.

**Regulatory guideline values (Naturvårdsverket):** for arsenic, generic guideline values for contaminated land are 10 mg/kg TS (KM, sensitive land use, e.g. homes/schools/gardens) and 25 mg/kg TS (MKM, less sensitive land use, e.g. offices/industry). The KM value corresponds to the 90th percentile of SGU's regional measurements. The dataset's 95th percentile for As (13.00 ppm) sits between KM and MKM.

**Supplementary geological layer, alum shale (Alunskiffer):**
- Product: [Berggrund 1:50 000-1:250 000](https://www.sgu.se/produkter-och-tjanster/geologiska-data/berggrund--geologiska-data/berggrund/) (SGU bedrock geology)
- Direct bulk download (GeoPackage, CC0 license): [berggrund50k-250k.zip](https://resource.sgu.se/data/oppnadata/berggrund50k-250k/berggrund50k-250k.zip)
- Alum shale polygons (n = 123) were extracted from this layer, reprojected to EPSG:3006, and overlaid on the Priority 1/Priority 2 results (see Figure 5).

---

## 3. Results

**Figure 1: Arsenic screening**
As hotspots (top 5%, red) among all cleaned sample points.
![Arsenic Screening Map](results/arsenic_screening_map.png)

**Figure 2: Uranium screening**
U hotspots (top 5%, blue) among all cleaned sample points.
![Uranium Screening Map](results/uranium_screening_map.png)

**Figure 3: Combined As + U priority**
Priority 1 (red) and Priority 2 (orange) points overlaid on the full sample set.
![Combined Priority Map](results/sweden_as_u_priority_map.png)

**Figure 4: Arsenic vs. Naturvårdsverket guideline values (KM/MKM)**
Points classified against Sweden's legal contaminated-land guideline values.
![Arsenic Guideline Screening Map](results/arsenic_guideline_screening_map.png)

**Figure 5: Combined As + U priority with alum shale (Alunskiffer) overlay**
Priority 1 and Priority 2 points, with mapped alum shale bedrock polygons (black, n=123).
![Alum Shale Priority Overlay](results/alum_shale_priority_overlay.png)

**Point counts per class (relative percentile screening, n=27,981):**

| Class | Count | % of total |
|---|---|---|
| As hotspot (top 5%) | 1,394 | 4.98% |
| U hotspot (top 5%) | 1,370 | 4.90% |
| Priority 1 (both hot) | 137 | 0.49% |
| Priority 2 (either hot) | 2,490 | 8.90% |
| Background | 25,354 | 90.61% |

**Point counts per class (Arsenic vs. Naturvårdsverket guideline values, n=27,981):**

| Class | Count | % of total |
|---|---|---|
| Below KM (<10 ppm) | 25,862 | 92.42% |
| KM-MKM (10-25 ppm) | 1,752 | 6.26% |
| Above MKM (>25 ppm) | 367 | 1.31% |

---

## 4. Interpretation

The screening shows clear geographic clustering of high As and U values, mainly in northern and parts of central Sweden. This method identifies points that are high relative to this dataset. It does not confirm regulatory exceedance on its own.

Priority 1 points (both As and U high) are the strongest signal, since two independent measurements agree. Priority 2 points (only one element high) are weaker and need more context before being treated as risk areas.

**Guideline comparison:** 7.6% of points (2,119 of 27,981) exceed the Naturvårdsverket KM guideline value (10 ppm As). This is more than the 5% implied by the relative screening threshold, because KM sits below this dataset's own 95th percentile (13.00 ppm). Guideline-based screening identifies more points than percentile-based screening.

**Alum shale overlay (Figure 5):** Priority 1 points in northern and central Sweden appear near mapped alum shale polygons on the map. Alum shale is known to carry elevated uranium and sulfide-hosted trace metals, including arsenic (Falk et al., 2006; Lecomte et al., 2017). Lecomte et al. (2017) also found that alum shale in northern Sweden underwent stronger metamorphism than in the south, which mobilized uranium into new minerals there. This gives a plausible geological reason for a north-south difference, but it has not been tested against this dataset.

This spatial pattern is a hypothesis, not a confirmed finding. Only 5 of 137 Priority 1 points (3.6%) fall inside a mapped shale polygon. This number alone does not confirm or rule out a real association, since no distance-based test or random baseline has been run. Section 6 lists this as a planned next step.

Southern clusters (Skåne, Gotland) show little nearby shale in this dataset. The cause is not known.

This is a screening tool, not a risk map. It does not account for bioavailability, land use, or exposure pathways.

---

## 5. Limitations

- Classification is based on total/leachable concentrations, not bioavailability or chemical speciation.
- Relative percentiles show anomalies within this dataset, not regulatory exceedances.
- No Naturvårdsverket generic guideline value exists for uranium in soil.
- Local land use, exposure pathways, and receptor information are not included in this version.
- Groundwater conditions, pH, redox environment, and carbonate chemistry are not part of the model.
- The `> 0` filter excludes "not analyzed" placeholders and below-detection-limit values, slightly narrowing the dataset.
- The alum shale overlay is untested. It does not account for glacial transport distance, ice-flow direction, or shale occurrences missing from this dataset.

---

## 6. Next Steps

1. Compare against relevant guideline values for uranium (drinking water or radiological guidance).
2. Add land use or receptor information: residential areas, agricultural land, drinking water interests, private wells.
3. Keep soil/till, sediment, surface water, and groundwater separate, since risk logic differs by medium.
4. Use Fe, Al, Ca, pH to interpret binding, mobility, and transport.
5. Test county-level thresholds instead of one national threshold, to account for natural geological background.
6. Consider log-transformation before percentile calculation, given the right-skewed nature of geochemical data.
7. Document data coverage, analytical method, sample medium, and spatial resolution in future versions.
8. Run a distance-based spatial join: compute nearest-distance to shale for each point, and compare Priority 1, Priority 2, and Background classes using a statistical test (e.g. Mann-Whitney U).

---

## 7. Way Forward: Toward a Machine Learning-Based Model

The current version is a percentile-based screening method. Planned extensions:

**More data:**
- Additional SGU layers (`moran_2mm_hno3_icpms`, sediment, surface soil) to compare As/U across sample media.
- Additional geochemical variables: pH, S, rare earth elements.
- External layers: bedrock geology, land use, drinking water proximity.

**Toward machine learning:**
KNN regression will test whether Ca concentration can help predict U concentration at unsampled points. Two versions are planned: a manual k=1 implementation, and a `scikit-learn` version sweeping k from 1 to 70 to observe the bias-variance tradeoff.

This is exploratory work, separate from the current screening pipeline. It answers a different question (can one variable predict another) than the percentile method (which points are relatively elevated). Planned future steps:

- Spatial interpolation (kriging, IDW, or KNN) to move from points to continuous surfaces.
- Regression/classification using Fe, Ca, Al, pH as predictors of mobility.
- Train/test validation for future predictive models.
- A risk-informed prioritization model combining geochemical prediction, land use, and exposure pathways.

---

## 8. References

- Andersson, A., Dahlman, B., Gee, D.G., Snäll, S. (1985). *The Scandinavian Alum Shales.* Sveriges Geologiska Undersökning, Ca 56.
- Falk, H., Lavergren, U., Bergbäck, B. (2006). Metal mobility in alum shale from Öland, Sweden. *Journal of Geochemical Exploration*, 90(3), 157-165.
- Lecomte, A., Cathelineau, M., Michels, R., Peiffert, C., Brouand, M. (2017). Uranium mineralization in the Alum Shale Formation (Sweden). *Ore Geology Reviews*, 88, 71-98.
- Naturvårdsverket. *Riktvärden för förorenad mark, Rapport 5976.* https://www.naturvardsverket.se/Stod-i-miljoarbetet/Vagledningar/Fororenade-omraden/Riktvarden-for-fororenad-mark/
- SGU. *Markgeokemi, regional provtagning.* https://www.sgu.se/produkter-och-tjanster/geologiska-data/geokemi--geologiska-data/markgeokemi/