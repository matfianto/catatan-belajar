# Catatan Belajar: Ekosistem AI (Klien, Agen, Gateway, Penyedia Model)

> Disusun dari percakapan pada **25 Oktober 2026**.
> Harga, kuota gratis, dan fitur layanan AI **cepat berubah**. Angka dalam catatan ini adalah hasil pencarian pada tanggal tersebut. Cek ulang sebelum mengandalkannya. Bagian yang belum terverifikasi ditandai **(belum diverifikasi)** dan dikumpulkan di Bagian 14.
> Jangan menyimpan API key di dalam catatan ini.
> **Pembaruan:** ditambahkan Bagian 10.7 sampai 10.9 tentang Claude Desktop (klien + agen dalam satu aplikasi), beserta skill, connector/MCP, dan extensions. Juga Bagian 9.6 sampai 9.10: pindah model otomatis di VS Code (Kilo Code), stack vibe coding yang populer, Google Antigravity, dan Google Jules.
> **Pembaruan terbaru:** Bagian 3.6 dan 3.7 (API Gateway dan Code Mate di Cherry Studio), 4.2 (**diluruskan**: pencarian web Cherry Studio), 4.4 (penyedia pengambil URL), 9.11 (vibe coding di VS Code lewat Cherry Studio), dan 11.3 (susunan rantai yang diluruskan).
> **Pembaruan ketiga:** Bagian 2.9 sampai 2.12 (Claude Console, model lokal, Unsloth Studio, Colab dan Kaggle), 3.8 dan 3.9 (Knowledge Base dan Channels di Cherry Studio; **koreksi**: Cherry Studio punya fitur Channels untuk Telegram), 5.6 (memory layer/Obsidian), 7.4 sampai 7.6 (Telegram diluruskan, Claude API di bot, batasan WhatsApp), 9.12 sampai 9.18 (Codex, GitHub Copilot, OpenCode, pesaing, herdr, orkestrasi agen), 10.10 dan 10.11 (keluarga Copilot, skill di claude.ai vs plugin lokal), dan 11.4 (matriks produk vs lapisan).
> **Pembaruan keempat:** Bagian 2.13 (waspada layanan API murah: model masking dan kecurangan reseller) dan 6.7 (freellmapi: router gratis lintas penyedia, perbandingan dengan OpenRouter, dan cara memakainya), plus baris baru di matriks 11.4, keamanan, glosarium, daftar belum terverifikasi, tanya-jawab, dan sumber.

---

## Daftar Isi

1. [Tujuan dan Gambaran Besar](#1-tujuan-dan-gambaran-besar)
2. [Fondasi: Penyedia Model (Lapisan 4)](#2-fondasi-penyedia-model-lapisan-4)
3. [Klien Chat (Lapisan 1)](#3-klien-chat-lapisan-1)
4. [Pencarian Web untuk AI (pelengkap klien chat)](#4-pencarian-web-untuk-ai-pelengkap-klien-chat)
5. [Agen AI (Lapisan 2)](#5-agen-ai-lapisan-2)
6. [Gateway dan Fallback Otomatis (Lapisan 3)](#6-gateway-dan-fallback-otomatis-lapisan-3)
7. [Bot Telegram Multi-Model](#7-bot-telegram-multi-model)
8. [Pengaturan Model: Reasoning Effort vs Service Tier](#8-pengaturan-model-reasoning-effort-vs-service-tier)
9. [AI untuk Coding dan Vibe Coding](#9-ai-untuk-coding-dan-vibe-coding)
10. [Agen Terkelola dan Agen di Aplikasi Desktop (Muse dan Claude Desktop)](#10-agen-terkelola-dan-agen-di-aplikasi-desktop-muse-dan-claude-desktop)
11. [Susunan Akhir dan Rekomendasi Langkah](#11-susunan-akhir-dan-rekomendasi-langkah)
12. [Keamanan dan Privasi (Ringkasan)](#12-keamanan-dan-privasi-ringkasan)
13. [Glosarium](#13-glosarium)
14. [Hal yang Belum Terverifikasi](#14-hal-yang-belum-terverifikasi)
15. [Tanya-Jawab Cepat](#15-tanya-jawab-cepat)
16. [Sumber Rujukan](#16-sumber-rujukan)

---

## 1. Tujuan dan Gambaran Besar

**Tujuan awal:** memakai semua akses AI yang dimiliki (Gemini plus model `deepseek-v4-flash`, `glm-5.3-flash`, `qwen3.8-flash`, `qwen3.8-max` lewat gateway geraikita) dalam **satu tempat**, dengan model yang bisa dipilih sesuai kebutuhan. Dengan begitu tidak perlu lagi berpindah antara gemini.google.com dan situs chat AI lain.

### 1.1 Empat lapisan

Sistem AI yang dirakit sendiri bisa dipahami sebagai empat lapisan:

```
LAPISAN 1  Klien chat       aplikasi yang kamu pakai       Cherry Studio, Chatbox, RikkaHub
LAPISAN 2  Agen             program yang mengerjakan tugas Hermes Agent, OpenClaw
LAPISAN 3  Gateway/router   pengatur lalu lintas + fallback 9Router, LiteLLM
LAPISAN 4  Penyedia model   yang menjalankan model         Gemini, OpenRouter, geraikita
```

Alur lengkap bila semuanya dipakai:

```
Cherry Studio (klien chat) ─┐
                            ├→ 9Router (gateway, opsional) → Gemini / OpenRouter / geraikita
Hermes Agent (agen) ────────┘                                 (penyedia API)
 + antarmuka sendiri (terminal / Telegram)
```

### 1.2 Label produk yang tepat

| Lapisan | Contoh | Label yang tepat |
|---|---|---|
| Aplikasi yang kamu pakai | Cherry Studio | **Klien chat / klien LLM** (BYOK) |
| Pengerja tugas | Hermes Agent | **AI agent** (program agen, bukan model) |
| Pengatur lalu lintas | 9Router | **Gateway / router AI** (proxy lokal) |
| Penyedia model | Gemini, OpenRouter, geraikita | **Penyedia API** (beberapa jenis, lihat 2.1) |
| Klien + agen dalam satu aplikasi | Claude Desktop | **Aplikasi klien dan agen sekaligus** (tab Chat = klien; tab Cowork dan Code = agen), lihat 10.7 |
| IDE berbasis agen | Google Antigravity, Cursor, Windsurf | **IDE + agen coding** (lapisan 1 dan 2 sekaligus), lihat 9.8 |
| Agen coding asinkron | Google Jules | **Agen coding di cloud** yang menghasilkan pull request, lihat 9.9 |

Catatan istilah:
- **"Workstation AI"** bukan istilah baku. Biasanya disebut klien chat AI, LLM client, atau BYOK client.
- **Fallback** adalah fitur, bukan jenis produk. 9Router disebut gateway, fallback adalah salah satu fungsinya.
- Batas klien dan agen tidak tegas: Cherry Studio punya mode agen otonom, tetapi pada dasarnya klien chat.
- **OpenRouter** (layanan cloud, agregator model) dan **9Router** (program lokal yang kamu jalankan sendiri) sama-sama mengandung kata "router" tetapi berbeda jenis.

### 1.3 Urutan baca dan susunan bab

Penomoran lapisan (1 sampai 4) mengikuti diagram dari sisi pengguna ke sisi model, tetapi urutan **bab** sengaja tidak sama dengan nomor lapisan:

- **Bab 2 lebih dulu (fondasi):** semua lapisan lain memakai tiga hal yang dijelaskan di sana, yaitu base URL, API key, dan nama model. Karena itu penyedia model (lapisan 4) dibaca pertama.
- **Bab 3 sampai 6:** klien chat (lapisan 1), pencarian web sebagai pelengkap klien chat (Bab 4), agen (lapisan 2), lalu gateway (lapisan 3).
- **Bab 7 sampai 10:** penerapan dan bahasan lanjutan (bot Telegram, pengaturan model, coding, agen terkelola).
- **Bab 11 sampai 16:** rangkuman susunan, keamanan, glosarium, daftar hal yang belum terverifikasi, tanya-jawab, dan sumber.

Bila ingin membaca berurutan mengikuti diagram: **Bab 3 → 5 → 6 → 2**.

---

## 2. Fondasi: Penyedia Model (Lapisan 4)

### 2.1 Konsep dasar

Untuk memakai model lewat aplikasi apa pun, yang dibutuhkan hanya tiga hal:

1. **Base URL / alamat API**, misalnya `https://api.groq.com/openai/v1`.
2. **API key** (rahasia, seperti kata sandi).
3. **Nama model**, misalnya `deepseek-v4-flash`.

Format yang paling umum disebut **OpenAI-compatible**: aplikasi mengirim permintaan ke `{base_url}/chat/completions`. Banyak penyedia mengikuti format ini sehingga cara menambahkannya ke aplikasi apa pun sama.

Jenis "penyedia" sebenarnya ada tiga:

| Jenis | Arti | Contoh |
|---|---|---|
| Pembuat model | Membuat dan menjalankan modelnya sendiri | Google (Gemini), Anthropic (Claude), DeepSeek, Zhipu (GLM), Alibaba (Qwen) |
| Agregator / marketplace | Tidak membuat model, menyalurkan banyak model lewat satu key | OpenRouter, Hugging Face Inference Providers, Eden AI |
| Reseller / gateway pihak ketiga | Menyediakan akses ke model pihak lain lewat satu API | geraikita (disimpulkan dari nama modelnya) |

Untuk keperluan memasang, ketiganya diperlakukan sama. Perbedaannya penting untuk soal data, kuota, dan tanggung jawab.

### 2.2 Google Gemini

- API key gratis bisa dibuat di **Google AI Studio** (aistudio.google.com), tanpa kartu. Chat gemini.google.com dan API key adalah dua hal berbeda.
- Gemini punya endpoint kompatibel OpenAI: `https://generativelanguage.googleapis.com/v1beta/openai` (path chat: `/chat/completions`). Bisa juga dipakai lewat tipe penyedia Google/Gemini bawaan di klien.
- Cek daftar model yang tersedia untuk key-mu (nama model berubah-ubah):
  ```bash
  curl -H "Authorization: Bearer KEY_GEMINI" https://generativelanguage.googleapis.com/v1beta/openai/models
  ```
- **Batas pemakaian paket gratis** ketat dan tidak lagi dipublikasikan per model di dokumentasi. Angkanya terlihat di konsol batas AI Studio proyekmu. Error "429" atau "quota exceeded" berarti batas menit atau hari terlewati.
- **Tidak semua model gratis.** Menurut satu sumber, sebagian besar model Flash/Flash-Lite dan Gemini 2.5 Pro gratis, sedangkan model gambar/video tertentu tidak.
- **Privasi:** pada layanan tanpa bayar, isi prompt dan jawaban dapat dipakai Google untuk meningkatkan produknya dan dapat dibaca peninjau manusia. Jangan kirim data pribadi/rahasia. Menurut satu sumber, menghubungkan penagihan dan mengisi saldo minimal $5 (naik ke Tier 1) menghentikan pemakaian data itu.

### 2.3 Claude (Anthropic) API

- Akun gratis **claude.ai tidak menghasilkan API key**. Chat Claude dan API adalah produk terpisah.
- API key (biasanya diawali `sk-ant-`) dibuat di **console.anthropic.com**. API Claude **tidak punya paket gratis permanen**. Yang ada paling banyak kredit awal kecil sekali pakai; setelah habis, key berhenti sampai kamu mengisi saldo.
- Cherry Studio punya penyedia **Anthropic** bawaan. Contoh nama model: `claude-haiku-5-5` (paling hemat), `claude-sonnet-5-5`, `claude-opus-5-5`. Nama model bisa berubah, cek dokumentasi resmi.
- Kalau "key Claude gratis" berasal dari penyedia lain (reseller/gateway), jangan pakai penyedia "Anthropic" di klien. Tambahkan sebagai model di penyedia OpenAI-compatible dengan base URL dan key dari penyedia itu.
- **Waspada key "gratis" dari situs/grup tidak resmi:** bisa dipakai membaca isi chat atau tiba-tiba mati. Jangan dipakai untuk data penting.
- Dokumentasi resmi: https://docs.claude.com/en/api/overview

### 2.4 OpenRouter

- Layanan cloud: **satu API key untuk banyak model**, termasuk model gratis berakhiran `:free`.
- Base URL: `https://openrouter.ai/api/v1`.
- Contoh batas gratis (dari satu daftar, bisa berubah): sekitar 50 permintaan per hari, naik ke 1.000 per hari bila top up $10. Biasanya berlaku **per akun**, bukan per model.
- **Fallback bawaan:** parameter `models` berisi daftar model berurutan. Bila model utama down, kena batas, atau menolak menjawab, OpenRouter mencoba model berikutnya. Daftar ini dikirim lewat isi permintaan (belum tentu bisa diisi dari klien seperti Cherry Studio) dan hanya mencakup model di dalam OpenRouter.

### 2.5 Hugging Face

HF punya beberapa fungsi:

1. **Gudang model/dataset:** tempat mengunduh model untuk dijalankan lokal (Ollama, LM Studio, dan lain-lain). Model besar butuh RAM/GPU kuat; HP di Termux umumnya tidak cocok.
2. **Inference Providers (penyedia API seperti OpenRouter):** satu token HF di `https://router.huggingface.co/v1` meneruskan permintaan ke penyedia seperti Groq, Together, Fireworks, Cerebras, Nebius. Penyedia bisa dikunci lewat akhiran, misalnya `meta-llama/Llama-3.3-70B-Instruct:groq`.
3. **Inference Endpoints:** server khusus berbayar per pelanggan.
4. **Spaces:** hosting aplikasi demo.

Catatan:
- Pengguna gratis hanya mendapat kuota bulanan **kecil** (satu forum menyebut sekitar $0,10, bukan sumber resmi). PRO mendapat kredit $2 per bulan. Kalau kredit habis, API mengembalikan `402 Payment Required`.
- Sumber berbeda pendapat soal markup harga: ada yang menyebut markup kecil, ada yang menyatakan tarif penyedia diteruskan tanpa tambahan. Cek halaman harga HF.
- Katalognya condong ke **model terbuka**, jadi sebagai cadangan gratis kurang berguna dibanding OpenRouter, Gemini, atau Groq.

### 2.6 Penyedia gratis lain

| Penyedia | Catatan | Base URL (bila diketahui) |
|---|---|---|
| Gemini (AI Studio) | Kuota gratis rate-limited | `https://generativelanguage.googleapis.com/v1beta/openai` |
| OpenRouter | Model `:free`, ~50 req/hari tanpa top up | `https://openrouter.ai/api/v1` |
| Groq | Sangat cepat | `https://api.groq.com/openai/v1` |
| Cerebras | Cepat, paket gratis cukup besar | cek dokumentasi |
| NVIDIA NIM | Model besar (mis. Qwen3 235B), sekitar 40 req/menit | `https://integrate.api.nvidia.com/v1` |
| Mistral | Sekitar 1 req/detik dan ~1 miliar token/bulan; menurut satu daftar, paket gratis mensyaratkan persetujuan data pelatihan dan verifikasi telepon | cek dokumentasi |
| Zhipu AI (GLM) | Varian Flash gratis, batas tidak terdokumentasi | `https://open.bigmodel.cn/api/paas/v4/` |
| GitHub Models | Sekitar 50 sampai 150 permintaan/hari | cek dokumentasi |
| Hugging Face | Kredit gratis kecil | `https://router.huggingface.co/v1` |

Hal yang perlu diwaspadai pada paket gratis:
- **Batas sering berubah.** Cek angka terbaru.
- **Model terbaik biasanya berbayar.** Sejak April 2026, banyak model unggulan dipindah ke paket berbayar.
- **Privasi.** Sebagian layanan memakai prompt untuk melatih model berikutnya. Jangan kirim data pribadi/rahasia lewat paket gratis.
- **Hindari "key gratis" tidak resmi.**

### 2.7 geraikita

- Gateway/reseller OpenAI-compatible yang menyediakan `deepseek-v4-flash`, `glm-5.3-flash`, `qwen3.8-flash`, `qwen3.8-max`.
- Pakai seperti penyedia OpenAI-compatible lain: base URL (bot lama memakai `https://ai.geraikita.com/v1`, samakan dengan yang kamu pakai), API key, nama model.

### 2.8 Perbandingan: Gemini vs Claude vs OpenRouter

| | Gemini (Google) | Claude (Anthropic) | OpenRouter |
|---|---|---|---|
| Kunci gratis dari | Google AI Studio | Hanya kredit awal kecil (bila ada) | Daftar akun |
| Paket gratis permanen | Ada (rate-limited) | Tidak ada | Model `:free` terbatas |
| Format API | Native + OpenAI-compatible | Native Anthropic | OpenAI-compatible |

### 2.9 Claude Console (panel developer Anthropic)

**Apa itu:** dasbor web untuk developer Anthropic di platform.claude.com (console.anthropic.com mengarah ke konsol yang sama; juga disebut Anthropic Console atau Claude Platform). Fungsinya: membuat API key, mengelola organisasi dan workspace beserta peran, penagihan dan kredit, batas pengeluaran, laporan pemakaian dan biaya, serta **Workbench** (tempat mencoba prompt di browser).

**Bukan penyedia model.** Console adalah panel pengelola untuk penyedia (lapisan 4). Penyedianya adalah Anthropic lewat Claude API di `api.anthropic.com`.

| Peran | Google | Anthropic |
|---|---|---|
| Klien chat (lapisan 1) | gemini.google.com | claude.ai |
| Dasbor developer + playground + pembuat key | Google AI Studio | **Claude Console** (Workbench) |
| Penyedia / API (lapisan 4) | Gemini API | Claude API |
| Paket gratis API | Ada (terbatas) | Tidak ada paket gratis permanen |

**Perbedaan biaya dengan Google AI Studio:**
- Wajib ada metode pembayaran atau kredit sebelum permintaan berhasil. AI Studio memberi key gratis tanpa kartu.
- Kredit prabayar bisa kedaluwarsa, dan pemakaian berhenti saat saldo habis.
- Langganan claude.ai dan pemakaian API di Console ditagih **terpisah**. Akun gratis claude.ai tidak menghasilkan API key.

**Diagram: Console, aplikasi yang memakai key, dan Claude API**

```
      ┌──────────────────────────────────────────────────────────────┐
      │            CLAUDE CONSOLE  (platform.claude.com)             │
      │   buat API key · isi saldo · batas pengeluaran · Workbench   │
      └──────────────────────────────┬───────────────────────────────┘
                                     │ API key disalin ke aplikasi:
         ┌─────────────┬─────────────┼─────────────┬─────────────┐
         ▼             ▼             ▼             ▼             ▼
   Cherry Studio Cline / Kilo   Claude Code  Bot Telegram     9Router
     provider      OpenCode     (auth API)     buatanmu       LiteLLM
     Anthropic    (Anthropic)                  (Python)      (gateway)
         └───────────────────────────┬───────────────────────────┘
                                     │ permintaan + key
                                     ▼
             ┌───────────────────────────────────────────────┐
             │        CLAUDE API  (api.anthropic.com)        │
             │             penyedia / lapisan 4              │
             └───────────────────────────────────────────────┘

   Terpisah dari semua di atas: claude.ai (klien chat), langganan sendiri,
   tanpa API key, ditagih terpisah.
```

**Cara membacanya:**
- Console **tidak menjalankan model dan tidak dilalui permintaanmu**. Ia hanya tempat membuat key, mengisi saldo, dan memasang batas. Key disalin ke aplikasi, lalu aplikasi itu **langsung** memanggil Claude API.
- Semua aplikasi yang memakai key yang sama masuk ke saldo yang sama, sehingga satu aplikasi boros bisa menghabiskan jatah yang lain.
- Cherry Studio, 9Router, dan LiteLLM bisa menjadi perantara untuk alat lain.

**Bila dipakai untuk agen (lapisan 2):**

```
LAPISAN 1  Antarmuka / pengelola   Terminal │ VS Code │ herdr │ Telegram
           │
LAPISAN 2  Agen                    Claude Code │ OpenCode │ Cline/Kilo │ agen buatanmu
           │  permintaan + API key
LAPISAN 3  Gateway (opsional)      9Router │ LiteLLM │ Cherry API Gateway
           │
LAPISAN 4  Penyedia                Claude API  (api.anthropic.com)
           ▲
           │ kelola key, saldo, batas pengeluaran (bukan jalur permintaan)
           ┌──────────────────────────────────────┐
           │ CLAUDE CONSOLE (platform.claude.com) │
           └──────────────────────────────────────┘
```

- Console adalah rute autentikasi API bawaan untuk Claude Code. Untuk OpenCode, Cline, dan Kilo Code, pilih penyedia **Anthropic** dan tempel key-nya. Pakai API native, bukan lapisan kompatibel OpenAI.
- Dokumentasi menyebut Claude API juga memberi akses ke Claude Managed Agents (detailnya belum diperiksa).
- **Agen boros token:** mulai dengan `claude-haiku-5-5`. **Pisahkan key per agen** dengan workspace dan batas pengeluaran terpisah, supaya kerugian terbatas bila agen lepas kendali atau key bocor.
- **Privasi:** prompt Workbench, key, metadata pemakaian, dan data organisasi dikelola di Console Anthropic. Hindari memasukkan data sensitif tanpa kebijakan yang jelas, dan jangan menempelkan key ke Slack, email, atau dokumen bersama.

### 2.10 Model lokal: menjalankan model di komputer sendiri

"Lapisan 4 versi lokal" berarti penyedianya adalah **komputermu sendiri**: tanpa API key dan tanpa kuota harian, tetapi kecepatan dan kualitasnya bergantung pada RAM dan GPU. Internet tetap dibutuhkan saat mengunduh model (dan saat memakai alat pencarian web).

| Alat | Menjalankan model | Melatih model | Catatan |
|---|---|---|---|
| **Unsloth Studio** | Ya | Ya | Antarmuka web lokal, Beta (lihat 2.11) |
| **LM Studio** | Ya | Tidak | Aplikasi desktop berantarmuka; menelusuri Hugging Face dan mengunduh model dengan satu klik; proprietary |
| **Ollama** | Ya | Tidak | Berbasis CLI dan REST API yang kompatibel OpenAI, ramah untuk dihubungkan ke alat lain; kini punya aplikasi desktop |
| **Jan** | Ya | Tidak | Desktop sumber terbuka, mengelola model di dalam aplikasi, punya integrasi MCP |
| **GPT4All** | Ya | Tidak | Mudah dipasang, pilihan model lebih sedikit |
| **Pinokio** | Ya | Tidak | Pemasang aplikasi dan model AI lokal dengan antarmuka grafis |
| **H2O LLM Studio** | Terbatas | Ya | GUI tanpa kode khusus untuk fine-tuning |

Transformer Lab dan LLaMA-Factory juga sering disebut untuk pelatihan (dari pengetahuan saya, belum dicek di pencarian).

**Cherry Studio tidak mengunduh dan menjalankan model sendiri.** Ia klien yang terhubung ke aplikasi lain: untuk model lokal kamu memasang Ollama atau LM Studio terpisah, lalu di Cherry Studio (Settings → Model Services) menambahkannya sebagai penyedia. Ollama harus tetap berjalan, dan modelnya diunduh dan dikelola di aplikasi itu.

```
Cherry Studio (lapisan 1: antarmuka)
        │  Settings → Model Services
        ▼
Ollama / LM Studio (lapisan 4 lokal: unduh + jalankan model)
        │
        ▼
 model di komputermu
```

**Catatan kualitas:** model lokal kecil sering lemah di tool calling dan konteks panjang. Untuk agen coding, hasilnya biasanya di bawah Gemini atau model besar, kecuali perangkatmu cukup kuat. Model lokal juga bisa dipakai alat lain lewat endpoint OpenAI-compatible (OpenCode, Cline, Cherry Studio, dan lain-lain).

### 2.11 Unsloth Studio

**Apa itu:** antarmuka web sumber terbuka tanpa kode untuk **melatih, menjalankan, dan mengekspor** model terbuka secara lokal. Status: **Beta**. Bukan klien BYOK ke penyedia cloud (fitur itu tidak ditemukan).

**Fitur:**
- **Menjalankan:** mencari, mengunduh, dan menjalankan model (GGUF, LoRA adapter, safetensors); chat dengan unggahan gambar, audio, PDF, kode, dan DOCX; tool calling, pencarian web, dan eksekusi kode.
- **Melatih:** fine-tuning banyak model, diklaim pengembang lebih cepat dengan VRAM jauh lebih sedikit; Data Recipes untuk membuat dataset dari PDF, CSV, DOCX, dan lain-lain.
- **Mengekspor:** ke GGUF, safetensors, dan format lain.
- **Endpoint API inferensi:** model lokal bisa dipakai di alat seperti Claude Code dan Codex, jadi Unsloth bisa menjadi **penyedia lokal untuk alat lain**.

**Platform dan cara memasang:**
- Berjalan di Windows, Linux, WSL, dan macOS. Pemasangan lewat skrip perintah (`curl ... | sh` atau `irm ... | iex`), lalu dijalankan dengan `unsloth studio -p 8888` dan **dibuka di browser**. Jadi ia server web lokal, bukan aplikasi desktop dengan penginstal seperti Cherry Studio. Ada juga image Docker.
- **Tidak ada versi Android** di dokumentasi yang dibaca; di Termux kemungkinan tidak praktis (belum diuji).
- Bawaan hanya lokal. `-H 0.0.0.0` membukanya ke LAN (HP di Wi-Fi yang sama bisa membukanya lewat browser), dan opsi `--cloudflare` memberi URL publik. Jangan buka ke internet tanpa pengamanan.
- **Perangkat keras:** pelatihan butuh GPU NVIDIA (RTX 30/40/50 dan seterusnya); CPU mendukung chat dan Data Recipes; dukungan AMD dan Intel berubah cepat, cek dokumentasi.
- **Keamanan:** alat sisi server (pencarian web, eksekusi kode Python dan terminal) berjalan sebagai penggunamu dan aktif secara bawaan. Biarkan lokal dan baca skrip pemasangan.
- **Tidak ada fitur bot Telegram atau WhatsApp** yang ditemukan. Jika ingin, pakai Hermes sebagai gerbang pesan dengan endpoint Unsloth sebagai penyedia model, atau bot buatan sendiri yang memanggil endpoint Unsloth. Path endpoint belum diverifikasi.

**Label dalam empat lapisan:** lapisan 1 (UI web) + lapisan 4 versi lokal (mesin model di komputermu) + alat agen terbatas di lapisan 2 (tool calling, pencarian, eksekusi kode; bukan agen coding penuh). Lapisan 3 tidak ada. Pelatihan model berada **di luar** empat lapisan, karena empat lapisan menggambarkan jalur permintaan saat *memakai* model, sedangkan pelatihan *membuat atau mengubah* modelnya.

```
CHERRY STUDIO (BYOK)
┌───────────────────┐                          ┌───────────────────────────┐
│ LAPISAN 1         │  internet + API key      │ LAPISAN 4                 │
│ Cherry Studio     │ ──────────────────────→  │ Gemini / geraikita / dst. │
│ (di PC-mu)        │ ←──────────────────────  │ (di server penyedia)      │
└───────────────────┘                          └───────────────────────────┘

UNSLOTH STUDIO (lokal)
┌──────────────────────────────────────────────────────────┐
│ KOMPUTERMU: model lokal, tanpa API key, tanpa kuota      │
│ ┌──────────────────────┐     ┌─────────────────────────┐ │
│ │ LAPISAN 1            │     │ LAPISAN 4 (lokal)       │ │
│ │ UI web di browser    │ ──→ │ model yang kamu unduh   │ │
│ │ (chat, unggah file)  │ ←── │ (mis. GGUF) + llama.cpp │ │
│ └──────────────────────┘     └─────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

```
┌───────────────────────────────┐                ┌────────────────────────────────────┐
│ BENGKEL (di luar 4 lapisan)   │                │ PEMAKAIAN (4 lapisan)              │
│ 1. unduh model dasar          │ ── model ────→ │ Lapisan 1: UI chat di browser      │
│ 2. latih / fine-tuning        │                │ Lapisan 2: tool calling, pencarian │
│ 3. ekspor GGUF / safetensors  │                │            web, eksekusi kode      │
└───────────────────────────────┘                │ Lapisan 3: tidak ada               │
                                                 │ Lapisan 4: mesin model lokal       │
                                                 └────────────────────────────────────┘
                                                                     │ endpoint API lokal
                                                                     ▼
                                              Cherry Studio / OpenCode / Claude Code / Codex
```

### 2.12 Google Colab dan Kaggle: bukan VPS

Menjalankan Unsloth Studio lewat notebook (Colab atau Kaggle) memberi mesin virtual **sementara** dengan GPU. Itu bukan VPS.

```
┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│ VPS (sewa server)               │   │ Google Colab (pinjam komputer)  │
├─────────────────────────────────┤   ├─────────────────────────────────┤
│ menyala terus (24/7)            │   │ hidup selama sesi saja          │
│ disk permanen                   │   │ disk ikut terhapus saat selesai │
│ IP tetap, SSH, akses root       │   │ tanpa IP tetap, SSH dibatasi    │
│ bebas memasang layanan sendiri  │   │ khusus notebook interaktif      │
│ bayar sewa bulanan              │   │ ada paket gratis (terbatas)     │
└─────────────────────────────────┘   └─────────────────────────────────┘
```

**Colab (paket gratis):**
- Sesi terputus sekitar 90 menit tanpa aktivitas dan ada batas keras sekitar 12 jam (angka pihak ketiga). Google menyatakan batas penggunaan, waktu idle, dan umur VM bervariasi dan tidak dipublikasikan.
- Tidak ada penyimpanan permanen; model yang diunduh hilang setelah sesi.
- GPU tidak dijamin (bisa dapat T4, bisa tidak sama sekali). T4 sekitar 15 GB VRAM.
- FAQ Colab menyebut contoh pembatasan seperti menjalankan antarmuka web alih-alih UI notebook dan pekerja komputasi terdistribusi; pengguna gratis sering mengalami penghentian runtime saat menjalankan UI web. Panduan lain menyebut SSH atau remote desktop bisa memicu peringatan, penghentian sesi, atau pembatasan akun. Paket berbayar mengurangi sebagian pembatasan.

**Kaggle Notebooks:**
- Dua GPU T4 (masing-masing 16 GB, total sekitar 32 GB) atau P100 16 GB. Satu sumber menyebut dual T4 mampu menjalankan model 70B, tetapi hanya dengan kuantisasi berat; anggap sebagai klaim.
- Kuota **30 jam GPU per minggu** dan batas sesi sekitar 9 sampai 12 jam (sumber berbeda). Kuotanya tetap dan terlihat, tidak sedinamis Colab.
- Akses internet di notebook butuh akun terverifikasi nomor telepon.
- Sesi interaktif yang aktif menghabiskan kuota meski tidak ada proses berjalan, dan punya batas tidak aktif sekitar 1 jam. **Hentikan sesi saat selesai.** Dua notebook GPU sekaligus menguras kuota dua kali lebih cepat.
- Simpan hasil ke `/kaggle/working`. Model yang diunduh umumnya harus diunduh ulang tiap sesi kecuali persistence diaktifkan.
- Aturan Kaggle soal menjalankan UI web lewat tunnel tidak ditemukan dengan jelas; jangan dipakai untuk layanan publik jangka panjang.

**Analisis skrip notebook Unsloth Studio (versi Colab yang dilampirkan):**
1. Memeriksa GPU (`nvidia-smi`), memasang Unsloth Studio lewat `curl | sh`, dan memastikan PyTorch CUDA terpasang.
2. Mengunduh `cloudflared`, menjalankan Studio di `127.0.0.1:9000` dengan **kata sandi acak** dan opsi `--disable-tools`.
3. Membuat tautan publik `trycloudflare.com` dan mencetak URL dan kata sandi di keluaran sel.
4. Menjalankan loop yang terus mencetak log dan statistik GPU agar sesi tidak dianggap idle, dan menyarankan memuat model GGUF 9B lewat kolom pencarian Studio.

Yang baik: kata sandi acak, server hanya `127.0.0.1` (pintu masuk satu-satunya adalah tunnel dengan autentikasi), dan alat sisi server dimatikan. Yang diwaspadai:
- Loop "anti idle" dan akses lewat UI web adalah dua hal yang dipersoalkan Colab di paket gratis; penghentian bisa terjadi kapan saja.
- **Jangan bagikan keluaran sel** (URL dan kata sandi tercetak). Hentikan sesi sebelum membagikan notebook.
- Skrip memanggil `shutil.rmtree` pada `~/.unsloth/studio`: aman di VM sementara, tetapi **jangan jalankan di PC sendiri**.
- `UNSLOTH_STUDIO_PASSWORD` dan `--disable-tools` belum diverifikasi berlaku di versi Unsloth yang terpasang; pastikan halaman Studio benar-benar meminta kata sandi.
- Colab gratis punya satu T4 (~15 GB), jadi model yang jauh lebih besar tidak muat. Di Kaggle, skrip hanya mencetak GPU nomor 0; belum diketahui apakah Studio memakai kedua GPU.
- **Kode notebook orang lain berjalan dengan akunmu.** Baca dulu kodenya, jangan menempelkan Kaggle Secrets, token Hugging Face, atau API key kecuali perlu, dan pakai token sekali pakai berizin minimal. Tunnel membuka sesimu ke internet: pastikan ada autentikasi.

**Cocok dipakai untuk:** mencoba model lokal tanpa GPU sendiri, eksperimen fine-tuning singkat, belajar. **Tidak cocok:** bot Telegram/WhatsApp 24/7, gateway seperti 9Router, penyimpanan model jangka panjang, layanan yang harus selalu tersedia. Alternatif notebook GPU gratis lain: Kaggle Notebooks, Hugging Face ZeroGPU (demo singkat), dan kredit bulanan kecil dari vendor serverless (semua sementara dan tanpa jaminan ketersediaan). Untuk server yang menyala terus: VPS berbayar, komputer di rumah, atau HP di Termux.

### 2.13 Waspada layanan API murah: model masking dan kecurangan reseller

Di media sosial banyak akun menjual "API key AI murah" lewat web app buatan sendiri. Komentar netizen **"masking gak?"** pada intinya bertanya: *apakah model yang dijual disamarkan?* Saya tidak menemukan sumber berbahasa Indonesia yang mendefinisikan kata itu, jadi ini tafsiran dari konteks dan istilah teknis yang setara.

**Arti "masking" (model masking / model substitution):** penjual mengiklankan akses ke model premium (Claude, GPT, dan lain-lain), tetapi di belakang layar memakai model yang lebih murah yang diberi nama model premium, sering dengan system prompt seperti "You are Claude, made by Anthropic". Di komunitas Tiongkok praktik ini disebut "model dilution" (掺水, "dicampur air").

```
Kamu minta : model = "claude-opus"
                                  │
                                  ▼
              ┌────────────────────────────────────────┐
              │        WEB APP / RELAY PENJUAL         │
              │  (isi dalamnya tidak bisa kamu lihat)  │
              │     kamu hanya melihat jawabannya      │
              └─────────┬────────────────────┬─────────┘
                        │                    │
                 tanpa masking          dengan masking
                        ▼                    ▼
        Claude Opus asli            model murah (mis. DeepSeek/Qwen)
          (harga mahal)             + nama dibuat "claude-opus"
                                    + prompt rahasia "Kamu Claude"
                                    → jawaban terlihat mirip
```

**Trik lain yang sering menyertai** (dari alat audit relay yang ditemukan):
- **Konteks dipotong diam-diam:** diklaim 1 juta token, kenyataannya dipotong (mis. ke 32K).
- **Streaming palsu:** jawaban ditunggu penuh lalu dipotong-potong, tanpa keuntungan kecepatan.
- **Tagihan digelembungkan:** prompt tersembunyi disuntikkan untuk menambah jumlah token yang ditagih.
- **Penggantian sebagian:** sebagian lalu lintas dialihkan ke model murah, tidak selalu (asli satu jam, diganti jam berikutnya). Karena itu satu kali tes tidak cukup.

**Kenapa netizen curiga:** harga yang jauh di bawah harga resmi adalah tanda bahaya (satu alat deteksi punya pemeriksaan anomali harga). Selisih harga bisa berasal dari penggantian model, akses tidak resmi, atau subsidi; penjual tertentu tidak bisa dinilai dari sini.

**Cara mengecek sendiri:**
1. Bandingkan harga dengan harga resmi model yang diklaim.
2. Tanyakan identitas model dengan banyak cara (langsung, tidak langsung, batas pengetahuan) dan cari jawaban yang tidak konsisten. Ini lemah, karena system prompt bisa memalsukan identitas.
3. Uji konteks: taruh satu fakta di tengah teks panjang lalu tanyakan kembali (needle-in-a-haystack).
4. Bandingkan dengan sumber tepercaya: kirim soal sulit yang sama ke API resmi dan ke penjual; model murah gagal pada soal yang model asli lulus.
5. Hitung token sendiri dan bandingkan dengan angka pemakaian di tagihan.
6. Ulangi di waktu berbeda karena penggantian bisa sebagian.

**Alat audit sumber terbuka** (belum diuji; hasilnya statistik, bukan bukti pasti; jalankan lokal dengan key uji berisaldo kecil): `zing` (identitas model, jendela konteks, tagihan, streaming), `relay-radar` (deteksi Opus yang diganti Sonnet), `llm-verify`, `tokenwatt-verify`, `fpverify`, `api-model-spy`.

**Risiko lain bagi pembeli:**
- **Privasi:** semua prompt, kode, dan isi file proyek (termasuk `.env` pada agen coding) lewat server penjual yang tidak dikenal dan bisa dicatat.
- **Saldo hilang** bila layanan tutup mendadak, terutama jika sumber aksesnya tidak resmi.
- **Ketentuan penyedia:** menjual ulang akses bisa melanggar aturan penyedia, dan key atau akun di baliknya bisa dicabut.
- **Sulit membuktikan kecurangan** setelah uang dibayar.

**Saran:** waspadai harga jauh di bawah resmi, klaim "unlimited", penjual yang tidak menyebut sumber akses, bayar di muka dalam jumlah besar, dan tidak ada jalur refund. Prinsip pengujian ini berlaku untuk penyedia pihak ketiga mana pun, termasuk gateway yang sudah dipakai. Alternatif lebih aman untuk hemat: model yang memang murah secara resmi (mis. Claude Haiku atau DeepSeek resmi) atau paket gratis resmi (Gemini AI Studio, dan lain-lain), ketimbang "diskon" untuk model unggulan.

---

## 3. Klien Chat (Lapisan 1)

### 3.1 Cherry Studio (desktop)

- Klien AI desktop gratis dan sumber terbuka untuk Windows, macOS, Linux. Menyatukan banyak penyedia dalam satu aplikasi. Data (riwayat chat, pengaturan) **disimpan lokal** di perangkat.
- Fitur: jawaban beberapa model dibandingkan berdampingan, MCP (alat untuk model), mode agen otonom, pencarian web.
- Ada aplikasi mobile Cherry Studio dan fitur "koneksi perangkat" lewat LAN/VPN. Apakah riwayat chat ikut tersinkron lewat jalur itu **(belum diverifikasi)**.

**Menambahkan penyedia (Settings → Model Service):**

1. **Gemini:** pilih penyedia Gemini, tempel API key, tambahkan model Gemini.
2. **Anthropic:** pilih penyedia Anthropic bila punya key resmi.
3. **geraikita / gateway apa pun:** klik **Add**, tipe **OpenAI-compatible**, isi nama, API key, Base URL, lalu tambahkan nama model satu per satu.
4. Pilih model dari dropdown saat mengobrol.

### 3.2 Chatbox (desktop dan HP)

- Tersedia di Windows, macOS, Linux, dan Android/iOS, sehingga tampilannya bisa sama di semua perangkat.
- Penyedia kustom memakai mode **OpenAI API Compatible**: API Host (mis. `https://ai.geraikita.com/v1`) dan API Path `/chat/completions`.

### 3.3 RikkaHub (Android)

- Aplikasi chat AI khusus Android. Mendukung banyak penyedia (OpenAI, Google, Anthropic, dan API lain), model bisa diganti bebas, menerima input gambar, dokumen, dan PDF. Situs: rikka-ai.com.
- Setelan: penyedia Gemini (tipe Google/Gemini) dan satu penyedia OpenAI-compatible untuk geraikita. Nama menu bisa berbeda antar versi.

### 3.4 Alternatif klien chat

| Klien | Catatan |
|---|---|
| Jan | Desktop; model lokal atau penyedia cloud |
| Msty | Berbayar (sekitar $50/tahun); chat bercabang; penyedia online |
| LobeChat, Open WebUI, LibreChat | Berbasis web, dijalankan di server/komputer sendiri; LibreChat condong ke multi-pengguna |
| AnythingLLM | Fokus chat dengan dokumen dan agen |
| 5ire | Desktop, mendukung MCP |

### 3.5 Pengaturan sinkronisasi antar perangkat

- Pengaturan dan riwayat chat Cherry Studio/Chatbox disimpan lokal per perangkat. API key harus dimasukkan lagi di tiap perangkat.
- **Diluruskan:** Cherry Studio **punya** fitur **Channels** untuk menghubungkan Agent ke aplikasi pesan, termasuk Telegram (lihat 3.9). Klaim lama di catatan ini bahwa Cherry Studio tidak punya bot Telegram bawaan salah/usang. Untuk alternatif satu riwayat bersama, lihat 7.4.

### 3.6 Cherry Studio API Gateway (gateway lokal)

Menu: **Settings → API Gateway** (dulu bernama "API Server"). Keterangan di aplikasi: mengekspos kemampuan AI Cherry Studio lewat HTTP API yang kompatibel dengan OpenAI dan Anthropic.

**Fungsi:** menjadikan Cherry Studio **pintu masuk lokal untuk program lain**. Penyedia dan key yang sudah kamu atur di Cherry Studio bisa dipakai ulang oleh alat lain (ekstensi coding, skrip, agen) tanpa memasukkan key penyedia lagi di alat itu. Alat lain hanya menyimpan alamat lokal dan key dari gateway. Gateway juga menjadi layanan internal yang wajib menyala untuk fitur **agen** Cherry Studio; untuk chat, gambar, dan terjemahan biasa tidak perlu dinyalakan.

```
Cline / Kilo Code / skrip / agen → Cherry Studio API Gateway (127.0.0.1:23333) → Gemini / geraikita / dst.
                                    (memakai key lokal dari Cherry)               (memakai key aslimu)
```

**Isi menunya:**
- **Status (start/stop/restart):** menyalakan atau mematikan server. Port bawaan **23333**, bisa diganti ke port bebas 1000 sampai 65535, dan **hanya bisa diubah saat gateway berhenti**. Error "address already in use" berarti port dipakai program lain atau instans Cherry Studio kedua.
- **Alamat URL:** `http://127.0.0.1:<port>` (menurut dokumentasi memakai `http`, bukan `https`).
- **API key dan header Authorization:** kunci untuk **masuk ke gateway** (bukan key Gemini/geraikita). Program lain mengirim `Authorization: Bearer <API Key>`. Header bisa disalin langsung dari halaman. Key dan port tidak bisa diubah saat gateway berjalan.

**Cara memakai dan menguji:**
1. Pastikan modelnya bisa mengobrol normal di Settings → Model Services (gateway tidak menyediakan model sendiri; ia butuh minimal satu penyedia dan model yang berfungsi).
2. Klik **Start** sampai status **Running**; halaman menampilkan URL dan tautan ke dokumentasi API.
3. Uji bertahap: minta daftar model dulu, lalu kirim teks pendek, baru sambungkan aplikasi penuh. Contoh (path dan format nama model perlu dicocokkan dengan dokumentasi API di halaman gateway): `curl http://127.0.0.1:23333/v1/models -H "Authorization: Bearer KEY"`.
4. Kriteria berhasil: status Running, daftar model terbaca, permintaan teks pendek sukses, dan klien tidak bisa memanggil lagi setelah gateway dihentikan.

**Keamanan dan batasan:**
- Dokumentasi lama menyebut server hanya mendengarkan di `127.0.0.1` (tidak terbuka ke jaringan lokal atau internet; untuk lintas mesin disarankan SSH reverse tunnel). Dokumentasi baru membahas akses dari perangkat lain dan menegaskan bahwa itu **memperluas paparan**, jadi harus memeriksa sendiri pengaturan listening, firewall, dan kontrol akses. Tetap gunakan lokal saja kecuali sangat perlu.
- Key bisa memakai **semua penyedia** di Cherry Studio. Jangan taruh di repositori, tangkapan layar, atau log bersama. Bila bocor: hentikan gateway, klik **Regenerate**, perbarui semua klien lokal.
- Akibat praktis: **bot Telegram di HP (Termux) tidak bisa memakainya** karena `127.0.0.1` hanya menunjuk ke komputer yang sama.
- Cherry Studio kemungkinan harus tetap berjalan agar gateway aktif. Menghentikan gateway memutus agen dan koneksi kanal yang sedang berjalan.
- Biaya dan kuota tetap dikenakan oleh penyedia sebenarnya; rekap pemakaian bisa dilihat di Settings → Usage Statistics.
- Dokumentasi tidak menyebut **fallback otomatis**. Satu laporan bug di repositori Cherry Studio menyebut pesan error dari penyedia hulu bisa disamarkan dari pengguna, yang menyulitkan pelacakan error kuota.

**Perbandingan dengan 9Router:**

| | Cherry Studio API Gateway | 9Router |
|---|---|---|
| Sumber konfigurasi | Penyedia dan key yang sudah ada di Cherry Studio | Dikonfigurasi terpisah di dasbornya |
| Fallback otomatis | Tidak ditemukan di dokumentasi | Ya (combo, tiga tingkat) |
| Dasbor kuota | Tidak ada | Ada |
| Jangkauan | Komputer yang sama | Bisa diatur lebih luas |
| Cocok untuk | Memakai ulang setup Cherry untuk alat lain | Fallback dan banyak akun |

### 3.7 Code Mate (alat coding CLI di Cherry Studio)

**Code Mate** adalah halaman di Cherry Studio untuk **memasang, mengonfigurasi, dan menjalankan alat coding berbasis command line**. Ia mengenali salinan yang dikelola Cherry Studio dan juga CLI yang sudah ada di sistem (PATH). Daftar alat yang terlihat di dokumentasi antara lain Claude Code, OpenAI Codex, Antigravity CLI, OpenCode, Qwen Code, Kimi Code, GitHub Copilot CLI, Hermes Agent, OpenClaw, dan lain-lain (daftar berubah antar versi).

- **Menyambungkan model:** untuk CLI yang memakai model dari Cherry Studio, daftar penyedia yang kompatibel ditampilkan di sisi kanan, termasuk **Unified Gateway** ("Any CLI, every model") yang mengarahkan CLI lewat gateway Cherry Studio sehingga bisa memakai model apa pun yang sudah kamu atur. CLI dengan login akun sendiri tetap memakai login aslinya.
- **Alur:** buka Code Mate dari Launchpad → pilih alat dan **Install** → pilih penyedia/model → pilih **folder proyek** dan terminal → **Launch** → jalankan perintah baca-saja dulu untuk memastikan akun, model, dan direktori benar sebelum mengizinkan perubahan file.
- **Claude Code:** ada dua mode model, **General** (semua permintaan memakai satu model) dan **Detailed** (model berbeda per peran: Opus, Sonnet, Haiku, Subagent, dan lain-lain). Pakai Detailed hanya bila memang perlu.
- **Sumber instalasi:** Cherry Studio Managed (dikelola di Code Mate), System PATH (diperbarui lewat package manager aslinya), atau bawaan aplikasi. Bila CLI tidak terdeteksi, pastikan perintahnya jalan di terminal login lalu mulai ulang aplikasi.
- Hasil dengan model non-Claude pada Claude Code bisa bervariasi (lihat 9.3).

Untuk memakai kedua fitur ini bersama VS Code, lihat 9.11.

### 3.8 Knowledge Base (RAG): menjawab dari dokumenmu, mirip NotebookLM

Cherry Studio bisa menjawab pertanyaan seputar sumber yang kamu sediakan lewat fitur **Knowledge Base** (sejak v0.9.1). Ide dasarnya mirip NotebookLM, tetapi caranya berbeda.

**Sumber yang bisa ditambahkan:** file (PDF, DOCX, TXT, dan lain-lain), folder (file berformat yang didukung divektorkan otomatis), URL, sitemap (XML), dan catatan teks.

```
Dokumen (PDF / Word / URL / catatan)
        │ dipecah menjadi potongan kecil
        ▼
Model embedding ──→ vektor ──→ database lokal (libSQL) di komputermu
                                      ▲
Pertanyaanmu ──→ model embedding ─────┘ cari potongan paling mirip
                                      │
                                      ▼
   potongan relevan + pertanyaan ──→ model chat ──→ jawaban
```

Dokumen dipecah menjadi potongan sekitar 200 sampai 500 karakter (angka dari dokumentasi versi Jepang), diubah model embedding menjadi vektor, dan disimpan bersama potongan aslinya di database lokal. Data dan salinan dokumen disimpan lokal di direktori data Cherry Studio.

**Langkah singkat:**
1. Di Model Services, cari model embedding (ada filter "Embedding") dan tambahkan ke "My Models". Dokumentasi memakai `bge-m3` sebagai contoh.
2. Klik ikon Knowledge Base di bilah kiri, **Add**, beri nama dan pilih model embedding.
3. Tambahkan file, folder, URL, atau catatan; tunggu vektorisasi selesai.
4. Uji lewat tombol cari di halaman itu untuk melihat potongan yang ditemukan.
5. Di chat, pilih knowledge base sebagai sumber jawaban.

**PDF hasil scan:** prapemrosesan dokumen dengan OCR membutuhkan Cherry Studio v1.4.8 atau lebih baru. **Ilustrasi dalam dokumen belum diubah menjadi vektor** dan perlu diubah manual menjadi teks.

| | Cherry Studio Knowledge Base | NotebookLM |
|---|---|---|
| Dasar | RAG: hanya potongan paling relevan yang dikirim ke model | Dirancang khusus menjawab dari sumbermu (setahu saya) |
| Pertanyaan menyeluruh ("rangkum seluruh buku") | Bisa lebih lemah (model hanya melihat potongan) | Umumnya lebih baik |
| Model | Bebas (BYOK, termasuk lokal) | Model Google |
| Fitur ekstra (ringkasan audio, peta pikiran, panduan belajar) | Tidak ada | Ada (setahu saya) |
| Penyimpanan | Lokal di komputermu | Cloud Google |
| Kualitas | Tergantung model embedding, ukuran potongan, OCR | Dikelola Google |

Rincian NotebookLM dalam tabel berasal dari pengetahuan saya, bukan dari pencarian.

**Hal yang perlu diketahui:**
- **Butuh model embedding.** Belum diverifikasi apakah Gemini atau geraikita-mu menyediakannya. Bila tidak, `bge-m3` bisa dijalankan lewat Ollama atau penyedia lain.
- **Privasi:** potongan teks dikirim ke penyedia embedding dan potongan yang ditemukan dikirim ke model chat. Untuk dokumen sensitif jalankan embedding dan chat secara lokal (embedding lokal lewat Ollama/LM Studio memungkinkan proses offline, dan model chat sebaiknya juga lokal). Model gratis (mis. Gemini gratis) datanya bisa dipakai penyedia.
- Untuk sedikit dokumen, lebih sederhana mengunggah file langsung ke chat dengan model berkonteks panjang seperti Gemini. Knowledge Base cocok untuk banyak dokumen.
- Bila dokumen berubah atau bertambah, perbarui isi knowledge base.

### 3.9 Channels: Agent Cherry Studio ke Telegram dan aplikasi pesan (koreksi)

> **Koreksi.** Versi catatan sebelumnya menyatakan Cherry Studio tidak punya bot Telegram bawaan. Itu **salah atau usang**. Dokumentasi dan berkas lokalisasi aplikasinya menunjukkan fitur **Channels**.

- **Fungsi:** menghubungkan **Agent** Cherry Studio ke platform pesan (Feishu, Telegram, Discord; berkas aplikasi juga memuat pengaturan Slack). WhatsApp tidak terlihat di daftar.
- **Syarat:** API Gateway Cherry Studio (lihat 3.6) harus aktif; menghentikannya memutus agen dan koneksi kanal yang berjalan.
- **Telegram:** pengaturannya berupa token bot dan daftar ID chat yang diizinkan, memakai long polling. **Cherry Studio harus tetap berjalan** agar bisa menerima pesan. Menurut string aplikasi, daftar ID kosong berarti semua chat diizinkan: isi dengan ID-mu supaya kuota tidak dipakai orang lain.
- **Belum diverifikasi:** apakah percakapan Telegram dan sesi agen di Cherry Studio saling terhubung, kemampuan agen dibanding bot buatan sendiri, dan apakah kanal bekerja saat jendela utama ditutup.
- Konsekuensi arsitektur: PC (atau perangkat yang menjalankan Cherry Studio) harus menyala. Untuk perbandingan dengan Hermes dan bot sendiri, lihat 7.4.

---

## 4. Pencarian Web untuk AI (pelengkap klien chat)

### 4.1 Mengapa model tidak tahu peristiwa terbaru

Model punya **batas pengetahuan** (tanggal akhir pelatihan). Peristiwa sesudahnya (misalnya juara Piala Dunia 2026) tidak diketahui kecuali model diberi hasil pencarian web. Menambahkan tanggal hari ini ke system prompt membantu untuk "hari ini tanggal berapa", tetapi **tidak** membuat model tahu siapa juaranya. Pengetahuan itu hanya datang dari hasil pencarian.

Bot Telegram yang hanya meneruskan chat ke `/chat/completions` tidak mencari apa pun. Pencarian baru terjadi lewat perintah khusus (`/cari`). Cherry Studio bisa menjawab karena pencarian web (ikon globe) menyala.

### 4.2 Pencarian web di Cherry Studio (Settings → Web Search)

> **Diluruskan.** Versi sebelumnya menyebut jalur tanpa key adalah "pencarian lokal" (jendela browser tersembunyi yang mengambil hasil Google/Bing). Itu berasal dari rujukan lama. Di versi Cherry Studio yang diperiksa langsung lewat menu pengaturan, jalur tanpa key bawaannya adalah **exaMCP**.

**Temuan dari menu pengaturan:**
- **Search service providers** (pilih satu): **exaMCP (default, tanpa API key)**, zhipu, tavily, searxng, exa, bocha, querit, jina, firecrawl, parallel, serply, duckduckgo.
- **Pengaturan bawaan:** API host `https://mcp.exa.ai/mcp`, jumlah hasil 5, metode kompresi "cutoff", batas 2.000 karakter. Arti "cutoff" saya tafsirkan sebagai pemotongan isi tiap hasil hingga batas karakter **(belum diverifikasi)**. Nilai ini hemat token dan layak ditiru di bot.
- Ikon globe harus diaktifkan **di setiap jendela chat**; pengaturan global tidak otomatis berlaku untuk semua sesi.

**Alur yang terjadi:**

```
Pertanyaan → Cherry Studio mengirim kueri ke exaMCP (mcp.exa.ai)
→ 5 hasil kembali, tiap hasil dipotong sekitar 2.000 karakter
→ teks hasil disisipkan ke prompt → model menjawab dari teks itu
```

Karena jawaban diambil dari teks hasil pencarian di dalam prompt, **batas pengetahuan model tidak lagi menghalangi**. Itulah sebabnya Cherry Studio bisa menjawab juara Piala Dunia 2026 meskipun modelnya berhenti dilatih sebelum turnamen dimulai.

**Exa MCP (default):**
- Server MCP hosted milik Exa: pencarian web dan pengambilan halaman dengan isi yang sudah bersih. Bisa dipakai **tanpa API key** pada paket gratis yang dibatasi (rate-limited). Key hanya perlu bila terkena batas (error 429).
- Angka batas MCP **tidak diterbitkan** Exa. Satu artikel menyebut sekitar 2 permintaan/detik dan 50 permintaan/hari per IP tanpa key; anggap itu perkiraan.
- Bila kena batas: isi API key Exa di pengaturan penyedia, atau pindah ke penyedia lain di daftar.

**Kelompok penyedia pencarian (hasil pengamatan, kebutuhan key tiap penyedia belum semuanya diperiksa):**

| Kelompok | Contoh | Catatan |
|---|---|---|
| Tanpa key | exaMCP, duckduckgo | Gratis, ada batas pemakaian |
| API dengan key | tavily, exa, jina, firecrawl, parallel, serply, bocha, zhipu, querit | Kuota gratis bervariasi |
| Dijalankan sendiri | searxng | Butuh server sendiri |
| Bawaan model | mis. Gemini | Jalur terpisah di pengaturan model |

**Privasi dan keamanan:** kueri (dan kemungkinan konteks pertanyaan) dikirim ke server pihak ketiga (Exa, atau penyedia yang dipilih), jadi jangan menulis data pribadi/rahasia di pertanyaan yang memicu pencarian. Isi halaman web masuk ke prompt dan diperlakukan sebagai masukan yang **tidak sepenuhnya tepercaya** (halaman bisa memuat instruksi yang mengarahkan model); verifikasi jawaban penting dari sumbernya.

### 4.3 Pencarian untuk bot Telegram (di Termux)

Bot tidak punya jendela browser sungguhan, jadi menarik halaman Google/Bing langsung lewat `httpx` cepat diblokir atau kena CAPTCHA. Pilihan realistis adalah API pencarian:

| Layanan | Kuota gratis (per Okt 2026) | Catatan |
|---|---|---|
| Tavily | 1.000 kredit/bulan, tanpa kartu | Pencarian dan ekstraksi halaman |
| Firecrawl | 1.000 kredit/bulan, tanpa kartu | Sumber lama menyebut 500; cek dashboard |
| Brave Search | Tidak lagi gratis | Paket gratis dihapus Februari 2026; kini kredit bulanan $5 dan butuh kartu |
| DuckDuckGo tanpa key | Gratis | Tidak resmi, hasil dari satu sumber, sering tidak stabil; beberapa library butuh komponen yang sulit dipasang di Termux |

- Tavily/Firecrawl bisa dipanggil lewat REST dengan `httpx`, tanpa library tambahan.
- Hasil pencarian masuk ke prompt, sehingga **memakai token model**. Batasi 3 sampai 5 hasil dengan potongan teks pendek.
- Pencarian otomatis di bot: deteksi pertanyaan bertema "terbaru/juara/sekarang/2026", jalankan pencarian, lalu kirim hasilnya ke model.
- Meniru exaMCP di bot lebih rumit daripada REST biasa: endpoint MCP butuh inisialisasi sesi lewat protokol MCP. Tavily/Firecrawl lewat `httpx` tetap jalur paling sederhana.
- Membaca satu URL di bot cukup satu permintaan GET lewat `httpx` ke `https://r.jina.ai/{url}` (lihat 4.4), tanpa library tambahan. Bisa dijadikan perintah `/baca <url>`.

### 4.4 Penyedia pengambil URL (URL fetch provider) di Cherry Studio

Selain **Search service providers**, menu Web Search punya bagian **URL fetch provider** dengan pilihan: querit, fetch, **jina (default)**, firecrawl, crawl4AI. Default Jina tampil dengan kolom API key kosong dan API host `https://r.jina.ai`.

**Fungsinya: mencari dan membaca adalah dua tahap berbeda.**

| | Search service provider | URL fetch provider |
|---|---|---|
| Tugas | Menemukan halaman yang relevan | Mengambil isi lengkap **satu URL tertentu** |
| Masukan | Kata kunci atau pertanyaan | Alamat halaman (URL) |
| Keluaran | Daftar hasil (judul, URL, kutipan, kadang isi) | Isi halaman yang sudah dibersihkan menjadi teks/Markdown |
| Contoh | exaMCP, Tavily, DuckDuckGo | Jina, Firecrawl, Crawl4AI |

Pengambil halaman biasanya dipakai ketika mesin pencari hanya mengembalikan URL dan kutipan singkat, atau ketika kamu memberi URL langsung dan meminta model merangkum isinya. Karena exaMCP sudah mengembalikan isi halaman yang bersih, fetch provider kemungkinan jarang dibutuhkan untuk pencarian biasa. **Kapan persisnya Cherry Studio memanggilnya belum ditemukan di dokumentasi** (lihat Bagian 14).

**Pilihan di daftar:**
- **Jina Reader (default):** mengubah URL yang sudah ada menjadi konten siap pakai LLM (Markdown). Cukup menaruh URL setelah alamat Reader: `https://r.jina.ai/https://example.com`. **Mode dasar bisa dipakai tanpa key** (anonim, batas lebih rendah); fitur ReaderLM-v2 butuh key. Isi key gratis dari Jina hanya bila muncul error batas.
- **Firecrawl:** butuh key; lebih lengkap (crawling, pencarian, ekstraksi terstruktur).
- **Crawl4AI:** bekerja lokal memakai browser Chromium headless dengan rendering JavaScript, tanpa key eksternal; cocok untuk situs berat JavaScript. Kemungkinan butuh pemasangan lokal (belum diperiksa).
- **fetch** dan **querit:** keterangannya tidak ditemukan. "fetch" kemungkinan pengambilan HTTP sederhana (dugaan).

**Cara menguji:** nyalakan web search, minta "ringkas isi halaman ini: [URL]", lalu cek apakah isi ringkasan sesuai halaman.

**Privasi:** URL yang dibaca lewat Jina melewati server Jina; jangan dipakai untuk halaman pribadi/internal. Halaman yang butuh login umumnya tidak bisa diambil. Isi halaman tetap masukan yang tidak sepenuhnya tepercaya.

---

## 5. Agen AI (Lapisan 2)

### 5.1 Non-agent vs agent

| | Non-agent (chatbot biasa) | Agent |
|---|---|---|
| Cara kerja | Kamu bertanya, model menjawab sekali | Kamu memberi tujuan; model merencanakan, memakai alat, melihat hasil, mengulang |
| Alat | Tidak ada | Cari web, browser, baca/tulis file, jalankan kode, API |
| Contoh | gemini.google.com, bot Telegram biasa | Claude Code, Claude in Chrome, deep research, Hermes Agent |

Tiga hal yang membedakan: **alat**, **siklus beberapa langkah**, dan **otonomi**. Modelnya sendiri sama; yang membuat sesuatu menjadi agen adalah program di sekelilingnya (siklus kerja, alat, memori, izin). Model hanya "otaknya".

Pertimbangan:
- Agen jauh lebih berguna untuk tugas banyak langkah, tetapi satu tugas bisa memakai puluhan panggilan model. Kuota gratis cepat habis.
- Agen lebih berisiko (bisa menghapus file, mengirim sesuatu yang keliru). Biasanya ada persetujuan di langkah penting.
- Untuk bertanya, menulis, menerjemahkan, belajar: non-agent cukup.
- Pencarian web satu kali dengan pipeline tetap (ikon globe) berada di tengah spektrum, belum agen penuh.

### 5.2 Hermes Agent

- **Bukan model AI.** Program agen sumber terbuka dari Nous Research (dirilis Februari 2026, lisensi MIT, dijalankan sendiri). Nous juga membuat model bernama "Hermes", itu hal berbeda dari Hermes Agent.
- **BYOK:** mendukung Nous Portal, OpenRouter, NVIDIA NIM, z.ai/GLM, Kimi, OpenAI, atau **endpoint kustom OpenAI-compatible** (URL, key, nama model). Ganti model dengan `hermes model` atau `/model`.
- Berbasis terminal (CLI). Punya gateway ke aplikasi pesan (Telegram, Discord, Slack, WhatsApp, Signal, email), 40+ alat, memori, "skill" yang dibuat dari pengalaman, dan tugas terjadwal. Bisa dipasang di Termux. Dukungan Windows asli masih beta awal.
- Punya penyedia cadangan dengan failover otomatis ke LLM lain saat model utama error. Bisa jadi backend untuk Open WebUI, LobeChat, LibreChat, NextChat, atau Chatbox lewat endpoint OpenAI-compatible.
- Perbedaan dengan Cherry Studio: Cherry Studio = klien chat grafis; Hermes = agen terminal/pesan yang mengerjakan tugas.
- Peringatan: baca skrip pemasangan sebelum menjalankan, agen bisa menjalankan perintah di perangkatmu (baca bagian keamanannya), dan satu tugas bisa menghabiskan kuota gratis.

### 5.3 Syarat model agar bisa jadi otak agen

1. **Mendukung tool calling (function calling)** lewat endpoint. Tidak semua model mendukung, dan gateway harus meneruskan parameter `tools`.
2. **Konteks panjang.** Hermes mensyaratkan sekitar 64.000 token (sebagian sumber menyebut 32.000 sebagai minimum). Agen menghabiskan konteks dengan cepat.
3. **Kuota memadai.** Banyak panggilan, prompt panjang.
4. **Kualitas model.** Model kecil/ringan sering salah memanggil alat atau hanya menjelaskan tanpa memanggil. Aktifkan "tool-use enforcement" bila tersedia.

Saran: mulai dari Gemini sebagai otak agen (konteks panjang, kuota gratis), model geraikita sebagai cadangan.

### 5.4 Menguji tool calling

**Cara A: skrip uji (pengujian sebenarnya).** Kirim satu permintaan dengan daftar alat ("menu" alat, fungsinya sendiri tidak ada), lalu lihat apakah model membalas dengan `tool_calls`:

```python
import httpx
URL = "https://ai.geraikita.com/v1/chat/completions"   # untuk Gemini: https://generativelanguage.googleapis.com/v1beta/openai/chat/completions
KEY = "ISI_KEY"
body = {
    "model": "qwen3.8-flash",
    "messages": [{"role": "user", "content": "Cuaca di Jakarta sekarang? Gunakan alat."}],
    "tools": [{"type": "function", "function": {
        "name": "get_weather", "description": "Ambil cuaca kota",
        "parameters": {"type": "object",
                       "properties": {"city": {"type": "string"}},
                       "required": ["city"]}}}],
}
r = httpx.post(URL, headers={"Authorization": f"Bearer {KEY}"}, json=body, timeout=60)
print(r.json()["choices"][0]["message"].get("tool_calls"))
```

Membaca hasil:
- Daftar berisi `get_weather` dan `city` = Jakarta: **lolos**.
- `None` dan model menjawab teks biasa: **gagal** (model tidak mendukung, atau gateway membuang `tools`).
- Error 400 / "tools not supported": model atau gateway menolak parameter alat.
- Error 401: key salah.
- Lolos belum menjamin andal di tugas agen panjang; jalankan 2 sampai 3 kali.

**Cara B: lewat Cherry Studio dengan MCP.** Tambahkan server MCP sederhana (mis. fetch: tipe STDIO, perintah `uvx`, argumen `mcp-server-fetch`) di Settings → MCP Server, aktifkan di kolom input chat, lalu minta "Buka https://example.com dan ringkas isinya". Bila muncul status pemanggilan alat dan jawaban berisi isi halaman, tool calling bekerja melalui gateway itu. Label kemampuan (alat/jaringan/penalaran) di pengaturan model hanyalah penanda, bukan hasil uji.

**Cara C: uji tiruan lewat chat biasa (lemah).** Minta model membalas hanya dengan JSON seolah memanggil alat. Ini hanya menguji kepatuhan format teks, bukan tool calling tingkat API. Kegagalan di sini informatif, keberhasilan belum membuktikan apa-apa.

### 5.5 Alternatif agen

| Agen | Catatan |
|---|---|
| OpenClaw | Pelopor agen pribadi yang dijalankan sendiri; berorientasi gateway ke aplikasi chat. Februari 2026 ditemukan masalah malware di marketplace ClawHub, hati-hati dengan skill pihak ketiga. Hermes menyediakan perintah `migrate` dari OpenClaw. |
| NanoClaw / ZeroClaw / PicoClaw | NanoClaw: sesi terisolasi dalam kontainer; ZeroClaw: runtime Rust kecil; PicoClaw: perangkat berdaya rendah |
| Agen terkelola | Claude Cowork, Manus, Perplexity Computer, hosted OpenClaw |
| Agen coding | Claude Code, Codex, Cline (bukan pengganti langsung karena fungsinya lebih sempit) |

### 5.6 Memory layer untuk agen (contoh: Obsidian)

**Memory layer** adalah penyimpanan di luar model tempat agen menulis dan membaca pengetahuan lintas sesi. Model dan agen pada dasarnya **tanpa ingatan**: begitu sesi selesai, apa yang dibahas hilang, dan jendela konteks terbatas. Memory layer menyelesaikannya dengan membiarkan agen **mencari sendiri** di folder yang tertata, bukan memuat semua konteks sekaligus.

**Peran Obsidian:** aplikasi catatan gratis berbasis **file Markdown lokal**. Satu "vault" hanyalah folder di disk, sehingga agen cukup membaca dan menulis file teks. Catatan bisa memiliki metadata di bagian atas (status, proyek, tipe) agar mudah dicari.

```
    ┌─────────────────────────────────────────────────────────────┐
    │                      AGEN  (lapisan 2)                      │
    │           Claude Code │ Hermes │ OpenCode │ Codex           │
    └─────────────┬─────────────────────────────────┬─────────────┘
                  │ pakai model                     │ baca / tulis
                  ▼                                 ▼
    ┌────────────────────────────┐    ┌───────────────────────────┐
    │       LAPISAN 3 + 4        │    │       MEMORY LAYER        │
    │ 9Router → Claude API dll.  │    │ vault Obsidian (file .md) │
    └────────────────────────────┘    └───────────────────────────┘
                                                    ▲
                                                    │ kamu juga membuka dan mengedit
                                                      lewat aplikasi Obsidian
```

Memory layer bukan bagian dari empat lapisan, melainkan **pendamping lapisan 2**: ia tidak dilalui permintaan ke model.

**Contoh isi vault dan alur sesi:**

```
vault/
├── 00-index.md            ← pintu masuk: daftar proyek dan aturan
├── preferensi.md          ← gaya kerja, bahasa, hal yang dihindari
├── proyek/web-app/
│   ├── keputusan.md       ← "kenapa pilih Supabase, bukan Firebase"
│   └── todo.md
└── harian/2026-10-26.md   ← log apa yang dikerjakan hari ini
```

1. Sesi baru dimulai: agen membaca indeks dan catatan proyek.
2. Agen bekerja dengan konteks itu.
3. Di akhir, agen menulis ringkasan dan keputusan ke vault.
4. Sesi berikutnya (atau agen lain) mulai dari catatan yang sama.

**Cara menghubungkannya:**
1. **Langsung lewat folder:** jalankan agen di folder vault dan beri tahu lewat file instruksi (mis. `CLAUDE.md`) untuk membaca indeksnya. Untuk Claude Code, MCP tidak wajib.
2. **Lewat MCP:** plugin Obsidian Local REST API ditambah jembatan MCP (mis. `mcp-obsidian` atau `obsidian-mcp-server`) memberi alat cari, baca, buat, perbarui catatan, dan telusuri backlink. Berguna untuk Claude Desktop dan agen lain.
3. **Server memori khusus berbasis vault:** contoh `devbrain` (menghubungkan Antigravity, Claude Code, Hermes, OpenCode ke vault Obsidian lokal lewat MCP dengan pencarian hibrida) dan `engram` (server MCP self-hosted di atas vault Markdown yang disimpan di Git). Keduanya proyek komunitas kecil; cek dulu sebelum memakai. Hermes mendukung server MCP lewat `~/.hermes/config.yaml`.

Keunggulan terbesar: **memori yang sama dipakai banyak agen** (satu sumber kebenaran), transparan, bisa diversi dengan Git, dan tidak terkunci vendor.

**Bedanya dengan `claude-mem`:**

| | Vault Obsidian | claude-mem |
|---|---|---|
| Cara menulis | Eksplisit (agen atau kamu menulis catatan) | Otomatis (hook menangkap aktivitas lalu dipadatkan AI) |
| Bentuk | File Markdown yang bisa dibaca dan diedit | SQLite + Chroma, kurang ramah dibaca manusia |
| Kendali | Tinggi, versi lewat Git | Lebih otomatis, kurang transparan |
| Cocok untuk | Pengetahuan terkurasi, keputusan, preferensi | Mengingat jejak kerja sesi demi sesi |

Keduanya bisa dipakai bersamaan (lihat 10.11 untuk detail claude-mem).

**Risiko dan kebiasaan baik:**
- Agen bisa menimpa atau menghapus catatan: simpan vault di Git atau cadangkan, batasi folder yang boleh ditulis.
- **Jangan simpan rahasia** (API key, kata sandi, data pribadi) di vault yang dibaca agen.
- Isi catatan yang berasal dari web atau dokumen luar tidak sepenuhnya tepercaya (prompt injection).
- Jangan memuat seluruh vault: pakai indeks dan pencarian. Model gratis berkonteks kecil cepat penuh.
- Plugin REST API dan server MCP komunitas membuka akses lokal ke vault: pasang hanya yang dipercaya.
- Untuk bot Telegram sendiri, ide serupa bisa dibuat sederhana: satu file Markdown preferensi dan ringkasan per pengguna yang disisipkan ke pesan sistem.

---

## 6. Gateway dan Fallback Otomatis (Lapisan 3)

### 6.1 Masalah dan konsep

Masalah: kuota gratis sebuah model habis (harus menunggu besok). Keinginan: otomatis pindah ke model gratis lain yang kuotanya masih ada.

- Dari pencarian, **tidak ada aplikasi chat desktop** (Cherry Studio dan sejenisnya) yang punya pindah otomatis antar-penyedia sebagai fitur bawaan. Cherry Studio hanya punya rotasi beberapa API key dalam satu penyedia (bukan pindah model).
- Pola umum: aplikasi hanya bicara ke **satu endpoint**; **gateway** di tengah yang mencoba model berurutan. Dengan begitu aplikasi apa pun (Cherry Studio, Chatbox, RikkaHub, Hermes) memakai gateway yang sama dan aturan fallback diatur di satu tempat.
- Aturan perancangan yang baik: pindah ke model berikutnya hanya untuk galat yang bisa dicoba ulang (429, 503, timeout); berhenti langsung untuk galat autentikasi atau permintaan tidak valid. Kode penanda kuota habis berbeda antar penyedia (429, 402, atau 403).

Tiga cara mendapatkan fallback:
1. **OpenRouter** dengan parameter `models` (terbatas pada model di dalam OpenRouter).
2. **Gateway lokal** (9Router, LiteLLM, atau skrip kecil buatan sendiri).
3. **Di bot Telegram:** rantai model berurutan di kode bot.

### 6.2 9Router

- **Apa:** gateway/router AI sumber terbuka (MIT, pembuat: decolua), dijalankan lokal/self-hosted. Alat atau agen mengirim permintaan ke endpoint 9Router (biasanya `http://localhost:20128/v1`), lalu 9Router memilih model dan penyedia, dan berpindah bila perlu. Mudah dipasang (`npm install -g 9router`).
- **Fitur:**
  - Fallback tiga tingkat: **langganan → murah → gratis**.
  - "Combo": beberapa penyedia digabung menjadi satu penyedia virtual dengan round-robin dan fallback otomatis.
  - Rotasi banyak akun dan pelacakan kuota waktu nyata (dasbor menampilkan pemakaian token dan hitung mundur reset).
  - Penghemat token yang memampatkan isi hasil alat (`tool_result`), sekitar 20% sampai 40% (klaim situs: 20 sampai 65% dengan fitur tambahan).
  - 60+ penyedia, endpoint OpenAI-compatible, banyak alat didukung (Claude Code, Cursor, Cline, Hermes Agent, dan lain-lain).
- **Untuk gratis?** Sebagian, tergantung penyedia gratis yang disambungkan. Situs mencantumkan beberapa penyedia "gratis" (Kiro, iFlow, Qwen, OpenCode Free, OpenRouter, NVIDIA NIM, Gemini, Cloudflare AI), tetapi sebuah ulasan menyebut iFlow, Qwen, dan Gemini CLI tidak lagi menawarkan akses gratis. Dua sumber bertentangan, cek sendiri.
- **Risiko:** beberapa penyedia disambungkan lewat OAuth atau cookie browser, dan ada fitur "MITM Bridge" yang mencegat langganan IDE. Saya belum memeriksa ketentuan tiap penyedia; memakai token langganan/cookie lewat perantara bisa melanggar aturan layanan dan berisiko akun dikunci, dan cookie/token lebih berbahaya bila bocor daripada API key biasa. **Gunakan jalur API key resmi** (Gemini, OpenRouter, NVIDIA NIM, geraikita).
- Proyek komunitas yang baru: cek repositori GitHub sebelum memasang. geraikita kemungkinan bisa ditambahkan sebagai penyedia kustom **(belum diverifikasi)**.
- Lebih nyaman dijalankan di PC daripada di Termux (instalasi Node di Termux belum diverifikasi).

### 6.3 Alur pemasangan dengan 9Router

1. Pasang dan jalankan 9Router, buka dasbornya.
2. Tambahkan penyedia lewat API key (OpenRouter, Gemini, geraikita bila didukung). Buat **combo** dengan urutan fallback.
3. Buat API key di 9Router sendiri.
4. **Hermes Agent:** `hermes model` → Custom endpoint → URL `http://localhost:20128/v1`, key dari 9Router, nama model/combo dari dasbor.
5. **Cherry Studio:** Settings → Model Service → Add → tipe OpenAI-compatible, Base URL `http://localhost:20128/v1`, key 9Router, model = nama combo. Pilih nama **combo** di chat supaya fallback jalan.

Catatan penting:
- `localhost` hanya berlaku bila 9Router dan aplikasi ada di komputer yang sama. Bila 9Router di HP dan klien di PC, pakai IP HP di jaringan Wi-Fi yang sama.
- Kuota gratis dipakai bersama oleh Hermes dan Cherry Studio. Agen yang boros panggilan bisa menguras jatah chat.
- Proyek lebih berfokus ke alat pemrograman; kelancaran untuk chat biasa (mis. streaming) belum bisa dipastikan, coba dulu dengan satu model.
- Penghemat token bekerja pada hasil alat, hampir tidak berpengaruh di chat biasa tanpa alat.
- Pencarian web Cherry Studio berjalan sendiri dan tidak lewat 9Router.
- Fallback tetap butuh minimal satu penyedia yang masih punya kuota; kalau semua habis, muncul error.
- Cherry Studio mengirim ulang seluruh riwayat tiap permintaan, jadi percakapan tetap nyambung walau model berganti, tetapi gaya jawaban bisa berubah, dan pesanmu bisa sampai ke semua penyedia dalam combo (hati-hati dengan penyedia gratis yang memakai data untuk pelatihan).
- Apakah jawaban yang sedang mengalir bisa dilanjutkan bila penyedia gagal di tengah **(belum diverifikasi)**; biasanya fallback terjadi sebelum jawaban dimulai.

**Cara menguji fallback:** taruh penyedia yang kuotanya hampir habis di urutan pertama combo, kirim beberapa pesan dari Cherry Studio sambil memantau dasbor. Bila jawaban tetap datang dari penyedia kedua, fallback bekerja.

### 6.4 9Router vs OpenRouter

| | OpenRouter | 9Router |
|---|---|---|
| Jenis | Layanan cloud (pasar model) | Program lokal buatan komunitas, kamu yang menjalankan |
| Penyedia model | Menyalurkan banyak model dengan satu key | Tidak punya model; kamu menyambungkan penyedia/akunmu |
| Fallback | Parameter `models`, hanya antar model dalam OpenRouter | Lintas penyedia dan akun (langganan → murah → gratis) |
| Tambahan | Satu tagihan, model `:free` | Dasbor kuota, rotasi akun, penghemat token |
| Datamu | Lewat server OpenRouter | Lewat perangkatmu, lalu ke tiap penyedia |

Keduanya tidak bersaing: OpenRouter adalah penyedia, 9Router adalah pengatur lalu lintas. OpenRouter bisa ditambahkan sebagai salah satu penyedia di dalam 9Router.

### 6.5 Alternatif gateway

| Gateway | Catatan |
|---|---|
| LiteLLM | Sumber terbuka paling populer; API OpenAI-compatible untuk 100+ penyedia; gratis dijalankan sendiri; instalasi lebih berat di Termux |
| Bifrost | Sumber terbuka berbasis Go; failover bawaan; overhead sangat kecil |
| Portkey | Inti gateway sumber terbuka sejak Maret 2026; lapisan terkelola berbayar; diakuisisi Palo Alto Networks (2026) |
| Cloudflare AI Gateway | Gratis, caching dan analitik; routing lebih ringan |
| Helicone | Condong ke observabilitas dan pencatatan |
| Eden AI | Terkelola, 500+ model, fallback bawaan |
| freellmapi | Router sumber terbuka yang menggabungkan paket gratis banyak penyedia; kamu memakai key penyedia sendiri (lihat 6.7) |

### 6.6 Opsi lain bila ingin gateway sendiri

Skrip Python kecil berbasis `httpx` (tanpa library berat) yang menerima permintaan OpenAI-compatible, mencoba daftar model berurutan, dan berpindah saat 429/503/timeout. Cocok dengan lingkungan Termux. Perlu menentukan: urutan model/penyedia, dijalankan di HP atau PC.

### 6.7 freellmapi: router gratis lintas penyedia (self-hosted)

**Apa itu:** router sumber terbuka (MIT) yang dijalankan sendiri dan menggabungkan paket gratis dari banyak penyedia di balik **satu endpoint `/v1` kompatibel OpenAI**. README menyebut 34 penyedia dan 635 endpoint model gratis (Google, Groq, Cerebras, Mistral, OpenRouter, NVIDIA, Hugging Face, Cloudflare, Cohere, Zhipu, OpenCode Zen, dan lain-lain), plus penyedia kustom untuk endpoint OpenAI-compatible apa pun. Dalam empat lapisan: **gateway (lapisan 3)** di atas banyak penyedia (lapisan 4), sejenis 9Router tetapi berfokus pada paket gratis. Repositori: github.com/tashfeenahmed/freellmapi.

**Kenapa situs resminya tidak memberi API key:** ia **tidak menjalankan model dan tidak punya akun pengguna**. Kamu memasukkan key gratis dari **masing-masing penyedia asli** ke dasbor lokal, lalu router membuat satu "unified key" berawalan `freellmapi-…`. Itu hanya kunci lokal untuk masuk ke router-mu sendiri. Situs freellmapi.co hanya berisi katalog model, pemasang, dan layanan katalog opsional.

```
          Aplikasi: Cherry Studio / OpenCode / Cline / skrip
                                   │ Base URL: http://localhost:3001/v1
                                   │ API key : freellmapi-... (kunci lokal dari router-mu)
                                   ▼
        ┌──────────────────────────────────────────────────────┐
        │         FREELLMAPI  (berjalan di komputermu)         │
        │     routing · fallback otomatis · pelacak kuota      │
        │ key penyedia disimpan terenkripsi di database lokal  │
        └─────────┬────────────────┬────────────────┬──────────┘
                  │                │                │
                  ▼                ▼                ▼
          Google AI Studio       Groq       OpenRouter dst.
           (key Google-mu)  (key Groq-mu)  (key masing-masing)
```

**Perbedaan dengan model free OpenRouter:**

| | OpenRouter (model `:free`) | freellmapi |
|---|---|---|
| Jenis | Layanan cloud (agregator) | Router yang dijalankan sendiri |
| Akun dan key | Daftar ke OpenRouter, dapat key | Daftar ke **tiap penyedia** (OpenRouter pun bisa jadi salah satunya) |
| Kuota | Satu jatah gratis per akun OpenRouter | Gabungan jatah dari banyak penyedia sekaligus |
| Jalur data | Lewat server OpenRouter | Dari komputermu **langsung** ke penyedia yang diaktifkan |
| Pindah saat kuota habis | Terbatas di dalam OpenRouter | Lintas penyedia, otomatis |
| Biaya | Gratis (ada opsi top up) | Router gratis; katalog live opsional berbayar |
| Usaha setup | Rendah | Lebih tinggi (banyak key, pasang router) |

Angka "7,4 miliar token per bulan" di README adalah **jumlah seluruh paket gratis yang terdaftar**. Yang bisa dipakai hanya dari penyedia yang kamu beri key.

**Fitur penting:**
- **Routing cerdas** dengan enam strategi dan skor per model; fallback otomatis pada 429/5xx dengan jeda dan rotasi key; pelacak batas per `(platform, model, key)` (RPM/RPD/TPM/TPD).
- **Sticky session:** percakapan tetap di satu model sekitar 30 menit; catatan serah-terima konteks opsional bila model berganti di tengah.
- **Permukaan API:** `/v1/chat/completions`, `/v1/responses` (untuk Codex CLI), `/v1/completions` (ghost-text editor), embeddings, gambar, video, suara, dan `/v1/messages` (format Anthropic, untuk Claude Code). Tool calling lintas penyedia (panggilan alat berbentuk teks biasa dipulihkan menjadi `tool_calls`).
- **Fusion:** model virtual `fusion` mengirim prompt ke beberapa model gratis lalu model hakim menyintesis satu jawaban.
- **Key terenkripsi** (AES-256-GCM) di SQLite lokal; dasbor dengan analitik; server MCP; cache respons opsional; berjalan di Node 20+ (~40 MB RSS saat idle).

**Cara memakai:**
1. **Pasang:** Windows/macOS lewat pemasang desktop (`.exe`/`.dmg`) dari Releases GitHub (berjalan dari tray, tanpa akun atau kata sandi); Linux/server lewat Docker: `curl -fsSL https://freellmapi.co/install.sh | bash` (skripnya bisa dibaca dulu); Android lewat panduan Termux yang berlabel **eksperimental**.
2. **Ambil key gratis** dari penyedia yang diinginkan (Google AI Studio, Groq, Cerebras, Mistral, NVIDIA, OpenRouter, dan lain-lain).
3. **Buka dasbor** di `http://localhost:3001` (atau tray → Open Dashboard), halaman **Keys**, tambahkan key tiap penyedia. Atur urutan di **Fallback Chain**.
4. **Salin unified key** dari header halaman Keys (di aplikasi desktop, dari popover tray).
5. **Pakai di aplikasi:** Base URL `http://localhost:3001/v1`, API key = unified key, model `auto` (atau `auto:fast`, `auto:smart`, nama profil, atau ID model). Di Cherry Studio, tambahkan sebagai penyedia OpenAI-compatible. Header respons `X-Routed-Via` menunjukkan penyedia yang menjawab.
6. **Agen coding:** kebanyakan dikonfigurasi satu perintah, mis. `npx freellmapi setup-claude --url http://localhost:3001 --api-key <unified-key>`; Cline, OpenCode, Aider, Hermes Agent, dan lain-lain juga tercantum.
7. Key juga bisa dikelola dari terminal: `npx freellmapi keys add|list|remove|test <platform>` (butuh token dasbor).

**Katalog dan Premium (opsional):** router menarik katalog model bertanda tangan dari freellmapi.co dua kali sehari. Instalasi gratis memakai snapshot bulanan (sebuah model muncul 30 hari setelah masuk umpan live; README menyebut tertinggal sekitar 303 model). Katalog live (same-day) berbayar $19 per tahun atau $49 sekali bayar; routernya tetap MIT dan gratis. Menurut README, server katalog tidak melihat prompt, jawaban, atau key penyedia.

**Hal yang perlu diwaspadai:**
- **Kualitas dan konsistensi:** README mengakui tidak ada model frontier, latensi bervariasi, tanpa SLA, dan kecerdasan efektif menurun menjelang akhir hari saat model teratas mencapai batas harian (reset tengah malam UTC).
- **Privasi:** router bisa berpindah penyedia kapan saja, sehingga kamu tidak mengontrol penyedia mana yang melihat prompt. Banyak paket gratis memakai data untuk pelatihan: jangan kirim data sensitif.
- **Syarat penyedia:** deskripsi proyek berlabel "Personal experimentation only". Menumpuk banyak paket gratis atau banyak akun bisa melanggar ketentuan sebagian penyedia; bagian disclaimer belum dibaca. Pakai satu key sah per penyedia.
- **Koneksi keluar** untuk katalog; **jangan buka port ke internet**, buka akses LAN hanya bila perlu; baca skrip `curl | bash` sebelum menjalankan.
- Populer (41,7 ribu bintang) bukan jaminan aman; tetap perangkat lunak pihak ketiga.
- Uji tool calling sebelum dipakai untuk agen (lihat 5.4).

**Dibanding 9Router:** freellmapi menekankan key API resmi dari tiap penyedia gratis; 9Router juga menyediakan jalur OAuth/cookie/MITM yang lebih berisiko (lihat 6.2). Keduanya bisa diarahkan dari Cherry Studio, OpenCode, Cline, dan lain-lain lewat endpoint OpenAI-compatible. **Saran:** bila Gemini dan geraikita sudah cukup, OpenRouter atau skrip fallback sendiri memadai; freellmapi layak dicoba bila ingin memanfaatkan banyak paket gratis sekaligus, hanya untuk eksperimen pribadi.

---

## 7. Bot Telegram Multi-Model

### 7.1 Gagasan

Bot Telegram lama (python-telegram-bot + httpx, di Termux) dikembangkan menjadi **satu pintu untuk banyak penyedia**. File `bot_multi.py` (disediakan terpisah di percakapan) berisi:

- Registri **penyedia** (`PROVIDERS`: base URL + key) dan **model** (`MODELS`: nama model → penyedia). Menambah model baru cukup menambah satu baris.
- Perintah `/start`, `/model` (tombol inline berisi semua model, yang aktif diberi centang), `/reset` (hapus riwayat).
- Riwayat chat dibagi antar model (10 pesan terakhir per pengguna), sehingga model bisa diganti di tengah percakapan.
- Setiap jawaban diberi penanda model yang menjawab.
- `ALLOWED_IDS`: hanya ID Telegram tertentu yang bisa memakai bot, supaya kuota API tidak dipakai orang lain.
- Pesan panjang dipecah per 4.000 karakter (batas Telegram 4.096).
- Gemini dipanggil lewat endpoint OpenAI-compatible, jadi satu jalur kode untuk semua penyedia, tanpa library tambahan.

### 7.2 Cara memakai

1. Isi token Telegram, key geraikita, key Gemini, dan ID Telegram di bagian atas file.
2. Samakan `base_url` geraikita dengan yang dipakai bot lama (alamat sebelum `/chat/completions`).
3. Cek nama model Gemini lewat `/models` (lihat 2.2), ganti di daftar `MODELS` bila perlu.
4. Hentikan bot lama, jalankan `python bot_multi.py` (dua bot dengan token yang sama akan bentrok).
5. Bila error, bot menampilkan kode error di chat.

### 7.3 Keterbatasan dan pengembangan

- Hanya teks (tanpa upload gambar/file, tanpa jawaban bertahap seperti gemini.google.com). Dukungan gambar untuk model yang mendukung (mis. Gemini) bisa ditambahkan.
- Riwayat disimpan di memori bot dan **terpisah** dari Cherry Studio. Menyatukannya butuh penyimpanan bersama yang diprogram sendiri.
- Fallback antar model dan pencarian otomatis bisa ditambahkan (lihat 6.1 dan 4.3).

### 7.4 Telegram dan Cherry Studio (diluruskan)

> **Koreksi.** Versi sebelumnya menyatakan Cherry Studio tidak punya bot Telegram bawaan. Itu salah atau usang: ada fitur **Channels** (lihat 3.9). Yang tetap berlaku: Cherry Studio adalah aplikasi desktop, jadi perangkat yang menjalankannya harus menyala dan aplikasinya harus berjalan.

Pilihan untuk "satu riwayat, bisa dipakai tanpa membuka jendela Cherry Studio":

1. **Cherry Studio Channels** (Agent ↔ Telegram): memakai penyedia dan model yang sudah diatur di Cherry Studio; butuh API Gateway menyala dan aplikasi berjalan. Sinkronisasi riwayat dengan sesi agen belum diverifikasi.
2. **Hermes Agent sebagai pusat** (riwayat dan memori Hermes di satu tempat): lebih berat dan lebih boros kuota untuk sekadar mengobrol.
3. **Bot Telegram sendiri (`bot_multi.py`) + Cherry Studio ke gateway yang sama**, tanpa riwayat bersama.
4. **Aplikasi mobile dan koneksi perangkat Cherry Studio** (sinkronisasi riwayat belum diverifikasi; PC tampaknya tetap harus menyala).
5. **Klien chat berbasis web di server** (Open WebUI, LibreChat, LobeChat): riwayat di server, bisa dibuka dari mana saja, tanpa Telegram; butuh komputer/VPS yang menyala terus.

Pilih berdasarkan prioritas: Telegram dengan fitur siap pakai (Channels), Telegram dengan kendali penuh (Hermes atau bot sendiri), riwayat bersama di semua perangkat (web berbasis server), atau cukup model yang sama di mana saja (gateway bersama).

### 7.5 Memakai Claude API di bot Telegram

Key dari Claude Console (lihat 2.9) bisa dipakai bot Python-mu. Dua cara memanggilnya:

1. **Messages API native (disarankan).** Header `x-api-key` dan `anthropic-version`, `max_tokens` wajib, pesan sistem dikirim terpisah dari daftar pesan. Cukup `httpx`:
   ```python
   async def ask_claude(model, system, messages):
       async with httpx.AsyncClient(timeout=120) as client:
           r = await client.post(
               "https://api.anthropic.com/v1/messages",
               headers={"x-api-key": ANTHROPIC_KEY,
                        "anthropic-version": "2023-06-01",
                        "content-type": "application/json"},
               json={"model": model, "max_tokens": 1024,
                     "system": system, "messages": messages},
           )
           r.raise_for_status()
           return r.json()["content"][0]["text"]
   ```
   Model contoh: `claude-haiku-5-5` (paling hemat).
2. **Lapisan kompatibel OpenAI:** ganti `base_url` ke `https://api.anthropic.com/v1/` dan model ke nama Claude, sehingga kode `bot_multi.py` hampir tidak berubah. Anthropic menyebutnya untuk menguji dan membandingkan, bukan solusi jangka panjang atau siap produksi; untuk fitur penuh pakai API native.

Cara ketiga tanpa mengubah kode bot: tambahkan Anthropic sebagai penyedia di 9Router atau LiteLLM, lalu bot memanggil gateway itu.

**Catatan:** biaya per token dari saldo Console, jadi tetap isi `ALLOWED_IDS`. Buat workspace dan key khusus bot dengan batas pengeluaran. Simpan key di variabel lingkungan.

### 7.6 WhatsApp: pembatasan resmi

- Sejak 15 Januari 2026, Meta melarang chatbot AI serba guna di WhatsApp Business Platform (penyedia AI yang menawarkan asisten umum sebagai layanan utama). Bot dengan fungsi bisnis tertentu (dukungan, penjualan, notifikasi) masih boleh.
- Kebijakannya ditulis cukup luas sehingga Meta bebas menentukan apa yang termasuk "AI serba guna"; posisi bot AI pribadi tidak jelas.
- Jalur tidak resmi (pustaka yang meniru WhatsApp Web) biasanya melanggar ketentuan dan berisiko nomor diblokir (belum diverifikasi lewat pencarian).
- Telegram tidak punya pembatasan serupa yang diketahui, jadi pilihan paling aman untuk bot pribadi.

---

## 8. Pengaturan Model: Reasoning Effort vs Service Tier

Dua pengaturan berbeda yang sering membingungkan:

### 8.1 Low / Medium / High: kedalaman berpikir (reasoning effort)

- Menentukan seberapa banyak model "berpikir" sebelum menjawab. Tingkat umum: minimal, low, medium, high, xhigh. Rekomendasi umum: low untuk tugas sederhana, medium sebagai default, high untuk tugas kompleks (matematika, logika, debugging rumit, arsitektur).
- Di Gemini maknanya sama; label di Cherry Studio bisa berbeda (mis. panjang rantai pikiran).
- **Semakin tinggi, semakin boros token dan lebih lambat.** Token berpikir dihitung sebagai token keluaran dan memengaruhi tagihan/kuota token. Belum tentu lebih baik untuk pertanyaan ringan (model bisa berpikir berlebihan).
- Tidak semua model mendukung; pada model non-penalaran, pengaturan bisa diabaikan. Apakah model di geraikita merespons pengaturan ini **(belum diverifikasi)**.

### 8.2 Standard / Fast / Flex: tier layanan (service tier)

- Parameter `service_tier` di OpenRouter yang mengatur kompromi biaya dan kecepatan, **bukan** kedalaman berpikir.
- **Flex:** biaya lebih rendah, latensi lebih tinggi (di OpenAI, sekitar 50% dari harga standar dengan imbalan lebih lambat dan sesekali sumber daya tidak tersedia, jadi bisa gagal).
- **Fast (= priority):** lebih cepat, lebih mahal per token (pengali tergantung model).
- **Standard/default:** perlakuan normal.
- Hanya untuk penyedia/model tertentu (mis. OpenAI, Google Vertex, Google AI Studio). Untuk model lain biasanya tidak berpengaruh.
- Tier tidak mengubah **jumlah** token, hanya harga per token dan kecepatannya.

### 8.3 Ringkasan

| Pengaturan | Kualitas jawaban | Kecepatan | Jumlah token | Harga per token |
|---|---|---|---|---|
| Effort **low → high** | Lebih dalam di tugas sulit | Lebih lambat | Naik (token berpikir) | Sama |
| Tier **flex** | Sama | Lebih lambat | Sama | Lebih murah |
| Tier **fast** | Sama | Lebih cepat | Sama | Lebih mahal |

### 8.4 Saran untuk pemakai kuota gratis

- Biarkan tier di **standard/default** (model `:free` umumnya tidak mendukung tier).
- Effort **low/medium** untuk chat harian; **high** hanya untuk soal sulit.
- Agen/coding dengan effort tinggi paling cepat menguras kuota; mulai dari medium.

---

## 9. AI untuk Coding dan Vibe Coding

Ada tiga jalur untuk memakai AI di proyek web dev tanpa meninggalkan VS Code biasa:

### 9.1 Ekstensi di VS Code (disarankan)

Terpasang dari Marketplace dan jalan di VS Code biasa; pengaturan dan pintasan tetap.

| Ekstensi | Catatan |
|---|---|
| **Cline** | Sumber terbuka, gratis, BYOK (bayar hanya pemakaian model). Agen: merencanakan, mengedit banyak file, menjalankan perintah terminal dengan persetujuanmu. Tidak punya autocomplete. Menerima backend OpenAI-compatible, jadi geraikita, Gemini, OpenRouter, atau 9Router bisa dipakai. |
| **Kilo Code** | Sumber terbuka, BYOK; mode orkestrator dan penyelesaian kode sebaris |
| **GitHub Copilot** | Paket gratis bisa dipakai mencoba mode agen; tidak memakai key sendiri |

Peringatan: satu sumber menyebut Roo Code, Continue, Amazon Q, dan Gemini Code Assist (individu) dihentikan pada 2026, sementara sumber lain tetap membahas Continue. Cek status terbaru.

### 9.2 IDE khusus AI (fork VS Code)

Cursor dan Windsurf adalah aplikasi terpisah, bukan ekstensi: pengaturan, ekstensi, dan pintasan harus dipindahkan. Berlangganan (Cursor Pro sekitar $20/bulan, Windsurf sekitar $15/bulan) dan modelnya lebih terikat ke layanan mereka. Lebih mulus, tetapi kurang cocok bila tujuannya memakai kuota gratis dan model dari gateway sendiri.

### 9.3 Agen terminal

Claude Code berjalan sebagai ekstensi VS Code dan tercatat paling banyak dipasang. Secara bawaan memakai akun/kredit Anthropic, jadi bukan jalur gratis.

**Koreksi:** ia **bisa diarahkan ke model lain lewat gateway** (misalnya variabel lingkungan ke endpoint OpenRouter yang kompatibel dengan format Anthropic, atau lewat 9Router yang mencantumkan Claude Code sebagai alat yang didukung). Namun itu bukan jalur yang dirancang Anthropic, dan hasil dengan model non-Claude bisa bervariasi.

Dokumentasi resmi: https://docs.claude.com/en/docs/claude-code/overview

Kategori lain: pembuat aplikasi berbasis web yang menghasilkan situs dari deskripsi tanpa editor kode (cocok untuk prototipe cepat, kendali atas kode lebih sedikit).

### 9.4 Langkah awal

1. Pasang Cline atau Kilo Code dari Marketplace VS Code.
2. Di pengaturan ekstensi, pilih penyedia OpenAI-compatible atau Google Gemini, isi key dan nama model. Mulai dengan Gemini (konteks panjang, kuota gratis).
3. Mulai dari tugas kecil sebelum menyerahkan proyek besar.

### 9.5 Catatan penting

- **Kuota gratis cepat terkuras** (banyak file, banyak langkah). Model gratis/ringan lebih sering salah mengedit. Di sini gateway fallback berguna.
- **Pakai Git:** commit sebelum tiap sesi dengan AI agar bisa dibatalkan.
- **Jaga key:** agen membaca file proyek. Simpan API key di `.env` yang masuk `.gitignore`, bukan di kode.
- **Baca perubahan dan perintah terminal** sebelum menyetujui, terutama perintah yang menghapus atau memasang paket.

### 9.6 Ekstensi BYOK dan pindah model otomatis saat kuota habis

**BYOK di VS Code:** Cline dan Kilo Code (sumber terbuka, memakai key sendiri). Roo Code dan Continue juga BYOK, tetapi statusnya simpang siur (lihat 9.1). Di terminal: Aider, OpenCode, Goose.

Tiga tingkat pindah model otomatis saat kuota habis:

| Cara | Di mana fallback diatur | Catatan |
|---|---|---|
| **Kilo Code: Virtual Quota Fallback** | Di dalam ekstensi | Paling mudah, tanpa program tambahan |
| **OpenRouter sebagai penyedia** | Di sisi OpenRouter | Terbatas pada model di OpenRouter |
| **Gateway lokal** (9Router dan sejenisnya) | Di luar ekstensi | Fleksibel, satu aturan untuk semua alat |

**1. Kilo Code, "Virtual Quota Fallback".** Penyedia "meta" yang tidak terhubung ke layanan LLM langsung, melainkan **mengelola profil penyedia lain** yang sudah kamu atur.
- Batas per penyedia bisa diisi berdasarkan jumlah token atau permintaan per menit, jam, atau hari. Saat batas terlewati atau API mengembalikan error, penyedia itu dinonaktifkan sementara dan sistem pindah ke penyedia berikutnya di daftar.
- Notifikasi muncul di VS Code setiap kali perpindahan terjadi.
- Bila batas tidak diisi, pindah hanya terjadi saat API mengembalikan error (rate limit, masalah jaringan, key tidak valid).
- Syarat: sudah ada minimal satu profil penyedia lain; profil fallback tidak bisa disarangkan di dalam profil fallback lain.
- Dokumentasinya ada di dua situs Kilo dan menu bisa berbeda antar versi. **Pastikan fiturnya ada di versi yang kamu pasang.**

Cara setelnya (kira-kira):
1. Buat satu profil penyedia untuk tiap sumber: Gemini, geraikita, OpenRouter bila ada.
2. Di dropdown penyedia, pilih "Virtual Quota Fallback", klik **Add Profile**, lalu urutkan (misalnya Gemini → geraikita → OpenRouter `:free`).
3. Isi batas sesuai kuota gratis tiap penyedia (token per jam atau permintaan per menit) agar pindah terjadi sebelum error; kosongkan bila ingin pindah hanya saat error.

**2. OpenRouter sebagai penyedia.** Cline dan Kilo Code bisa memakai OpenRouter lewat pengaturan penyedia, dan kegagalan di lapisan routing tidak terlihat oleh agen. Ada dua jenis: failover otomatis antarpenyedia untuk model yang sama (tanpa konfigurasi) dan fallback berurutan antarmodel. Catatan: **percobaan yang gagal tetap dihitung ke kuota harian**, dan model gratis populer bisa terkena batas di jam sibuk.

**3. Gateway lokal.** 9Router mengklaim bekerja dengan Claude Code, Codex, Cursor, Cline, Roo, Continue, dan Kilo Code, cukup dengan menunjuk ke `http://localhost:20128/v1`. Keunggulannya: aturan fallback yang sama bisa dipakai bersama Cherry Studio dan Hermes. Ada juga proyek sejenis yang berfokus pada kuota gratis (melacak batas per key, mencoba model berikutnya saat 429/5xx, dan menahan percakapan di satu model sekitar 30 menit untuk menjaga koherensi). Proyek-proyek ini baru: cek repositorinya dulu.

**Cline:** tidak ditemukan fitur fallback bawaan yang setara dengan Kilo Code. Jalurnya lewat OpenRouter atau gateway.

**Supaya vibe coding tetap lancar saat memakai fallback:**
- **Setiap model di rantai harus lolos tes tool calling dan punya konteks panjang** (lihat 5.3 dan 5.4). Model lemah di rantai bisa merusak edit file.
- **Pindah model di tengah tugas bisa membuat konteks tidak konsisten.** Selesaikan tugas kecil satu per satu dan **commit Git** sebelum mulai.
- **Fallback tidak membuat kuota tak terbatas.** Kalau semua penyedia gratis habis, kamu tetap berhenti. Siapkan satu cadangan murah berbayar di akhir rantai bila ingin benar-benar lancar.
- Gemini cocok di urutan pertama (konteks panjang, kuota gratis).

### 9.7 Stack vibe coding yang populer

Rekomendasi yang paling sering muncul adalah **stack per tahap**, bukan satu aplikasi tunggal: satu alat untuk tiap tahap (desain, kode, backend, deploy). Peringkat berasal dari blog dan artikel perbandingan, bukan survei resmi; harga bisa berubah.

| Tahap | Pilihan populer | Catatan |
|---|---|---|
| **Kode (pembuat aplikasi)** | Lovable, Bolt.new, Replit, v0 | Untuk non-teknis. Lovable disebut pilihan bawaan pendiri non-teknis (backend Supabase); Bolt untuk build instan di browser; Replit untuk membangun sekaligus hosting. Harga mulai sekitar $20 sampai $25/bulan; beberapa punya paket gratis. |
| **Kode (editor/agen)** | Cursor, Claude Code, Windsurf, GitHub Copilot, VS Code + Cline/Kilo Code | Untuk yang mau mengatur kode sendiri. Cursor bila akan menyentuh kode asli; Claude Code untuk agen paling otonom. |
| **Backend** | **Supabase** | Pilihan bawaan yang jelas: database PostgreSQL, autentikasi, penyimpanan. Lovable dan Bolt menyambungkannya otomatis. |
| **Deploy** | Vercel, Netlify, Cloudflare | Bolt tidak menyediakan hosting, jadi deploy ke platform ini. |
| **Kontrol versi** | GitHub | Wajib. |
| **UI** | v0 (Vercel) | Sering dipasangkan sebagai lapisan UI. |

Jalur belajar yang sering disarankan: Replit → Lovable/Bolt → Cursor → pengembangan tradisional. Satu rangkuman tren menyebut ada "stack $0": Claude + Supabase + Google Antigravity (detail Antigravity: lihat 9.8).

**Tiga paket yang masuk akal:**
1. **Pemula/non-teknis:** Lovable atau Replit + Supabase (otomatis) + GitHub. Paling cepat, tetapi kodenya kurang bisa diatur dan kreditnya boros bila agen mengulang-ulang.
2. **Mau mengatur kode sendiri:** Cursor, atau VS Code + Cline/Kilo Code, + GitHub + Supabase + Vercel/Netlify/Cloudflare.
3. **Gratis dan BYOK:** VS Code + Kilo Code (dengan Virtual Quota Fallback) atau Cline + Gemini gratis sebagai model utama + gateway fallback (geraikita/OpenRouter sebagai cadangan) + Supabase + GitHub + hosting gratis (umumnya ada paket gratis, cek batasnya).

**Alur kerja yang sehat:**
1. Tulis ringkasan proyek dan aturan pendek (misalnya di `README`/file aturan): tujuan, framework, struktur folder. Aturan proyek tidak selalu dipatuhi di percakapan panjang, jadi ulangi bila perlu.
2. `git init`, lalu commit sebelum dan sesudah tiap fitur.
3. Kerjakan satu fitur kecil per sesi, uji di browser, baru lanjut.
4. Minta AI menjelaskan perubahan sebelum kamu menyetujui.
5. Simpan key di `.env` yang masuk `.gitignore` (di Bolt, `.env` tidak dienkripsi dan key terlihat di daftar file).
6. Bila memakai Supabase, aktifkan **Row Level Security (RLS)** di tabelmu; tanpa itu data bisa terbuka lewat API publik. Cek dokumentasi Supabase.
7. Pilih alat yang kodenya bisa diekspor; alat tanpa ekspor kode sering berarti harus membangun ulang proyek.

**Peringatan umum:** kebanyakan alat memakai kredit, dan build kompleks atau putaran agen bisa menguras anggaran dengan cepat. Tugas kecil dan fallback model membantu.

### 9.8 Google Antigravity

- **Apa:** IDE berbasis agen (agent-first IDE) dari Google. Diumumkan 18 November 2025 bersamaan dengan Gemini 3, langsung tersedia sebagai pratinjau publik gratis. Agen AI merencanakan, menulis, menjalankan, dan menguji kode di editor, terminal, dan browser bawaan.
- **Struktur:** dua tampilan: **editor** (IDE biasa dengan penyelesaian kode dan agen di panel samping) dan **manajer (Agent Manager)** untuk menjalankan beberapa agen sekaligus. Agen menghasilkan **Artifacts** (rencana, tangkapan layar, rekaman) agar pekerjaannya bisa diverifikasi.
- **Dasar:** disebut sebagai fork VS Code, sehingga ekstensi dan pintasan bisa dipindahkan.
- **Model:** Gemini, plus model dari Anthropic dan OpenAI. Tidak ditemukan keterangan BYOK; akses model tampaknya lewat kuota akun Google.
- **Biaya:** gratis dalam batas pemakaian, batas lebih tinggi untuk pelanggan paket AI Google berbayar. Desember 2025 pengguna gratis dipindah ke batas mingguan; Maret 2026 dilaporkan ada protes atas penurunan kuota dan kenaikan harga. Cek batas terbaru di antigravity.google.
- **Perkembangan 2026:** Antigravity 2.0 menjadi pengganti Agent Manager yang berfungsi independen dari IDE; ada Antigravity CLI (menggantikan Gemini CLI untuk konsumen) dan SDK. Editor aslinya tetap sebagai produk terpisah. Per Oktober 2026 IDE-nya belum didukung untuk pelanggan enterprise.
- **Keamanan:** agen bisa diatur seberapa banyak bekerja tanpa pengawasan. Dalam mode "Turbo", agen menjalankan perintah terminal tanpa meminta persetujuan. Saran: jalankan di workspace tanpa hak istimewa, batasi akses agen ke folder proyek, commit sering, tetap tinjau kode, hindari Turbo sebelum nyaman dengan perilaku agen.

**Label:** agen AI khusus coding yang dibungkus IDE (bukan klien chat umum). Dalam empat lapisan: lapisan 1 dan 2 sekaligus; model dari Google, Anthropic, dan OpenAI. Kategorinya sama dengan Cursor dan Windsurf.

```
Antigravity = [IDE (fork VS Code) + agen coding + Agent Manager + browser/terminal terintegrasi]
              model: Gemini, plus Claude/OpenAI tertentu; kuota lewat akun Google
```

### 9.9 Google Jules

- **Apa:** agen coding **asinkron berbasis cloud**. Kamu memberi tugas; Jules menyalin repositori ke VM Google Cloud yang aman, menulis rencana, menjalankan perubahan di banyak file, lalu **membuka pull request** untuk kamu tinjau.
- **Cara kerja:** diakses lewat jules.google tanpa pemasangan lokal; ada Jules Tools (CLI dan TUI). Tugas bisa berasal dari issue GitHub (beri label `jules`). Rencana bisa diedit sebelum dieksekusi. Fitur 2026: bila CI gagal pada PR yang dibuatnya, Jules menganalisis error, memperbaiki, dan mendorong ulang commit.
- **Model:** keluarga Gemini; paket gratis berjalan di Gemini 3 Flash. Tidak ada indikasi BYOK.
- **Batasan akun:** per Juni 2026 hanya akun Gmail pribadi, bukan Google Workspace enterprise.
- **Biaya (indeks pihak ketiga, Juli 2026):** gratis 15 tugas/hari; Pro $19,99/bulan untuk 100 tugas; Ultra $124,99/bulan untuk 300 tugas. Pada peluncuran paket gratis juga dibatasi tiga tugas bersamaan. Cek jules.google untuk angka terbaru.

**Label:** agen AI khusus coding berbasis cloud. Secara arsitektur paling mirip Muse (bekerja di VM cloud; kamu bisa menutup komputer lalu kembali ke hasilnya), tetapi domainnya kode dan repositori GitHub.

**Cocok untukmu atau tidak:**
- Bukan alat vibe coding interaktif: eksekusinya hanya asinkron, jadi iterasi cepat kurang nyaman. Cocok untuk tugas jelas dan terukur (memperbaiki bug, menambah tes, fitur kecil dari satu issue).
- Kuota gratisnya terpisah dari API key-mu, jadi bisa jadi pelengkap gratis untuk VS Code + Kilo Code.
- Kode disalin ke VM Google: cek kebijakan data sebelum memakainya untuk proyek sensitif.
- Butuh repositori GitHub.

### 9.10 Perbandingan alat coding dan labelnya

| Alat | Jenis | Cara kerja | Tempat kerja | Model | BYOK |
|---|---|---|---|---|---|
| Cline / Kilo Code | Ekstensi VS Code (agen) | Interaktif, dengan persetujuan | Komputermu | Bebas | Ya |
| Cursor / Windsurf | IDE berbasis agen (fork) | Interaktif | Komputermu | Terutama lewat layanan mereka | Terbatas |
| **Antigravity** | IDE berbasis agen + Agent Manager | Interaktif/terawasi, multi-agen | Komputermu | Gemini + Claude/OpenAI tertentu | Tidak ditemukan |
| **Jules** | Agen coding asinkron | Delegasi, hasil berupa PR | VM Google Cloud | Gemini | Tidak |
| Claude Code | Agen terminal/ekstensi | Interaktif | Komputermu | Claude (bisa dialihkan lewat gateway) | Lewat gateway |

Antigravity dan Jules sama-sama dari Google tetapi berbeda: Antigravity bekerja terawasi di mesinmu, Jules bekerja sendiri di cloud dan mengirim hasil sebagai PR. Keduanya bukan klien chat umum seperti Cherry Studio.

### 9.11 Vibe coding di VS Code memakai AI dari Cherry Studio

Bisa, lewat dua jalur. Keduanya memakai penyedia dan key yang sudah diatur di Cherry Studio.

**Jalur A: Code Mate (berbasis terminal; lihat 3.7)**
1. Buka **Code Mate** dari Launchpad Cherry Studio, pilih alat (mis. OpenCode atau Qwen Code), lalu **Install**.
2. Pilih penyedia dan model, bisa lewat **Unified Gateway**.
3. Pilih folder proyek dan terminal, lalu **Launch**.
4. Mulai dengan perintah baca-saja untuk memastikan akun, model, dan direktori benar.
Alat ini berjalan di terminal. Menjalankan CLI yang sama di terminal terintegrasi VS Code pada folder proyek yang sama kemungkinan bisa, tetapi belum diperiksa.

**Jalur B: ekstensi di dalam VS Code + API Gateway (lihat 3.6)**
1. Pastikan model bisa mengobrol normal di Settings → Model Services.
2. **Settings → API Gateway → Start** sampai **Running**; salin URL dan API key.
3. Uji: minta daftar model, lalu kirim teks pendek (dokumentasi API ada di halaman gateway; path dan format nama model dicocokkan di sana).
4. Di Cline atau Kilo Code pilih penyedia **OpenAI Compatible**: Base URL = alamat gateway (biasanya berakhiran `/v1`), API key = key gateway, Model ID = persis seperti di daftar model. Nama kolom bisa berbeda antar versi.
5. Mulai dari tugas kecil dan **commit Git** sebelum agen mengubah file.

**Perlu gateway Cherry Studio atau tidak?**
- Untuk vibe coding saja, Cline atau Kilo Code bisa langsung disambungkan ke Gemini atau geraikita dengan key masing-masing. Itu lebih sederhana dan tidak bergantung pada Cherry Studio yang harus tetap berjalan.
- Gateway Cherry Studio berguna bila ingin **menyimpan key di satu tempat** atau memakai Code Mate.
- Secara teori beberapa profil Kilo Code bisa mengarah ke gateway Cherry Studio dengan model berbeda, lalu memakai Virtual Quota Fallback untuk pindah otomatis. Belum diuji.

**Syarat dan catatan:**
- Cherry Studio harus berjalan dengan gateway **Running**; setelah dihentikan, klien tidak bisa memanggil.
- Biaya dan kuota tetap di penyedia aslinya. Tidak ada fallback otomatis di gateway Cherry Studio, jadi saat kuota habis muncul error.
- Model harus mendukung tool calling dan konteks panjang (uji dengan skrip di 5.4; skrip yang sama bisa diarahkan ke alamat gateway). Mulai dengan Gemini.
- Soal rantai yang lebih panjang lewat 9Router, lihat 11.3.

### 9.12 Codex (OpenAI)

**Bukan IDE seperti VS Code.** Codex adalah **agen coding** dari OpenAI, sekategori Hermes tetapi khusus pemrograman. Yang membingungkan: satu agen punya banyak antarmuka.

```
                      ┌─ CLI (terminal, sumber terbuka)
                      ├─ Ekstensi IDE (VS Code, JetBrains, dll.)
Agen Codex (OpenAI) ──┼─ Aplikasi desktop (macOS / Windows)
                      ├─ Codex Cloud (di ChatGPT, berjalan di sandbox)
                      └─ Aplikasi mobile (memantau/mengontrol tugas)
```

- Hubungan dengan VS Code: **VS Code adalah tempat kerja, Codex agen yang dipasang di dalamnya**, seperti Cline atau Kilo Code. Ekstensinya punya tiga mode persetujuan: Chat (hanya baca), Agent (bawaan), dan Agent Full Access.
- Antarmuka berbagi satu akun, satu file konfigurasi, dan satu set batas pemakaian. Mode cloud menjalankan tugas di lingkungan cloud terisolasi, beberapa tugas sekaligus (mirip Jules).
- **Model** (per Juni 2026): GPT-5.5 untuk sesi lokal, GPT-5.3-Codex untuk cloud dan code review. **MCP** lokal (STDIO) dan jarak jauh (HTTP) diatur di `~/.codex/config.toml`.
- **Biaya:** tidak ada langganan Codex terpisah; ikut paket ChatGPT. Satu sumber (Juli 2026) menyebut Free (terbatas), Go $8, Plus $20, Pro $100 atau $200 per bulan, tetapi sumber lain menyebut skema tagihan berubah per April dan ada yang menyebut tidak ada paket gratis. Cek halaman harga ChatGPT. Dengan API key OpenAI, menurut satu panduan fitur cloud (delegasi cloud, code review di GitHub) tidak tersedia.
- **BYOK:** Codex tercantum sebagai alat yang didukung 9Router dan Code Mate Cherry Studio, jadi kemungkinan bisa diarahkan ke model lain. Belum diverifikasi; hasil dengan model non-OpenAI bisa bervariasi.

**Pemetaan empat lapisan:** Codex mencakup lapisan 1, 2, dan 4 sekaligus (modelnya milik OpenAI, terhubung bawaan); lapisan 3 kosong secara bawaan.

```
Lapisan 1  Klien      CLI / ekstensi VS Code / aplikasi desktop / ChatGPT
              │
Lapisan 2  Agen       Codex (merencanakan, mengedit file, menjalankan perintah)
              │
Lapisan 3  Gateway    tidak ada bawaan (opsional: 9Router, Code Mate Cherry Studio)
              │
Lapisan 4  Penyedia   OpenAI (model GPT), lewat login ChatGPT atau API key
```

### 9.13 GitHub Copilot

"Copilot" adalah merek yang dipakai banyak produk (satu analisis menemukan 75 produk berbrand Copilot). Yang dibahas di sini adalah **GitHub Copilot** untuk coding; Microsoft Copilot (chat umum) dibahas di 10.10.

- **Bukan IDE:** ia ekstensi untuk VS Code, Visual Studio, JetBrains, Neovim, dan lain-lain.
- **Bukan sekadar chat:** ia tumpukan fitur: penyelesaian kode sebaris, Copilot Chat, agent mode, agen coding cloud asinkron, code review PR, dan Copilot CLI.

```
VS Code (IDE, tempat kerja)
 └─ Ekstensi GitHub Copilot
     ├─ Penyelesaian kode ─ saran teks abu-abu saat mengetik (tekan Tab)
     ├─ Chat (Ask) ──────── tanya jawab, snippet, boilerplate, jelaskan kode
     └─ Agent mode ──────── tugas multilangkah, edit banyak file

Di luar editor:
 ├─ Copilot CLI ─────────── agen di terminal
 ├─ Cloud agent ─────────── mengerjakan issue di GitHub lalu membuka PR (asinkron, mirip Jules)
 └─ Code review ─────────── meninjau pull request
```

- **Model:** multi-model (OpenAI, Anthropic, Google) dipilih lewat GitHub; sebagian model dibatasi ke paket tertentu.
- **Paket (Juni sampai Oktober 2026):** Free memberi 2.000 penyelesaian kode per bulan plus chat dan agen terbatas; Pro $10, Pro+ $39, Max $100 per bulan; Business $19 dan Enterprise $39 per pengguna. Tagihan pindah ke berbasis pemakaian pada Juni 2026 (GitHub AI Credits). Satu sumber menyebut pendaftaran mandiri Pro, Pro+, dan pelajar yang baru dijeda sejak 20 April 2026, sementara sumber lain tetap menyebut Pro $10 tersedia: cek halaman paket GitHub.
- **BYOK:** Copilot SDK (untuk membangun aplikasi sendiri) mendukung key dari OpenAI, Anthropic, Azure AI Foundry, AWS Bedrock, Google AI Studio, atau penyedia OpenAI-compatible, tetapi itu bukan Copilot di editor. Untuk chat di VS Code belum diverifikasi; penyelesaian kode otomatis kemungkinan tetap memakai model GitHub.
- **Lapisan:** 1 (ekstensi dan CLI) + 2 (agent mode dan cloud agent) + 4 (model lewat GitHub), tanpa gateway bawaan.
- **Cocok untukmu:** paket Free bisa jadi pelengkap gratis berupa penyelesaian kode di VS Code, sementara tugas agen memakai Kilo Code atau Cline dengan key sendiri. Kuotanya terbatas dan syarat paketnya sering berubah.

### 9.14 OpenCode

**Utamanya agen coding sumber terbuka (MIT), model-agnostic**, sejenis Hermes tetapi untuk pemrograman. Bisa dipakai lewat TUI terminal, aplikasi desktop, ekstensi VS Code, dan web. Lewat registri Models.dev ia terhubung ke lebih dari 75 penyedia (BYOK), dengan arsitektur client-server (tampilan dan mesin agen terpisah).

**Sisi mirip penyedia (opsional, tidak wajib):**
- **Zen:** gateway bayar-per-pemakaian dengan daftar model terpilih, dari sekitar $0,45 per 1 juta token masukan (model hemat) sampai $10 (Opus).
- **Go:** langganan $5 untuk bulan pertama lalu $10 per bulan, untuk model open-weight pilihan; menurut satu sumber bisa dipakai agen coding lain, tidak hanya OpenCode.
- **Gratis:** satu sumber (panduan pribadi di GitHub) menyebut 200 permintaan per hari lewat model tertentu; cek situs resmi.

```
OpenCode (agen coding, sumber terbuka)
 ├─ Antarmuka: TUI terminal / aplikasi desktop / ekstensi VS Code / web
 └─ Sumber model (pilih salah satu atau campur):
     ├─ BYOK  ── Gemini, geraikita (OpenAI-compatible), Groq, model lokal, dll.
     ├─ Zen   ── gateway berbayar per pemakaian (opsional)
     └─ Go    ── langganan murah model open-weight (opsional)
```

**Peran tiap lapisan:**

```
LAPISAN 1  Antarmuka     TUI terminal │ aplikasi desktop │ ekstensi VS Code │ web
              │
LAPISAN 2  Agen          OpenCode: merencanakan, mengedit file, menjalankan perintah
              │          └── alat tambahan: MCP, skill, perintah kustom, plugin
              │
LAPISAN 3  Gateway       (opsional) 9Router │ LiteLLM │ Cherry API Gateway │ Zen
              │
LAPISAN 4  Penyedia      Gemini │ geraikita │ OpenRouter │ Groq │ Go │ model lokal
```

- OpenCode sendiri mengisi lapisan 1 dan 2. Ia **bukan gateway**; Zen adalah layanan milik tim OpenCode yang berfungsi seperti gateway, dan Go adalah langganan di lapisan penyedia.
- **"Agen murni"** = memakai lapisan 1 dan 2 dari OpenCode, lalu menyediakan lapisan 4 sendiri dengan key milikmu, tanpa Zen dan Go. Lapisan 3 boleh kosong.

**Bisa terhubung dengan apa saja:**
- *Penyedia (lapisan 4):* OpenAI, Anthropic (API key), Google Gemini, AWS Bedrock, Azure OpenAI, Groq, DeepSeek; model lokal (Ollama, LM Studio, llama.cpp); endpoint OpenAI-compatible apa pun (geraikita, OpenRouter, Venice); Zen dan Go; akun GitHub Copilot atau langganan ChatGPT. Hati-hati dengan akun langganan konsumen: OpenCode berhenti mendukung login Claude Pro/Max setelah Anthropic memblokir jalur tidak resmi.
- *Gateway (lapisan 3):* `baseURL` kustom berguna untuk proxy, jadi bisa diarahkan ke 9Router, LiteLLM, atau API Gateway Cherry Studio. OpenCode juga tercantum di Code Mate Cherry Studio.
- *Alat tambahan (lapisan 2):* MCP lokal dan jarak jauh (mis. context7 untuk dokumentasi library terbaru, chrome-devtools untuk otomasi browser), skill (`SKILL.md`), perintah kustom, plugin, dan SDK dari spesifikasi OpenAPI untuk skrip atau klien kustom.
- *Antarmuka:* editor yang kompatibel ACP dan alur kerja GitHub.

**Contoh konfigurasi geraikita** (`~/.config/opencode/opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "geraikita/qwen3.8-flash",
  "provider": {
    "geraikita": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "geraikita",
      "options": {
        "baseURL": "https://ai.geraikita.com/v1",
        "apiKey": "{env:GERAIKITA_KEY}"
      },
      "models": {
        "qwen3.8-flash": { "name": "Qwen 3.8 Flash" },
        "deepseek-v4-flash": { "name": "DeepSeek V4 Flash" }
      }
    }
  }
}
```

Catatan: `npm` diisi `@ai-sdk/openai-compatible` untuk endpoint `/v1/chat/completions`, baseURL sebaiknya berakhiran `/v1`, key dibaca dari variabel lingkungan. Pola sama untuk Gemini dengan base URL `https://generativelanguage.googleapis.com/v1beta/openai`. Nama kunci konfigurasi bisa berbeda antar versi (`provider` atau `providers`): cek opencode.ai/docs/providers. Model harus lolos tes tool calling.

**Dua catatan penting:**
- **Nama mirip:** repositori lama `opencode-ai/opencode` (berbasis Go) berhenti dipelihara sejak September 2025. Proyek aktif: github.com/anomalyco/opencode (tim di balik Serverless Stack/SST). Pastikan memasang yang aktif.
- **Jangan menyambungkan langganan konsumen secara tidak resmi.** Pada Januari 2026 Anthropic memblokir alat pihak ketiga yang memakai langganan Claude konsumen lewat jalur tidak resmi. Pakai API key resmi.

**Cocok untukmu:** alternatif gratis dan BYOK untuk Cline atau Kilo Code dengan tampilan terminal, dengan key dan gateway yang sama.

### 9.15 Pesaing OpenCode dan mana yang terbaik untuk vibe coding

**Tidak ada yang terbaik untuk semua orang.** Peringkat di internet tidak netral: sebagian artikel ditulis vendor yang menjual salah satu alatnya (mis. artikel tim Kilo yang menempatkan Kilo CLI di urutan pertama).

| Alat | Jenis | Catatan |
|---|---|---|
| **Aider** | Agen terminal | Berbasis git, menyunting banyak file dengan rapi; Apache-2.0 |
| **Cline** | Ekstensi VS Code + CLI + SDK | Berdiri di dalam editor, Apache-2.0 |
| **Kilo Code / Kilo CLI** | Ekstensi VS Code/JetBrains + CLI | Cabang dari Cline dengan tampilan lebih baik; punya Virtual Quota Fallback |
| **Goose** | Agen umum berbasis MCP | Tidak hanya kode; sudah pindah ke Linux Foundation |
| **Crush** | Agen terminal | Disebut punya tampilan terminal terbaik, bisa ganti model di tengah sesi; lisensi "source available" (FSL), bukan sepenuhnya sumber terbuka |
| **Qwen Code** | Agen terminal | Cabang terbuka dari Gemini CLI untuk model Qwen; paket gratis OAuth Qwen dihentikan 15 April 2026, jadi pakai API key |
| **Pi, OpenHands, Continue** | Agen lain | Pi ringan; OpenHands untuk tugas otonom; Continue untuk CI |

**Berbayar atau terikat satu vendor:** Claude Code (Anthropic, tertutup), Codex CLI (OpenAI), GitHub Copilot CLI, Antigravity CLI. Satu daftar menyebut Gemini CLI dipensiunkan 18 Juni 2026 untuk digantikan penerus tertutup.

**Perubahan 2026:**
- **Roo Code diarsipkan pada Mei 2026** (status yang tadinya simpang siur di catatan ini kini jelas); penggunanya banyak pindah ke Kilo Code.
- OpenCode berhenti mendukung login Claude Pro/Max setelah sengketa dengan Anthropic.

**Mana yang terbaik untuk vibe coding?** Vibe coding bergantung pada hasil AI, jadi yang penting adalah **keamanan kerja dan kemudahan meninjau perubahan**:
1. **Untuk VS Code, gratis, BYOK: Kilo Code (atau Cline).** Perubahan tampil sebagai diff di editor, kamu menyetujui langkah demi langkah, dan Kilo Code punya pindah model otomatis.
2. **Nyaman dengan terminal: OpenCode.** Paling banyak dipakai dan fleksibel di antara agen terbuka; kamu harus lebih teliti meninjau perubahan.
3. **Ingin aman dengan git: Aider** (tiap perubahan jadi commit, mudah dibatalkan).
4. **Siap membayar dan ingin kualitas: Claude Code atau Cursor.** Satu sumber menilai Claude Code terbaik dalam orkestrasi agen, tetapi berbayar dan terkunci ke satu vendor.

**Pilihan alat kalah penting dibanding modelnya.** Dengan model gratis atau "flash", semua agen terasa mirip, dan keterbatasannya ada di kualitas model, tool calling, dan konteks. Uji model dengan skrip tool calling (5.4) sebelum memilih alat.

### 9.16 herdr (pengelola banyak agen di terminal)

**Apa itu:** multiplexer terminal untuk agen coding, seperti tmux tetapi sadar bahwa yang berjalan di dalamnya adalah agen AI. Satu binary Rust sekitar 10 MB, lokal, tanpa akun, tanpa telemetri.

- Tiap agen berjalan di pane sendiri; sidebar menampilkan status (blocked, working, done, idle). Agen dikenali lewat nama proses dan pola keluaran terminal (Claude Code, Codex, OpenCode, dan lain-lain).
- **Sesi persisten:** proses pane bertahan saat klien dilepas dan bisa disambung kembali lewat SSH. Dukungan sentuh lewat SSH memungkinkan mengelola agen dari HP (belum diuji).
- **Otomasi:** socket API lokal, perintah CLI, plugin, dan skill agen.
- Status proyek: pra-1.0, rilis sangat sering; deteksi agen berbasis heuristik sehingga bisa perlu sering diperbarui.

**Bukan gateway:**

| | herdr | 9Router |
|---|---|---|
| Yang diatur | Sesi terminal banyak agen | Lalu lintas permintaan ke model |
| Dilalui permintaan ke model? | **Tidak** | **Ya** |
| Fallback dan kuota | Tidak ada | Ada |
| Padanan | tmux | Proxy / gateway |
| Posisi di lapisan | Pendamping lapisan 1 (pengelola agen) | Lapisan 3 |

**herdr mengelola agen-nya, 9Router mengelola jalur ke model-nya.** Keduanya saling melengkapi.

**Peringatan pemasangan:** hasil pencarian menunjukkan beberapa repositori bernama herdr (`maxjendrall`, `naaive`, `ogulcancelik`). Satu sumber menyebut halaman utamanya di `ogulcancelik/herdr` dan situs resminya herdr.dev; mana yang resmi tidak bisa dipastikan. Pasang lewat herdr.dev dan baca skrip pemasangan sebelum menjalankannya.

**Cocok untukmu?** Berguna bila menjalankan beberapa agen sekaligus. Bila hanya satu agen di satu editor (mis. Kilo Code di VS Code), belum perlu.

### 9.17 Diagram: VS Code + herdr + agen + 9Router + Claude API

**Koreksi penempatan:** herdr **bukan lapisan 2**. Lapisan 2 adalah yang mengerjakan tugas (agen seperti Claude Code, OpenCode, Codex); herdr hanya menampung dan memantau agen, jadi ia pengelola di lapisan 1 bersama VS Code.

```
            ┌──────────────────────────────────────────────────────────────┐
LAPISAN 1   │   VS Code → terminal bawaan → herdr (memantau banyak agen)   │
            └──────────────────────────────────────────────────────────────┘
                                           ▼
            ┌──────────────────────────────────────────────────────────────┐
            │        AGEN  (tiap agen berjalan di satu pane herdr)         │
            │ ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
LAPISAN 2   │ │  Claude Code   │  │    OpenCode    │  │Codex / lainnya │   │
            │ └────────────────┘  └────────────────┘  └────────────────┘   │
            └──────────────────────────────────────────────────────────────┘
                                           │ satu alamat: http://localhost:20128/v1
                                           ▼
            ┌──────────────────────────────────────────────────────────────┐
LAPISAN 3   │      9Router  (combo: urutan fallback + pelacak kuota)       │
            └──────────────────────────────────────────────────────────────┘
                                           │ utama → cadangan → gratis
                                           ▼
            ┌──────────────────────────────────────────────────────────────┐
LAPISAN 4   │      Claude API │ Gemini │ OpenRouter :free │ geraikita      │
            └──────────────────────────────────────────────────────────────┘
                      ▲
              CLAUDE CONSOLE: kelola key, saldo, batas (bukan jalur permintaan)
```

**Catatan:**
- Saya belum memverifikasi apakah herdr tampil baik di terminal VS Code (sumbernya menyarankan terminal modern seperti Kitty, WezTerm, Ghostty). Bila bermasalah, pakai jendela terminal terpisah.
- Agen di extension VS Code (Cline, Kilo Code) tidak berjalan di pane herdr, tetapi bisa diarahkan ke 9Router yang sama.
- **Biaya berlipat:** banyak agen sekaligus memakai saldo Claude API yang sama. Pasang batas pengeluaran di workspace Console.
- **Format API:** Claude Code memakai format Anthropic, OpenCode dan Codex format OpenAI. 9Router mencantumkan Claude Code dan Codex sebagai alat yang didukung; pastikan alamat dan key tiap jalur benar.
- **Fallback dari Claude ke model lain belum tentu mulus:** kualitas dan tool calling bisa turun; uji tiap model di rantai.
- **Saldo habis di Anthropic belum tentu dikenali sebagai "kuota habis" oleh 9Router**, sehingga fallback mungkin tidak terpicu. Simulasikan dengan key berisaldo hampir nol atau key yang sengaja salah.
- Banyak agen paralel lebih cepat memicu batas laju (429) pada Gemini dan OpenRouter gratis.
- herdr punya socket API lokal agar agen bisa berinteraksi dengannya; biarkan lokal.

### 9.18 Claude Code mengendalikan Codex lewat herdr

Dari sebuah unggahan Instagram (captionnya tidak bisa dibuka langsung; penjelasan bersandar pada kutipan caption dan dokumentasi herdr): Claude Code bisa "menyetir" Codex sebagai subagent, dan sesi tetap hidup walau terminal ditutup.

**Mekanismenya:** herdr membuka socket lokal. Agen yang berjalan di dalamnya bisa memakai perintah `herdr` untuk menjalankan agen lain di pane sebelahnya, membaca keluaran pane, mengirim teks dan tombol ke pane lain, dan menunggu agen lain selesai. Claude Code diberi **skill** herdr (folder skill disalin ke `.claude/skills`; skill berisi instruksi, bukan kode, dan tidak terikat model tertentu). Contoh dari dokumentasi skill (agennya saya ganti Codex):

```bash
herdr pane split 1-2 --direction right --no-focus    # buat pane baru di sebelah
herdr pane run 1-3 "codex"                           # jalankan Codex di pane itu
herdr wait output 1-3 --match ">" --timeout 15000    # tunggu Codex siap
herdr pane run 1-3 "review test coverage di src/api/"   # kirim tugas
herdr wait agent-status 1-3 --status done --timeout 120000   # tunggu selesai
herdr pane read 1-3 --source recent --lines 100      # baca hasilnya
```

Versi dokumentasi yang lebih baru juga punya perintah `agent start`, `agent prompt`, dan `agent wait`.

**Kenapa disebut "subagent" dan bisa lintas model:** ini orkestrasi **tingkat terminal**, bukan tingkat model. Claude Code mengetik ke terminal Codex dan membaca teks yang muncul, seperti manusia. Tiap agen tetap punya model, login, dan kuota sendiri; tidak ada memori atau konteks bersama. Ini beda dengan subagent bawaan Claude Code yang memakai model Claude yang sama. Untuk DeepSeek atau model lain: jalankan agen lain di pane (mis. OpenCode dengan API DeepSeek), atau arahkan agen ke endpoint lain (tabel kompatibilitas mencantumkan endpoint Anthropic-compatible DeepSeek di `https://api.deepseek.com/anthropic`, yang secara teori memungkinkan Claude Code memakai DeepSeek). Cara persis di video itu tidak bisa dipastikan.

```
             ┌───────────────────────────────────────────────────────────────────────────┐
LAPISAN 1    │              herdr: sesi tetap hidup walau terminal ditutup               │
             │          socket lokal: pane run · wait agent-status · pane read           │
             └───────────────────────────────────────────────────────────────────────────┘
                        ▼                          ▼                          ▼
             ┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
             │       pane 2        │    │       pane 1        │    │       pane 3        │
LAPISAN 2    │        Codex        │◄── │     Claude Code     │──► │   OpenCode / lain   │
             │   (dikendalikan)    │    │    (orkestrator)    │    │   (mis. DeepSeek)   │
             └─────────────────────┘    └─────────────────────┘    └─────────────────────┘
                        ▼                          ▼                          ▼
             ┌───────────────────────────────────────────────────────────────────────────┐
LAPISAN 3    │ (opsional) 9Router: satu pintu + fallback untuk agen yang memakai API key │
             └───────────────────────────────────────────────────────────────────────────┘
                        ▼                          ▼                          ▼
             ┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
LAPISAN 4    │       OpenAI        │    │     Claude API      │    │   DeepSeek / dst.   │
             └─────────────────────┘    └─────────────────────┘    └─────────────────────┘
                                                   ▲
                                     CLAUDE CONSOLE: key & saldo
```

Panah mendatar di lapisan 2 adalah kendali lewat herdr (lalu lintas antaragen, bukan ke model); panah vertikal adalah jalur tiap agen ke modelnya. Codex dengan login ChatGPT kemungkinan langsung ke OpenAI tanpa gateway (belum diverifikasi).

**Risiko:**
- **Agen otomatis tanpa persetujuan.** Pekerja di pane tanpa pengawasan macet di pertanyaan y/n, sehingga panduan skill menyarankan persetujuan otomatis (`codex --yolo`, `claude --dangerously-skip-permissions`). Itu menonaktifkan pengaman; pembuat skill menyebut ini aman hanya karena memasang hook daftar-tolak global. **Jangan menyalin tanpa pengaman:** pakai proyek yang bisa dibatalkan (Git) dan idealnya terisolasi.
- Komposer TUI agen seperti Claude Code menganggap `pane run` sebagai paste dan menelan tombol Enter; solusinya `send-text` lalu `send-keys enter`.
- Biaya berlipat (dua atau tiga agen = dua atau tiga tagihan/kuota).
- Claude Code hanya melihat teks keluaran pane Codex; hasil panjang atau layar TUI bisa terpotong.
- herdr masih pra-1.0 dan deteksi agennya heuristik.

---

## 10. Agen Terkelola dan Agen di Aplikasi Desktop (Muse dan Claude Desktop)

### 10.1 Apa itu

- **Muse** adalah personal AI agent dari Meta (diluncurkan 8 September 2026). Menjalankan tugas atas namamu: email, kalender, pemesanan (travel, restoran), belanja, pelacakan harga, dokumen, gambar, tujuan jangka panjang.
- Berjalan di **Muse Secure VM**: komputer Linux persisten dan terisolasi di cloud, dengan browser penuh yang dipakai bersama olehmu dan agennya. Ada agen penjaga **Sentinel** yang mengatur apa yang boleh sampai ke internet.
- Memakai model buatan Meta sendiri (**Muse Spark 1.3**).
- Diakses lewat aplikasi Muse (iOS/Android), muse.ai, dan WhatsApp. Muse meminta persetujuan sebelum mengirim email atau melakukan pembelian; ada jejak audit aktivitas. Meta menyatakan data di VM tidak dibagikan ke sistem iklannya (klaim Meta sendiri).
- Dibedakan dari **Meta AI** (asisten, dicantumkan Meta sebagai "personal AI assistant").

### 10.2 Kategori produknya

Disebut **personal AI agent** (istilah Meta) atau secara umum **agen AI terkelola (hosted/managed agent)**. **Bukan** provider API seperti Gemini/OpenRouter, melainkan produk jadi untuk pengguna akhir.

Produk sejenis: Claude Cowork (agen desktop terkelola dari Anthropic; lihat 10.7), Manus (agen cloud dengan komputer virtual), Perplexity Computer, KimiClaw, OpenClaw Cloud (sekitar $59/bulan).

### 10.3 Posisi di empat lapisan

Muse menggabungkan klien chat + agen + komputer cloud + model sendiri dalam satu paket tertutup. Tidak ada gateway dan tidak ada BYOK (tidak terlihat di sumber yang dibaca). Penyedia modelnya Meta sendiri.

```
Muse by Meta = [klien + agen + komputer cloud (VM) + model Muse Spark]
               satu paket tertutup, tanpa BYOK dan tanpa gateway
```

### 10.4 Mengapa produk seperti ini ada

1. Model kini mampu memakai alat dengan andal (tool calling).
2. Agen butuh tempat bekerja yang aman: komputer cloud terisolasi memisahkan tindakan agen dari perangkat pengguna.
3. Pengguna awam tidak mau memasang apa-apa.
4. Pengamanan lewat persetujuan sebelum tindakan berisiko.

### 10.5 Soal memasang Hermes dan 9Router di dalam Muse

Belum bisa diverifikasi. Secara teknis VM Linux persisten memungkinkannya, tetapi Sentinel bisa membatasi akses internet dan aturan layanan Meta bisa melarangnya. Tumpang tindih fungsi juga: agen (Hermes) di dalam agen (Muse). Bila tujuannya menjalankan Hermes dan 9Router sendiri, VPS biasa lebih jelas haknya dan kendalinya.

### 10.6 Hosted vs self-hosted

| | Agen terkelola (Muse, Manus, Cowork) | Self-hosted (Hermes + 9Router) |
|---|---|---|
| Pemasangan | Praktis | Dipasang dan dirawat sendiri |
| Pilihan model | Biasanya terkunci | Bebas (BYOK) |
| Data | Disimpan di perusahaannya | Di server milikmu |
| Biaya | Langganan atau gratis untuk fitur tertentu | Server + pemakaian model |
| Kendali | Dibatasi pengembangnya | Penuh |

Harga dan ketersediaan Muse di wilayahmu belum ditemukan jelas; cek sebelum berencana memakainya.

### 10.7 Claude Desktop: klien dan agen dalam satu aplikasi

Claude Desktop tidak masuk satu label saja, karena ia aplikasi dengan beberapa tab yang berperan berbeda. Aplikasi desktop (Mac, Windows, ChromeOS) menggabungkan **Chat, Cowork, dan Code**. Claude Code sudah tertanam di aplikasinya, sehingga tidak perlu memasang CLI terpisah.

| Tab | Label yang tepat | Penjelasan |
|---|---|---|
| **Chat** | Klien chat (dengan alat) | Tanya-jawab biasa, tetapi bisa memakai connector/MCP dan skill |
| **Cowork** | **Agen AI** | Bukan chat dengan tampilan baru, melainkan agen desktop otonom: kamu memberi tujuan, Claude menentukan langkahnya |
| **Code** | Agen coding | Claude Code bawaan |

Posisi dalam empat lapisan:

```
Claude Desktop = [Tab Chat   : klien chat + connector/skill]
                 [Tab Cowork : agen otonom, bekerja di komputermu]
                 [Tab Code   : agen coding (Claude Code tertanam)]
                 model: Claude (Anthropic); tanpa BYOK/gateway (setahu saya)
```

Catatan:
- Mencakup **lapisan 1 (klien) dan lapisan 2 (agen)** sekaligus; lapisan 4 (model) adalah Claude dari Anthropic. Setahu saya tidak ada BYOK atau gateway untuk memakai model non-Claude **(belum diverifikasi dari sumber yang dibaca)**.
- Cowork menjalankan tugas multilangkah **di mesinmu** (membaca, mengedit, dan menyusun file di komputer), dengan akses folder dan connector yang diatur lewat kontrol aplikasi. Fitur Cowork, Code, computer use, dan extensions hanya berjalan di aplikasi desktop native, bukan di antarmuka web.
- Beberapa fitur khusus desktop: Quick Entry (macOS, pintasan keyboard dari aplikasi mana pun), extensions MCP, dan Cowork.
- Menurut satu panduan, aplikasinya bisa dipakai di paket gratis, tetapi **Cowork membutuhkan paket berbayar**. Cek batas paket terbaru di support.claude.com.

### 10.8 Skill, connector (MCP), dan extensions

| Istilah | Arti |
|---|---|
| **Connector** | Sama dengan server MCP (nama berbeda): cara Claude tersambung ke aplikasi lain. Ada **web connector** (jalan di server perusahaan di cloud) dan **desktop connector** (dipasang dan jalan di mesin lokal) |
| **Skill** | Kumpulan instruksi tersimpan, seperti kartu resep untuk tugas tertentu |
| **Extensions** | Server MCP yang dikemas untuk pemasangan sekali klik (format `.mcpb`, format lama `.dxt` masih jalan), mirip ekstensi browser |

Ringkasannya: **connector memberi akses, skill menjelaskan cara memakai akses itu.**

Poin penting untuk memahami kategori:
- **Banyaknya skill dan MCP tidak mengubah kategori aplikasi.** Keduanya hanya menambah kemampuan (alat dan instruksi). Cherry Studio juga mendukung MCP, tetapi itu tidak menjadikannya agen.
- Yang menentukan kategori adalah **ada tidaknya siklus otonom** (rencana, alat, hasil, ulang). Di tab Chat, setiap giliran masih percakapan; di Cowork, Claude bekerja sendiri sampai tujuan tercapai.
- Menurut satu panduan pihak ketiga, skill yang diunggah lewat pengaturan Desktop dipakai bersama di tab Chat, Cowork, dan Code, tetapi panel plugin Cowork dan Code terpisah, dan konfigurasi MCP tidak selalu sama antara aplikasi dan CLI. Cek dokumentasi resmi sebelum mengandalkannya.

### 10.9 Perbandingan: Cherry Studio vs Claude Desktop vs Muse

| | Cherry Studio | Claude Desktop | Muse by Meta |
|---|---|---|---|
| Jenis | Klien chat multi-penyedia | Klien + agen dalam satu aplikasi | Agen terkelola (+ klien) |
| Tempat agen bekerja | Mode agen sebagai fitur tambahan | **Di komputermu**, dengan akses folder yang diatur | **Di cloud** (Muse Secure VM) |
| Model | Bebas (BYOK) | Claude saja | Muse Spark saja |
| Gateway / BYOK | Ya | Tidak | Tidak |
| MCP / skill | MCP didukung | MCP, skill, extensions | Connectors ke aplikasi tertentu |
| Biaya | Gratis, bayar pemakaian API | Aplikasi di paket gratis; Cowork berbayar (menurut satu panduan) | Belum jelas |

Kesimpulan:
- **Paling mirip Muse dari sisi tujuan:** agen yang mengerjakan tugas atas namamu, berbasis model satu vendor. Bedanya ada di **tempat kerja**: Claude Desktop di mesinmu, Muse di komputer cloud Meta.
- **Mirip Cherry Studio dari sisi struktur aplikasi:** aplikasi desktop dengan MCP, tetapi tanpa kebebasan memilih penyedia.
- Sebagian rujukan bagian ini adalah artikel pihak ketiga, bukan dokumentasi resmi. Nama tab dan batas paket bisa berubah; rujukan resmi: https://support.claude.com

### 10.10 Keluarga Copilot: Microsoft Copilot, Microsoft 365 Copilot, GitHub Copilot

"Copilot" adalah satu merek untuk banyak produk. Tiga yang sering tertukar:

| Produk | Jenis | Fungsi |
|---|---|---|
| **Microsoft Copilot** (konsumen) | Klien chat + integrasi Windows/Edge | Chat, jawaban berbasis web, gambar, suara |
| **Microsoft 365 Copilot** | Asisten + agen di aplikasi kantor | Menulis, merangkum, menganalisis di Word, Excel, Outlook, Teams |
| **GitHub Copilot** | Asisten + agen coding | Penyelesaian kode, chat, agent mode, cloud agent (lihat 9.13) |

**Microsoft Copilot konsumen** (copilot.microsoft.com atau aplikasi) pada dasarnya klien chat gratis sejenis gemini.google.com dan chatgpt.com:
- Web, aplikasi iOS/Android, dan aplikasi desktop; pemilih model (GPT-5.2 dan model MAI buatan Microsoft); jawaban berbasis web dengan kutipan; pembuatan gambar, suara, Copilot Vision. Modelnya disediakan Microsoft; **tidak ada indikasi BYOK**.
- **Lebih terintegrasi ke sistem:** di Windows 11 ada tombol Copilot, kata panggil "Hey Copilot", dan Copilot Vision; **Copilot Mode di Edge** bisa melakukan tindakan lintas tab (Actions dan Journeys saat ini hanya untuk pengguna AS dan masih eksperimental), yang mulai berperan seperti agen.
- Copilot Pro dihentikan akhir 2025 dan fiturnya dipindah ke Microsoft 365 Premium (sekitar $19,99 per bulan) dengan agen seperti Researcher dan Analyst.

```
Lapisan 1  Klien      copilot.microsoft.com / aplikasi / Windows / Edge
              │
Lapisan 2  Agen       sebagian (Copilot Mode di Edge, agen di paket berbayar)
              │
Lapisan 3  Gateway    tidak ada
              │
Lapisan 4  Penyedia   Microsoft + OpenAI (juga Claude di sebagian produk 365)
```

Untuk versi konsumen gratis, kategorinya lapisan 1 dan 4 dalam satu paket tertutup, sama seperti gemini.google.com dan chatgpt.com. Ia tidak menggantikan Cherry Studio untuk keperluan BYOK.

### 10.11 Skill di claude.ai (web) vs plugin lokal (claude-mem)

**Skill di claude.ai:**
- **Skill bawaan** (PowerPoint, Excel, Word, PDF) aktif otomatis saat membuat dokumen, tanpa pengaturan. Syaratnya "Code execution and file creation" aktif di Settings > Capabilities.
- **Skill kustom tidak otomatis:** Customize > Skills > "+" > Create skill > Upload a skill, lalu unggah ZIP berisi folder skill (folder dengan `SKILL.md`). Skill kustom bersifat pribadi untuk akunmu.
- **Tidak tersinkron** antara claude.ai, API, dan Claude Code; unggah terpisah di tiap tempat.
- **Soal paket:** Pusat Bantuan menyebut skill tersedia untuk Free, Pro, Max, Team, Enterprise, sedangkan dokumentasi platform menyebut skill kustom di claude.ai untuk Pro, Max, Team, Enterprise dengan eksekusi kode aktif. Cek di menu akunmu.

**`claude-mem` bukan sekadar skill.** Ia sistem memori persisten untuk Claude Code dan agen lokal lain (README menyebut Claude Code, OpenClaw, Codex, Gemini, Hermes, Copilot, OpenCode): hook siklus hidup (SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop, SessionEnd), layanan worker lokal berbasis Bun, database SQLite, database vektor Chroma, alat MCP, dan skill `mem-search`. Kebutuhan: Node.js 20+, Claude Code terbaru dengan dukungan plugin, serta Bun dan uv (dipasang otomatis bila belum ada). Pemasangan lewat `npx claude-mem install` atau perintah `/plugin` di Claude Code.

Semua itu berjalan **di komputermu** sebagai proses latar belakang, sehingga **tidak bisa dipakai di claude.ai web** (sandbox tanpa hook dan tanpa daemon). README menyebut "Claude Desktop Skill" untuk mencari memori dari percakapan Claude Desktop; itu jalur lokal dan detailnya belum diverifikasi.

```
claude.ai (web)                         Claude Code / agen lokal
┌───────────────────────────┐           ┌─────────────────────────────────┐
│ sandbox eksekusi kode     │           │ komputermu                      │
│ skill = folder SKILL.md   │           │ skill + hook + MCP + worker     │
│ diunggah sebagai ZIP      │           │ + SQLite + Chroma (claude-mem)  │
│ tanpa hook, tanpa daemon  │           │ berjalan terus di latar belakang│
└───────────────────────────┘           └─────────────────────────────────┘
```

**Padanan di web:** fitur memori bawaan claude.ai (pencarian riwayat chat dan memori dari riwayat) di pengaturan.

**Kehati-hatian soal claude-mem:**
- Ia merekam semua yang dilakukan agen (hasil alat, isi file), lalu memampatkannya dengan AI. Isi bisa mengandung rahasia; README menyebut penanda `<private>` untuk mengecualikan konten.
- Pemampatan memakai model: installer mengajak masuk dengan email dan memberi percobaan gratis 30 hari, lalu kamu memilih penyedia (observer bawaan, key OpenRouter atau Gemini milikmu, atau paket Anthropic). Bisa dilewati dengan flag `--provider`.
- `npx` dan skrip `curl | bash` menjalankan kode dari internet; baca repositori dan skripnya. README juga menyebut token kripto CMEM dari pihak ketiga; itu tidak diperlukan untuk memakai plugin, pertimbangkan sendiri.

---

## 11. Susunan Akhir dan Rekomendasi Langkah

### 11.1 Susunan yang disarankan untuk keperluanmu

```
Cherry Studio (PC) / Chatbox / RikkaHub (HP) ─┐
Cline / Kilo Code (ekstensi di VS Code) ──────┤
Hermes Agent (opsional, untuk tugas)  ────────┼→ Gateway (9Router atau LiteLLM) → Gemini / Groq / OpenRouter :free / geraikita
Bot Telegram (opsional, lewat gateway yang sama) ┘
```

Prinsip:
- **Satu tempat** untuk mengatur urutan fallback (gateway), bukan di tiap aplikasi.
- **Gemini dulu** untuk pemakaian harian (kuota gratis, konteks panjang); geraikita dan lainnya sebagai cadangan.
- Pencarian web diatur terpisah di tiap klien (Cherry Studio sendiri) atau lewat Tavily/Firecrawl untuk bot.

### 11.2 Urutan langkah (checklist)

1. Pasang Cherry Studio; tambahkan Gemini dan geraikita; coba chat dan pencarian web.
2. Tambahkan Chatbox/RikkaHub di HP dengan dua penyedia yang sama.
3. Daftar penyedia gratis tambahan (OpenRouter, Groq, Cerebras, NVIDIA NIM) bila perlu kuota lebih.
4. Jalankan tes tool calling (5.4) untuk tiap model sebelum dipakai sebagai agen.
5. Bila sering kehabisan kuota: pasang gateway (9Router atau skrip sendiri) dan buat combo fallback; arahkan Cherry Studio ke sana; uji fallback (6.3).
6. Opsional: Hermes Agent untuk tugas otomatis (baca keamanan dulu); bot Telegram (`bot_multi.py`) untuk akses dari HP.
7. Coding: Kilo Code (dengan Virtual Quota Fallback) atau Cline di VS Code, dengan Git dan `.env` untuk key (lihat 9.6 dan 9.7). Antigravity dan Jules bisa jadi pelengkap gratis (lihat 9.8 dan 9.9).
8. Opsional: manfaatkan API Gateway dan Code Mate Cherry Studio untuk memakai ulang penyedia yang sudah diatur (lihat 3.6, 3.7, dan 9.11). Untuk fallback tetap pakai gateway khusus (9Router/LiteLLM).

### 11.3 Susunan ekosistem yang diluruskan (jangan rantai dua gateway)

Pertanyaan: apakah rantai berikut benar?

```
VS Code → Cline / Kilo Code → Cherry Studio (klien chat) ─┐
                                                           ├→ 9Router (gateway) → Gemini / OpenRouter / geraikita
```

Secara teknis bisa dijalankan, tetapi perlu diluruskan:

1. **Peran Cherry Studio di rantai itu bukan "klien chat".** Yang dipakai adalah fitur **API Gateway**-nya, jadi ia berperan sebagai **gateway lokal**. Label "klien chat" hanya berlaku saat kamu mengobrol di jendelanya.
2. **VS Code tidak mengirim permintaan.** VS Code hanya tempat ekstensi berjalan; yang mengirim permintaan ke model adalah Cline atau Kilo Code.
3. **Dua gateway berurutan itu berlebihan.** 9Router sudah bisa menjadi gateway tunggal dan mengklaim mendukung Cline dan alat sejenis, jadi ekstensi bisa langsung diarahkan ke 9Router. Menambah Cherry Studio di tengah:
   - menambah satu titik gagal (Cherry Studio harus tetap berjalan dengan gateway menyala);
   - menyulitkan pelacakan error, karena pesan error penyedia hulu bisa disamarkan sehingga sulit membedakan 429 dari galat lain, padahal untuk fallback kuota itu penting;
   - tidak menambah kemampuan, karena fallback, rotasi, dan pemantauan kuota sudah ada di 9Router.

**Susunan yang lebih tepat: semuanya sejajar ke satu gateway.**

```
VS Code (editor)
 └─ Cline / Kilo Code (ekstensi agen coding) ──┐
Cherry Studio (klien chat) ────────────────────┼→ 9Router (gateway + fallback) → Gemini / OpenRouter / geraikita
Hermes Agent (agen, opsional) ─────────────────┘                                 (penyedia API)
```

Semua alat memakai alamat 9Router (biasanya `http://localhost:20128/v1`), key dari 9Router, dan nama combo sebagai model. Urutan fallback diatur di satu tempat.

**Kapan rantai lewat Cherry Studio masuk akal?** Hanya bila sengaja ingin menjadikan Cherry Studio satu-satunya tempat kunci atau memakai Code Mate. Caranya: di Cherry Studio tambahkan 9Router sebagai penyedia OpenAI-compatible, lalu arahkan Cline/Kilo Code ke gateway Cherry Studio (Cline/Kilo Code → Cherry Studio API Gateway → penyedia "OpenAI-compatible" → 9Router → penyedia API). Bisa jalan tetapi **belum diuji** dan kurang disarankan karena alasan di atas.

**Dua pilihan untuk vibe coding gratis:**
- **Sederhana:** Kilo Code → langsung ke Gemini, dengan Virtual Quota Fallback ke geraikita sebagai cadangan.
- **Satu pusat fallback:** Kilo Code atau Cline → 9Router → Gemini / OpenRouter / geraikita; Cherry Studio ikut mengarah ke 9Router juga.

### 11.4 Matriks produk vs empat lapisan

Produk yang menggabungkan beberapa lapisan dalam satu paket mudah dipakai tetapi model terkunci pada satu vendor; produk yang memisahkan lapisan lebih bebas tetapi harus dirakit sendiri.

| Produk | L1 Klien | L2 Agen | L3 Gateway | L4 Penyedia |
|---|---|---|---|---|
| Cherry Studio | Ya | Fitur tambahan (Agent) | API Gateway lokal, tanpa fallback | Bebas (BYOK) |
| Hermes Agent | CLI/pesan | Ya | Tidak | Bebas (BYOK) |
| Claude Desktop | Ya | Ya (Cowork/Code) | Tidak | Claude |
| Muse by Meta | Ya | Ya | Tidak | Muse Spark |
| Codex | Ya | Ya | Tidak (opsional) | OpenAI |
| GitHub Copilot | Ya (ekstensi/CLI) | Ya (agent mode, cloud agent) | Tidak | Multi-model lewat GitHub |
| Microsoft Copilot (konsumen) | Ya | Sebagian | Tidak | Microsoft + OpenAI |
| OpenCode | Ya (TUI/desktop/VS Code/web) | Ya | Zen (opsional) | BYOK + Zen/Go |
| Antigravity | Ya | Ya | Tidak | Gemini + Claude/OpenAI tertentu |
| Jules | Ya (web/CLI) | Ya | Tidak | Gemini |
| Unsloth Studio | Ya (UI web) | Sebagian (tool calling) | Tidak | Lokal |
| 9Router | Tidak | Tidak | **Ya** | Tidak |
| herdr | Pendamping (pengelola agen) | Tidak | Tidak | Tidak |
| Claude Console | Panel kontrol lapisan 4, bukan jalur permintaan | | | |
| freellmapi | Tidak | Tidak | **Ya** (lintas paket gratis) | Tidak (memakai key penyedia gratis milikmu) |

---

## 12. Keamanan dan Privasi (Ringkasan)

- **Jangan menempel API key di chat, screenshot, atau repositori.** Simpan di `.env` dan `.gitignore`. Key di aplikasi klien tersimpan di perangkat.
- **Paket gratis sering memakai data untuk pelatihan** (Gemini gratis, Mistral gratis, dan lain-lain). Jangan kirim data pribadi/rahasia lewat paket gratis.
- **Hindari "key gratis" dari situs/grup tidak resmi.**
- **Gateway/agen pihak ketiga (9Router, Hermes, OpenClaw):** proyek baru dan viral di medsos bukan jaminan aman. Cek repositori resmi, baca skrip pemasangan, jauhi jalur OAuth/cookie/MITM bila bisa, dan waspada skill/marketplace pihak ketiga (ada kasus malware di ClawHub).
- **Agen bisa menjalankan perintah:** baca persetujuan tiap langkah penting; gunakan isolasi bila tersedia.
- **Bot Telegram:** set `ALLOWED_IDS` agar kuota tidak dipakai orang lain.
- **Fallback lintas penyedia:** pesan bisa sampai ke semua penyedia dalam rantai.
- **Agen terkelola (Muse dan sejenis):** data berada di perusahaannya; pertimbangkan kepercayaanmu sebelum menyambungkan email atau data keuangan.
- **Agen coding:** hindari mode tanpa persetujuan (Turbo di Antigravity) sebelum nyaman; jalankan di workspace tanpa hak istimewa dan commit sering. Jules menyalin kodemu ke VM Google, jadi cek kebijakan data untuk proyek sensitif.
- **Supabase:** aktifkan Row Level Security di tiap tabel agar data tidak terbuka lewat API publik.
- **API Gateway Cherry Studio:** key-nya bisa memakai semua penyedia di aplikasi. Jangan taruh di repositori/tangkapan layar/log; bila bocor, hentikan gateway lalu Regenerate. Biarkan gateway hanya lokal.
- **Pencarian web dan pengambilan URL:** kueri dikirim ke server pihak ketiga (Exa, Jina, dan lain-lain); jangan menulis data pribadi/rahasia saat pencarian menyala. Isi halaman yang masuk ke prompt tidak sepenuhnya tepercaya (risiko prompt injection).
- **Notebook pihak ketiga (Kaggle/Colab):** kode berjalan dengan akunmu. Baca dulu; jangan menempelkan Secrets, token Hugging Face, atau API key kecuali perlu (pakai token sekali pakai berizin minimal). Jangan bagikan keluaran sel yang memuat URL dan kata sandi. Jangan menjalankan skrip yang menghapus folder (`rmtree`) di PC sendiri. Tunnel membuka sesimu ke internet: pastikan ada autentikasi.
- **Unsloth Studio:** alat sisi server (pencarian web, eksekusi kode) aktif secara bawaan dan berjalan sebagai penggunamu. Biarkan hanya lokal; `-H 0.0.0.0` dan `--cloudflare` memperluas paparan.
- **Agen otomatis tanpa persetujuan** (mis. Claude Code mengendalikan Codex lewat herdr dengan `--yolo`/`--dangerously-skip-permissions`): pengaman dinonaktifkan. Pakai Git, proyek terisolasi, dan hook daftar-tolak.
- **Memory layer dan claude-mem:** jangan simpan rahasia di vault yang dibaca agen; claude-mem merekam aktivitas agen (hasil alat, isi file) dan memampatkannya dengan model (data dan biaya). Gunakan penanda `<private>` dan baca skrip `npx`/`curl | bash` sebelum menjalankan.
- **Key Claude API:** buat workspace dan key terpisah per agen/bot dengan batas pengeluaran.
- **Cherry Studio Channels (Telegram):** isi daftar ID chat yang diizinkan (kosong berarti semua chat, menurut string aplikasi).
- **Knowledge Base:** potongan dokumen dikirim ke penyedia embedding dan model chat; untuk dokumen sensitif jalankan keduanya lokal.
- **Langganan konsumen lewat jalur tidak resmi** (mis. login Claude Pro/Max di alat pihak ketiga) berisiko diblokir; pakai API key resmi.
- **API murah dari reseller tak dikenal:** waspadai model masking (model murah diberi nama model mahal), konteks dipotong, streaming palsu, dan tagihan digelembungkan. Semua prompt, kode, dan isi `.env` lewat server mereka; gunakan key uji berisaldo kecil dan jangan kirim data sensitif (lihat 2.13).
- **freellmapi:** jangan buka port 3001 ke internet; key penyedia tersimpan terenkripsi tetapi tetap di komputermu; prompt bisa dilayani penyedia mana pun yang aktif; pakai satu key sah per penyedia dan hormati ketentuan tiap paket gratis (lihat 6.7).

---

## 13. Glosarium

| Istilah | Arti singkat |
|---|---|
| **API key** | Kunci rahasia untuk memakai API sebuah layanan |
| **BYOK** | Bring Your Own Key: memakai key milikmu sendiri di aplikasi |
| **Base URL / endpoint** | Alamat API yang dituju aplikasi |
| **OpenAI-compatible** | Format API yang meniru OpenAI (`/chat/completions`), dipakai banyak penyedia |
| **Penyedia (provider)** | Layanan yang menjalankan model dan melayani API |
| **Agregator** | Penyedia yang menyalurkan banyak model dari banyak pembuat (OpenRouter) |
| **Gateway / router AI** | Perantara yang mengatur ke penyedia mana permintaan dikirim |
| **Fallback / failover** | Pindah otomatis ke model/penyedia lain saat gagal atau kuota habis |
| **Round-robin** | Bergiliran memakai beberapa key/penyedia |
| **Combo** | Gabungan beberapa penyedia jadi satu penyedia virtual (istilah 9Router) |
| **Klien chat** | Aplikasi untuk mengobrol dengan banyak model (Cherry Studio) |
| **Agen (agent)** | Program yang menjalankan siklus: rencana, alat, hasil, lalu mengulang |
| **Tool calling / function calling** | Kemampuan model meminta pemanggilan alat dalam format terstruktur |
| **MCP** | Model Context Protocol: standar untuk menyambungkan alat ke model |
| **Connector** | Istilah lain untuk server MCP (web atau desktop) yang memberi Claude akses ke aplikasi lain |
| **Skill** | Instruksi tersimpan untuk tugas tertentu ("kartu resep"); connector memberi akses, skill menjelaskan caranya |
| **Extensions (.mcpb)** | Server MCP yang dikemas untuk pemasangan sekali klik di Claude Desktop |
| **Cowork** | Tab agen otonom di Claude Desktop yang bekerja di komputer pengguna |
| **Agent-first IDE** | Editor kode yang dirancang di sekitar agen AI (Antigravity, Cursor, Windsurf) |
| **Agen asinkron** | Agen yang bekerja di belakang layar (biasanya di cloud) dan mengirim hasil setelah selesai, mis. Jules |
| **Pull request (PR)** | Usulan perubahan kode di GitHub yang bisa ditinjau sebelum digabung |
| **Virtual Quota Fallback** | Fitur Kilo Code yang berpindah otomatis antar-profil penyedia saat batas kuota terlewati atau terjadi error |
| **Row Level Security (RLS)** | Aturan keamanan database (Supabase/PostgreSQL) yang membatasi baris data yang boleh diakses tiap pengguna |
| **Supabase** | Layanan backend (database PostgreSQL, autentikasi, penyimpanan) yang jadi pilihan bawaan banyak stack vibe coding |
| **Exa MCP (exaMCP)** | Server MCP hosted milik Exa untuk pencarian web; bisa dipakai tanpa key pada paket gratis yang dibatasi; penyedia pencarian default di Cherry Studio |
| **Jina Reader** | Layanan yang mengubah URL menjadi teks/Markdown siap pakai LLM (`https://r.jina.ai/<URL>`); penyedia pengambil URL default di Cherry Studio |
| **URL fetch provider** | Penyedia yang mengambil isi satu halaman web (Jina, Firecrawl, Crawl4AI), terpisah dari penyedia yang mencari |
| **Crawl4AI** | Pengambil halaman yang berjalan lokal dengan Chromium headless dan rendering JavaScript |
| **Prompt injection** | Instruksi tersembunyi di isi halaman/dokumen yang mencoba mengarahkan model |
| **API Gateway (Cherry Studio)** | Server lokal di Cherry Studio yang mengekspos penyedia/model yang sudah diatur lewat API kompatibel OpenAI dan Anthropic |
| **Code Mate** | Halaman Cherry Studio untuk memasang, mengonfigurasi, dan menjalankan CLI coding (Claude Code, OpenCode, Hermes Agent, dan lain-lain) |
| **Unified Gateway** | Opsi di Code Mate yang mengarahkan CLI lewat gateway Cherry Studio agar bisa memakai model apa pun yang sudah diatur |
| **Konteks (context window)** | Jumlah token yang bisa "diingat" model dalam satu permintaan |
| **Token** | Satuan potongan teks yang dihitung untuk kuota dan biaya |
| **Rate limit / 429** | Batas jumlah permintaan; 429 berarti batas terlewati |
| **Kuota** | Jatah pemakaian gratis per menit/hari/bulan |
| **Reasoning effort** | Seberapa dalam model berpikir (low/medium/high) |
| **Service tier** | Kompromi biaya dan kecepatan pemrosesan (standard/flex/fast) |
| **Streaming** | Jawaban muncul bertahap, bukan sekaligus |
| **Self-hosted** | Dijalankan di perangkat/server milik sendiri |
| **VPS / VM** | Komputer virtual di cloud |
| **Fork (VS Code)** | Salinan VS Code yang dimodifikasi menjadi aplikasi terpisah (Cursor, Windsurf) |
| **Vibe coding** | Membuat perangkat lunak dengan mendeskripsikan keinginan ke AI |
| **Memory layer** | Penyimpanan di luar model tempat agen menulis dan membaca pengetahuan lintas sesi (mis. vault Obsidian) |
| **Vault (Obsidian)** | Folder berisi file Markdown yang dikelola aplikasi Obsidian; bisa dibaca dan ditulis agen |
| **RAG** | Retrieval-Augmented Generation: potongan dokumen relevan dicari lalu dikirim ke model bersama pertanyaan |
| **Embedding** | Perubahan teks menjadi vektor angka agar kemiripan makna bisa dicari |
| **Chunk** | Potongan kecil dokumen (sekitar 200 sampai 500 karakter) yang divektorkan |
| **Knowledge Base (Cherry Studio)** | Fitur RAG untuk menjawab dari file, folder, URL, dan catatanmu |
| **Channels (Cherry Studio)** | Fitur menghubungkan Agent ke aplikasi pesan (Telegram, Discord, Feishu, Slack) |
| **Multiplexer (herdr)** | Terminal yang menampung banyak sesi/agen sekaligus dan menjaganya tetap hidup |
| **Notebook (Colab/Kaggle)** | Lingkungan Jupyter di cloud dengan mesin virtual sementara dan GPU terbatas |
| **Zen / Go (OpenCode)** | Layanan opsional milik OpenCode: gateway bayar-per-pemakaian (Zen) dan langganan model open-weight (Go) |
| **Claude Console** | Dasbor developer Anthropic untuk membuat key, saldo, batas, dan Workbench; bukan penyedia model |
| **Messages API** | API native Claude (`/v1/messages`) dengan `x-api-key`, `anthropic-version`, dan `max_tokens` wajib |
| **Plugin (Claude Code)** | Paket yang menggabungkan skill, hook, dan MCP; jalan lokal, beda dengan skill ZIP di claude.ai |
| **Model masking (model dilution)** | Penjual API menyamarkan model murah sebagai model mahal (nama model dan identitas dipalsukan) |
| **Relay / reseller API** | Layanan perantara yang menjual akses API model, sering dengan harga jauh di bawah resmi |
| **Streaming palsu** | Jawaban ditunggu penuh lalu dipotong-potong seolah mengalir bertahap |
| **Needle-in-a-haystack** | Uji konteks dengan menaruh satu fakta di tengah teks panjang lalu menanyakannya kembali |
| **Unified key (freellmapi)** | Kunci lokal `freellmapi-…` buatan router-mu sendiri; bukan key dari penyedia atau dari situs freellmapi |

---

## 14. Hal yang Belum Terverifikasi

Dikumpulkan agar tidak dianggap fakta:

- Apakah aplikasi mobile/koneksi perangkat Cherry Studio menyinkronkan riwayat chat.
- Apakah gateway geraikita meneruskan parameter `tools` dan model-modelnya merespons reasoning effort.
- Apakah geraikita bisa ditambahkan sebagai penyedia kustom di 9Router.
- Perilaku persis 9Router untuk tiap penyedia saat kuota habis; kelancaran streaming di Cherry Studio; apakah jawaban yang sedang mengalir bisa dilanjutkan saat penyedia gagal di tengah.
- Pemasangan 9Router (Node) di Termux.
- Daftar penyedia "gratis" 9Router (sumber bertentangan).
- Apakah Hermes dan 9Router bisa dipasang di Muse Secure VM, serta harga dan ketersediaan Muse di wilayahmu.
- Claude Desktop: apakah benar tanpa BYOK (tidak bisa memakai model non-Claude), nama tab, dan batas paket untuk Cowork. Sebagian rujukan adalah artikel pihak ketiga; cek support.claude.com.
- Kilo Code: apakah Virtual Quota Fallback masih ada dan menunya sama di versi terbaru (dokumentasi ada di dua situs). Cline: tidak ditemukan fallback bawaan, bukan berarti tidak ada.
- Claude Code via gateway/OpenRouter: kualitas hasil dengan model non-Claude belum diuji.
- Stack vibe coding dan peringkat alat berasal dari blog/artikel perbandingan (sebagian milik vendor), bukan survei resmi; harga cepat berubah.
- Antigravity: apakah mendukung BYOK, model apa yang tersedia di paket gratis, dan batas kuota terbaru (sedang berubah-ubah).
- Jules: angka kuota dan harga berasal dari indeks pihak ketiga (Juli 2026); kebijakan data untuk paket gratis belum diperiksa.
- Proyek gateway gratis sejenis 9Router yang disebut di 9.6: baru dan belum dinilai keamanannya.
- Status Roo Code/Continue (sumber bertentangan).
- Angka kuota gratis: OpenRouter, Hugging Face (sumber forum), Tavily, Firecrawl, dan lain-lain.
- Model Gemini yang tepat untuk key-mu (cek `/models`).
- Cherry Studio, pencarian web: arti persis "compression method: cutoff"; apakah jalur "pencarian lokal" (Local) masih ada di versi lain; kebutuhan key tiap penyedia pencarian; batas MCP gratis Exa (tidak dipublikasikan; angka 50/hari per IP hanya perkiraan dari satu artikel).
- URL fetch provider: kapan persisnya dipanggil Cherry Studio, apa itu "fetch" dan "querit", serta cara memasang Crawl4AI.
- API Gateway Cherry Studio: path persis (mis. `/v1`) dan format nama model yang diterima; apakah parameter `tools` diteruskan; apakah Cherry Studio harus tetap berjalan (kemungkinan ya); apakah gateway bisa dibuka ke perangkat lain dan caranya.
- Code Mate: apakah CLI-nya bisa dijalankan di terminal terintegrasi VS Code; kualitas Claude Code dengan model non-Claude lewat Unified Gateway.
- Rantai Cline/Kilo Code → Cherry Studio → 9Router, dan profil Kilo Code ganda yang mengarah ke gateway Cherry Studio: belum diuji.
- Cherry Studio Channels: apakah percakapan Telegram dan sesi agen tersinkron, apakah kanal bekerja saat jendela utama ditutup, dan kemampuan agen dibanding bot buatan sendiri.
- Knowledge Base: apakah Gemini atau geraikita menyediakan model embedding, apakah jawaban menampilkan kutipan sumber, dan ukuran potongan di versi terbaru.
- Unsloth Studio: path endpoint API, dukungan Android/Termux, penggunaan multi-GPU di Kaggle, dan apakah `UNSLOTH_STUDIO_PASSWORD` serta `--disable-tools` berlaku di versi terpasang.
- Kaggle: aturan soal tunnel/UI web, batas sesi (9 atau 12 jam), dan angka kuota 30 jam (bukan angka resmi yang ditemukan). Colab: seberapa aman notebook UI web dari penghentian di paket gratis.
- Codex: BYOK lewat gateway, skema biaya terbaru, dan model bawaan. GitHub Copilot: BYOK untuk chat di VS Code dan status pendaftaran paket individu.
- OpenCode: nama kunci konfigurasi antar versi (`provider`/`providers`), paket gratis (sumber pribadi), dan harga Zen/Go.
- herdr: repositori resmi, tampilan di terminal VS Code, dan akses sentuh lewat SSH dari HP. Video Instagram yang dibahas tidak bisa dibuka, jadi cara persis yang dipakai tidak diverifikasi.
- Claude Console: detail Claude Managed Agents dan penamaan produk (Console vs Platform).
- Skill claude.ai: paket yang mendukung skill kustom (sumber Anthropic berbeda). claude-mem: detail "Claude Desktop Skill".
- Memory layer: kualitas dan keamanan `devbrain`, `engram`, dan proyek komunitas sejenis.
- 9Router + Claude API: apakah saldo habis di Anthropic dikenali sebagai kuota habis, dan kualitas fallback dari Claude ke model non-Claude.
- WhatsApp: perlakuan bot AI pribadi dan risiko jalur tidak resmi.
- freellmapi: isi bagian disclaimer, kebijakan tiap penyedia soal menumpuk paket gratis, kualitas routing dan konsistensi di pemakaian nyata, aplikasi Android/iOS yang dicantumkan di README, dan keandalan panduan Termux (berlabel eksperimental).
- Arti "masking" dalam konteks Indonesia ditafsirkan dari istilah teknis setara; alat audit relay (zing, relay-radar, llm-verify, tokenwatt-verify, fpverify, api-model-spy) belum diuji dan hasilnya probabilistik.

---

## 15. Tanya-Jawab Cepat

**Bagaimana memakai semua model dalam satu tempat?**
Pakai klien chat (Cherry Studio di PC, Chatbox/RikkaHub di HP) dengan penyedia Gemini dan satu penyedia OpenAI-compatible untuk geraikita. Tambahkan gateway bila ingin fallback.

**Apakah API key Gemini gratis seperti akun gratis Gemini chat?**
Ya, bisa dibuat di Google AI Studio dengan paket gratis rate-limited. Claude berbeda: akun gratis claude.ai tidak menghasilkan API key, dan API tidak punya paket gratis permanen.

**Apakah Cherry Studio otomatis pindah model saat kuota habis?**
Tidak (tidak ada fitur bawaan). Gunakan gateway (9Router/LiteLLM) atau fallback OpenRouter.

**Apa bedanya OpenRouter dan 9Router?**
OpenRouter = layanan cloud penyalur model; 9Router = program lokal pengatur lalu lintas yang bisa memakai OpenRouter sebagai salah satu penyedia.

**Apa bedanya agen dan non-agen?**
Non-agen menjawab; agen mengerjakan (alat, siklus, otonomi). Modelnya bisa sama.

**Apakah Hermes Agent punya model sendiri?**
Bukan. Ia program agen BYOK; Nous Research memang punya model "Hermes", tetapi itu hal berbeda.

**Bisakah model gratisku jadi otak agen?**
Bisa bila mendukung tool calling, konteks panjang, dan kuotanya cukup. Uji dulu (5.4).

**Mengapa model di bot tidak tahu juara Piala Dunia 2026, tetapi di Cherry Studio tahu?**
Bot tidak mencari web untuk chat biasa; Cherry Studio memakai pencarian web. Model tidak tahu peristiwa setelah batas pengetahuannya.

**Apakah pencarian web di bot gratis?**
Bisa, lewat kuota gratis Tavily/Firecrawl (sekitar 1.000/bulan). Hasil pencarian tetap memakai token.

**Mengapa Cherry Studio bisa mencari web gratis?**
Penyedia pencarian bawaannya, exaMCP (server MCP hosted milik Exa), bisa dipakai tanpa API key pada paket gratis yang dibatasi. Ada juga DuckDuckGo tanpa key dan penyedia lain berkey dengan kuota gratis (lihat 4.2). Penjelasan lama tentang "pencarian lokal" sudah diluruskan.

**Apa bedanya low/medium/high dengan standard/fast/flex?**
Yang pertama kedalaman berpikir (high = lebih banyak token, lebih lambat); yang kedua kecepatan/harga (flex = murah dan lambat, fast = cepat dan mahal), tanpa mengubah jumlah token.

**Apakah Hugging Face sama seperti OpenRouter?**
Sebagian. HF adalah gudang model plus penyedia API (Inference Providers) dengan kuota gratis kecil, katalog condong ke model terbuka.

**Bagaimana memakai AI di VS Code?**
Pasang Cline atau Kilo Code (BYOK), atau Copilot gratis; Cursor/Windsurf adalah aplikasi terpisah berlangganan.

**Muse by Meta itu apa?**
Personal AI agent terkelola: klien + agen + komputer cloud + model sendiri dalam paket tertutup; bukan provider API.

**Claude Desktop (dengan banyak skill dan MCP) itu klien chat, agen, atau seperti Muse?**
Tergantung tabnya: Chat = klien chat dengan alat; Cowork = agen otonom yang bekerja di komputermu; Code = agen coding. Banyaknya skill/MCP tidak mengubah kategori. Mirip Muse dari sisi tujuan (agen satu vendor), tetapi bekerja di mesinmu, bukan di cloud (lihat 10.7 sampai 10.9).

**Apakah ada ekstensi VS Code selain Claude Code yang BYOK, dan bisa ganti model otomatis saat kuota habis?**
Ada: Cline dan Kilo Code. Kilo Code punya Virtual Quota Fallback yang pindah otomatis antar-profil penyedia berdasarkan batas atau error. Cline tidak ditemukan fitur setara; jalurnya lewat OpenRouter atau gateway (lihat 9.6). Claude Code juga bisa dialihkan ke model lain lewat gateway, tetapi itu bukan jalur resmi.

**Setup vibe coding yang populer seperti apa?**
Stack per tahap: alat kode (Lovable/Bolt/Replit atau Cursor/Claude Code/VS Code + Cline/Kilo Code), backend Supabase, deploy Vercel/Netlify/Cloudflare, GitHub untuk versi. Paket gratis BYOK: VS Code + Kilo Code + Gemini + gateway fallback (lihat 9.7).

**Google Antigravity itu apa, dan labelnya?**
IDE berbasis agen dari Google (fork VS Code, dengan Agent Manager dan Artifacts): lapisan klien + agen khusus coding, model Gemini plus Claude/OpenAI tertentu. Bukan klien chat umum dan tidak seperti Muse (lihat 9.8).

**Google Jules itu apa, dan labelnya?**
Agen coding asinkron di VM Google Cloud yang menghasilkan pull request dari repositori GitHub. Agen AI khusus coding; secara arsitektur paling mirip Muse, tetapi untuk kode. Bukan untuk vibe coding interaktif (lihat 9.9).

**Apa fungsi "URL fetch provider" di pengaturan web search Cherry Studio?**
Mengambil isi lengkap satu halaman (URL) menjadi teks bersih; terpisah dari penyedia pencarian yang hanya menemukan halaman. Default Jina Reader bisa dipakai tanpa key pada mode dasar (lihat 4.4).

**Apa fungsi API Gateway di Cherry Studio?**
Menjadikan Cherry Studio gateway lokal (`127.0.0.1:23333`) yang mengekspos penyedia/model yang sudah diatur lewat API kompatibel OpenAI dan Anthropic, sehingga alat lain tidak perlu key penyedia sendiri. Key gateway sangat berkuasa, hanya lokal, dan tidak ada fallback otomatis (lihat 3.6).

**Bisakah vibe coding di VS Code memakai AI dari Cherry Studio?**
Bisa: lewat Code Mate (CLI di terminal) atau lewat API Gateway yang dipakai Cline/Kilo Code sebagai penyedia OpenAI-compatible. Untuk vibe coding saja, menyambungkan ekstensi langsung ke Gemini/geraikita lebih sederhana (lihat 9.11).

**Apakah rantai VS Code → Cline/Kilo Code → Cherry Studio → 9Router benar?**
Bisa jalan, tetapi Cherry Studio di sana berperan sebagai gateway (bukan klien chat), VS Code hanya tempat ekstensi, dan dua gateway berurutan berlebihan. Lebih baik semua alat sejajar ke satu gateway (lihat 11.3).

**Apa itu Claude Console, dan apakah itu "provider"?**
Dasbor developer Anthropic (platform.claude.com) untuk membuat API key, mengisi saldo, memasang batas, dan Workbench. Bukan penyedia model; penyedianya Claude API. Perannya sepadan dengan Google AI Studio, tetapi API Claude tidak punya paket gratis permanen (lihat 2.9).

**Bisakah API Claude Console dijadikan bot Telegram atau WhatsApp?**
Telegram: ya, lewat Messages API native (atau lapisan kompatibel OpenAI, atau lewat gateway); lihat 7.5. WhatsApp: Meta melarang chatbot AI serba guna di Business Platform sejak 15 Januari 2026, jalur tidak resmi berisiko (lihat 7.6).

**Bisakah Cherry Studio menjawab dari dokumen saya (mirip NotebookLM)?**
Ya, lewat Knowledge Base (RAG) dengan model embedding; berbeda dari NotebookLM dalam pertanyaan menyeluruh dan fitur ekstra (lihat 3.8).

**Apakah Cherry Studio punya bot Telegram bawaan?** (diluruskan)
Ada, lewat fitur Channels untuk Agent (Telegram, Discord, Feishu, Slack); Cherry Studio harus tetap berjalan dan API Gateway aktif. Klaim lama "tidak punya" sudah dikoreksi (lihat 3.9 dan 7.4).

**Apakah Cherry Studio bisa mengunduh dan menjalankan model lokal?**
Tidak sendiri; ia terhubung ke Ollama atau LM Studio yang mengunduh dan menjalankan model (lihat 2.10).

**Apa itu Unsloth Studio, dan apakah mendukung Telegram/WhatsApp atau Android?**
Antarmuka web lokal untuk menjalankan dan melatih model terbuka (Beta). Tidak ada fitur Telegram/WhatsApp maupun versi Android yang ditemukan; bisa membuka endpoint API untuk alat lain (lihat 2.11).

**Apakah Colab atau Kaggle itu VPS?**
Bukan: mesin virtual sementara per sesi. Kaggle punya kuota GPU mingguan yang tetap, Colab lebih tidak terduga; keduanya tidak cocok untuk layanan 24/7 (lihat 2.12).

**Apa itu Codex?**
Agen coding OpenAI dengan banyak antarmuka (CLI, ekstensi IDE, desktop, cloud); bukan IDE (lihat 9.12).

**Apa bedanya GitHub Copilot dan Microsoft Copilot?**
GitHub Copilot = asisten dan agen coding; Microsoft Copilot = klien chat konsumen; Microsoft 365 Copilot = asisten di aplikasi kantor (lihat 9.13 dan 10.10).

**OpenCode itu agen atau provider?**
Utamanya agen coding sumber terbuka (lapisan 1 dan 2) dengan layanan opsional mirip penyedia (Zen dan Go); bisa dipakai sebagai agen murni dengan key sendiri (lihat 9.14).

**Apa pesaing OpenCode dan mana terbaik untuk vibe coding?**
Aider, Cline, Kilo Code, Goose, Crush, Qwen Code, dan lain-lain. Tidak ada yang terbaik untuk semua; untuk VS Code gratis BYOK, Kilo Code atau Cline; kualitas model lebih menentukan daripada alat (lihat 9.15).

**Apa itu herdr, dan apakah gateway seperti 9Router?**
Multiplexer terminal untuk banyak agen coding (seperti tmux). Bukan gateway: ia mengelola sesi agen, 9Router mengelola jalur ke model (lihat 9.16).

**Bagaimana Claude Code bisa menyetir Codex?**
Lewat socket dan CLI herdr (skill herdr): Claude Code membuat pane, menjalankan Codex, mengirim prompt, menunggu status, dan membaca keluaran; orkestrasi tingkat terminal, bukan tingkat model (lihat 9.18).

**Apakah skill claude.ai otomatis, dan bisakah memakai claude-mem di web?**
Skill bawaan otomatis; skill kustom diunggah sebagai ZIP. claude-mem adalah plugin lokal (hook, worker, SQLite/Chroma) sehingga tidak bisa di web (lihat 10.11).

**Apa itu memory layer dan kombinasi agen + Obsidian?**
Penyimpanan di luar model tempat agen menulis dan membaca pengetahuan lintas sesi; Obsidian adalah folder Markdown yang bisa dibaca dan ditulis agen (lihat 5.6).

**Apa bedanya model free OpenRouter dengan freellmapi, dan kenapa freellmapi tidak memberi API key?**
OpenRouter adalah layanan cloud dengan akun dan key sendiri; freellmapi adalah router yang dijalankan sendiri dan tidak menerbitkan key. Kamu memasukkan key gratis dari tiap penyedia asli; "unified key"-nya hanya kunci lokal (lihat 6.7).

**Bagaimana cara memakai freellmapi?**
Pasang (desktop, Docker, atau Termux eksperimental), tambahkan key penyedia di halaman Keys, salin unified key, lalu arahkan aplikasi ke `http://localhost:3001/v1` dengan model `auto` (lihat 6.7).

**Apa maksud "masking gak?" pada layanan API murah?**
Menanyakan apakah penjual menyamarkan model murah seolah-olah model mahal (model masking atau model dilution). Trik terkait: konteks dipotong, streaming palsu, tagihan digelembungkan (lihat 2.13).

**Bagaimana mengecek apakah API murah benar-benar memakai model aslinya?**
Bandingkan harga dengan harga resmi, uji identitas dan batas pengetahuan, uji konteks (needle-in-a-haystack), bandingkan soal sulit dengan API resmi, hitung token sendiri, dan ulangi di waktu berbeda. Alat audit seperti zing dan relay-radar bisa membantu (lihat 2.13).

---

## 16. Sumber Rujukan

Tautan yang muncul dalam hasil pencarian selama percakapan (cek versi terbarunya):

- Gemini API, kompatibilitas OpenAI: https://ai.google.dev/gemini-api/docs/openai
- Dokumentasi Cherry Studio: https://docs.cherry-ai.com (MCP: https://docs.cherry-ai.com/docs/en-us/advanced-basic/mcp/config)
- Rilis Cherry Studio: https://github.com/CherryHQ/cherry-studio/releases
- RikkaHub: https://rikka-ai.com/en
- Hermes Agent: https://pypi.org/project/hermes-agent dan https://hermes-agent.nousresearch.com/docs/integrations
- Perbandingan Hermes Agent vs OpenClaw: https://www.websiterating.com/tools/hermes-agent-vs-openclaw-comparison/
- 9Router: https://9router.com/ dan https://docs.9router.com/
- OpenRouter, model fallbacks: https://openrouter.ai/docs/guides/routing/model-fallbacks
- OpenRouter, service tiers: https://openrouter.ai/docs/guides/features/service-tiers.md
- OpenRouter, perbandingan API LLM gratis: https://openrouter.ai/blog/tutorials/free-llm-apis-compared/
- Hugging Face Inference Providers: https://huggingface.co/docs/inference-providers/en/index
- API pencarian web gratis untuk agen: https://parallel.ai/articles/best-free-web-search-api dan https://www.firecrawl.dev/blog/best-free-web-search-apis
- Perbandingan gateway LLM: https://dupple.com/learn/best-llm-gateways
- Agen coding untuk VS Code: https://kilo.ai/articles/coding-agents-for-vscode
- Muse by Meta: https://ai.meta.com/muse/ dan https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/
- Claude Desktop (panduan pihak ketiga): https://fast.io/resources/claude-desktop-app-guide/, https://www.productcompass.pm/p/claude-cowork-guide, https://chellehoniker.com/claude-for-authors-part-3-connectors/
- Kilo Code, Virtual Quota Fallback: https://kilo.ai/docs/ai-providers/virtual-quota-fallback dan https://kilocode.ai/docs/providers/virtual-quota-fallback
- OpenRouter dengan agen coding apa pun: https://openrouter.ai/blog/tutorials/any-coding-agent/
- 9Router (daftar alat yang didukung): https://github.com/decolua/9Router
- Stack dan alat vibe coding: https://rapidevelopers.com/vs/best-vibe-coding-tools dan https://vp0.com/blogs/best-vibe-coding-tools-2026
- Google Antigravity: https://en.wikipedia.org/wiki/Google_Antigravity dan https://blog.openreplay.com/google-antigravity-ide-guide/ (resmi: https://antigravity.google)
- Google Jules: https://theaiagentindex.com/agents/jules, https://www.morphllm.com/comparisons/jules-google-coding-agent, https://dataconomy.com/2025/08/07/google-launches-jules-ai-coding-agent-out-of-beta/ (resmi: https://jules.google)
- Cherry Studio, API Gateway: https://cherryai.com/docs/en/advanced-basic/developer-tools/api-gateway/ (versi lama: https://docs.cherry-ai.com/docs/en-us/advanced-basic/api-server)
- Cherry Studio, Code Mate: https://cherryai.com/docs/en/cherrystudio/preview/code-cli/
- Cherry Studio, laporan bug gateway: https://github.com/CherryHQ/cherry-studio/issues/22268
- Exa MCP (hosted, tanpa key): https://www.gamut.so/mcp/search-ai/exa, https://agentman.ai/agentskills/connections/mcp-server/exa, dan https://dev.to/juholee/exa-mcp-for-claude-code-ai-native-web-search-with-clean-page-content-no-key-needed-191e (angka batas dari artikel ini bukan resmi)
- Jina Reader vs Firecrawl: https://macmyths.com/jina-ai-vs-firecrawl-for-web-llm-extraction/
- Anthropic (resmi): https://docs.claude.com/en/api/overview, https://docs.claude.com/en/docs/claude-code/overview, https://support.claude.com
- Claude API dan skill (resmi): https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/openai-sdk, https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview, https://support.claude.com/en/articles/12512180-use-skills
- claude-mem: https://github.com/thedotmack/claude-mem
- herdr: https://herdr.dev/docs/preview/agent-automation/ dan https://skills.cat/skills/ogulcancelik/herdr
- Cherry Studio, Knowledge Base: https://docs.cherry-ai.com/docs/en-us/knowledge-base/knowledge-base dan https://docs.cherry-ai.com/en-us/knowledge-base/document-preprocessing
- Cherry Studio, Channels (berkas lokalisasi aplikasi): https://raw.githubusercontent.com/CherryHQ/cherry-studio/55b402e401b21b897df7531bb35cf727c5bc3684/src/renderer/i18n/locales/en-us.json
- Memory layer / Obsidian: https://pasqualepillitteri.it/en/news/962/obsidian-claude-code-second-brain-persistent-memory, https://glama.ai/mcp/servers/DycandX/devbrain, https://github.com/PatrickNoFilter/eling, https://enterprisedna.co/directories/mcp/rwnalds-engram
- Kebijakan WhatsApp: https://techcrunch.com/2025/10/18/whatssapp-changes-its-terms-to-bar-general-purpose-chatbots-from-its-platform dan https://respond.io/blog/whatsapp-general-purpose-chatbots-ban
- Notebook Unsloth Studio (Kaggle, dibagikan pengguna): https://www.kaggle.com/code/talhakhallid/unsloth-studio
- OpenCode (resmi): https://opencode.ai/docs/providers
- freellmapi: https://github.com/tashfeenahmed/freellmapi dan https://freellmapi.co
- Audit relay API dan masking model: https://pypi.org/project/zing-audit/, https://github.com/AetherCore-Dev/relay-radar, https://github.com/mintesnot-teshome/llm-verify, https://pypi.org/project/tokenwatt-verify/, https://github.com/dabaibian/api-model-spy, https://github.com/Mohamed7415/fpverify

---

*Akhir catatan. Perbarui bagian harga, kuota, dan status layanan secara berkala.*
