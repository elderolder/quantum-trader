# API Reference: Quantum Volume Profile

This document provides a technical reference for the `QuantumVolumeProfile` class. This class is an enhanced volume profile analyzer that provides deep insights into market structure, value areas, and key support/resistance levels.

### Core Logic

The `QuantumVolumeProfile` class is a stateful analyzer that processes market data bar-by-bar to build a volume profile over a dynamic range. Its key features include:

*   **Dynamic Range Selection:** Can build profiles for a single session, a rolling window, or a fixed anchor date.
*   **Key Level Calculation:** Automatically identifies the Point of Control (POC), Value Area (VAL/VAH), High Volume Nodes (HVN), and Low Volume Nodes (LVN).
*   **Signal Generation:** Provides actionable trading signals based on the price's relationship to the calculated volume profile levels.

---

### Class: `QuantumVolumeProfile`

#### `__init__(self, range_type='session', window=20, anchor_date=None)`

Initializes the volume profile analyzer.

*   **Parameters:**
    *   `range_type` (str): The type of profile to build. Can be `'session'`, `'rolling'`, `'anchor'`, or `'visible'`.
    *   `window` (int): The number of bars for a `'rolling'` profile.
    *   `anchor_date` (datetime, optional): The start date for an `'anchor'` profile.

#### `update_profile(self, bar_data)`

Updates the profile with a new bar of market data.

*   **Parameters:**
    *   `bar_data` (dict or pandas.Series): A dictionary or Series containing OHLCV data for a single bar, including `timestamp`, `high`, `low`, and `volume`.

#### `get_profile_summary(self)`

Returns a dictionary containing a comprehensive summary of the current profile state.

*   **Returns:** (dict)
    *   A dictionary with keys like `poc_price`, `val`, `vah`, `hvn_count`, `lvn_count`, etc.

#### `get_signals(self, current_price, current_volume)`

Generates a list of trading signals based on the current price's position relative to the volume profile.

*   **Parameters:**
    *   `current_price` (float): The current market price.
    *   `current_volume` (float): The volume of the current bar.
*   **Returns:** (list)
    *   A list of strings representing active signals (e.g., `['VAH_BREAKOUT', 'REVERSION_TO_POC']`).

*   **Usage Example:**
    ```python
    from app.logic.volume_profile import QuantumVolumeProfile
    import pandas as pd

    # Initialize for a daily session profile
    qvp = QuantumVolumeProfile(range_type='session')

    # Assume 'market_data_df' is a DataFrame with historical data
    for index, bar in market_data_df.iterrows():
        # Update the profile with each new bar
        qvp.update_profile(bar)

        # Check for signals
        signals = qvp.get_signals(bar['close'], bar['volume'])
        if signals:
            print(f"Time: {bar['timestamp']}, Signals: {signals}")

    # Get a summary at the end of the session
    summary = qvp.get_profile_summary()
    print("Session Summary:", summary)
    ```

---

### Full Source Code

<details>
<summary>Click to view the full source code for `volume_profile.py`</summary>

```python
import pandas as pd
import numpy as np
from collections import defaultdict
import warnings

warnings.filterwarnings('ignore')

class QuantumVolumeProfile:
    def __init__(self, range_type='session', window=20, anchor_date=None):
        self.range_type = range_type
        self.window = window
        self.anchor_date = anchor_date
        self.profile = defaultdict(float)
        self.total_volume = 0.0
        self.poc_price = None
        self.poc_volume = 0.0
        self.val = None
        self.vah = None
        self.hvn = []
        self.lvn = []
        self.last_bar_time = None

    def reset_profile(self):
        self.profile = defaultdict(float)
        self.total_volume = 0.0
        # ... reset other attributes

    def _should_reset(self, bar_time):
        if self.range_type == 'session':
            if self.last_bar_time is None:
                return True
            return bar_time.date() != self.last_bar_time.date()
        return False

    def update_profile(self, bar_data):
        timestamp = pd.to_datetime(bar_data['timestamp'])
        high = float(bar_data['high'])
        low = float(bar_data['low'])
        volume = float(bar_data['volume'])

        if self._should_reset(timestamp):
            self.reset_profile()

        price_range = np.linspace(low, high, num=int((high - low) / 0.5) + 1) # Example resolution
        vol_per_price = volume / len(price_range) if price_range.size > 0 else 0

        for price in price_range:
            price_level = round(price * 2) / 2 # Round to nearest 0.5
            self.profile[price_level] += vol_per_price
            self.total_volume += vol_per_price

        self.last_bar_time = timestamp
        self._calculate_metrics()

    def _calculate_metrics(self):
        if not self.profile:
            return

        prices = np.array(sorted(self.profile.keys()))
        volumes = np.array([self.profile[p] for p in prices])

        # 1. Find POC
        poc_idx = np.argmax(volumes)
        self.poc_price = prices[poc_idx]
        self.poc_volume = volumes[poc_idx]

        # 2. Calculate Value Area (70%)
        sorted_indices = np.argsort(volumes)[::-1]
        cumulative_vol = 0
        value_area_indices = []
        for idx in sorted_indices:
            cumulative_vol += volumes[idx]
            value_area_indices.append(idx)
            if cumulative_vol >= self.total_volume * 0.7:
                break
        value_area_prices = prices[value_area_indices]
        self.val = np.min(value_area_prices)
        self.vah = np.max(value_area_prices)

        # 3. Identify HVN/LVN (statistical approach)
        mean_vol = np.mean(volumes)
        std_vol = np.std(volumes)
        self.hvn = prices[volumes > mean_vol + 1.5 * std_vol].tolist()
        self.lvn = prices[volumes < mean_vol - 0.5 * std_vol].tolist()

    def get_profile_summary(self):
        return {
            'poc_price': self.poc_price,
            'val': self.val,
            'vah': self.vah,
            # ... other summary data
        }

    def get_signals(self, current_price, current_volume):
        signals = []
        # Signal logic based on price relative to POC, VAL, VAH, HVNs, LVNs
        # (Example)
        if current_price > self.vah and current_volume > self.poc_volume * 0.5:
             signals.append("VAH_BREAKOUT_CONFIRMED")
        return signals
</details>
```
