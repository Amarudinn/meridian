# Meridian — Setup Guide

## Prasyarat

| Kebutuhan | Keterangan |
|-----------|------------|
| Node.js | >= 18.0.0 |
| npm | Bawaan Node.js |
| Solana Wallet | Private key (base58 atau JSON array) |
| RPC URL | Solana mainnet RPC (Helius/Quicknode/Alchemy recommended) |
| LLM API Key | OpenRouter, atau proxy OpenAI-compatible lainnya |
| Telegram Bot (opsional) | Untuk kontrol remote & notifikasi |
| Helius API Key (opsional) | Untuk data balance yang lebih akurat |

---

## Instalasi

```bash
cd C:\Users\User\OneDrive\Desktop\all-folder\meridian
npm install
```

> `postinstall` script akan otomatis jalankan `scripts/patch-anchor.js` untuk patch kompatibilitas.

---

## Setup Wizard (Recommended)

```bash
npm run setup
```

Wizard interaktif akan memandu kamu mengisi:
1. API keys (OpenRouter / custom LLM)
2. Wallet private key
3. RPC URL
4. Telegram bot token & chat ID
5. Risk preset (Degen / Moderate / Safe / Custom)
6. Deploy parameters
7. Exit rules
8. Scheduling intervals

Hasil wizard tersimpan di `.env` dan `user-config.json`.

---

## Setup Manual (Tanpa Wizard)

### 1. Buat file `.env`

```env
# === WAJIB ===
WALLET_PRIVATE_KEY=your_base58_private_key_here
RPC_URL=https://your-rpc-endpoint.com
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxx

# === OPSIONAL ===
HELIUS_API_KEY=your_helius_key
TELEGRAM_BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
TELEGRAM_CHAT_ID=your_chat_id
DRY_RUN=false

# === CUSTOM LLM (jika tidak pakai OpenRouter) ===
LLM_BASE_URL=http://localhost:4141/v1
LLM_API_KEY=your_api_key
LLM_MODEL=claude-opus-4.6

# === HIVEMIND (opsional) ===
HIVE_MIND_URL=https://api.agentmeridian.xyz
HIVE_MIND_API_KEY=your_hive_key

# === OKX DEX API (opsional, untuk risk analysis) ===
OKX_API_KEY=your_okx_key
OKX_SECRET_KEY=your_okx_secret
OKX_PASSPHRASE=your_passphrase
OKX_PROJECT_ID=your_project_id

# === SAFETY ===
ALLOW_SELF_UPDATE=false
```

### 2. Buat file `user-config.json` (minimal)

```json
{
  "deployAmountSol": 0.5,
  "maxPositions": 3,
  "stopLossPct": -30,
  "takeProfitPct": 15,
  "managementIntervalMin": 5,
  "screeningIntervalMin": 30
}
```

---

## Bagian yang WAJIB Diganti

| File | Key | Yang Harus Diisi |
|------|-----|-----------------|
| `.env` | `WALLET_PRIVATE_KEY` | Private key wallet Solana kamu (JANGAN share) |
| `.env` | `RPC_URL` | RPC endpoint mainnet (jangan pakai public default — rate limited) |
| `.env` | `OPENROUTER_API_KEY` atau `LLM_API_KEY` | API key untuk LLM provider |
| `.env` | `TELEGRAM_BOT_TOKEN` | Token dari @BotFather (jika mau kontrol via Telegram) |
| `.env` | `TELEGRAM_CHAT_ID` | Chat ID kamu (kirim /start ke bot, cek via API) |

---

## Bagian yang Perlu Disesuaikan

| File | Key | Pertimbangan |
|------|-----|-------------|
| `user-config.json` | `deployAmountSol` | Berapa SOL per posisi (sesuaikan dengan balance) |
| `user-config.json` | `maxPositions` | Berapa posisi maks terbuka sekaligus |
| `user-config.json` | `stopLossPct` | Kapan cut loss (-20% sampai -50%) |
| `user-config.json` | `takeProfitPct` | Target profit (5% sampai 50%) |
| `user-config.json` | `positionSizePct` | % wallet per deploy (default 0.35 = 35%) |
| `user-config.json` | `gasReserve` | SOL yang disisakan untuk gas (min 0.2) |

---

## Menggunakan Custom LLM (Ultramen Proxy)

Jika kamu punya proxy OpenAI-compatible sendiri:

```env
LLM_BASE_URL=http://localhost:4141/v1
LLM_API_KEY=ulm-your_key_here
LLM_MODEL=claude-opus-4.6
```

Tambahkan juga di `user-config.json`:

```json
{
  "managementModel": "claude-opus-4.6",
  "screeningModel": "claude-opus-4.6",
  "generalModel": "claude-opus-4.6"
}
```

> **Syarat:** Proxy harus support OpenAI Chat Completions format + Function Calling (tool use).

---

## Mendapatkan Telegram Chat ID

1. Buat bot via @BotFather → dapat token
2. Kirim pesan apapun ke bot kamu
3. Buka: `https://api.telegram.org/bot<TOKEN>/getUpdates`
4. Cari `"chat":{"id": 123456789}` — itu chat ID kamu

---

## Keamanan

⚠️ **JANGAN PERNAH:**
- Commit `.env` ke git
- Share private key
- Pakai wallet utama (buat wallet khusus untuk LP)
- Jalankan tanpa `DRY_RUN=true` sebelum yakin config benar

✅ **SEBAIKNYA:**
- Encrypt `.env` dengan `npm run env:encrypt`
- Gunakan wallet terpisah dengan SOL secukupnya
- Test dulu dengan `DRY_RUN=true`
- Set `gasReserve` minimal 0.2 SOL
- Monitor via Telegram
