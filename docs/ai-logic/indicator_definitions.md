# Indicator Definitions

This document contains the pure Python source code for the proprietary trading indicators used in Quantum Trader Pro. This serves as the ground truth for the core signal generation logic.

**Audience:** Gemini (Implementation)
**Goal:** To provide a single, clean source for the Python code that should be placed in the `backend/app/logic/` directory.

---

### 1. Quantum Cipher (`quantum_cipher.py`)

This indicator combines momentum, volume, and trend analysis to generate buy/sell signals.

```python
import pandas as pd
import numpy as np
import ccxt

def calculate_quantum_cipher(df):
    """
    Calculate Quantum Cipher signals from market data DataFrame.
    """
    # 1. Calculate core price values
    df['hlc3'] = (df['high'] + df['low'] + df['close']) / 3

    # 2. Adaptive Momentum Oscillator
    df['mom'] = df['hlc3'].diff()
    df['abs_mom'] = df['mom'].abs()
    df['smooth_mom'] = df['mom'].ewm(span=14, adjust=False).mean()
    df['smooth_abs'] = df['abs_mom'].ewm(span=14, adjust=False).mean()
    df['osc'] = 100 * df['smooth_mom'] / (df['smooth_abs'] + 1e-9)

    # 3. Volume-Weighted Confirmation
    cumulative_vp = (df['hlc3'] * df['volume']).cumsum()
    cumulative_vol = df['volume'].cumsum()
    df['vwap'] = cumulative_vp / (cumulative_vol + 1e-9)
    df['volume_sma20'] = df['volume'].rolling(window=20).mean()
    df['v_ratio'] = df['volume'] / (df['volume_sma20'] + 1e-9)

    # 4. Trend Filter
    df['ema_fast'] = df['close'].ewm(span=8, adjust=False).mean()
    df['ema_slow'] = df['close'].ewm(span=21, adjust=False).mean()

    # 5. Signal Conditions
    df['bull_vol'] = (df['close'] > df['vwap']) & (df['v_ratio'] > 1.2)
    df['bear_vol'] = (df['close'] < df['vwap']) & (df['v_ratio'] > 1.2)
    df['uptrend'] = df['ema_fast'] > df['ema_slow']
    df['downtrend'] = df['ema_fast'] < df['ema_slow']

    # 6. Generate Signals
    df['buy_signal'] = (df['osc'] < -30) & (df['osc'] > df['osc'].shift(1)) & df['bull_vol'] & df['uptrend']
    df['sell_signal'] = (df['osc'] > 70) & (df['osc'] < df['osc'].shift(1)) & df['bear_vol'] & df['downtrend']
    
    return df

def fetch_market_data(exchange, symbol, timeframe, limit=1000):
    """
    Fetch historical market data using ccxt.
    """
    try:
        exchange_class = getattr(ccxt, exchange)()
        ohlcv = exchange_class.fetch_ohlcv(symbol, timeframe, limit=limit)
        df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
        df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
        return df
    except Exception as e:
        print(f"Error fetching market data: {e}")
        return pd.DataFrame()
```

2. Quantum Volume Profile (quantum_volume.py)

This class provides an enhanced volume profile analysis, identifying key levels like POC, VAL, and VAH.

```python
import pandas as pd
import numpy as np
from collections import defaultdict

class QuantumVolumeProfile:
    """
    Enhanced Volume Profile analyzer with dynamic range selection and statistical node classification.
    """
    def __init__(self, range_type='session', window=20, anchor_date=None):
        self.range_type = range_type
        self.window = window
        self.anchor_date = anchor_date
        self.profile = defaultdict(float)
        self.total_volume = 0.0
        self.poc_price = None
        self.val = None
        self.vah = None
        self.last_bar_time = None

    def reset_profile(self):
        self.profile = defaultdict(float)
        self.total_volume = 0.0
        self.poc_price = None
        self.val = None
        self.vah = None

    def _should_reset(self, bar_time):
        if self.range_type == 'session':
            if self.last_bar_time is None: return True
            return bar_time.date() != self.last_bar_time.date()
        return False

    def update_profile(self, bar_data):
        timestamp = pd.to_datetime(bar_data['timestamp'])
        high = float(bar_data['high'])
        low = float(bar_data['low'])
        volume = float(bar_data['volume'])

        if self._should_reset(timestamp):
            self.reset_profile()

        # Approximate volume distribution across the bar's price range
        price_range = np.arange(low, high, 0.5) # Example: using a 0.5 price tick size
        if len(price_range) > 0:
            vol_per_tick = volume / len(price_range)
            for price in price_range:
                self.profile[price] += vol_per_tick
                self.total_volume += vol_per_tick
        
        self.last_bar_time = timestamp
        self._calculate_metrics()

    def _calculate_metrics(self):
        if not self.profile: return

        prices = np.array(sorted(self.profile.keys()))
        volumes = np.array([self.profile[p] for p in prices])

        # Point of Control (POC)
        poc_idx = np.argmax(volumes)
        self.poc_price = prices[poc_idx]

        # Value Area (VA) - 70% of volume
        target_volume = self.total_volume * 0.7
        cumulative_volume = 0
        value_area_prices = []

        # Start from POC and expand outwards
        sorted_by_vol_desc = sorted(self.profile.items(), key=lambda item: item[1], reverse=True)
        
        for price, vol in sorted_by_vol_desc:
            value_area_prices.append(price)
            cumulative_volume += vol
            if cumulative_volume >= target_volume:
                break
        
        if value_area_prices:
            self.val = min(value_area_prices)
            self.vah = max(value_area_prices)
```
