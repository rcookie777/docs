# Historical PNL Calculation System

## Overview

This document describes the implementation of the historical PNL (Profit and Loss) tracking system for Polymarket positions.

## Features

### 1. Accurate Historical Reconstruction
- Fetches all filled orders for a wallet
- Reconstructs position history at each time point
- Handles both active and inactive markets
- Supports partially filled orders

### 2. Time Range Support
- **1D**: 1 day with hourly intervals
- **1W**: 7 days with 2-hour intervals  
- **1M**: 30 days with 6-hour intervals
- **3M**: 90 days with 12-hour intervals
- **1Y**: 365 days with 24-hour intervals
- **ALL**: 2 years with 24-hour intervals

### 3. Position Tracking
- Uses `filled_shares` field for LIMIT orders (can be partially filled)
- Uses `shares` field for MARKET, STOPLOSS, and COPYTRADE orders (always fill completely)
- Tracks both BUY and SELL orders
- Maintains accurate cost basis using FIFO accounting
- Separates realized and unrealized PNL

## Backend Implementation

### Algorithm

1. **Fetch Orders**: Get all FILLED/TRIGGERED orders for the wallet
2. **Get Historical Prices**: Fetch price history for each unique token from Polymarket API
3. **Generate Time Points**: Create time series based on requested interval
4. **Calculate PNL at Each Point**:
   - Replay all orders up to that timestamp
   - Calculate position shares and cost basis
   - Get market price at that timestamp
   - Compute unrealized PNL: (shares × current_price) - cost_basis
   - Add any realized PNL from SELL orders

### Key Logic

```rust
// For each time point
for timestamp in time_points {
    for each token {
        // Replay orders up to this timestamp
        position_shares = 0
        total_cost = 0
        
        for order in orders_up_to_timestamp {
            if BUY:
                position_shares += filled_shares
                total_cost += filled_shares × price
            else if SELL:
                avg_cost = total_cost / position_shares
                position_shares -= filled_shares
                total_cost -= filled_shares × avg_cost
                realized_pnl += filled_shares × (price - avg_cost)
        }
        
        // Calculate unrealized PNL
        if position_shares > 0:
            market_value = position_shares × current_price
            unrealized_pnl = market_value - total_cost
            total_pnl += unrealized_pnl
    }
}
```

### Handling Edge Cases

1. **Partial Fills**: 
   - LIMIT orders use `filled_shares` (can be partially filled)
   - MARKET, STOPLOSS, COPYTRADE orders use `shares` (always fill completely)
2. **Inactive Markets**: Still fetches historical prices from Polymarket API
3. **Missing Price Data**: Uses closest available price or first price if none before timestamp
4. **Empty Positions**: Returns empty array if no orders exist

## Frontend Implementation

### Data Flow

1. User selects time range (1D, 1W, 1M, etc.)
2. Frontend calculates `days` and `interval` parameters
3. Calls `/api/v1/wallet/pol/{wallet_id}/pnl-chart?days={days}&interval={interval}`
4. Receives array of `{ timestamp, pnl }` points
5. Formats and displays in chart

### Response Format

```json
[
  {
    "timestamp": 1697500800,
    "pnl": "123.45"
  },
  {
    "timestamp": 1697522400,
    "pnl": "145.67"
  }
]
```

### Chart Formatting

The `formatPnlDataForChart` function converts timestamps to readable dates:

```typescript
{
  date: "Oct 16",  // Human-readable date
  value: 123.45    // PNL as number
}
```

## API Endpoint

### Request

```
GET /api/v1/wallet/pol/{wallet_id}/pnl-chart?days=30&interval=6
```

### Parameters

- `wallet_id` (path): The wallet ID to get PNL for
- `days` (query): Number of days to look back (1-730)
- `interval` (query): Hours between data points (1-24)

### Response

```json
{
  "success": true,
  "response": [
    {
      "timestamp": 1697500800,
      "pnl": "123.45"
    }
  ]
}
```

## Performance Considerations

1. **Price History Caching**: Price histories are fetched once per token
2. **Efficient Time Series**: Only calculates PNL at requested intervals
3. **Sorted Data**: Orders are fetched sorted by timestamp for efficient replay
4. **Early Exit**: Breaks order replay loop once timestamp exceeded

## Future Enhancements

1. **Caching**: Cache calculated PNL data for recently accessed wallets
2. **Real-time Updates**: WebSocket support for live PNL updates
3. **Multi-wallet**: Aggregate PNL across multiple wallets
4. **Breakdown**: Show PNL by market or category
5. **Benchmark**: Compare against market performance

