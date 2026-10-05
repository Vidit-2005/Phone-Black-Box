# Comprehensive Telematics & IMU Dataset Analysis Report

## 1. Executive Summary

This investigation analyzed every dataset present in the project workspace to build a production-grade master dataset for real-time smartphone sensor classification. The target is detecting vehicle safety and dynamics events—including **Crash, Near-Miss, Normal Driving, Braking, Hard Braking, Turn, Sharp Turn, Bump, and Pothole**—for mobile deployment on Android.

---

## 2. In-Depth Dataset Audit & Quality Assessment

| Dataset | Format & Size | Contents | Usability Status | Action Taken |
| :--- | :--- | :--- | :--- | :--- |
| **`dataset/VZCrash`** | 16 Parquet files (Train: 11, Val: 3, Test: 2), ~1.5 GB | 189,303 driving events. Each has 16s of 100 Hz tri-axial acceleration ($g$), 100 Hz tri-axial gyro ($^\circ$/s), and 1 Hz GPS speed ($km/h$). Verified 3-way consensus labels. | **High Value (Usable)** | Extracted Crash (0), Near Miss (1), Hard Braking, Braking, Sharp Turn, Normal Turn, and Normal Driving. Vehicle size dropped. |
| **`dataset/road_accident_imu_dataset_8000.csv`** | CSV (1.7 MB, 8,000 rows) | IMU acceleration ($m/s^2$), Gyroscope ($rad/s$), Speed ($km/h$), Motion Intensity. Labeled: 7,000 Normal (0) and 1,000 Crash (1). | **High Value (Usable)** | Integrated into crash and normal baseline validation. |
| **`dataset/Isolated Data`** | 3 RAR archives (141 MB total, 393 CSVs) | Android smartphone sensor data (100 Hz): Accel ($m/s^2$), Gyro ($rad/s$), Location, annotated for **Bump**, **Pothole**, and **Normal Road**. | **High Value (Usable)** | Extracted Bump, Pothole, and Normal Road windows with SI-standard units. |
| **`dataset/GIS`** | CSV (11 MB) & RAR (41 MB) | `combined_gis_dataset.csv` contains only Latitude, Longitude, Elevation ($m$). `NextGIS.rar` contains QGIS DEM GeoTIFF rasters and contour lines. | **Unuseful (Excluded)** | **Excluded from Master Dataset**: Lacks inertial motion sensors (no accelerometer, gyroscope, or driving event annotations). |
| **`dataset/IO-VNBD-master`** | 187 CSV files & 2 ZIP files (Git LFS) | Purported Inertial Odometry Vehicle Navigation Benchmark Dataset. | **Corrupted / Unusable (Excluded)** | **Excluded from Master Dataset**: All 187 CSV files and both ZIP files are 134-byte Git LFS pointer stubs (`version https://git-lfs.github.com/spec/v1`). No binary payload exists in the repository. |

---

## 3. Analysis of Usable Sensor Telemetry

### 3.1 VZCrash (Verizon Connect Research)
* **Sample Count**: 137,954 (Train), 27,175 (Val), 24,174 (Test) = **189,303 total events**.
* **Sensors**:
  * `gsensor`: $[a_x, a_y, a_z]$ tri-axial acceleration at 100 Hz (1,600 readings per 16-second event). Converted from $g$ to standard SI $m/s^2$ ($1g = 9.80665 \, m/s^2$).
  * `gyro`: $[\omega_x, \omega_y, \omega_z]$ tri-axial angular velocity at 100 Hz in $^\circ$/s.
  * `gps_speed`: Vehicle ground speed at 1 Hz in $km/h$.
* **Ground Truth & Event Extraction**:
  * **Crash (Class 0)**: Severe deceleration and impact shock wave ($> 3.0g$ to $6.0g+$, extreme jerk $> 20g/s$, rapid tumble and speed drop to 0).
  * **Near Miss (Class 1)**: Emergency evasive maneuvers (sudden avoidance steering combined with sharp braking, rapid speed loss without impact).
  * **Hard Braking (Class 4)**: Longitudinal deceleration $<-3.5 \, m/s^2$ (speed drop $> 12 \, km/h/s$) without structural crash signature.
  * **Braking (Class 3)**: Controlled deceleration (longitudinal deceleration between $-1.0$ and $-3.0 \, m/s^2$, speed drop $4\text{--}10 \, km/h/s$).
  * **Sharp Turn (Class 6)**: Aggressive cornering/swerve (yaw rate $> 28^\circ$/s or lateral accel $> 0.35g$).
  * **Turn (Class 5)**: Smooth curve/cornering (yaw rate $10^\circ\text{--}25^\circ$/s).
  * **Normal Driving (Class 2)**: Steady cruising ($|\Delta v| < 3 \, km/h/s$, acceleration $\approx 1g$ gravity, gyro $< 8^\circ$/s).

### 3.2 Road Accident IMU Dataset (8,000 Records)
* **Sample Count**: 8,000 continuous time-steps.
* **Telemetry**: Tri-axial acceleration, tri-axial gyroscope, Speed ($km/h$), Motion Intensity.
* **Findings**:
  * Non-crash ($N=7,000$): Mean motion intensity $9.73 \, m/s^2$, cruising speed mean $50.05 \, km/h$.
  * Crash ($N=1,000$): Mean motion intensity $13.49 \, m/s^2$, speed collapse down to mean $10.23 \, km/h$ (min $0.01 \, km/h$).

### 3.3 Isolated Road Anomalies & Normal Road
* **Sample Count**: 393 extracted files.
* **Telemetry**: High-frequency Android smartphone sensor recordings (100 Hz).
* **Ground Truth**: Ground-truth tagged segments of **Bump** (speed bump, road hump), **Pothole** (asphalt cavity/drop), and **Normal Road**.

---

## 4. Master Dataset Architecture

To support real-time sliding-window inference on Android phones:

1. **Window Size**: 100 consecutive sensor samples (1.0 second at 100 Hz).
2. **Units Standardized to Android OS Sensor API**:
   * Acceleration: $m/s^2$ (`Sensor.TYPE_ACCELEROMETER`)
   * Gyroscope: $deg/s$ (converted from `Sensor.TYPE_GYROSCOPE` rad/s via $\times \frac{180}{\pi}$)
   * Speed: $km/h$ (converted from `Location.getSpeed()` m/s via $\times 3.6$)
3. **Engineered Feature Set (51 Features per Window)**:
   * **Statistical (30 features)**: Mean, std, min, max, peak-to-peak for $a_x, a_y, a_z, \omega_x, \omega_y, \omega_z$.
   * **Kinematic & Magnitudes (9 features)**: Vector magnitude $\sqrt{a_x^2+a_y^2+a_z^2}$ (mean, std, min, max, ptp, rms), horizontal acceleration $\sqrt{a_x^2+a_y^2}$ (mean, std, max).
   * **Jerk Dynamics (3 features)**: $\Delta a / \Delta t$ mean, std, max (distinguishes collisions from road bumps).
   * **Rotational Energy (4 features)**: Total angular rate $\sqrt{\omega_x^2+\omega_y^2+\omega_z^2}$ (mean, std, max, ptp).
   * **Velocity Delta (2 features)**: Speed mean ($km/h$) and speed delta $\Delta v$ across the window.
   * **Spectral & Frequency (3 features)**: Total FFT spectral energy, low-band energy (0–5 Hz: maneuvers), high-band energy (>10 Hz: shocks and bumps).
4. **Target Classes**:
   * `0`: `crash`
   * `1`: `near_miss`
   * `2`: `normal_driving`
   * `3`: `braking`
   * `4`: `hard_braking`
   * `5`: `turn`
   * `6`: `sharp_turn`
   * `7`: `bump`
   * `8`: `pothole`
5. **Splits**: Strict Train (70%), Validation (15%), Test (15%) with zero data leakage across recording events.
