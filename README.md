# WiFi-Radar-2D

**Multi-node Wi-Fi CSI-based indoor human localization and direction-tracking research prototype**

An experimental, privacy-conscious indoor sensing project using Wi-Fi Channel State Information (CSI), ESP32-S3 receivers, signal processing, and machine learning. The long-term research goal is to estimate a person's occupied zone and direction of movement and display those estimates on a 2D room dashboard.

> **Research status:** early prototype scaffold. The repository currently supports serial capture and exploratory CSI analysis. It does **not** yet provide validated continuous (x, y) localization, calibrated distance/angle measurements, or reliable multi-person tracking. Those are research goals, not existing features.

## Current hardware

- M5Stack Cardputer ADV (ESP32-S3 / 2.4 GHz Wi-Fi) — initial receiver candidate.
- JioFiber router — potential 2.4 GHz access point / packet source. Exact router model and firmware are not yet known, so compatibility must be tested.
- Windows laptop — ESP-IDF build environment, serial capture, analysis, and dashboard.

## Project roadmap

| Milestone | Goal | Status |
|---|---|---|
| M0 | Confirm board, router band/channel, USB serial and firmware recovery path | Planned |
| M1 | Capture CSI on one ESP32-S3 using Espressif's router example | Planned |
| M2 | Save CSI records and inspect signal changes during controlled movement | Starter software included |
| M3 | Collect labeled room-zone fingerprints across repeated sessions | Planned |
| M4 | Evaluate a baseline zone classifier with session-separated testing | Starter training script included |
| M5 | Add two more spatially separated receiver nodes and synchronization | Planned |
| M6 | Build a calibrated 2D visualization and evaluate localization error | Planned |
| M7 | Study robustness, privacy, and physical-layer security limitations | Planned |

## Architecture (target)

```text
                  2.4 GHz Wi-Fi packets
             +------------------------------+
             |                              |
        [JioFiber AP]                 (controlled traffic)
             |                              |
       +-----+------------------------------+
       |                  |                 |
 [Cardputer ADV]   [ESP32-S3 node 1]  [ESP32-S3 node 2]
       |                  |                 |
       +------------------+-----------------+
                          USB serial / timestamped records
                                      |
                              [Windows laptop]
                                      |
                       parse -> filter -> features
                                      |
                       zone model -> evaluation
                                      |
                         [2D room dashboard]
```

The diagram is a proposed architecture. Three receivers do not automatically triangulate a person. CSI-based positioning generally requires stable packet generation, careful node placement, calibration, labeled measurements, synchronization, and validation against ground truth. A first practical target is **zone classification**, followed by more ambitious continuous position estimation if the data supports it.

## Repository layout

```text
WiFi-Radar-2D/
├── README.md
├── LICENSE
├── CITATION.cff
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── hardware.md
│   ├── phase1-plan.md
│   └── safety-and-privacy.md
├── firmware/
│   └── README.md
└── software/
    ├── requirements.txt
    ├── wifi_radar/
    │   ├── __init__.py
    │   ├── csi_parser.py
    │   ├── capture.py
    │   ├── features.py
    │   └── dashboard.py
    └── tests/
        └── test_parser.py
```

## Quick start (Windows 10/11)

### 1. Install Python dependencies

```powershell
cd software
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 2. Flash an ESP32-S3 CSI receiver

Use Espressif's maintained `esp-csi` example rather than assuming the Cardputer's factory firmware can expose CSI. See [`firmware/README.md`](firmware/README.md) for the tested starting point and board-specific caveats.

### 3. Capture serial output

Find the Cardputer COM port in Windows Device Manager, then:

```powershell
python -m wifi_radar.capture --port COM5 --baud 115200 --node-id cardputer --label room_center --session-id session_01 --out ..\data\room_center_session_01.csv
```

Replace `COM5` and the label with your actual values. Use a separate label for each marked position/zone. Do not put Wi-Fi passwords in command history or commit them to Git.

### 4. Inspect a capture

```powershell
python -m streamlit run wifi_radar/dashboard.py -- ..\data\room_center_session_01.csv
```

The dashboard is for exploratory signal inspection. It does not claim to show a person's measured physical coordinates. Run this command from the `software` directory.

## Tests

```powershell
python -m unittest discover -s tests -v
```

Run from the `software` directory after installing dependencies.

## Hardware purchasing guidance

Do not purchase an antenna module for the Cardputer ADV at this stage. Start with the built-in radio and one receiver. After a successful CSI capture, consider two additional **matching ESP32-S3 development boards** as separate receiver nodes. Confirm the exact board variant, USB interface, antenna implementation, and ESP-IDF support before ordering. See [`docs/hardware.md`](docs/hardware.md).

## Research evaluation

Report at least:

- CSI packet capture rate and packet-loss rate.
- Repeatability across multiple sessions and days.
- Detection performance for stationary, walking, and empty-room conditions.
- Zone classification accuracy, macro-F1, and confusion matrix.
- For continuous localization: median / 90th-percentile position error in metres, with a documented ground-truth method.
- Robustness to people moving outside the target area, doors, fans, furniture changes, and router channel changes.

Do not report accuracy from a random split of adjacent packets as if it represented generalization. Prefer session-separated or day-separated evaluation.

## Safety, privacy, and scope

Use only networks and devices you own or are authorized to test. Obtain informed consent from people participating in sensing experiments. Do not use this prototype for covert surveillance, identity recognition, or safety-critical decisions. Wi-Fi sensing is affected by multipath, interference, hardware variation, and environmental changes. See [`docs/safety-and-privacy.md`](docs/safety-and-privacy.md).

## References

- Espressif, [ESP-CSI repository](https://github.com/espressif/esp-csi) — CSI receiver and router examples, data parsing utilities, and radar/activity-detection examples.
- Espressif, [Wi-Fi CSI configuration guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/wifi-driver/wifi-vendor-features.html).
- M5Stack, [Cardputer-Adv documentation](https://docs.m5stack.com/en/core/Cardputer-Adv).
- IEEE 802.11bf-2025, WLAN sensing standard (consult the official IEEE catalogue for the standard text and licensing).

## License

MIT License. See [LICENSE](LICENSE).
