# MATLAB Financial Technical Indicator Analysis

This MATLAB script performs technical analysis on financial time series data. It loads stock data, calculates various common technical indicators, normalizes selected indicators, and prepares input features and output labels, likely for use in machine learning models for price prediction or analysis.

The script is tailored for data from the Belgrade Stock Exchange (BELEX), specifically using a file named `belex-arhiva-NIIS.csv`.

## Features

*   **Data Loading:** Imports data from a CSV file, expecting specific column names.
*   **Data Preprocessing:** Flips the order of the data, assuming the input CSV is in reverse chronological order (newest first).
*   **Indicator Calculation:**
    *   Simple Moving Average (SMA) of 'Cena' (Closing Price)
    *   Momentum
    *   Stochastic Oscillator (%K and %D)
    *   Relative Strength Index (RSI)
    *   Moving Average Convergence Divergence (MACD, Signal Line, MACD Histogram) using a custom EMA function.
    *   Commodity Channel Index (CCI)
    *   Larry Williams' %R
*   **Normalization:** Normalizes selected indicators (Momentum, RSI, SMA, MAD, CCI, Larry Williams' %R) to a range of [-1, 1].
*   **Output Label Generation:** Creates target outputs (`izlazMAX`, `izlazMIN`) representing the maximum 'Max' price and minimum 'Min' price over the next 5 periods.
*   **Data Combination:**
    *   Combines selected normalized indicators into a `combined_input` matrix.
    *   Combines the generated `izlazMAX` and `izlazMIN` into a `combined_output` matrix.

## Prerequisites

*   **MATLAB Environment:** A working installation of MATLAB.
*   **Statistics and Machine Learning Toolbox:** Required for functions like `movmean`, `movmad`, `movmax`, `movmin`.
*   **Input Data File:** A CSV file named `belex-arhiva-NIIS.csv` must be present in the same directory as the MATLAB script.

## Input Data File (`belex-arhiva-NIIS.csv`)

The script expects the CSV file `belex-arhiva-NIIS.csv` to have the following characteristics:

*   Data starts from the second row (header is skipped).
*   The data is assumed to be sorted in reverse chronological order (newest entries first), as the script uses `flipud` to process it chronologically.
*   **Required Columns:**
    *   `Datum_trgovanja`: Trading Date
    *   `Cena`: Closing Price (used as `Vrednost` for most calculations)
    *   `Promena`: Change
    *   `Promet`: Volume/Turnover
    *   `Open`: Opening Price
    *   `Min`: Minimum Price for the period
    *   `Max`: Maximum Price for the period

## Script Parameters

Several parameters for indicator calculations are defined at the beginning of the script:

*   `n = 7`: Period for Momentum and initial RSI calculations.
*   `kPeriod = 9`: Period for %K calculation in Stochastic Oscillator.
*   `dPeriod = 3`: Period for %D calculation (smoothing of %K) in Stochastic Oscillator.
*   `shortPeriod = 10`: Short period for MACD's Exponential Moving Average (EMA).
*   `longPeriod = 20`: Long period for MACD's EMA.
*   `signalPeriod = 9`: Period for MACD's signal line EMA.
*   `period = 14`: Period for CCI calculation and Larry Williams' %R.
*   `movingAvg` period is hardcoded to `10`.

These can be adjusted to experiment with different indicator sensitivities.

## How to Use

1.  **Ensure Prerequisites:** Make sure you have MATLAB and the Statistics and Machine Learning Toolbox installed.
2.  **Prepare Data:** Place the `belex-arhiva-NIIS.csv` file in the same directory as the `.m` script.
3.  **Open Script:** Open the MATLAB script in the MATLAB editor.
4.  **Run Script:** Execute the script by clicking the "Run" button or typing the script's name in the MATLAB Command Window (if it's in the current path).
5.  **Access Results:** After execution, the following variables will be available in the MATLAB workspace:
    *   `tabela`: The loaded and flipped data table.
    *   Individual indicator arrays (e.g., `momentum`, `RSI`, `MACDLine`, `CCI`, `RPercent`).
    *   Normalized indicator arrays (e.g., `normalizedMomentum`, `normalizedRSI`).
    *   `combined_input`: A matrix of selected normalized indicators (first 3355 rows).
    *   `combined_output`: A two-column matrix with future max and min prices.

## Calculated Indicators & Outputs

### Main Indicators Calculated:

*   `movingAvg`: 10-period simple moving average of `Cena`.
*   `momentum`: `n`-period price difference.
*   `stochasticK`, `stochasticD`: Stochastic Oscillator %K and %D lines.
*   `RSI`: Relative Strength Index.
*   `MACDLine`, `signalLine`, `MACDHistogram`: MACD components.
*   `CCI`: Commodity Channel Index.
*   `RPercent`: Larry Williams' %R.

### Normalization:

The script normalizes the following indicators to the range `[-1, 1]` using the formula:
`normalized_value = (x - min(x)) / (max(x) - min(x)) * 2 - 1`

*   `normalizedMomentum`
*   `normalizedRSI`
*   `normalizedSMA` (SMA of Typical Price used in CCI calculation)
*   `normalizedMAD` (Mean Absolute Deviation of Typical Price used in CCI calculation)
*   `normalizedCCI`
*   `normalizedLarryWilliamsR`

### Final Combined Datasets:

*   **`combined_input`**: A matrix of size `3355 x 5`.
    *   The script takes the first `3355` rows from the combined normalized data.
    *   **Columns:**
        1.  `normalizedMomentum`
        2.  `normalizedRSI`
        3.  `normalizedSMA` (of Typical Prices)
        4.  `normalizedMAD` (of Typical Prices)
        5.  `normalizedLarryWilliamsR`

*   **`combined_output`**: A matrix of size `(total_length - 4) x 2`.
    *   **Column 1 (`izlaz1`):** The maximum of the 'Max' price over the *next* 5 trading periods. For a given day `i`, this is `max(Max(i:i+4))`.
    *   **Column 2 (`izlaz2`):** The minimum of the 'Min' price over the *next* 5 trading periods. For a given day `i`, this is `min(Min(i:i+4))`.

    This structure suggests `combined_input` could be used to predict the price range defined by `combined_output` for a future 5-day window. Note that `combined_input` and `combined_output` will need careful alignment if used for supervised learning due to the look-ahead nature of `combined_output` and the truncation of `combined_input`.

## Helper Function

*   **`calculateEMA(data, period)`:**
    *   A custom function to calculate the Exponential Moving Average (EMA).
    *   It initializes the first EMA value as the simple mean of the first `period` data points.
    *   Subsequent EMA values are calculated using the standard EMA formula:
        `EMA(i) = (data(i) - EMA(i-1)) * alpha + EMA(i-1)`
        where `alpha = 2 / (period + 1)`.

## Potential Uses

The generated `combined_input` (features) and `combined_output` (targets) are well-suited for:

*   Training machine learning models (e.g., regression, neural networks) to predict future stock price highs and lows.
*   Quantitative trading strategy development.
*   Financial market analysis and research.

