# UAV_mavlink-network-forensics
# MAV-Link Attack Detection - PCAP-to-CSV Forensic Converter

**MAV-Link Attack Detection** is a high-performance desktop forensic tool and ML dataset pipeline designed to decapsulate, parse, and analyze MAVLink v1.0 / v2.0 protocol traffic from raw network packet captures (`.pcap`, `.pcapng`, `.cap`).

---

## 🚀 Key Features

- **Dual Format Support:** Seamlessly decapsulates standard `.pcap` and modern `.pcapng` capture streams.
- **Per-Stream Sequence Dynamics:** Computes sequence delta ($\Delta\text{Seq} \pmod{256}$) and timing dynamics ($\Delta t$ in ms) separately per `(sys_id, comp_id)` stream, preventing false anomalies caused by interleaved UAV and GCS traffic.
- **BAD_DATA Frame Shield:** Automatic detection and elimination of framing errors, checksum corruptions, and uninitialized `sys_id == 0 / comp_id == 0` frames.
- **Packet-Level Ground-Truth Labeling:** Eliminates flawed whole-file labeling; individual packets are classified as Benign (`0`), Command Injection (`1`), or Replay (`2`) using embedded packet tags, sidecar metadata, or forensic semantics. Routine telemetry (e.g. `HEARTBEAT`, `ATTITUDE`) retains `label=0`.
- **Authentic Real-Flight Attack Injection:** Built-in `FlightAttackInjector` blends cyber-attacks directly into authentic flight captures with matching timestamps and sensor distributions, eliminating synthetic distribution shifts and bogus 100% ML accuracy.
- **Command & Parameter Extraction:** Full decoding of MAVLink commands (`COMMAND_LONG`, `COMMAND_INT`, `MISSION_ITEM`, `SET_MODE`, etc.) with 7-parameter vector extraction.
- **Machine Learning Pipeline:** Train file-isolated Random Forest or XGBoost models from Reader CSV folders, optionally balance training rows with SMOTE, and detect Normal / Command Injection / Replay predictions from uploaded Reader CSV files.
- **Investigator Views:** The ML detector presents per-file verdicts, SHA-256 evidence metadata, searchable attack events, packet-level predictions, a clickable timeline, batch capture summaries, and exportable HTML/CSV reports.
- **CSV Attack Checks:** Detect attacks directly in one or more Reader-exported CSVs or the currently loaded Reader table, with optional post-prediction label scoring and packet-level prediction exports.
- **Modern Cyber-Dark GUI:** Built on PyQt5 with paginated table inspection, anomaly highlighting, BAD_DATA counter, and real-time forensic logging.
- **Integrated ML Detector UI:** A second in-window tab provides model loading, CSV-folder training, CSV-only attack detection, packet results, attack-event summaries, and console progress in the same cyber-dark layout.

### ML Attack Detector

1. Convert each `.pcap`, `.pcapng`, or `.cap` capture to a structured CSV in the **PCAP → CSV** tab: select the capture, choose **Execute Extraction & Parsing**, then **Export Structured CSV**.
2. Select a folder containing multiple labeled Reader CSV exports (class labels `0`, `1`, and `2`). Choose Random Forest or XGBoost, optionally enable SMOTE, then select **Train Model**. The application keeps capture files wholly on either the training or held-out test side and saves `model.joblib` in the selected folder.
3. In the **ML Attack Detector** tab, load the saved model and upload one or more converted Reader CSV files (or a folder). Choose **Check for Attacks** to analyze them. The ML tab accepts CSV input only.
4. Review the file verdict, searchable event list, packet-level predictions, clickable timeline, model-performance charts, and the batch **Files** table. Selecting an event filters its packet range; selecting a file opens that CSV's events. **Export Report** writes an events CSV and a self-contained HTML report with an embedded timeline image and available model metrics.

The Files tab summarizes each checked CSV and selecting a row opens its events. CSV labels are not used to produce predictions; if present, they are used only after prediction for scoring. Attack events require at least three predicted attack frames; smaller groups do not trigger the file-level attack verdict.

To check CSV data, load a model and use the CSV upload controls on the detector page: drop/browse one or more Reader CSVs or a folder, or choose **Use Reader CSV** after parsing a capture in the Reader tab. Then choose **Check for Attacks**. CSV labels are excluded from model features and are consulted only after predictions for true-attack/caught/false-alarm counts. The Files tab summarizes checked CSVs; selecting a row loads its events and packets. **Export predictions CSV** saves packet rows with predictions and confidence, while **Export Report** saves the current file's events CSV and HTML report.

---

## 📦 Extracted Feature Schema

| Column | Description |
| :--- | :--- |
| `timestamp` | Packet epoch timestamp in seconds (microsecond precision) |
| `delta_time_ms` | Per-stream inter-packet arrival delta time in milliseconds ($\Delta t$) |
| `msg_id` | Numerical MAVLink message ID |
| `msg_name` | Canonical message name (e.g., `HEARTBEAT`, `COMMAND_LONG`) |
| `seq` | Packet sequence counter ($0\text{--}255$) |
| `delta_seq` | Per-stream sequence step delta ($\Delta\text{Seq} \pmod{256}$ per `sys_id, comp_id`) |
| `sys_id` | Source System ID |
| `comp_id` | Source Component ID |
| `command_id` | Operational MAVLink command ID |
| `command_name` | Human-readable command name (e.g., `MAV_CMD_NAV_LAND`) |
| `param1` – `param7` | Extracted operational parameters $1\text{--}7$ |
| `label` | Target ML ground truth class ($0$: Benign, $1$: Injection, $2$: Replay) |

---

## 🛠️ Installation

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/LonathG/Mavlink-Reader.git
   cd Mavlink-Reader
   ```

2. **Install Dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## 🖥️ Running the Application

Launch the desktop user interface:
```bash
python main.py
# or using npm
npm start
```

### Standalone CSV Attack Detector

The separate PyQt5 detector accepts Reader-exported `.csv` files only; it does not read or parse PCAP/PCAPNG files. Convert captures to CSV with SkyLink Analysis Pro before loading them into the detector.

Run it from the project folder:
```bash
python detector_app/main.py
```

For a headless detector test run:
```bash
QT_QPA_PLATFORM=offscreen python -m unittest test_detector_app
```

### Running Automated Test Suite
To verify the extraction pipeline, per-stream sequence calculation, packet-level labeling, and BAD_DATA filtering:
```bash
python test_pipeline.py
python test_gui.py
QT_QPA_PLATFORM=offscreen python -m unittest test_ml_pipeline
QT_QPA_PLATFORM=offscreen python -m unittest test_csv_attack_check
# or using npm
npm test
```

---

## 📄 License
MIT License.
