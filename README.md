# ⚡ Smart Surveillance & Vehicle Trajectory Engine

A distributed, edge-to-cloud ANPR (Automatic Number Plate Recognition) and geospatial vehicle tracking system. The platform ingests real-time video feeds, runs edge-based deep learning inference, triggers sub-millisecond watchlist alerts via WebSockets, and reconstructs vehicle paths on an interactive GIS map.

---

## 🏗️ Technical Approach: End-to-End System Workflow & Architecture

```text
[ CCTV / Camera Feed ]
          │
          ▼
[ Edge Vision Pipeline: OpenCV ➔ YOLOv8 ➔ PaddleOCR ]
          │ (Lightweight JSON Metadata: ~150 Bytes)
          ▼
[ Central Backend: FastAPI Server ]
     ├── Check Watchlist In-Memory (O(1) Set) ──► [ Instant WebSocket Alert ] ──► [ Admin Dashboard ]
     └── Insert Record ──► [ MySQL Database ]
                                 │
     ┌───────────────────────────┴───────────────────────────┐
     ▼                                                       ▼
[ Trajectory Reconstruction Query ]             [ Macro Traffic Flow Analytics ]
(`ORDER BY sighted_at ASC`)                      (Density / Volume Aggregations)
     │                                                       │
     └───────────────────────────┬───────────────────────────┘
                                 ▼ (REST API / JSON)
                [ Frontend: Leaflet.js & Tailwind UI ]
                (Markers, Trajectory Polylines & Heatmaps)
```

---

## ⚙️ Backend Architecture & Processing

### Edge Vision & OCR Pipeline
* **Frame Ingestion:** OpenCV captures live video frames from geographically distributed cameras or RTSP streams.
* **Plate Localization:** A fine-tuned YOLOv8 model scans each frame, predicts bounding box coordinates `[x1, y1, x2, y2]`, and isolates the license plate region.
* **Text Extraction:** The cropped plate image is passed directly to PaddleOCR, which handles character segmentation and alphanumeric recognition to produce a text string (e.g., `"MP09AB1234"`).
* **Image Optimization:** Rather than storing the full 1080p frame, the system crops and compresses only the license plate snapshot (3–5 KB), saving it to local storage and referencing its path in the payload.

### Central API & Real-Time Event Engine (FastAPI)
* **Ingestion Endpoint:** Exposes a secure REST endpoint (`POST /api/detections`) to receive structured metadata payloads:
  ```json
  {
    "plate_number": "MP09AB1234",
    "camera_id": "CAM_12",
    "latitude": 22.7196,
    "longitude": 75.8577,
    "timestamp": "2026-09-04T01:00:00Z",
    "image_url": "/static/plates/cam12_snap.jpg"
  }
  ```
* **Watchlist Checking:** Cross-checks the recognized plate against a pre-loaded in-memory blacklist set in sub-millisecond time.
* **WebSocket Broadcast:** If a plate matches the watchlist, the server immediately pushes an alert payload over an active WebSocket channel to all connected admin clients.

### Data Persistence & Trajectory Engine (MySQL)
* **Sightings Table:** Stores each camera observation as an immutable discrete event row with an auto-incrementing ID.
* **Composite Indexing:** A composite B-tree index on `(plate_number, sighted_at)` enables instant retrieval across millions of rows.
* **Trajectory Engine:** An endpoint (`GET /api/trajectory/{plate_number}`) queries sightings for a specific vehicle within a target time window, returning chronologically ordered coordinate pairs via `ORDER BY sighted_at ASC`.
* **Macro Analytics:** Aggregates total vehicle counts per camera node over hourly bins to calculate volume, peak congestion, and density metrics.

---

## 🖥️ Frontend Architecture & GIS Visualization

### User Interface & Layout (HTML5 + Tailwind CSS)
* Provides a modern single-page dashboard featuring an interactive navigation bar, vehicle search panel, active camera status cards, and an emergency alert log.
* Includes a sliding modal dialog that displays the camera ID, timestamp, and cropped vehicle image whenever a watchlist alert fires.

### Client-Side Integration & Asynchronous State (Vanilla JavaScript)
* **`fetch()` API:** Handles user-driven queries (e.g., entering a license plate into the search bar) by sending asynchronous GET requests to FastAPI and injecting the returned data into dashboard tables.
* **WebSocket Client:** Maintains an uninterrupted, low-latency socket connection (`ws://localhost:8000/ws/alerts`) to receive pushed security events without requiring page refreshes.

### Geospatial & Trajectory Layer (Leaflet.js + Leaflet.heat)
* **Base Map:** Loads responsive OpenStreetMap vector tiles centered on the target city.
* **Trajectory Reconstruction:** Plots sequential camera sightings as circular red markers (`L.circleMarker`). It connects these markers in chronological order using dashed route polylines (`L.polyline`), visually reconstructing the vehicle's evaluated movement across town.
* **Marker Interactivity:** Clicking any point opens a popup containing the stop sequence number, camera name, exact timestamp, and the cropped snapshot.
* **Macro Density Heatmap:** Leverages `Leaflet.heat` to project an aggregated spatial density layer across all camera locations, visually highlighting traffic bottlenecks and high-congestion corridors.

---

## 🔄 How They Link Together: The End-to-End Data Lifecycle

* **Detection:** A vehicle passes Camera #14. The local script runs YOLOv8 + PaddleOCR, producing `"MP09AB1234"` in 45 ms.
* **Transmission:** The script sends a 150-byte JSON metadata packet to FastAPI via HTTP POST.
* **Database Write:** FastAPI inserts the record into MySQL with a sub-second commit.
* **Immediate Alert (If Blacklisted):** FastAPI detects a watchlist match and broadcasts a WebSocket packet. The frontend catches it in <10 ms, plays an alert sound, and drops a flashing marker at Camera #14's coordinates.
* **Historical Query:** An investigator searches for `"MP09AB1234"` on the frontend. JavaScript calls the trajectory endpoint; MySQL fetches all matching records ordered by time; Leaflet.js renders the full evaluated route on the city map.
