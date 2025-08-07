# API Reference: Quantum Cipher

This document provides a technical reference for the `Quantum Cipher` signal generation module. This module is responsible for calculating momentum, volume, and trend-based signals.

### Core Logic

The Quantum Cipher combines several standard indicators into a single, cohesive signal:
1.  An **Adaptive Momentum Oscillator** to gauge the speed and direction of price changes.
2.  A **Volume-Weighted Confirmation** using VWAP and a volume spike ratio to ensure conviction.
3.  A **Dual EMA Trend Filter** to ensure signals are only taken in the direction of the prevailing trend.

---

### Functions

#### `calculate_quantum_cipher(df)`

Calculates the full set of Quantum Cipher signals from a market data DataFrame.

*   **Parameters:**
    *   `df` (pandas.DataFrame): A DataFrame containing OHLCV data with the following columns: `timestamp`, `open`, `high`, `low`, `close`, `volume`.

*   **Returns:** (pandas.DataFrame)
    *   A new DataFrame with the original OHLCV data plus several calculated columns, including:
        *   `vwap`: The Volume-Weighted Average Price.
        *   `osc`: The final Momentum Oscillator value.
        *   `buy_signal` (bool): `True` if a buy signal is generated for a given candle.
        *   `sell_signal` (bool): `True` if a sell signal is generated for a given candle.

*   **Usage Example:**
    ```python
    import pandas as pd
    from app.logic.cipher import fetch_market_data, calculate_quantum_cipher

    # 1. Fetch market data
    market_df = fetch_market_data('binance', 'BTC/USDT', '1h')

    # 2. Calculate signals
    if not market_df.empty:
        signals_df = calculate_quantum_cipher(market_df)
        print(signals_df[['timestamp', 'close', 'buy_signal', 'sell_signal']].tail())
    ```

#### `fetch_market_data(exchange, symbol, timeframe, limit=1000)`

Fetches historical OHLCV data from a specified cryptocurrency exchange using the `ccxt` library.

*   **Parameters:**
    *   `exchange` (str): The ID of the exchange (e.g., `'binance'`).
    *   `symbol` (str): The trading pair symbol (e.g., `'BTC/USDT'`).
    *   `timeframe` (str): The candle timeframe (e.g., `'15m'`, `'1h'`, `'1d'`).
    *   `limit` (int, optional): The number of candles to fetch. Defaults to `1000`.

*   **Returns:** (pandas.DataFrame)
    *   A DataFrame containing the requested OHLCV data.

---

### Full Source Code

<details>
<summary>Click to view the full source code for `cipher.py`</summary>

```python
import pandas as pd
import numpy as np
import ccxt
import time

def calculate_quantum_cipher(df):
    """
    Calculate Quantum Cipher signals from market data
    """
    # 1. Calculate core price values
    df['hlc3'] = (df['high'] + df['low'] + df['close']) / 3

    # 2. Adaptive Momentum Oscillator
    df['mom'] = df['hlc3'].diff()
    df['abs_mom'] = df['mom'].abs()
    df['smooth_mom'] = df['mom'].ewm(span=14, adjust=False).mean()
    df['smooth_abs'] = df['abs_mom'].ewm(span=14, adjust=False).mean()
    df['osc'] = 100 * df['smooth_mom'] / (df['smooth_abs'] + 1e-9) # Avoid division by zero

    # 3. Volume-Weighted Confirmation
    cumulative_vp = (df['hlc3'] * df['volume']).cumsum()
    cumulative_vol = df['volume'].cumsum()
    df['vwap'] = cumulative_vp / (cumulative_vol + 1e-9) # Avoid division by zero
    df['volume_sma20'] = df['volume'].rolling(window=20).mean()
    df['v_ratio'] = df['volume'] / (df['volume_sma20'] + 1e-9) # Avoid division by zero

    # 4. Trend Filter
    df['ema_fast'] = df['close'].ewm(span=8, adjust=False).mean()
    df['ema_slow'] = df['close'].ewm(span=21, adjust=False).mean()

    # 5. Signal Conditions
    df['bull_vol'] = (df['close'] > df['vwap']
</details>
```
