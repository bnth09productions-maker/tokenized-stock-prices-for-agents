# How to get tokenized stock prices (xStocks, Ondo) from an AI agent

Tokenized stocks like **TSLAx, SPYx and NVDAx (xStocks on Solana)** and **Ondo Global Markets tokens (on Ethereum and BNB Chain)** trade on-chain all day, every day. An agent that wants to price them, or spot a DEX premium or discount against the real stock, needs one place that lists the tokens and their prices.

This guide shows how to get that data from [scraplingdata.xyz](https://scraplingdata.xyz). You can use a free HTTP call, a $0.005 pay-per-call x402 route, or MCP tools.

## 1. Free: the top 10 by supply value (no key, no payment)

```bash
curl https://scraplingdata.xyz/v1/free/tokenized-stocks
```

Here is real output, fetched on Oct 29, 2026 and trimmed to 2 of the 10 rows:

```json
{
  "tokens_tracked": 954,
  "chains": [
    "BSC",
    "Ethereum",
    "Solana"
  ],
  "top10": [
    {
      "rank": 1,
      "symbol": "TSLAx",
      "underlying_ticker": "TSLA",
      "issuer": "xStocks (Backed)",
      "chain": "Solana",
      "total_supply": 229633.89186,
      "supply_value_usd": 92202600.0,
      "oracle_price_usd": 401.52
    },
    {
      "rank": 2,
      "symbol": "SPYx",
      "underlying_ticker": "SPY",
      "issuer": "xStocks (Backed)",
      "chain": "Solana",
      "total_supply": 95774.749309,
      "supply_value_usd": 73642641.0,
      "oracle_price_usd": 768.915
    }
  ],
  "free": true,
  "upgrade": {
    "note": "Paid: every token with contract address, DEX price vs oracle and premium/discount %, ticker/chain filters",
    "endpoint": "https://scraplingdata.xyz/v1/lookup/tokenized-stocks",
    "price_usd": 0.005,
    "payment": "x402 v2 USDC (Base, Polygon, Arbitrum, Solana)"
  },
  "source": "On-chain only: totalSupply via public RPCs (Ethereum, BNB Chain, Solana), Uniswap v3 / PancakeSwap v3 pool prices, Chainlink equity feeds read on-chain, else Pyth Network equity oracle."
}
```

These are the top 10 from that same call (prices move, so run the call yourself for current numbers):

| # | Token | Issuer | Chain | Oracle price | Supply value |
|---|---|---|---|---|---|
| 1 | TSLAx | xStocks (Backed) | Solana | $401.52 | $92,202,600 |
| 2 | SPYx | xStocks (Backed) | Solana | $768.915 | $73,642,641 |
| 3 | NVDAx | xStocks (Backed) | Solana | $212.1855 | $68,282,629 |
| 4 | MSFTx | xStocks (Backed) | Solana | $528.085 | $55,759,906 |
| 5 | AAPLx | xStocks (Backed) | Solana | $349.835 | $53,967,056 |
| 6 | STRCon | Ondo Global Markets | Ethereum | $99.9499 | $46,892,124 |
| 7 | GOOGLx | xStocks (Backed) | Solana | $283.335 | $45,555,884 |
| 8 | METAx | xStocks (Backed) | Solana | $622.65 | $45,193,700 |
| 9 | SPYon | Ondo Global Markets | Ethereum | $768.915 | $41,874,993 |
| 10 | AMZNx | xStocks (Backed) | Solana | $236.985 | $40,831,135 |

## 2. Paid: every token, with filters ($0.005 per call over x402)

```
GET https://scraplingdata.xyz/v1/lookup/tokenized-stocks?ticker=TSLA
GET https://scraplingdata.xyz/v1/lookup/tokenized-stocks?chain=Solana&limit=50
GET https://scraplingdata.xyz/v1/lookup/tokenized-stocks?issuer=Ondo
```

- **Filters:** `ticker`, `chain`, `issuer`, `limit`.
- **What it returns:** every tracked token with its issuer, chain, contract or mint address, supply, oracle price, and the DEX price plus premium/discount % where a pool exists.
- **Payment:** the first request returns HTTP `402 Payment Required` with x402 payment details. Pay with USDC on Base, Polygon, Arbitrum or Solana. Any x402 client works, for example `x402-fetch` or `x402-axios`, or Coinbase's `npx awal` wallet:

```bash
npx awal x402 pay "https://scraplingdata.xyz/v1/lookup/tokenized-stocks?ticker=TSLA"
```

## 3. MCP tools (Claude, Cursor and other MCP clients)

The store's MCP server exposes two tools:
- `free_tokenized_stocks`: the free top 10
- `tokenized_stocks`: the paid full lookup, with the same filters

To set it up, see [scrapling-x402-mcp](https://github.com/bnth09productions-maker/scrapling-x402-mcp) and https://scraplingdata.xyz/llms.txt.

## What's covered

- **Tokens:** 954 in total. Ondo Global Markets tokens on Ethereum and BNB Chain, and xStocks (Backed) tokens on Solana.
- **Supply:** `totalSupply` read directly from public RPCs.
- **Prices:** Chainlink equity feeds read on-chain first, with the Pyth Network equity oracle as the fallback.
- **DEX price and premium/discount:** from Uniswap v3, PancakeSwap v3 and Orca pools, wherever a pool has enough liquidity.

## Honest gaps (as of Oct 2026)

- Robinhood Chain, Securitize and DTCC tokens aren't covered yet.
- Holder counts and 24h volume aren't filled in yet.
- Most tokens have no liquid DEX pool, so their DEX price and premium fields are empty. About 14 tokens have a DEX price today.
- After a server restart, the data can take about 2 minutes to refill.

## License

The guide text is MIT. All data comes from public on-chain sources.
