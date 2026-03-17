# RACEMake PitGPT Challenge


# Documentation Export: RACEMAKE Challenge Resolution

## 1. Level 1: Sorting Bug Fix
**Summary:** Resolved an issue where the coaching pipeline identified the wrong problem sector by outputting the fastest sector instead of the slowest.  

**Legacy State:** The `analyzeLap` function sorted sector findings in ascending order (`a.delta - b.delta`). This incorrectly placed the sector with the lowest time lost (or most time gained) at index `0`, skipping the actual problem sector.  

**Modifications:** Reversed the sorting logic in `analyzeLap` to descending order (`b.delta - a.delta`). This safely routes the sector with the highest positive delta to the front of the array.  

**Current State:** The lap analysis accurately flags the sector with the highest time lost, resolving the validation failure without restructuring the underlying data pattern. 

## 2. Level 2: Stint Analysis Extension
**Summary:** Extended the system to process arrays of consecutive laps (a "stint"), tracking how a driver’s issues evolve as tyres degrade. 

**Operational Logic:**
*   **Data Wrapping:** Introduced an `analyzeStint` function that takes the `ReferenceLap` and an array of `DriverLap` objects. It iterates the proven `analyzeLap` function over each lap to maintain functional purity.
*   **Pattern Tracking:** A summary engine tracks the primary issue of each lap. It compares the primary issue from the beginning of the stint (Lap 1) to the end of the stint (Lap 14).
*   **Coaching Output:** If a shift is detected (e.g., from *Late Braking* to *Traction Loss*), the engine dynamically appends a stint-level summary acknowledging the degraded traction limits and tyre wear. The system now natively supports both discrete lap analysis and continuous stint monitoring.

## 3. Level 3: Real-Time Telemetry Scaling (Architectural Review)
**Summary:** Analysis of system breaking points when scaling to 120Hz live telemetry for grids of 20+ cars, along with the required structural shifts safely documented.

**Operational Logic:**
*   **Failure Point:** The current synchronous, array-based mapping processes will block the Node/Bun event loop. Holding discrete multi-lap objects entirely in process memory will eventually trigger memory exhaustion and system crashes.
*   **Proposed Architecture:**
    1.  Transition from monolithic in-memory mapping to an **event-stream architecture** (e.g., Apache Kafka, Redis Streams) designed to easily handle 120Hz throughput.
    2.  Replace the full-lap calculation wait with **sliding window processors** via distributed instances or worker threads, making coaching instantaneous.
    3.  Offload raw historical streams into a **Time-Series Database** (like InfluxDB), caching only the live, actionable stint metadata in fast-access memory (Redis).
