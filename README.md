# 🚦 Smart Vehicle Tracking & Traffic Intelligence System

**Team Hacktivators | Smart India Hackathon 2026**

An AI-powered, city-scale vehicle tracking and traffic analytics platform that uses CCTV camera networks, edge computing, and computer vision to detect, recognize, and trace vehicles across a city in real time — while also generating actionable urban traffic insights.

---

## 📌 Problem Statement

Cities need a reliable way to track a vehicle's movement across multiple camera locations (e.g., for law enforcement, missing vehicle search, or traffic planning) without relying on continuous video streaming, which is bandwidth-heavy and difficult to scale. This project solves that by performing detection and recognition **at the edge**, transmitting only lightweight metadata, and reconstructing vehicle trajectories centrally.

---

## 🧠 Technical Approach

### 1. Technology Stack

| Layer | Technologies |
|---|---|
| **Frontend** | HTML5, Tailwind CSS, JavaScript, Leaflet.js |
| **Backend** | Python, FastAPI, WebSocket |
| **AI / Computer Vision** | OpenCV, YOLOv8, PaddleOCR |
| **Database** | MySQL, SQL |
| **Communication** | HTTPS, MQTT, 4G/5G, Ethernet/Fiber |
| **Edge Computing** | Edge AI, Local Buffer, Store & Forward |

---

## 🏗️ System Architecture

The system is organized into 8 interconnected modules:

### 1️⃣ CCTV / Camera Network
A distributed network of cameras deployed across multiple city locations (e.g., Bhawarkua, Naulakaha, Bengali, Palasia), continuously capturing live footage.

### 2️⃣ Edge Processing Box (Near Each Camera)
Runs locally next to each camera to minimize bandwidth usage and latency:
- **OpenCV** – Frame processing
- **YOLOv8** – Vehicle & plate detection
- **PaddleOCR** – Number plate recognition
- Plate normalization + deduplication
- Plate image optimization (crop + compress)
- **Edge Buffer** with store-and-forward capability for network resilience

### 3️⃣ Lightweight Vehicle Metadata
Instead of transmitting raw video, only structured JSON metadata is sent:
```json
{
  "plate": "MP09AB1234",
  "camera": "CAM_12",
  "time": "14:27:16",
  "location": "22.7196, 75.8577",
  "accuracy": "96%",
  "image_ref": "img_045.jpg"
}
```

### 4️⃣ Event Broker (Optional, for Scale)
**Kafka / RabbitMQ / MQTT** used to:
- Buffer events
- Handle burst load
- Retry on failure
- Decouple cameras from the backend

### 5️⃣ Central Backend — FastAPI
- **Detection API** ingests metadata from edge devices
- **Watchlist / Tracking Check** module splits processing into:
  - **A. Normal Operation** — standard vehicle detection, stored in the database
  - **B. Priority Tracking** — target plates requested by admin trigger priority edge detection, immediate metadata transmission, and real-time alerts

### 6️⃣ Database (MySQL)
Stores vehicle sighting records indexed for fast retrieval:

| Field | Description |
|---|---|
| ID (PK) | Unique record ID |
| Plate Number | Detected vehicle plate |
| Timestamp | Detection time |
| Camera ID | Source camera |
| Latitude / Longitude | Geolocation |
| Accuracy | OCR confidence score |
| Image Reference | Snapshot reference |

Indexed by **Plate + Timestamp**, with automated **30-day retention policy**.

### 7️⃣ Intelligence Modules
- **7A. Trajectory Reconstruction** — Chronological reconstruction of a vehicle's movement across time and camera locations (e.g., Bhawarkua → Naulakaha → Bengali → Palasia).
- **7B. Urban Traffic Analytics** — City-wide traffic heatmaps covering:
  - Vehicle volume
  - Traffic density
  - Peak periods
  - Average travel time
  - Congestion hotspots
  - Origin–destination patterns

### 8️⃣ Admin GIS Dashboard
A web-based command center (built with **Leaflet.js + OpenStreetMap**) offering:
- 🔍 Vehicle search by plate number
- 🛣️ Vehicle trajectory visualization
- 🔔 Live alerts
- 🌡️ Traffic heatmap view
- 📷 Real-time camera status monitoring (24/24 online)

---

## 🔄 End-to-End Data Flow

```
CCTV Cameras
   ↓
Edge Processing Box (Detection + OCR + Compression)
   ↓
Lightweight JSON Metadata
   ↓
Event Broker (Kafka/RabbitMQ/MQTT) [optional, for scale]
   ↓
Central Backend (FastAPI) → Watchlist/Tracking Check
   ↓                              ↓
Normal Operation            Priority Tracking → Real-Time Alert
   ↓                              ↓
MySQL Database  ←──────────────────
   ↓
Intelligence Modules (Trajectory Reconstruction + Traffic Analytics)
   ↓
Admin GIS Dashboard
```

---

## ✨ Key Features

- ⚡ **Edge-first design** — detection and OCR run near the camera, drastically reducing bandwidth vs. streaming raw video
- 🔁 **Store & forward** — resilient to network outages at the edge
- 🎯 **Priority tracking** — admins can flag a target plate for real-time, low-latency alerting
- 🗺️ **Trajectory reconstruction** — reconstructs a vehicle's full path across the city, chronologically
- 📊 **Traffic analytics** — city-wide heatmaps for planning and congestion management
- 🔒 **Secure communication** — HTTPS/MQTT over 4G/5G, Ethernet, or Fiber
- 🗃️ **Automated data retention** — 30-day rolling policy for sightings data

---

## 🛠️ Tech Stack Summary

**Frontend:** HTML5, Tailwind CSS, JavaScript, Leaflet.js
**Backend:** Python, FastAPI, WebSocket
**AI/CV:** OpenCV, YOLOv8, PaddleOCR
**Database:** MySQL / SQL
**Messaging:** Kafka / RabbitMQ / MQTT (optional at scale)
**Communication:** HTTPS, MQTT, 4G/5G, Ethernet/Fiber
**Edge Computing:** Edge AI, Local Buffer, Store & Forward
**Maps:** Leaflet.js + OpenStreetMap

---

## 👥 Team

**Team Hacktivators** — Smart India Hackathon 2026

---

## 📄 License

This project is developed for **Smart India Hackathon 2026**. License to be determined by the team.
