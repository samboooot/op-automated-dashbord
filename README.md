# Autamated dashboard opinion trade
## Instructions
---

## 1. Setup

Fill in the `.env` file:

```
MULTISIG_ADDRESS=
```

> 💡 **Where to find MULTISIG_ADDRESS:**  
> Open your profile on the website → **Balance Spot** → address under the balance.

In `opinion_client.py`, line 37, specify the `fingerprint`. You can get it from the browser: paste the code below and navigate to any page:
```bash
(function() {
    const origOpen = XMLHttpRequest.prototype.open;
    const origSetHeader = XMLHttpRequest.prototype.setRequestHeader;
    
    XMLHttpRequest.prototype.setRequestHeader = function(name, value) {
        if (name.toLowerCase() === 'x-device-fingerprint') {
            console.log('Fingerprint:', value);
            navigator.clipboard.writeText(value).then(() => {
                console.log('✅ Copied!');
            });
        }
        return origSetHeader.apply(this, arguments);
    };
    
    console.log('Navigate to any page on the site...');
})();
```

---

## 2. Installation

### MacOS:
```bash
cd <your_folder>
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Windows:
```bash
cd <your_folder>
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

---

## 3. Running

### MacOS:
```bash
source venv/bin/activate
python -m uvicorn web.app:app --reload --port 8080
```

### Windows:
```bash
venv\Scripts\activate
python -m uvicorn web.app:app --reload --port 8080
```

Open in your browser: `http://localhost:8080`

---

## How to get Auth Token

> ⚠️ **Important:** The token is valid for 24 hours.

### Steps:
1. Open the prediction website
2. Log in to your account
3. Open DevTools (F12) → Console
4. Paste the content of the `get_auth_token` script, or simply drag and drop the file into the console and the code will be pasted. Press Enter.
5. Navigate to any page — the token will be automatically copied to your clipboard.
6. Save it in the dashboard settings.

> 💡 **Tip:** An hour or two before the old token expires, log out and log in again to get a new token.

---

## Split & Sell

Converts USDT into YES + NO shares and automatically puts them up for sale.

### Parameters:
1. Sell Steps — how many stages to split the sale into. If 1, then without splitting, it will simply put the received Shares up for sale with a limit order.
2. Aggressive Mode — sell more of the position that is more expensive. (Available if you specify Steps 2 or higher)

---

## Aggressive Mode — Examples

With Aggressive Mode enabled, the bot sells more of the position that is more expensive by the percentage difference between the two prices.

**Formula:**
- `(high_price - low_price) / high_price = % value`

For example:
- `(0.66 - 0.34) / 0.66 = 48%`

The more expensive side will be listed for sale 48% more aggressively.

---

### Example 1: YES = 0.34, NO = 0.66 | Steps = 2

NO is more expensive → we sell more NO
- `(0.66 - 0.34) / 0.66 = 48%`

**Split 100 USDT → 100 YES + 100 NO shares**

| Step | YES | NO |
|------|-----|-----|
| 1 | 37.9 | 62.1 |
| 2 | 62.1 | 37.9 |

---

### Example 2: YES = 0.78, NO = 0.22 | Steps = 3

YES is more expensive → we sell more YES
- `(0.78 - 0.22) / 0.78 = 72%`

**Split 100 USDT → 100 YES + 100 NO shares**

| Step | YES | NO |
|------|-----|-----|
| 1 | 45.3 | 21.4 |
| 2 | 45.3 | 21.4 |
| 3 | 9.4 | 57.2 |

---

### Example 3: YES = 0.78, NO = 0.22 | Steps = 4

YES is more expensive → we sell more YES
- `(0.78 - 0.22) / 0.78 = 72%`

**Split 100 USDT → 100 YES + 100 NO shares**

| Step | YES | NO |
|------|-----|-----|
| 1 | 34.0 | 16.0 |
| 2 | 34.0 | 16.0 |
| 3 | 34.0 | 16.0 |
| 4 | -2.0 | 52.0 |

> ⚠️ **Note:** If the shares of one side run out early, the remainder is carried over to the last step. In this example, it would be:

| Step | YES | NO |
|------|-----|-----|
| 1 | 34.0 | 16.0 |
| 2 | 34.0 | 16.0 |
| 3 | 34.0 | 68.0 |

## Buy Shares (Market Maker)

This is the main operating mode. It both buys and sells. Sell is an emergency mode if the "connection is lost".

Places BUY orders for YES and/or NO positions, automatically adjusts the price.

### Modes:
- **Standard** — places at the best price in the order book.
- **Spread** — places at +0.001 above the best price (first in line).

### Single Order:
If enabled — places only YES or only NO.

---

## Sell Shares

Sells all available positions at the best price with automatic adjustment.

### How it works:
1. Gets all your positions.
2. Places SELL orders at the best price.
3. If someone sets a better price — automatically moves the order.
4. Works until all are sold or Stop is pressed.

---

## Automatic Price Adjustment

The bot monitors the order book and automatically moves orders:

| Order Type | Condition | Action |
|------------|-----------|--------|
| BUY | Your price < best bid | Moves to the best bid |
| SELL | Your price > best ask | Moves to the best ask |

> 💡 **Min Volume:** Orders with an amount less than Min Volume are ignored when determining the best price.

---

## Useful Tips

- **Min Volume** — use to ignore small orders.
- When you press **Stop**, all active orders are canceled.
- Monitoring works with the period set in the settings.
- Logs are updated every 5 minutes or when an order status changes.

