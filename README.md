# autoregression//@version=5
indicator("Autoregression Model", overlay=true)

// Input parameters
lookback = input(50, title="Lookback Period")
forecast_period = input(14, title="Forecast Period")
target = input(close, title="Target Series")

// Function to perform linear regression
linearReg(x, y, length) =>
    x1 = bar_index - length + 1
    y1 = y - ta.linreg(y, length, 0)
    sumX1Y1 = ta.sma(x1 * y1, length) - ta.sma(x1, length) * ta.sma(y1, length) / length
    sumX1X1 = ta.sma(x1 * x1, length) - ta.sma(x1, length) * ta.sma(x1, length) / length
    slope = sumX1Y1 / sumX1X1
    intercept = ta.sma(y1, length) / length - slope * ta.sma(x1, length) / length
    slope * (bar_index) + intercept  // Use bar_index without subtracting length

// Compute autoregression
autoregression = linearReg(ta.valuewhen(bar_index > lookback, target, 0), target, lookback)

// Plot autoregression line
plot(autoregression, color=color.blue, title="Autoregression Line")

// Function to calculate slope and intercept
calc_slope_intercept(x, y, length) =>
    x1 = bar_index - length + 1
    y1 = y - ta.linreg(y, length, 0)
    sumX1Y1 = ta.sma(x1 * y1, length) - ta.sma(x1, length) * ta.sma(y1, length) / length
    sumX1X1 = ta.sma(x1 * x1, length) - ta.sma(x1, length) * ta.sma(x1, length) / length
    slope = sumX1Y1 / sumX1X1
    intercept = ta.sma(y1, length) / length - slope * ta.sma(x1, length) / length
    [slope, intercept]

// Calculate slope and intercept for forecasted autoregression line
[slope, intercept] = calc_slope_intercept(ta.valuewhen(bar_index > lookback, target, 0), target, lookback)

// Plot forecasted autoregression line
for i = 0 to forecast_period - 1
    forecasted_index = bar_index[lookback - 1] + i
    forecasted_autoregression = autoregression[lookback - 1] + slope * (lookback - 1 + i)
    plot(forecasted_autoregression, color=color.red, title="Forecasted Autoregression Line")

