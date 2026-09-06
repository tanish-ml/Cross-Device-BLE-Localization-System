
## 1. General Setup

- Building area: EMI building corridor area on floor 0 and floor 1.
- Smartphone position for walking runs: trouser pocket unless explicitly noted otherwise.
- Sensor logging app: Android recording app exporting one CSV per recording.
- CSV streams:
  - `imu` rows with `accel`, `gyro`, `mag`, and `imu_processed`
  - `ble_rssi` rows for named beacon RSSI observations
  - `beacon` rows for raw BLE scan packets
  - `live_ble_snapshot` rows from the app snapshot stream
- Named beacons used in the project:
  - `arrive_emi1`
  - `arrive_emi2`
  - `arrive_emi3`
  - `arrive_emi4`
  - `arrive_emi8`
  - `arrive_emi10`
 

The beacons are mounted outside rooms / in the corridor. Room numbers are used as nearby landmarks, not as beacon locations inside rooms.

## 2. Ground Truth Policy

Use `measure.png` as the source of distance ground truth.

Do not use meter values written in timestamp files as final ground truth. Timestamp files are only for event times.

### How `measure.png` Was Produced

We used the web app `eleif.net/photomeasure` to estimate distances on the floor-plan images:

- Input image: floor-plan / beacon-location map image, especially `BeaconLocations_page1.png` and `BeaconLocations_page2.png`.
- Calibration dimension: the known straight hallway distance was set to `34.00 m`.
- Additional physical check: the corridor width was measured with a tape measure as approximately `3.26 m`.
- After calibration, line segments were drawn manually with the mouse on the floor plan to estimate corridor and connector lengths.

Important uncertainty:

- The known dimensions are trusted more than individual mouse-drawn line segments.
- Mouse placement is not perfectly precise, so each derived segment has small geometric error.
- this we are taking as ground truth.

Measured floor-1 segment lengths from `measure.png`:

| Segment | Length |
|---|---:|
| top hall from 124 to 118 | 34.00 m |
| west top connector | 3.26 m |
| 108 to west junction | 7.93 m |
| west lower connector | 2.70 m |
| 109a to 109b | 5.49 m |
| 109b to 109c | 5.48 m |
| 109c to 110a | 5.43 m |
| 110a to 110b | 5.57 m |
| 110b to 111 | 5.48 m |
| 111 to east turn | 2.79 m |
| east turn to 114 | 5.83 m |
| 114 to 115 | 3.26 m |

Derived coordinate convention in the notebook:
-
- `x = 0 m` at room `108`.
- Floor 1 lower-corridor positions are derived by summing the measured green segments from `measure.png`.
- Floor 0 uses the same corridor scale as floor 1 as an alignment assumption.
- Timestamp text files are still useful for matching events to elapsed time, but they are not used as meter-position truth.

## 3. Timing Convention

Manual timestamp notes use `minutes.seconds` notation, not decimal minutes.

Examples:

- `0.14` means 14 seconds.
- `1.10` means 1 minute 10 seconds, i.e. 70 seconds.
- `2.06` means 2 minutes 6 seconds, i.e. 126 seconds.

In the notebook, each CSV is synchronized to the first accelerometer sample. This is important because BLE scanning can start before IMU logging.

## 4. Data Files

### Original Localization Runs

| Run | CSV | Timestamp note | Purpose |
|---|---|---|---|
| R1 | `R1_S25Ultra.csv` | `Run1_TimeStamp.txt` | Floor 1 checkpoint-heavy baseline from 108 through the corridor and back/top side toward 124 | 
| R2 | `R2_S25Ultra.csv` | `Run2_TimeStamp.txt` | Slow floor transition from floor 0 west staircase up to floor 1 and toward 111 |
| R3 | `R3_S25Ultra.csv` | `Run3_TimeStamp.txt` | Floor 0 route with weaker/less favorable beacon geometry |
| R4 | `R4_S25Ultra.csv` | `Run4_TimeStamp.txt` | Fast downward floor transition from floor 1 toward floor 0 |
| R5 | `R5_S25Ultra.csv` | `Run5_TimeStamp.txt` | Long loop with floor transitions and return behavior |

in step one we checked the ground truth steps we walked was 150 steops
### R6 Static Cross-Device Test (done at 2m front of `arrive_emi2` at 1.5 height on table )

Purpose: compare whether phones record similar RSSI/IMU values when placed at the same physical location for approximately the same time.

| File | Device | Type |
|---|---|---|
| `R6_S23.csv` | Samsung S23 | static same-place recording |
| `R6_S25Ultra.csv` | Samsung S25 Ultra | static same-place recording |
| `R6_OnePlus11R.csv` | OnePlus 11R | static same-place recording |

R6 is not a walking trajectory. It is used to estimate device RSSI bias and stationary IMU differences.

### R7-R8 Two Runs Per Phone

Purpose: test whether the localization pipeline works across different phones after R6 showed device-dependent RSSI differences.

Route A: same-floor natural walking route.

- Start: floor 1 near `108`
- End: floor 1 near `115`
- No robotic checkpoint stopping
- stand approx 15 seconds at starting and ending

Route B: floor-transition route.

- Start on/near floor 0 west side near room 008
- Go up west stairs
- Continue on floor 1 to 115
- stand approx 15 seconds at starting and ending

| Run | CSV | Device | Route type |
|---|---|---|---|
| R7 | `R7_s23.csv` | Samsung S23 | Route A, natural floor-1 route |
| R8 | `R8_s23.csv` | Samsung S23 | Route B, floor-transition route |
| R7 | `R7_S25Ultra.csv` | Samsung S25 Ultra | Route A, natural floor-1 route |
| R8 | `R8_S25Ultra.csv` | Samsung S25 Ultra | Route B, floor-transition route |
| R7 | `R7_oneplus11R.csv` | OnePlus 11R | Route A, natural floor-1 route |
| R8 | `R8_oneplus11R.csv` | OnePlus 11R | Route B, floor-transition route |


### R9 Three-Phone Loop-Closure Test

Purpose: synchronized multi-phone route with both floor transitions and return to the same endpoint. This tests loop closure, floor-transition accuracy, and device robustness on the same path.

Route description:

- Start at `115`.
- Go down the staircase.
- Walk through the floor-0 corridor.
- Go up the next staircase.
- Return to `115`.

| Run | CSV | Device | Route |
|---|---|---|---|
| R9 | `R9_s23.csv` | Samsung S23 | 115 -> down -> floor 0 corridor -> up next stairs -> 115 |
| R9 | `R9_S25Ultra.csv` | Samsung S25 Ultra | same as R9 |
| R9 | `R9_oneplus11R.csv` | OnePlus 11R | same as R9 |


## 5. CSV Inventory Snapshot

Approximate durations from file timestamps:

| File | Duration | Notes |  Manual Step count
|---|---:|---|
| `R1_S25Ultra.csv` | 226.0 s | original floor-1 checkpoint run | 150
| `R2_S25Ultra.csv` | 139.1 s | original slow floor transition | 97
| `R3_S25Ultra.csv` | 192.3 s | original floor-0 run | 177
| `R4_S25Ultra.csv` | 103.8 s | original fast floor transition | 122
| `R5_S25Ultra.csv` | 206.4 s | original long loop | 238
| `R6_S23.csv` | 186.7 s | static phone comparison | 0
| `R6_S25Ultra.csv` | 205.3 s | static phone comparison |
| `R6_OnePlus11R.csv` | 226.3 s | static phone comparison |
| `R7_s23.csv` | 126.5 s | S23 route A |
| `R8_s23.csv` | 109.7 s | S23 route B |
| `R7_S25Ultra.csv` | 126.8 s | S25 Ultra route A |
| `R8_S25Ultra.csv` | 143.4 s | S25 Ultra route B |
| `R7_oneplus11R.csv` | 109.5 s | OnePlus route A |
| `R8_oneplus11R.csv` | 113.4 s | OnePlus route B |
| `R9_s23.csv` | 203.9 s | S23 loop-closure route |
| `R9_S25Ultra.csv` | 196.6 s | S25 Ultra loop-closure route |
| `R9_oneplus11R.csv` | 184.5 s | OnePlus loop-closure route |

stride length = 0.70 m