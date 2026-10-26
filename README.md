# 🔋 Kalkini Power Monitor (KPM) v3.0 — Enterprise Smart Grid Monitoring System

A decentralized, cloud-integrated IoT ecosystem designed to track real-time grid telemetry, automate fault localization, and decode sub-station anomalies without relying on expensive, legacy SCADA infrastructure. 

> 🔒 **Repository Note:** The production source code (including Firebase Security Rules, ESP32 Firmware C++, and Client Data-Masking Proxies) is kept strictly **PRIVATE** due to commercial sensitivity, local utility infrastructure security, and intellectual property rights. This README serves as a comprehensive **System Architecture & Technical Documentation Overview** for global tech evaluation.

---

## 🛠️ System Architecture Layer (Decentralized IoT Framework)

The system is engineered using a robust, decoupled **4-Tier Architecture** to achieve high scaling, sub-millisecond data synchronization, and zero server-side maintenance.

```mermaid
flowchart TB
    %% 💡 স্টাইলিং ও থিমিং কনফিগারেশন
    classDef hardware fill:#ef4444,stroke:#b91c1c,stroke-width:2px,color:#fff;
    classDef database fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#000;
    classDef server fill:#3b82f6,stroke:#1d4ed8,stroke-width:2px,color:#fff;
    classDef client fill:#10b981,stroke:#047857,stroke-width:2px,color:#fff;

    subgraph Edge_Hardware_Layer [⚡ EDGE HARDWARE LAYER]
        H1[ESP32 Field Node 1.0]:::hardware
        H2[ESP32 Field Node 2.0]:::hardware
        H3[ESP32 Field Node 3.0]:::hardware
    end

    subgraph Cloud_Database_Layer [🔥 CLOUD CORE LAYER]
        FB[(Firebase Realtime Database)]:::database
        FCM[Firebase Cloud Messaging API]:::server
    end

    subgraph Utility_Analytics_Hub [⚙️ UTILITY HUB]
        Admin[Secure Admin Portal: admin.html]:::server
        Engine[Intelligent Fault/Breaker Decoder]:::server
    end

    subgraph Consumer_Application_Layer [📱 CLIENT LAYER]
        PWA[Standalone Consumer App: index.html]:::client
        Worker[Service Worker: Offline Cache]:::client
    end

    %% 🔗 কানেক্টিভিটি ও ডেটা ফ্লো লুপ
    H1 & H2 & H3 -->|Continuous Handshake / JSON Payload| FB
    FB <-->|WebSocket Stream / Zero Latency| Engine
    Engine -->|Render Topology & Fault Sirens| Admin
    
    %% পুশ নোটিফিকেশন এপিআই রুট
    Engine -->|Trigger Alerts on Sequence Break| FCM
    FCM -->|Targeted Push Notification API| PWA
    
    %% ক্লায়েন্ট অ্যাবস্ট্রাকশন লেয়ার (ডাটা মাস্কিং)
    FB -->|Data Masking Proxy / Abstract Outages| PWA
    PWA <-->|Local Assets Validation| Worker
```

---

## 🧠 Core Engineering Innovations & Algorithms

### 1. Sequential Micro-Zone Fault Localization (Chain Breakdown Model)
Instead of expensive transformer-level hardware metrics, KPM maps local grid regions using a dynamic **Sequential Chaining Array** (`Node[1] -> Node[2] -> Node[3]`).
* **The Logic:** The backend processing loop continuously evaluates sorted node sequence indices. If `Node[i].status === "Online"` but `Node[i+1].status === "Offline"`, the network instantly detects a physical sequence break.
* **The Result:** The interface immediately raises a **CRITICAL FAULT** siren and isolates the hazard zone strictly between those two geo-locations, shrinking the search perimeter from a 10km grid to under 500 meters for dispatch teams.

### 2. The Breaker Tripping vs. Load-Shedding Decoder
A notorious constraint in rural grids is the absence of local **Dropout Fuses (DO Fuses)**, causing a localized grid failure to instantly trip the main sub-station breaker, dropping all downstream nodes simultaneously. KPM bypasses this physical bottleneck programmatically via a **Master-Switch State Machine**:
* **Scenario A (Official Outage):** If all concurrent nodes drop on a feeder and the sub-station's dynamic Master Switch state is tracked as **OFF**, the dashboard handles it as an intentional power-cut: `⚠️ INFO: Official Load-Shedding Active`.
* **Scenario B (Grid Failure / Surge):** If all nodes drop simultaneously but the Master Switch remains **ON**, the system decodes it as an emergency structural event: `🚨 BREAKING ALERT: Sub-station Breaker Tripped (Major Line Fault)`. It then extracts millisecond **Last-Ping Timestamps** to isolate where the high-voltage transient surge first impacted the array.

### 3. Asymmetric Security Proxies & Client Data-Masking
To prevent grid telemetry data leaks, avoid public panic, and secure critical infrastructure, the frontend implements an **asymmetric data abstraction layer**. 
* **Public Tier:** Raw sensor breakdowns and sequence crashes are filtered out on consumer clients; user interfaces render a simplified, abstracted `Online (🟢)` or `Offline (🔴)` state.
* **Admin Tier:** Full multi-feeder diagnostics, live hazard zones, line-loss estimations, and CRUD network configurations are accessible exclusively through an authenticated administration gate.

### 4. Conditional Edge-Failure Interceptor (Zero-Flicker Watchdog & State Recovery)
Microcontrollers (ESP32) operating in harsh rural grid sectors often experience abrupt hardware fatalities (e.g., total surge destruction or immediate power loss) before they can dispatch a graceful "Offline" payload to the cloud gateway. This introduces a major synchronization trap where the cloud server continuously retains stale "Online" states.
* **The Logic:** To bypass this constraint without overloading server-side operations, the framework implements a dual-layer **Deterministic Heartbeat Watchdog Interceptor** operating via client-side throttling windows (45-second fallback boundaries). It continuously audits millisecond differential delta gaps between transmission intervals and background timers.
* **The State Recovery Fix:** To ensure robust system testing during telemetry simulations, an explicit state validation flag (`window.isKpmOfflineForced`) controls the hard offline lock. When a live edge node or a simulated telemetry source re-establishes its connection (`status === "Online"`), the conditional logic instantly un-locks the forced isolation state layer across all active HTML5 DOM viewports without requiring manual user page refreshes.

---

## 📂 Production NoSQL Data Schematics (Firebase Model)

The database utilizes highly optimized NoSQL JSON node layers to allow fluid telemetry tracking and multi-day retrospective log calculations:

### 1. Active Feeder & Telemetry Node Hierarchy
Allows real-time state mapping and live area status parsing across unique hardware IDs:
```json
{
  "kalkini_power_monitor": {
    "system_control": {
      "last_updated": "2026-08-23 12:20:00"
    },
    "feeders": {
      "feeder_paurashava": {
        "name": "কালকিনি পৌরসভা ফিডার",
        "status": "Online"
      }
    },
    "areas": {
      "area_1877": {
        "name": "কালকিনি পৌরসভা",
        "feeder_id": "feeder_paurashava",
        "status": "Online",
        "last_ping": "22/06/2026, 11:52:01 pm",
        "nodes": {
          "node_1_106": {
            "name": "পৌরসভা মেইন মোড়",
            "status": "Online",
            "sequence": 1,
            "last_ping": "22/06/2026, 11:52:44 pm"
          },
          "node_2_460": {
            "name": "পাঙ্গাশিয়া",
            "status": "Offline",
            "sequence": 2,
            "heartbeat": 43307,
            "last_ping": "22/06/2026, 11:53:46 pm"
          }
        }
      }
    }
  }
}
```

### 2. Sequential Outage Log Structure (`power_status`)
The client analytics engine processes sequential `Offline` $\rightarrow$ `Online` transitions from the root database node. Durations are calculated dynamically on the fly based on specialized local string time formatting (`en-BD`) to populate the interactive retrospective bar charts:
```json
{
  "power_status": {
    "-OU1z8_MkBc9XyA1": {
      "status": "Offline",
      "time": "10/26/2026, 2:00:00 PM"
    },
    "-OU1z9_HkLp2WzR8": {
      "status": "Online",
      "time": "10/26/2026, 2:45:00 PM"
    }
  }
}
```

---

## 💻 Decoupled Testing & Local Telemetry Simulator

To run integration testing for multi-day retrospective charts without hardware node dependencies, the pipeline uses a standalone mock script powered by the **Firebase Admin SDK** and a secure Service Account Key structure.

### Setting Up the Telemetry Script
1. Clone the repository structure and initialize dependencies:
   ```bash
   npm install firebase-admin
   ```
2. Secure your project by placing your `serviceAccountKey.json` directly into the root folder directory tree.
3. Boot up the lightweight local simulation cluster to push live multi-day dynamic logs into the NoSQL cloud layers:
   ```bash
   node mock-server.js
   ```

---

## ⚡ Progressive Web App (PWA) Offline Engineering
To survive harsh deployment fields with unstable networks, the system is fully decoupled into an installable standalone application utilizing native browser storage optimization:
* **Standalone UI Context:** Custom configuration files target specific application parameters (`display: standalone`), entirely stripping away default browser search fields and URL frames to render a pure native app runtime across **Windows, Android, and iOS (Safari)**.
* **Asynchronous Caching Framework:** The background `service-worker.js` listens to incoming data pipelines, securely intercepting network payloads to execute isolated local caching via `caches.match`. This guarantees that if a lineman opens the application in a zero-network dead zone, the complete UI scaffolding, map structures, and offline charting elements remain fully functional.

---

## 📈 R&D Scalability Roadmap (Next Enterprise Phases)
* **Three-Phase Load Balance Optimization:** Interfacing **SCT-013 CT Sensors** with the ESP32 array to execute live amperage variance analysis; triggering alerts if current load deviates over 15% across R, Y, and B phases to prevent transformer burnouts.
* **Predictive Thermal Overhead Warnings:** Modeling historic power curves to detect overheating thresholds on critical transformers before a physical terminal explosion happens.
* **Differential Distribution Loss Analysis:** Calculating variance between sub-station feeder outputs.
* **Dual-Core Hardware Frequency Monitoring:** Interfacing ZMPT101B AC voltage step-down sensors with the ESP32’s secondary processing core to calculate dynamic Root Mean Square (RMS) line fluctuations and current load consumption in Amperes concurrently.


## Project Live link
<a href="https://kalkinipowermonitor.web.app/" target="_blank">
  <img src="https://img.shields.io/badge/View_Project-181717?style=for-the-badge&logo=github&logoColor=white"/>
</a>

