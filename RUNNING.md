# Meridian — Running Guide

## Cara Menjalankan

### Mode Development (DRY_RUN — tanpa transaksi on-chain)

```bash
npm run dev
```

Atau manual:

```bash
DRY_RUN=true node index.js
```

> Semua logic berjalan normal (screening, management, LLM calls) tapi TIDAK ada transaksi Solana yang dikirim. Cocok untuk testing config dan memastikan semuanya bekerja.

---

### Mode Production

```bash
npm start
```

Atau:

```bash
node index.js
```

---

### Mode Production dengan PM2 (Recommended)

```bash
# Install PM2 global (sekali saja)
npm install -g pm2

# Start
npm run pm2:start

# Restart (setelah ubah config/env)
npm run pm2:restart

# Lihat logs
npm run pm2:logs

# Stop
pm2 stop meridian

# Delete
pm2 delete meridian
```

PM2 akan auto-restart jika crash (max 10x, delay 5 detik).

---

### Menggunakan Custom LLM Proxy (Ultramen)

Jalankan proxy dulu di terminal terpisah:

```bash
node ultramen-proxy.mjs
```

Lalu di terminal lain, jalankan Meridian:

```bash
node index.js
```

Pastikan `.env` sudah di-set:

```env
LLM_BASE_URL=http://localhost:4141/v1
LLM_API_KEY=ulm-your_key_here
LLM_MODEL=claude-opus-4.6
```

> **Port 4141** = OpenAI-compatible (untuk Meridian)
> **Port 4142** = Anthropic Messages API (untuk Claude Code, BUKAN untuk Meridian)

---

## CLI Commands

Meridian juga punya CLI untuk operasi manual tanpa masuk REPL:

```bash
# Cek balance wallet
node cli.js balance

# Lihat posisi terbuka
node cli.js positions

# Lihat PnL
node cli.js pnl

# Lihat candidates
node cli.js candidates

# Info token
node cli.js token-info <query>

# Detail pool
node cli.js pool-detail <pool_address>

# Deploy manual
node cli.js deploy <pool_address> --amount 0.5

# Close posisi
node cli.js close <position_address>

# Claim fees
node cli.js claim <position_address>

# Swap token
node cli.js swap <input_mint> <output_mint> <amount>

# Jalankan screening sekali
node cli.js screen

# Jalankan management sekali
node cli.js manage

# Lihat/ubah config
node cli.js config get
node cli.js config set deployAmountSol 0.8

# Study top LPers
node cli.js study <pool_address>

# Lihat lessons
node cli.js lessons

# Lihat pool memory
node cli.js pool-memory <pool_address>

# Evolve thresholds manual
node cli.js evolve

# Blacklist token
node cli.js blacklist add <mint> --reason "scam"
node cli.js blacklist list

# Performance history
node cli.js performance
```

Flag global: `--dry-run`, `--silent`

---

## REPL Mode (Terminal Interaktif)

Saat dijalankan di terminal (TTY), Meridian masuk mode REPL:

```
┌─────────────────────────────────────┐
│  MERIDIAN DLMM LP Agent             │
│  [manage: 3m 45s | screen: 28m 12s] │
└─────────────────────────────────────┘
```

### REPL Commands:

| Input | Aksi |
|-------|------|
| `1`, `2`, `3`... | Deploy ke candidate nomor tersebut |
| `auto` | LLM pilih candidate terbaik dan deploy |
| `go` | Start cron (sudah auto-start) |
| `/status` | Refresh wallet + posisi |
| `/candidates` | Refresh daftar pool |
| `/briefing` | Tampilkan morning briefing |
| `/learn [addr]` | Study top LPers |
| `/thresholds` | Tampilkan screening thresholds |
| `/evolve` | Trigger threshold evolution manual |
| `/stop` | Shutdown graceful |
| Teks bebas | Kirim ke LLM sebagai chat (bisa tanya apa saja) |

---

## Telegram Control

Jika Telegram dikonfigurasi, kamu bisa kontrol dari HP:

```
/status      — Snapshot wallet + posisi
/positions   — List posisi dengan PnL
/close 1     — Close posisi #1
/screen      — Jalankan screening
/deploy 1    — Deploy ke candidate #1
/settings    — Menu button interaktif
/pause       — Stop semua cron
/resume      — Restart cron
/stop        — Shutdown agent
```

Pesan bebas juga bisa dikirim — akan diproses oleh LLM.

---

## Urutan Startup yang Benar

```
1. Pastikan .env dan user-config.json sudah benar
2. (Opsional) Jalankan proxy: node ultramen-proxy.mjs
3. Test dulu: DRY_RUN=true node index.js
4. Jika semua OK, jalankan production: node index.js
5. Monitor via Telegram atau terminal
```

---

## Logs

Log tersimpan di folder `./logs/`:

| File | Isi |
|------|-----|
| `agent-YYYY-MM-DD.log` | Log umum (info, warn, error) |
| `actions-YYYY-MM-DD.jsonl` | Audit trail setiap tool execution |

---

## Troubleshooting

| Masalah | Solusi |
|---------|--------|
| "Cannot find module @meteora-ag/dlmm" | Jalankan `npm install` ulang |
| Empty LLM response | Cek API key, pastikan model support function calling |
| "Insufficient SOL" | Isi wallet, atau turunkan `deployAmountSol` |
| Telegram tidak respon | Cek `TELEGRAM_BOT_TOKEN` dan `TELEGRAM_CHAT_ID` |
| Rate limited RPC | Ganti ke RPC berbayar (Helius/Quicknode) |
| Proxy connection refused | Pastikan `ultramen-proxy.mjs` sudah running di port 4141 |
| Position not closing | Cek RPC health, coba `node cli.js close <address>` manual |
