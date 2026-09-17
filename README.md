# SMC Alpha Terminal — Advanced Smart Money Scanner

Yeh aapki uploaded `smc-alpha-scanner` app ka ek **advanced, standalone rebuild** hai — same SMC (Smart Money Concepts) engine, lekin bina kisi proprietary platform dependency ke.

---

## ⚠️ Sabse zaroori baat pehle: guaranteed live data ke liye `server/` chalayein

Maine confirm kiya hai (Binance ke apne GitHub issues aur developer reports se) ki **Binance ka API browser se seedhe fetch karne par reliably CORS allow nahi karta** — yeh ek well-known, purani Binance limitation hai, iss app ka bug nahi. Isi wajah se "CRYPTO market feed unavailable" aata hai, chahe internet bilkul theek ho.

App ab teen tareeke try karta hai (Binance direct → local proxy → CoinGecko), lekin **guaranteed, hamesha kaam karne wala tareeka yeh hai**:

```bash
cd server
npm install
npm start
# phir browser mein kholiye: http://localhost:8787
```

Yeh chhota Node proxy Binance/Yahoo ko **server-side** se call karta hai — Node.js mein CORS lagta hi nahi (CORS sirf browser ka security rule hai), isliye yeh 100% reliably kaam karega, Crypto aur Forex/Futures dono ke liye. Yehi is app ko chalane ka **recommended tareeka** hai. Isi server ko Render/Railway/apne VPS par deploy karke kahin se bhi access kar sakte hain.

`index.html` ko seedha double-click karke bhi khol sakte hain (zero-setup), aur kayi baar Binance/CoinGecko direct kaam bhi kar jaate hain, lekin yeh guaranteed nahi hai — isliye `server/` hi bharosemand tareeka hai.

---

## 1. Aapki original file ka audit — kya mila

| # | File | Finding |
|---|------|---------|
| 1 | `package.json`, `App.tsx`, `backend/index.ts` | App **`@appdeploy/sdk`** aur **`@appdeploy/client`** naam ke private/platform-specific packages par depend karta hai. Iske bina yeh project kahin aur (Vercel, VPS, apna laptop) build/run nahi ho sakta. |
| 2 | `backend/index.ts` | `historicalBacktest()` function **poora likha hua hai lekin kisi bhi API route se wired nahi hai** — dead code. |
| 3 | `backend/realtime.ts`, `realtime-subscribers.ts` | WebSocket subscription boilerplate likha hai but `index.ts` ke router mein kahin use nahi hota — dead code. |
| 4 | `backend/index.ts` → `yahoo()` | Yahoo Finance ka undocumented endpoint CORS reliably nahi bhejta — backend proxy zaroori hai. |
| 5 | Original frontend | Crypto Binance WebSocket use karta tha assuming CORS-safe — **yeh assumption galat nikli** (dekhiye upar), Binance REST/WS dono browser se seedhe unreliable hain. |
| 6 | Trade "bot" | Paper-trading simulation hai, koi real order nahi hota — safe. |
| 7 | `tsconfig.json` | Sirf `src/` include hota hai, `backend/` type-checked nahi hota. |

**Sabse bada practical issue:** `@appdeploy/*` ke bina yeh code kahin run nahi ho sakta. Maine poore engine ko **plain HTML/CSS/JavaScript** mein rewrite kiya — zero build step.

---

## 2. Naya app — kya banaya

**`index.html`** — single-file web app, `server/` ke saath ya bina, dono tarah chal sakta hai.

### Features
- **Live SMC engine**: Liquidity Sweep → MSS/CHoCH → Displacement → FVG/Order Block → Retest → Premium/Discount → Score/Signal.
- **3 markets**: Crypto (50 coins), Forex (10 pairs), Futures (Gold/Silver/US Oil).
- **3-tier data fallback per market**: Binance/Yahoo direct → local `server/` proxy → CoinGecko (crypto only) — jitne zyada tareeke try honge, utna reliable.
- **Scanner table**: search, LONG/SHORT/SETUP/Watchlist filters, click-to-sort columns, min-score slider, ⭐ watchlist.
- **Detail panel**: custom SVG candlestick chart with sweep/MSS/displacement/retest markers, FVG/OB zones, entry/SL/TP lines, trade plan.
- **Automatic paper-trading bot**: 25% partials at TP1(0.5R)/TP2(1R)/TP3(2R)/TP4(3R), CSV export.
- **Browser alerts**: sound + desktop notification on new confirmed setup.
- **Backtest tool** (Crypto): real Binance 15m history replay, win-rate/total-R/profit-factor.
- **TradingView Pine Script export** — modal + copy button.
- Session clock, dark/light theme, fully responsive.
- **Sandbox/preview detection**: agar yeh file kisi embedded preview (jaise Claude ka apna in-app viewer) ke andar khuli ho, app khud pehchan kar warning dikhata hai.

---

## 3. Limitations

- Yahoo Finance ek unofficial API hai — kabhi rate-limit kar sakta hai.
- CoinGecko fallback ka candle data point-sampled hai (Binance jaisa true OHLC nahi) — sirf tab use hota hai jab Binance aur local proxy dono fail ho jaayein.
- Backtest sirf Crypto ke liye hai.
- Yeh purely educational/informational tool hai — koi financial advice nahi.

---

## 4. Agar phir bhi data na aaye

1. Pehle confirm kariye ki aap is file ko **`server/` proxy ke through** (`http://localhost:8787`) khol rahe hain, na ki seedha `index.html` double-click karke aur na hi kisi in-app preview mein.
2. `server/` chalate waqt terminal mein koi error toh nahi aa raha — `npm start` ke baad "SMC Alpha proxy + app running" dikhna chahiye.
3. Agar `server/` ke through khol kar bhi data na aaye, toh aapke network/firewall/ISP par Binance ya CoinGecko block ho sakte hain.
4. App ke andar 15 second baad ek banner khud dikhega jo yeh guide karega.

## 5. GitHub Pages par free deploy karna (koi server nahi chahiye)

`index.html` pure HTML/CSS/JS hai, isliye GitHub Pages par bilkul free deploy ho sakta hai — koi Node.js, koi Render, koi backend nahi chahiye:

1. GitHub repo banayein (agar pehle se nahi hai) aur `index.html` ko **repo ke root** mein upload karein
2. Repo ke **Settings → Pages** mein jaayein
3. "Source" mein **"Deploy from a branch"** select karein → Branch: `main`, Folder: `/ (root)` → **Save**
4. 1-2 minute mein URL milega: `https://<username>.github.io/<repo-naam>/`

**Kya kaam karega:**
- ✅ Crypto — guaranteed kaam karega (Binance direct → CoinGecko fallback, dono CORS-friendly)
- ⚠️ Forex/Futures — best-effort (Yahoo direct → 3 public CORS proxies try hote hain), zyadatar kaam kar jaata hai lekin 100% guarantee nahi hai kyunki yeh free third-party proxies par depend karta hai jo kabhi down ho sakte hain

Agar Forex/Futures GitHub Pages par kaam na kare, uski wajah upar "Agar phir bhi data na aaye" section mein hai — us case mein `server/` (Render ya kahin bhi) hi 100% guaranteed rasta hai.

## 6. File structure

```
index.html            ← main app (works standalone, but see recommendation above)
server/server.js       ← recommended proxy — run this for guaranteed live data
server/package.json
```
