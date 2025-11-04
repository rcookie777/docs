# PNL Calculation Example

## Scenario

A user makes the following trades on Token ABC:

| Time | Action | Shares | Price | Amount |
|------|--------|--------|-------|--------|
| T0 | BUY | 100 | $0.50 | $50 |
| T1 | BUY | 50 | $0.60 | $30 |
| T2 | SELL | 75 | $0.70 | $52.50 |
| T3 | (holding) | - | $0.80 (market) | - |

## Step-by-Step Calculation

### At Time T0
**After BUY 100 @ $0.50:**
- Position: 100 shares
- Total Cost: $50
- Avg Cost: $0.50/share
- Market Price: $0.50
- Market Value: 100 × $0.50 = $50
- Unrealized PNL: $50 - $50 = **$0**
- Realized PNL: $0
- **Total PNL: $0**

### At Time T1
**After BUY 50 @ $0.60:**
- Position: 100 + 50 = 150 shares
- Total Cost: $50 + $30 = $80
- Avg Cost: $80 / 150 = $0.533/share
- Market Price: $0.60
- Market Value: 150 × $0.60 = $90
- Unrealized PNL: $90 - $80 = **$10**
- Realized PNL: $0
- **Total PNL: $10**

### At Time T2
**After SELL 75 @ $0.70:**
- Avg Cost (before sell): $80 / 150 = $0.533/share
- Realized PNL from sale: 75 × ($0.70 - $0.533) = 75 × $0.167 = **$12.52**
- Remaining Position: 150 - 75 = 75 shares
- Remaining Cost: $80 - (75 × $0.533) = $80 - $40 = $40
- Market Price: $0.70
- Market Value: 75 × $0.70 = $52.50
- Unrealized PNL: $52.50 - $40 = **$12.50**
- Realized PNL: $12.52
- **Total PNL: $12.52 + $12.50 = $25.02**

### At Time T3
**No trades, just price movement:**
- Position: 75 shares
- Total Cost: $40
- Market Price: $0.80
- Market Value: 75 × $0.80 = $60
- Unrealized PNL: $60 - $40 = **$20**
- Realized PNL: $12.52
- **Total PNL: $12.52 + $20 = $32.52**

## Summary Table

| Time | Position | Cost | Market Price | Market Value | Unrealized PNL | Realized PNL | Total PNL |
|------|----------|------|--------------|--------------|----------------|--------------|-----------|
| T0 | 100 | $50.00 | $0.50 | $50.00 | $0.00 | $0.00 | **$0.00** |
| T1 | 150 | $80.00 | $0.60 | $90.00 | $10.00 | $0.00 | **$10.00** |
| T2 | 75 | $40.00 | $0.70 | $52.50 | $12.50 | $12.52 | **$25.02** |
| T3 | 75 | $40.00 | $0.80 | $60.00 | $20.00 | $12.52 | **$32.52** |

## Multiple Tokens Example

If the user also traded Token XYZ:

**Token XYZ:**
- BUY 200 @ $0.30 = $60 cost
- Current Price: $0.25
- Market Value: 200 × $0.25 = $50
- Unrealized PNL: $50 - $60 = **-$10**

**Combined Portfolio at T3:**
- Token ABC Total PNL: $32.52
- Token XYZ Total PNL: -$10.00
- **Portfolio Total PNL: $22.52**

## Partial Fill Example (LIMIT Orders)

If the T2 SELL was a **LIMIT order** that was only partially filled:

**LIMIT SELL order: 75 shares requested, 50 shares filled @ $0.70:**
- Use `filled_shares` (50) instead of `shares` (75)
- Realized PNL from sale: 50 × ($0.70 - $0.533) = **$8.35**
- Remaining Position: 150 - 50 = 100 shares
- Remaining Cost: $80 - (50 × $0.533) = $80 - $26.65 = $53.35

**Note**: MARKET, STOPLOSS, and COPYTRADE orders always use the full `shares` amount since they execute completely. Only LIMIT orders can be partially filled, which is why we check the order type.

## Backend JSON Response

```json
{
  "success": true,
  "response": [
    {
      "timestamp": 1697500800,
      "pnl": "0.00"
    },
    {
      "timestamp": 1697504400,
      "pnl": "10.00"
    },
    {
      "timestamp": 1697508000,
      "pnl": "25.02"
    },
    {
      "timestamp": 1697511600,
      "pnl": "32.52"
    }
  ]
}
```

## Chart Display

The frontend would render these points as a line chart showing the PNL progression over time:

```
$35 |                               •
    |                           /
$30 |                       /
    |                   /
$25 |               • 
    |           /
$20 |       /
    |   /
$15 |
    |
$10 | •
    |
$5  |
    |
$0  |•
    +---+---+---+---+---+---+---+---
     T0  T1  T2  T3
```

This visual representation helps users understand their trading performance over time.



