# Meridian — Panduan Konfigurasi

## Gambaran Umum

Konfigurasi Meridian tersimpan di dua tempat:
- **`.env`** — API keys, wallet, RPC (rahasia)
- **`user-config.json`** — Semua parameter strategi, threshold, scheduling

Kamu bisa ubah config via:
1. Edit `user-config.json` langsung (restart diperlukan)
2. Telegram: `/setcfg <key> <value>` (langsung aktif)
3. Telegram: `/settings` (menu button interaktif)
4. Chat ke bot: "ubah stopLossPct jadi -25" (LLM eksekusi)
5. CLI: `node cli.js config set <key> <value>`

---

## Konfigurasi LLM Provider

### OpenRouter (Default)

```env
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxx
```

### Ultramen Proxy (Claude via proxy lokal)

```env
LLM_BASE_URL=http://localhost:4141/v1
LLM_API_KEY=ulm-f9c689ab4be8c2e084f4571765046b9a6b7db5d4f0286f92
LLM_MODEL=claude-opus-4.6
```

Tambahkan di `user-config.json`:
```json
{
  "managementModel": "claude-opus-4.6",
  "screeningModel": "claude-opus-4.6",
  "generalModel": "claude-opus-4.6"
}
```

### LM Studio (Local)

```env
LLM_BASE_URL=http://localhost:1234/v1
LLM_API_KEY=lm-studio
LLM_MODEL=your-local-model-name
```

### Syarat LLM Provider:
- Harus support OpenAI Chat Completions format (`/v1/chat/completions`)
- Harus support Function Calling / Tool Use
- `max_tokens` minimal 2048 (model gratis sering terlalu rendah)

---

## Konfigurasi Screening (Filter Pool)

### Semua Parameter Screening:

| Key | Default | Fungsi | Risiko jika terlalu rendah |
|-----|---------|--------|---------------------------|
| `minFeeActiveTvlRatio` | 0.05 | Minimum fee/TVL ratio | Deploy ke pool dengan fee rendah |
| `minTvl` | 10000 | Minimum TVL ($) | Pool terlalu kecil, slippage tinggi |
| `maxTvl` | 150000 | Maximum TVL ($) | — |
| `minVolume` | 500 | Minimum volume ($) | Pool mati, tidak ada fee |
| `minOrganic` | 60 | Minimum organic score (0-100) | Bot/wash trading lolos |
| `minHolders` | 500 | Minimum holder count | Token terlalu baru/berisiko |
| `minMcap` | 150000 | Minimum market cap ($) | Micro-cap sangat volatile |
| `maxMcap` | 10000000 | Maximum market cap ($) | — |
| `minBinStep` | 80 | Minimum bin step | Fee per swap terlalu kecil |
| `maxBinStep` | 125 | Maximum bin step | Spread terlalu lebar |
| `minTokenFeesSol` | 30 | Minimum total fee token (SOL) | Token belum proven |
| `maxBundlePct` | 30 | Max bundler % | Insider manipulation |
| `maxBotHoldersPct` | 30 | Max bot holder % | Fake holders |
| `maxTop10Pct` | 60 | Max top 10 holder % | Whale dump risk |
| `blockPvpSymbols` | false | Block PVP symbol conflicts | — |
| `athFilterPct` | null | Filter by distance from ATH | — |
| `minTokenAgeHours` | null | Minimum token age | — |
| `maxTokenAgeHours` | null | Maximum token age | — |


---

## Preset Agresivitas

### Degen (High Risk, High Reward)

```json
{
  "minFeeActiveTvlRatio": 0.03,
  "minTvl": 5000,
  "maxTvl": 500000,
  "minOrganic": 40,
  "minHolders": 200,
  "minMcap": 50000,
  "maxMcap": 50000000,
  "minTokenFeesSol": 10,
  "maxTop10Pct": 80,
  "maxBotHoldersPct": 50,
  "stopLossPct": -50,
  "takeProfitPct": 30,
  "maxPositions": 5,
  "screeningIntervalMin": 15
}
```

### Moderate (Balanced)

```json
{
  "minFeeActiveTvlRatio": 0.05,
  "minTvl": 10000,
  "maxTvl": 150000,
  "minOrganic": 60,
  "minHolders": 500,
  "minMcap": 150000,
  "maxMcap": 10000000,
  "minTokenFeesSol": 30,
  "maxTop10Pct": 60,
  "maxBotHoldersPct": 30,
  "stopLossPct": -30,
  "takeProfitPct": 15,
  "maxPositions": 3,
  "screeningIntervalMin": 30
}
```

### Safe (Conservative)

```json
{
  "minFeeActiveTvlRatio": 0.10,
  "minTvl": 20000,
  "maxTvl": 100000,
  "minOrganic": 80,
  "minHolders": 1000,
  "minMcap": 500000,
  "maxMcap": 5000000,
  "minTokenFeesSol": 50,
  "maxTop10Pct": 40,
  "maxBotHoldersPct": 15,
  "stopLossPct": -20,
  "takeProfitPct": 10,
  "maxPositions": 2,
  "screeningIntervalMin": 60
}
```

---

## Konfigurasi Management & Exit Rules

| Key | Default | Fungsi |
|-----|---------|--------|
| `stopLossPct` | -50 | Cut loss otomatis jika PnL turun ke % ini |
| `takeProfitPct` | 15 | Take profit otomatis jika PnL naik ke % ini |
| `trailingTakeProfit` | true | Aktifkan trailing TP (lock profit saat turun dari peak) |
| `trailingTriggerPct` | 5 | PnL % yang mengaktifkan trailing mode |
| `trailingDropPct` | 3 | Drop dari peak yang trigger close |
| `outOfRangeWaitMinutes` | 30 | Menit tunggu sebelum close posisi OOR |
| `outOfRangeBinsToClose` | 10 | Bins di atas range yang trigger close langsung |
| `minFeePerTvl24h` | 0.01 | Minimum fee/TVL 24h (di bawah ini = low yield close) |
| `deployAmountSol` | 0.5 | Floor deploy amount (SOL) |
| `maxDeployAmount` | 50 | Ceiling deploy amount (SOL) |
| `positionSizePct` | 0.35 | % wallet yang di-deploy per posisi |
| `gasReserve` | 0.2 | SOL yang disisakan untuk gas fees |
| `maxPositions` | 3 | Maksimum posisi terbuka bersamaan |
| `minSolToOpen` | 0.55 | Minimum SOL di wallet untuk buka posisi baru |

### Formula Deploy Amount:

```
deployable = walletSol - gasReserve
amount = clamp(deployable * positionSizePct, floor=deployAmountSol, ceil=maxDeployAmount)
```

Contoh: Wallet 5 SOL, gasReserve 0.2, positionSizePct 0.35
- deployable = 5 - 0.2 = 4.8
- amount = 4.8 * 0.35 = 1.68 SOL per posisi

---

## Konfigurasi Strategy

| Key | Default | Fungsi |
|-----|---------|--------|
| `strategy` | "bid_ask" | Shape distribusi liquidity (spot/bid_ask) |
| `minBinsBelow` | 35 | Minimum bins di bawah active bin |
| `maxBinsBelow` | 69 | Maximum bins di bawah active bin |
| `defaultBinsBelow` | null | Override manual (bypass formula volatility) |

### Formula bins_below (otomatis dari volatility):

```
bins_below = round(minBinsBelow + (volatility / 5) * (maxBinsBelow - minBinsBelow))
clamped to [minBinsBelow, maxBinsBelow]
```

Contoh:
- Volatility 1.0 → bins_below = 35 + (1/5) * 34 = 42 bins
- Volatility 3.0 → bins_below = 35 + (3/5) * 34 = 55 bins
- Volatility 5.0+ → bins_below = 69 bins (maximum)

Semakin volatile → range semakin lebar → lebih aman tapi fee kurang terkonsentrasi.

---

## Konfigurasi Schedule

| Key | Default | Fungsi |
|-----|---------|--------|
| `managementIntervalMin` | 5-10 | Interval cek posisi (menit) |
| `screeningIntervalMin` | 30 | Interval cari pool baru (menit) |

Tips:
- Management 5 menit = responsif tapi lebih banyak API calls
- Management 15 menit = hemat tapi lambat react ke dump
- Screening 15 menit = agresif, lebih sering deploy
- Screening 60 menit = konservatif, jarang deploy

---

## Konfigurasi Darwinian Signal Weights

| Key | Default | Fungsi |
|-----|---------|--------|
| `darwinEnabled` | true | Aktifkan sistem belajar otomatis |
| `darwinWindowDays` | 60 | Rolling window data (hari) |
| `darwinRecalcEvery` | 5 | Recalculate setiap N posisi di-close |
| `darwinBoost` | 1.05 | Multiplier untuk sinyal top quartile |
| `darwinDecay` | 0.95 | Multiplier untuk sinyal bottom quartile |
| `darwinFloor` | 0.3 | Minimum weight (tidak bisa di bawah ini) |
| `darwinCeiling` | 2.5 | Maximum weight (tidak bisa di atas ini) |
| `darwinMinSamples` | 10 | Minimum data sebelum recalc pertama |

Sistem ini otomatis belajar sinyal mana yang predict profit. Tidak perlu di-tune manual kecuali mau percepat/perlambat adaptasi.

---

## Konfigurasi Chart Indicators

```json
{
  "chartIndicators": {
    "enabled": true,
    "entryPreset": "supertrend_break",
    "exitPreset": "supertrend_break",
    "intervals": ["5_MINUTE", "15_MINUTE"],
    "requireAllIntervals": false,
    "rsiLength": 14,
    "rsiOversold": 30,
    "rsiOverbought": 80
  }
}
```

### Entry Presets:

| Preset | Kondisi Entry |
|--------|--------------|
| `supertrend_break` | Supertrend flip bullish ATAU harga di atas bullish ST |
| `rsi_reversal` | RSI <= oversold (30) |
| `rsi_plus_supertrend` | RSI oversold DAN supertrend bullish |
| `supertrend_or_rsi` | Supertrend bullish ATAU RSI oversold |
| `bollinger_reversion` | Harga <= lower Bollinger band |
| `bb_plus_rsi` | Harga <= lower band DAN RSI oversold |
| `fibo_reclaim` | Harga cross UP melalui fib 0.618/0.5/0.786 |
| `fibo_reject` | Harga cross DOWN melalui fib 0.618/0.5 |

Jika `enabled: false`, chart indicators di-skip dan semua candidate lolos.

---

## Risiko Utama dan Mitigasi

| Risiko | Severity | Mitigasi Built-in |
|--------|----------|-------------------|
| **Impermanent Loss** | High | Stop loss otomatis, OOR timeout, wide range (35-69 bins), single-side SOL deploy |
| **Rug Pull / Scam** | Critical | Token blacklist, dev blocklist, OKX risk analysis, bundler detection, organic score, holder count, smart wallet confirmation |
| **Low Yield** | Medium | Auto-close setelah 60 menit jika fee terlalu rendah, pool cooldown 4 jam |
| **LLM Bad Decision** | Medium | 11 safety checks sebelum deploy, hard filters sebelum LLM lihat candidates, deterministic close rules tanpa LLM |
| **Double Deploy** | Medium | Busy flags, fresh position count, duplicate pool/token guard, once-per-session lock |
| **RPC Failure** | Medium | Retry logic, state sync dengan on-chain, verification loop setelah close |
| **Bin Array Cost** | Low | Pre-deploy assertion, tolak jika range butuh init (non-refundable ~0.07 SOL/array) |

### Cara Kurangi Risiko:

1. **Mulai dengan `DRY_RUN=true`** — test semua logic tanpa transaksi
2. **Gunakan wallet terpisah** — jangan wallet utama
3. **Set `stopLossPct` ketat** — misal -20% bukan -50%
4. **Set `maxPositions` rendah** — mulai dari 1-2
5. **Set `deployAmountSol` kecil** — mulai 0.3-0.5 SOL
6. **Naikkan `minTokenFeesSol`** — hanya token yang sudah proven
7. **Aktifkan `blockPvpSymbols`** — hindari symbol conflict
8. **Monitor via Telegram** — react cepat jika ada masalah

---

## Tips Konfigurasi

- **Baru mulai?** Gunakan preset Safe, DRY_RUN dulu 1-2 hari
- **Sudah yakin?** Mulai dengan 0.3 SOL per posisi, max 2 posisi
- **Sudah profit?** Naikkan positionSizePct dan maxPositions bertahap
- **Sering kena stop loss?** Lebarkan range (naikkan minBinsBelow) atau ketatkan screening
- **Jarang deploy?** Turunkan threshold (minOrganic, minHolders, minTokenFeesSol)
- **Sering OOR?** Naikkan outOfRangeWaitMinutes atau lebarkan range
- **Fee rendah?** Naikkan minFeeActiveTvlRatio atau turunkan management interval

### Jangan Lakukan:

- Jangan set `gasReserve` di bawah 0.15 (bisa gagal close)
- Jangan set `minBinsBelow` di bawah 35 (terlalu sempit, IL tinggi)
- Jangan set `maxPositions` terlalu tinggi tanpa SOL cukup
- Jangan matikan `trailingTakeProfit` tanpa alasan (kehilangan profit lock)
- Jangan set `stopLossPct` terlalu ketat (misal -5%) — normal volatility bisa trigger
