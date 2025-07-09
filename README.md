# 🚗 Summer Analytics 2025: Real-Time Dynamic Parking Pricing System

## 📌 Overview

This project, developed as part of the **Summer Analytics 2025** course by IIT Guwahati, focuses on building a real-time dynamic pricing system for parking lots based on occupancy, demand, traffic, and competitive behavior. Using real-world-like parking data, we implemented three progressively advanced pricing models and integrated them into a **real-time streaming pipeline** using **Pathway**, along with live visualization support using **Bokeh**.

The system ingests a time-series dataset of parking lot updates and emits dynamic pricing continuously for each parking lot.

---

## 💻 Tech Stack

| Tool/Library  | Purpose                                 |
| ------------- | --------------------------------------- |
| Python        | Core programming language               |
| Pathway       | Real-time data streaming and processing |
| Pandas, NumPy | Data preprocessing                      |
| Bokeh         | Live data visualization                 |
| Git & GitHub  | Version control and repository sharing  |

---

## 🧱 Architecture Diagram

```
                      ┌──────────────────────────┐
                      │      dataset.csv         │
                      │ (Simulated real-time)    │
                      └────────────┬─────────────┘
                                   ↓
                        ┌─────────────────────┐
                        │  Pathway Stream     │
                        │  (Streaming Mode)   │
                        └────────┬────────────┘
                                 ↓
                ┌─────────────────────────────────────┐
                │   Model 1, 2, 3 Logic Modules        │
                │   with Dynamic Pricing + Rerouting  │
                └────────┬────────────────────────────┘
                         ↓
              ┌─────────────────────┐
              │ JSON Output Stream  │
              └────────┬────────────┘
                       ↓
                ┌──────────────┐
                │   Bokeh UI   │◄──── Live Streaming Line Chart
                └──────────────┘
```

---

## 🔁 Project Workflow & Architecture

### 📥 1. **Data Ingestion**
- Parking updates were streamed row-by-row using Pathway’s `csv.read(..., mode='streaming')`.
- Each record contains timestamped information about occupancy, traffic, vehicle type, etc.

### 📊 2. **Model Logic**

We implemented three models:

#### 🔹 Model 1: Linear Pricing (Basic)
- Price increases linearly based on the occupancy ratio.
- Stateless update formula.

#### 🔹 Model 2: Demand-Based Pricing (Advanced)
- Adds complexity by considering:
  - Occupancy
  - TrafficLevel
  - QueueLength
  - Vehicle type weight
  - IsSpecialDay
- Demand is normalized per lot and used to scale pricing.

#### 🔹 Model 3: Competitive Pricing + Rerouting (Business Smart)
- Builds on Model 2 but adds **location intelligence** using latitude & longitude.
- Nearby lots within ~2 km are considered competitors.
- Price is adjusted based on competition:
  - If full and nearby lots are cheaper → reduce price and **suggest reroute**
  - If nearby lots are expensive → increase price to capture demand

### 📡 3. **Streaming Pipeline (Pathway)**
- Data is processed on-the-fly using Pathway.
- Stateful transformations are used to track and update pricing logic per lot.
- Emitted output is streamed to JSON or integrated with Bokeh.

### 📈 4. **Visualization (Bokeh)**
- A real-time line plot of `Model3_Price` vs. `TimeStep` is streamed via Bokeh.
- Supports tracking multiple lots.

---

## ✅ Model Assumptions

### 🔹 Model 1 Assumptions (Linear Pricing)
- Base Price at 8:00 AM is ₹10
- Alpha (α) = 2.0 (price sensitivity factor)
- Price is updated per parking lot (`SystemCodeNumber`) for each day over 18 time steps (8:00 AM to 4:30 PM)
- Time steps are 30-minute intervals derived from `LastUpdatedTime`
- Price update formula:

```
Price_{t+1} = Price_t + α * (Occupancy / Capacity)
```

---

### 🔹 Model 2 Assumptions (Demand-Based Pricing)
- Base Price = ₹10
- Alpha (α) = 2.0 → weight for Occupancy/Capacity
- Beta (β) = 0.3 → weight for QueueLength
- Gamma (γ) = 0.5 → negative weight for TrafficLevel
- Delta (δ) = 1.0 → weight for IsSpecialDay
- Epsilon (ε) = 0.2 → weight for VehicleWeight
- Lambda (λ) = 0.5 → scaling factor for demand impact
- Vehicle weights: Car = 1.0, Bike = 0.5, Truck = 1.5
- Demand normalized per lot, price:

```
Price_t = BasePrice * (1 + λ * NormalizedDemand)
```

---

### 🔹 Model 3 Assumptions (Competitive Pricing + Rerouting)
- Builds on Model 2 using `Model2_Price` as base
- Adds geo-competitive behavior:
  - Lots within ~2 km (0.02 deg) are competitors
- ETA (η) = 0.3 → price influence factor
- Logic:
  - If ≥90% full and neighbors are cheaper → reduce price & **suggest reroute** to cheaper nearby lot
  - If neighbors are more expensive → raise price slightly
- Rerouting is only applied in Model 3
- Output columns: `Model3_Price`, `SuggestedRerouteTo`

---

## 📎 Notes
- Model 3 is the only model **integrated with real-time streaming + Bokeh**.
- Models 1 and 2 can also be simulated in real-time but were initially prototyped offline.
- All code is self-contained and reproducible in Google Colab.

---

