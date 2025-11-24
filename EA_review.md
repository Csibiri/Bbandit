# BBandit_v14 Expert Advisor Review

## Compilation-level issues
- `CalculateLotSize()` calls `SymbolInfoString(symbol, SYMBOL_CURRENCY_PROFIT)` (line 462), but this API is only available in MQL5. Under MT4 the code will not compile. Use `MarketInfo(symbol, MODE_CURRENCY)` or other MT4-compatible functions to read the profit currency instead.

## Logic issues that can misprice orders
- `GetHL_Buy()`/`GetHL_Sell()` return bar indices rather than prices (lines 501-510). They pass the *shift* returned by `iHighest`/`iLowest` straight back instead of the corresponding high/low price. Any stop-loss values derived from these functions will be expressed in bar offsets instead of prices, leading to nonsensical SL levels when used by `ManageStopLoss()` and other callers.

## Risk/margin calculation correctness
- The cross-rate calculation in `CalculateLotSize()` assumes that the concatenated symbol (e.g., `EURUSD`) exists in the broker's symbol list (lines 470-483). On MT4 many brokers use suffixes/prefixes, so these lookups can return zero and fall back to the raw balance, overestimating position size. Consider checking `MarketInfo(..., MODE_BID)` return values and symbol availability more defensively.

## Runtime robustness
- Several functions (e.g., `UpdatePendingOrders`, `ManageStopLoss`) iterate all orders and rely solely on magic numbers to distinguish BUY vs SELL. Pending orders with the opposite magic number but different type could be modified unintentionally, because `isBuy` is inferred only from the magic number (line 574) instead of the order type. Guarding by `OrderType()` alongside the magic number would avoid misclassifying SELL pendings as BUY and vice versa.

