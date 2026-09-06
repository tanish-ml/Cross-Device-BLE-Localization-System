# Cross-Device BLE Localization System

An indoor positioning system that tracks a user's location across multiple floors by fusing smartphone **Inertial Measurement Unit (IMU)** data with **Bluetooth Low Energy (BLE)** beacon signal strength. 

This project solves the "global cold start" problem (locating a user with no prior location knowledge) and tackles the severe hardware fragmentation of Android smartphones (calibrating disparate RSSI biases across different devices) using a robust **Particle Filter**.

## Features
* **Hybrid Localization:** Fuses Step Detection (Accelerometer/Gyroscope) with Path-Loss Distance Estimation (BLE).
* **Multi-Floor Tracking:** Seamlessly tracks users navigating corridors and staircases.
* **Hardware Agnostic:** Includes pre-calculated RSSI offset calibration for various Android devices (e.g., Samsung, OnePlus) to ensure consistent tracking regardless of antenna hardware.

## Repository Structure

```text
.
├── Project_Description/    # Contains building floor maps and measuments
├── Runs/                   # Raw data CSVs collected from various devices (IMU + BLE streams), and timestamp notes
│   └── data_collection.md  # Detailed documentation of the data collection process and routes
├── run_path/               # Images showing the true walking paths for the baseline runs
├── testing.ipynb           # The main Jupyter Notebook harness containing the Particle Filter logic
├── blog.md                 # Technical blog post explaining the project architecture and challenges
└── README.md               # This file
```

## How It Works

1. **Step Detection:** Raw IMU streams are passed through a Butterworth low-pass filter. The algorithm detects rhythmic peaks to count physical steps, applying a standard stride length (0.70m).
2. **BLE Path-Loss Model:** The phone logs BLE packets from fixed beacons in the environment. A logarithmic path-loss model converts the Signal Strength (RSSI) into a noisy distance estimate.
3. **Particle Filter:** The filter generates hundreds of virtual "particles" (guesses) on the map. Every time a step is detected, particles move forward. Every time a BLE packet is heard, particles are scored based on the expected distance to the beacon. Impossible particles (in walls or wrong floors) are eliminated, collapsing the cloud onto the user's true location.

## Getting Started

### Requirements
You will need Python 3 installed along with the following data science packages:
* `numpy`
* `pandas`
* `matplotlib`
* `scipy`
* `jupyter`

### Running the Particle Filter
1. Clone this repository.
2. Open `testing.ipynb` in your preferred Jupyter environment.
3. The notebook is configured to run a global cold start on a given CSV recording from the `Runs/` folder (e.g., `R9_s23.csv`).
4. Execute the cells to process the IMU/BLE data and generate a visual trajectory over the floor maps. The final cell will generate an animated GIF of the particle filter in action!

## Cross-Device Calibration
To account for different smartphone antennas reading RSSI differently, we implemented a static cross-device calibration. If you test with a new device, you may need to measure its baseline RSSI offset against the system's path-loss model and update the calibration dictionary inside `testing.ipynb`.
