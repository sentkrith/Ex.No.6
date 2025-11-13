# Ex.No.6 Development of Python Code Compatible with Multiple AI Tools

# Date:13-11-2025
# Register no:212223060254
# Aim: Write and implement Python code that integrates with multiple AI tools to automate the task of interacting with APIs, comparing outputs, and generating actionable insights with Multiple AI Tools

# AI Tools Required:

ChatGPT (GPT-5 or GPT-4) — generate and explain Python code.

Google Gemini / Bard — compare code logic, alternative endpoints, and exchange-specific caveats.

GitHub Copilot — in-editor suggestions and refactors.

Replit / Jupyter Notebook — run and test the code.

Postman / RapidAPI — explore endpoints and test requests.

# Explanation:
Experiment the persona pattern as a programmer for any specific applications related with your interesting area. 
Generate the outoput using more than one AI tool and based on the code generation analyse and discussing that. 

# Objective:
To design effective prompts that guide AI tools (like ChatGPT) to generate Python code, compare outputs from multiple APIs, and suggest meaningful insights — improving students’ ability to communicate coding needs to AI.

# Stage 1: Generate Python Code for Interacting with Multiple APIs

## Prompt (to ChatGPT):
Write a Python program that connects to CoinGecko and Binance public APIs, fetches the current USD price for a given symbol (e.g., BTC, ETH), and prints the results neatly. Include basic error handling and symbol normalization.

## OUTPUT:

```python
import requests
import time
# Simple symbol map (user-friendly -> API-specific)

SYMBOL_MAP = {
    "BTC": {"coingecko": "bitcoin", "binance": "BTCUSDT"},
    "ETH": {"coingecko": "ethereum", "binance": "ETHUSDT"},
    # add more as needed
}

def get_coingecko_price(symbol_id):
    url = "https://api.coingecko.com/api/v3/simple/price"
    params = {"ids": symbol_id, "vs_currencies": "usd", "include_last_updated_at": "true"}
    resp = requests.get(url, params=params, timeout=10)
    resp.raise_for_status()
    data = resp.json()
    if symbol_id not in data:
        raise ValueError("CoinGecko: symbol not found in response")
    price = data[symbol_id]["usd"]
    ts = data[symbol_id].get("last_updated_at")
    return {"source": "CoinGecko", "price": float(price), "timestamp": ts}

def get_binance_price(symbol_pair):
    # Binance public ticker endpoint for symbol price
    url = "https://api.binance.com/api/v3/ticker/price"
    params = {"symbol": symbol_pair}
    resp = requests.get(url, params=params, timeout=10)
    resp.raise_for_status()
    data = resp.json()
    # response: {"symbol":"BTCUSDT","price":"27123.45000000"}
    price = data.get("price")
    if price is None:
        raise ValueError("Binance: price not found")
    return {"source": "Binance", "price": float(price), "symbol": data.get("symbol")}

if __name__ == "__main__":
    user_symbol = "BTC"  # or get from input()
    mapping = SYMBOL_MAP.get(user_symbol.upper())
    if not mapping:
        raise ValueError(f"Symbol {user_symbol} not supported. Add mapping in SYMBOL_MAP.")
    try:
        cg = get_coingecko_price(mapping["coingecko"])
    except Exception as e:
        print("CoinGecko error:", e)
        cg = None

    try:
        bn = get_binance_price(mapping["binance"])
    except Exception as e:
        print("Binance error:", e)
        bn = None

    print("Prices fetched:")
    if cg:
        print(f"- CoinGecko: ${cg['price']:.2f}")
    if bn:
        print(f"- Binance:   ${bn['price']:.2f}")
```

## AI Explanation:
The code queries CoinGecko’s simple price endpoint and Binance’s ticker endpoint, normalizes symbol names via SYMBOL_MAP, handles HTTP errors via raise_for_status(), and prints prices in USD. Add more symbols to SYMBOL_MAP as needed.

# Stage 2: Compare Outputs from Different APIs

## Prompt:
Modify the code to compare prices, compute percentage difference, and flag discrepancies larger than a threshold (e.g., 0.5%). Also compute a simple average price.

## OUTPUT:
```python
# additions / modifications to the previous script

def compare_prices(price_a, price_b):
    if price_a is None or price_b is None:
        return None
    diff_abs = abs(price_a - price_b)
    diff_pct = (diff_abs / ((price_a + price_b) / 2)) * 100  # percent relative to mean
    return {"abs_diff": diff_abs, "pct_diff": diff_pct}

 in main after fetching cg and bn:
cg_price = cg["price"] if cg else None
bn_price = bn["price"] if bn else None

if cg_price is None and bn_price is None:
    print("No prices available.")
else:
    if cg_price is not None:
        print(f"CoinGecko: ${cg_price:.2f}")
    if bn_price is not None:
        print(f"Binance:   ${bn_price:.2f}")

    comp = compare_prices(cg_price, bn_price)
    if comp:
        print(f"Absolute diff: ${comp['abs_diff']:.2f}")
        print(f"Percent diff: {comp['pct_diff']:.3f}%")
        if comp["pct_diff"] > 0.5:
            print("⚠️ Significant price divergence between sources (>0.5%).")
        else:
            print("✅ Prices consistent (<=0.5%).")

    # simple average
    present = [p for p in (cg_price, bn_price) if p is not None]
    avg = sum(present) / len(present)
    print(f"Average price (simple): ${avg:.2f}")
```

# AI Explanation:
The script computes absolute and percent differences (relative to the mean of the two prices), flags divergence beyond 0.5%, and prints a simple arithmetic mean as a quick fallback.

# Stage 3: Suggest Insights or Next Steps

## Prompt:
Based on compared crypto prices, suggest a way to choose the most reliable price (e.g., prefer exchange with higher volume, compute volume-weighted price if volume available, fall back to median, and add checks for stale data). Provide code sketch for using volume when available.

## OUTPUT:
```python
# sketch additions to incorporate volume-based weighting (illustrative)

def get_binance_price_and_volume(symbol_pair):
    # ticker 24hr endpoint provides price change and volume
    url = "https://api.binance.com/api/v3/ticker/24hr"
    params = {"symbol": symbol_pair}
    resp = requests.get(url, params=params, timeout=10)
    resp.raise_for_status()
    data = resp.json()
    price = float(data["lastPrice"])
    # volume is base asset volume over 24h (string)
    vol = float(data.get("volume", 0.0))
    return {"source": "Binance", "price": price, "volume": vol}

def choose_weighted_price(sources):
    # sources: list of dicts with 'price' and optional 'volume'
    weighted_sum = 0.0
    total_weight = 0.0
    for s in sources:
        p = s.get("price")
        w = s.get("volume") or 1.0  # fallback weight if no volume
        weighted_sum += p * w
        total_weight += w
    if total_weight == 0:
        return None
    return weighted_sum / total_weight
```
# AI Explanation:
When volume is available, compute a volume-weighted average prioritizing exchanges with more trading activity. If volumes are missing, use the median (robust to outliers) or simple average. Also check for stale timestamps and rate-limit responses.

Suggested Next Steps & Practical Tips:

Add timestamp checks: ensure data is recent (<30s for spot exchange, <2min for aggregated APIs).

Respect API rate limits — add exponential backoff and caching.

Expand SYMBOL_MAP and add a routine to auto-map symbols by querying APIs (safer than hard-coding).

Add unit tests that mock API responses (use requests-mock or pytest with fixtures).

For production: add retry logic, logging, and a small local cache (TTL) to avoid hitting rate limits.

# Reflection Note:

My early prompts omitted symbol mapping and volume considerations, leading AI to return simplistic code. Adding explicit requirements — “include symbol normalization”, “compute percent difference relative to mean”, “flag >0.5%”, “use volume-weighted average if volume exists” — produced more robust, production-minded code. Future prompts should include expected thresholds, freshness requirements, and preferred fallback strategies.

# Conclusion:

This exercise demonstrates how prompt engineering guides AI to produce practical integration code across multiple APIs. Comparing outputs from multiple AI tools highlights different heuristics: some tools will prioritize simplicity, others will automatically suggest production concerns (retries, volumes, caching). Clear, constrained prompts yield better, testable code that students can run in Jupyter/Replit and validate with Postman/RapidAPI.

# Result: 
The corresponding Prompt is executed successfully.
