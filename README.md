#  AI Lab Project: Uniform Cost Search (UCS) vs. A* Search on the Stanford SNAP Gowalla Dataset

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)
![Platform](https://img.shields.io/badge/Platform-Google%20Colab-orange?logo=googlecolab)
![Dataset](https://img.shields.io/badge/Dataset-Stanford%20SNAP%20Gowalla-red)
![Status](https://img.shields.io/badge/Accuracy-100%25%20Optimal-brightgreen)

## Project Overview
This repository contains the implementation, data engineering pipeline, performance benchmarking, and visual simulation of two classical Artificial Intelligence graph search algorithms—**Uniform Cost Search (UCS)** and **A* (A-Star) Search**—applied to the real-world **Gowalla Location-Based Social Network Dataset** from Stanford SNAP.

Because the raw Gowalla dataset separates social friendship connections (`edges`) from geographical check-in coordinates (`totalCheckins`), this project implements a complete **Data Cleaning & Merging Pipeline** to construct a spatially weighted social graph (`graph_weighted`) and admissible Euclidean heuristics ($h(n)$) for informed pathfinding.

---

## 📂 Dataset Information
**Source:** [Stanford Large Network Dataset Collection (SNAP) — Gowalla](https://snap.stanford.edu/data/loc-gowalla.html)

The raw dataset consists of two separate files that must be cleaned and merged:
1. **`loc-gowalla_edges.txt`** (`1,900,654` raw edges): Contains undirected friendship pairs (`user_u`, `user_v`) without edge weights or geographical coordinates.
2. **`loc-gowalla_totalCheckins.txt`** (`6,442,890` raw check-ins): Contains user check-in histories including `user_id`, `timestamp`, `latitude`, `longitude`, and `location_id`.

---

## ⚙️ Data Processing & Merging Pipeline
Neither UCS nor A* can run on the raw edges file alone because **UCS requires positive edge weights (step costs)** and **A* additionally requires spatial coordinates to compute heuristic estimates**. Our preprocessing pipeline performs the following steps:

1. **Data Cleaning (Check-ins):**
   - Extracts `user_id`, `lat` (Latitude), and `lon` (Longitude).
   - Drops corrupt/missing rows (`NaN`) and filters out invalid GPS coordinates outside $\text{Lat} \in [-90^\circ, 90^\circ]$ and $\text{Lon} \in [-180^\circ, 180^\circ]$.
   - **Deduplication:** Retains each user's primary (most recent) check-in coordinate (`107,092` unique users with valid GPS locations).
2. **Subgraph Filtering:**
   - Filters active nodes (`user_id < 2000`) to construct a dense, realistic social-geographic subgraph suitable for list-based $O(N^2)$ frontier benchmarking in Google Colab.
3. **Dataset Merging (Relational Inner Join):**
   - Performs a two-stage inner join matching `user_u` and `user_v` from the Edges dataset with their respective `(lat_u, lon_u)` and `(lat_v, lon_v)` coordinates from the cleaned Check-ins dataset.
4. **Edge Weight & Heuristic Formulation:**
   - **Edge Weight $c(u, v)$:** Computed as the Euclidean distance between connected users:
     $$w(u, v) = \sqrt{(\text{lat}_u - \text{lat}_v)^2 + (\text{lon}_u - \text{lon}_v)^2}$$
   - **A* Heuristic $h(n)$:** Computed as the straight-line Euclidean distance from any node $n$ to the `goal` node:
     $$h(n) = \sqrt{(\text{lat}_n - \text{lat}_{\text{goal}})^2 + (\text{lon}_n - \text{lon}_{\text{goal}})^2}$$
   - Since the straight-line distance never overestimates the actual multi-hop graph distance, $h(n)$ is **admissible and consistent**, guaranteeing **100% path optimality**.

---

##  Implemented Search Algorithms

### 1. Uniform Cost Search (UCS) — *Uninformed / Blind Search*
- Expands nodes strictly in order of cumulative path cost $g(n)$ from the `start` node.
- Has no knowledge of the `goal` node's direction (`h(n) = 0`), causing it to explore radially in all directions.
- Guarantees the optimal minimum-cost path but suffers from high execution time and memory consumption.

### 2. A* (A-Star) Search — *Informed / Heuristic Search*
- Evaluates nodes using the evaluation function:
  $$f(n) = g(n) + h(n)$$
- Combines the exact cost so far $g(n)$ with the estimated remaining distance $h(n)$ to prioritize nodes moving toward the `goal`.
- Achieves the **exact same optimal path cost as UCS** while expanding a fraction of the nodes.

---

## 📊 Experimental Results & Performance Comparison

Both algorithms were evaluated across **20 randomized multi-hop test cases** ($\ge 2$ hops) with Python Garbage Collection isolated (`gc.disable()`) and memory profiled via `tracemalloc`.

| Metric | Uniform Cost Search (UCS) | A* Search (Euclidean Heuristic) | Comparison / Improvement |
| :--- | :---: | :---: | :---: |
| **Path Optimality (Accuracy Score)** | `100.00%` (Baseline) | **`100.00%`** | Exact match on all 20 test cases |
| **Nodes Expanded** | High (Radial blind search) | **Low (Directed search)** | **~75% – 95% fewer nodes expanded** |
| **Execution Time (ms)** | Slow (`~50 – 380 ms`) | **Fast (`~0.5 – 15 ms`)** | **~15x – 100x+ speedup** |
| **Peak Memory Allocation (KB)** | High frontier footprint | **Minimal frontier footprint** | **~70% – 90% less RAM used** |

---

## 📈 Visualizations Included in the Notebook
1. **Dataset Inspection Tables:** Formatted previews of Dataset 1 (`user_locations`), Dataset 2 (`edges_df`), and the Merged Weighted Dataset (`merged_df`).
2. **Execution Time Bar Chart:** Side-by-side comparison of UCS vs. A* execution times across 20 multi-hop test cases.
3. **Peak Memory Allocation Bar Chart & Distribution Histogram:** Visualizes per-test-case peak memory (`KB`) and frequency distributions using `tracemalloc`.
4. **Optimality / Accuracy Verification Plot:** Overlays UCS optimal path costs with A* found costs to prove `100.0%` accuracy.
5. **18-Second Side-by-Side Search Simulation Video (`ucs_vs_astar_search_18s.mp4`):** Animated geographic plot (`Longitude` vs. `Latitude`) showing UCS expanding blindly across the map while A* locks directly onto the goal and highlights the optimal path.

---

## How to Run the Project (Google Colab)

1. **Upload the Datasets to Google Drive:**
   Upload the extracted SNAP Gowalla files to the root of your Google Drive (`MyDrive`):
   - `loc-gowalla_totalCheckins.txt`
   - `loc-gowalla_edges.txt`
2. **Open the Notebook in Google Colab:**
   Open `AI_Lab_Project.ipynb` in Google Colab.
3. **Run All Cells Sequentially:**
   - **Cell 1:** Mounts Google Drive and automatically locates both dataset files.
   - **Cell 2:** Cleans, deduplicates, filters, and merges the datasets into `merged_df`.
   - **Cell 3:** Builds the `graph_weighted` adjacency list and `compute_heuristics()` helper.
   - **Cell 4 & 5:** Defines the `ucs()` and `a_star_search()` algorithms.
   - **Cell 6 & 7:** Executes single-run and 20-case multi-hop benchmarks (Time, Nodes Expanded, Peak Memory, and Accuracy Score).
   - **Cell 8:** Generates the 4-panel performance and memory comparison charts.
   - **Cell 9:** Renders and embeds the 18-second side-by-side MP4 video animation (`ucs_vs_astar_search_18s.mp4`).

---

## 🛠️ Technologies & Libraries Used
- **Language:** Python 3
- **Data Processing:** `pandas`, `numpy`
- **Graph & Profiling:** `collections.defaultdict`, `math`, `time`, `tracemalloc`, `gc`
- **Visualization & Animation:** `matplotlib`, `ffmpeg` (HTML5 video rendering), `IPython.display`

---

##  Conclusion
This experiment demonstrates that while **Uniform Cost Search (UCS)** and **A* Search** both guarantee **100% optimal pathfinding** on spatially weighted social networks, **A* Search** drastically outperforms UCS in **execution speed**, **number of nodes expanded**, and **peak memory allocation** by leveraging an admissible geographic heuristic.
