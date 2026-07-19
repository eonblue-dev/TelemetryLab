# FastF1 API Cheat Sheet • Data Science
Quick reference and data contracts for feature engineering, telemetry analysis, and predictive modeling in F1.

---

## 1. Core Module and Session Ingestion

### `fastf1.get_session(year, gp, session)`
* **Return Type:** `fastf1.core.Session`
* **Parameter Signatures:**
  * `year` (`int`): Season year (>= 2018 for full telemetry).
  * `gp` (`str` | `int`): Grand prix name (e.g. 'Monaco') or round number.
  * `session` (`str`): 'FP1', 'FP2', 'FP3', 'Q', 'SQ', 'S', 'R'.

### `Session.load(laps=True, telemetry=False, weather=False, messages=False)`
* **Return Type:** `None` (in-place operation)
* **Purpose:** Injects relational databases into the current session instance.

---

## 2. Lap Abstraction and Filtering (`fastf1.core.Laps`)

### `Session.laps`
* **Return Type:** `fastf1.core.Laps` (inherits from `pandas.DataFrame`)
* **Key Column Schema:**
  * `Time` (`timedelta64[ns]`): Lap end timestamp.
  * `LapTime` (`timedelta64[ns]`): Lap duration.
  * `Driver` (`str`): Driver code (e.g. 'VER').
  * `LapNumber` (`int64`): Sequential index within the session.
  * `Stint` (`int64`): Tyre stint identifier.
  * `Compound` (`str`): Categorical ('SOFT', 'MEDIUM', 'HARD', 'INTERMEDIATE', 'WET').
  * `TyreLife` (`float64`): Accumulated tyre age in laps.
  * `TrackStatus` (`str`): Track flag code (e.g. '1' = Green, '4' = Safety Car).

### `Laps.pick_driver(driver_code)`
* **Return Type:** `fastf1.core.Laps`
* **Filter:** Isolates records for a single driver by their 3-letter code.

### `Laps.pick_fastest()`
* **Return Type:** `fastf1.core.Lap` (inherits from `pandas.Series`)
* **Filter:** Returns the lap with the minimum scalar value in `LapTime`.

### `Laps.pick_quicklaps(threshold=1.07)`
* **Return Type:** `fastf1.core.Laps`
* **Filter:** Removes outliers by excluding laps whose times exceed the median multiplied by the threshold.

---

## 3. Telemetry Streams and Physical Signals (`fastf1.core.Telemetry`)

### `Lap.get_telemetry()`
* **Return Type:** `fastf1.core.Telemetry` (inherits from `pandas.DataFrame`)
* **Description:** Returns sensor channels resampled and aligned via linear interpolation.
* **Key Column Schema:**
  * `SessionTime` (`timedelta64[ns]`): Index of the unified time series.
  * `Distance` (`float64`): Accumulated distance in meters from the line.
  * `Speed` (`float64`): Instantaneous linear speed in km/h.
  * `RPM` (`int64`): Engine revolutions per minute (0 - 15000).
  * `Throttle` (`int64`): Throttle opening percentage (0 - 100).
  * `Brake` (`bool`): Logical flag indicating brake pedal pressure.
  * `nGear` (`int64`): Gear engaged in the gearbox (1 - 8).
  * `X`, `Y`, `Z` | `float64`: Absolute spatial coordinates in millimeters.

### `Lap.get_car_data()`
* **Return Type:** `fastf1.core.Telemetry`
* **Description:** Returns mechanical variables only (`Speed`, `RPM`, `Throttle`, `Brake`) at native frequency (~20Hz) without interpolation.

### `Lap.get_pos_data()`
* **Return Type:** `fastf1.core.Telemetry`
* **Description:** Returns positioning/GPS spatial variables (`X`, `Y`, `Z`) at native frequency (~5Hz) without interpolation.

---

## 4. Environmental Context Structures and Classification

### `Session.weather_data`
* **Return Type:** `pandas.DataFrame`
* **Key Column Schema:**
  * `Time` (`timedelta64[ns]`): Session time of the sample (Frequency: ~1 min).
  * `AirTemp` (`float64`): Air temperature (°C).
  * `TrackTemp` (`float64`): Track surface temperature (°C).
  * `Humidity` (`float64`): Relative humidity percentage.
  * `Rainfall` (`bool`): Logical indicator of active precipitation on track.

### `Session.results`
* **Return Type:** `pandas.DataFrame`
* **Key Column Schema:**
  * `Position` (`float64`): Official finishing position in the session.
  * `ClassifiedPosition` (`str`): Official position or retirement code ('1', 'DNF', 'DSQ').
  * `GridPosition` (`float64`): Starting grid position.
  * `Status` (`str`): Race status reason (e.g., 'Finished', '+1 Lap', 'Collision').
  * `Points` (`float64`): World Championship points awarded.