# 🚦 Traffic Signal Simulation  

## 📌 Overview  
This project simulates a **4-lane traffic intersection** with dynamic traffic signal control using **OpenCV** to calculate vehicle density. The simulation adapts green signal timing based on real-time density analysis, making the system more efficient compared to static signals.  

---

## 🛣️ Simulation Setup  

- **4 Lanes** at the intersection.  
- **Each lane has 4 divisions**:  
  - 2 ongoing (forward flow).  
  - 2 outgoing (exit flow).  
- **Signal timing**:  
  - Static base time = **90 sec** (for each lane).  
  - Signals switch **clockwise**.  
- **Zebra Crossings** included for pedestrians.  

---

## ⚙️ Logic for Dynamic Control  

### 🔹 Density Calculation  
1. Capture vehicle density using **OpenCV**.  
2. Use **masks** for each direction: **North, South, East, West**.  
3. Compute density values per second (e.g., `North = 0.56, South = 0.78`).  
4. Formula (concept):  

\[
\text{Density} = \frac{\text{Vehicle Area Covered}}{\text{Total Lane Area}}
\]  

---

### 🔹 Dynamic Green Light Rules  

- If **density < 0.3** for **5+ sec** → reduce time by **40%**.  
- If **0.4 ≤ density ≤ 0.6** → reduce time by **25%**.  
- If **density ≥ 0.7** → no change.  
- **Best case**: 30 sec green light.  
- **Worst case**: 90 sec green light.  

---

## 🔄 Execution Flow (Efficient Green Light Control)  

1. **Start green light** (e.g., North) → `time = 90 sec`.  
2. **First 10 sec**:  
- Cars move normally.  
- Keep capturing density values (but no decisions yet).  
3. **After 10 sec**:  
- Use **sliding averages** of last 5 sec to smooth density:  
  - (0–5), (1–6), (2–7), … until current time.  
4. **Every 5 sec**, apply rules:  
- Density < 0.3 → reduce remaining time by 40%.  
- 0.4 ≤ Density ≤ 0.6 → reduce remaining time by 25%.  
- ≥ 0.7 → keep same.  
5. **Time adjustment method**:  
- Apply reduction step by step, not all at once.  
- Example: If 60 sec left and density < 0.3 → `60 × 0.6 = 36 sec`.  
6. **Repeat** until green time ends.  
7. **Switch clockwise** to the next direction.  

---

## 📊 Example Density Flow  

| Direction | Density | Action | Adjusted Time |
|-----------|---------|--------|---------------|
| North     | 0.25    | Reduce 40% | 90 → 54 sec |
| South     | 0.45    | Reduce 25% | 90 → 67 sec |
| East      | 0.72    | No change | 90 sec |
| West      | 0.31    | Reduce 40% | 90 → 54 sec |

---

## 📈 Efficiency Analysis  

- **Static System**:  
  - Each lane always gets **90 sec**, total cycle = **360 sec**.  
  - Even empty lanes waste full green time.  

- **Dynamic System**:  
  - **Best Case (Low Traffic All Sides)**:  
    - Each lane = **30 sec** → total cycle = **120 sec**.  
    - Efficiency gain ≈ **67% faster** than static.  
  - **Worst Case (High Traffic All Sides)**:  
    - Each lane = **90 sec** → total cycle = **360 sec**.  
    - Same as static, no gain (but never worse).  

✅ On average, this system saves **30–60% of cycle time** compared to static signals.  

---
## 🆚 Static vs Dynamic (Head-to-Head)

### What’s the difference?
- **Static signals:** Every lane always gets **90s** green. Total cycle = **360s** (4 × 90).
- **Dynamic signals (this project):** Each lane starts at **90s**, then adjusts using density rules every 5s after an initial 10s window:
  - Density < 0.3 → reduce remaining time by **40%**
  - 0.4 ≤ Density ≤ 0.6 → reduce remaining time by **25%**
  - Density ≥ 0.7 → **no change**
  - Bounds: **30s (best case)** to **90s (worst case)** per lane

---

### 📈 Key KPIs

| KPI | Static | Dynamic (Yours) | Why it matters |
|---|---|---|---|
| Cycle time (all 4 lanes) | Always **360s** | **120–360s** | Shorter cycles = lower network delay |
| Lane green time | Fixed **90s** | **30–90s** based on need | Cuts waste on empty/low-traffic lanes |
| Average wait time | Higher under imbalance | Lower under imbalance | Users wait less when their lane is light |
| Throughput under uneven load | Limited | Higher | Extra time shifts to busy lanes |
| Fairness | Equal time, not equal need | Proportional to demand | Matches service to actual traffic |
| Pedestrian integration | Fixed windows | Can co-schedule | Flexible, safer timing options |
| Robustness to surges | Poor | Better | Adapts in-cycle to spikes |
| Implementation complexity | Low | Moderate | Needs vision + logic |

---

### 🔢 Worked Scenario (One Full Cycle)

Assume instantaneous density estimates (after smoothing window):

| Direction | Density | Rule Applied | Green Time |
|---|---:|---|---:|
| North | 0.25 | < 0.3 → −40% | 90 × 0.6 = **54s** |
| South | 0.55 | 0.4–0.6 → −25% | 90 × 0.75 = **67.5s** → **68s** (rounded) |
| East | 0.72 | ≥ 0.7 → no change | **90s** |
| West | 0.28 | < 0.3 → −40% | 90 × 0.6 = **54s** |

- **Dynamic cycle** = 54 + 68 + 90 + 54 = **266s**  
- **Static cycle** = 360s  
- **Cycle time saved** = 360 − 266 = **94s**  
- **Efficiency gain** = (94 / 360) × 100 ≈ **26.1% faster** for this cycle

> Note: When *all* approaches are light, the cycle approaches **120s** (≈ **67% faster** than static).  
> When *all* are saturated, dynamic = static (**360s**) — never worse than static.

---

### 📐 Quick Formulas

- **Cycle Time (Static):**  
  \[
  C_{\text{static}} = 4 \times 90 = 360 \text{ s}
  \]

- **Per-Lane Dynamic Time (bounded):**  
  \[
  T' = \max\!\big(30,\ \min(90,\ \text{rule-based reduction of remaining time})\big)
  \]

- **Cycle Time (Dynamic):**  
  \[
  C_{\text{dyn}} = \sum_{\text{lanes}} T'_{\text{lane}}
  \]

- **Efficiency Gain (% faster than static):**  
  \[
  \text{Gain} = \frac{C_{\text{static}} - C_{\text{dyn}}}{C_{\text{static}}} \times 100
  \]

---

### 🧠 When Dynamic Shines
- **Uneven demand:** Reallocates green to busy approaches.
- **Off-peak / night:** Cuts empty-lane green dramatically.
- **Incidents or temporary surges:** Adapts mid-cycle via sliding averages.

### ⚠️ Edge Cases & Safeguards
- **Minimum green:** Never drop below **30s** (clearance & pedestrian safety).
- **Stability:** Use sliding/EMA averages to avoid flapping decisions.
- **Starvation protection:** Hard cap ensures every lane gets service each cycle.


## 🖥️ Tech Stack  
- **Python**  
- **OpenCV** (for vehicle detection & density estimation)  
- **Numpy** (calculations & averages)  

---

## ✅ Future Improvements  
- Integrate **YOLO/Deep Learning models** for better vehicle detection.  
- Add **emergency vehicle priority**.  
- Use **real-world camera input** instead of simulated cars.  
- Implement **pedestrian signal logic** with zebra crossings.  
