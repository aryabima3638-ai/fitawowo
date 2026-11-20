# Bot Build Vann — GramJS (GitHub Actions REAL)

Versi bersih + **server.js sudah real** (bukan dummy).

## Fitur
- GramJS auto-connect + session
- Build Flutter & Web2APK lewat **GitHub Actions**
- Credit, ban, reseller, maintenance, force-join, admin panel

## Setup

```bash
npm install
```

### Wajib isi environment

```bash
export BOT_TOKEN="123:ABC..."
export API_ID="12345678"
export API_HASH="abcdef..."
export OWNER_ID="8584716706"
export ADMIN_IDS="8584716706,6808178075"
export CHANNEL_USERNAME="@channel1"

# === GitHub Actions (WAJIB untuk build asli) ===
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxx"
export GITHUB_OWNER="username-atau-org"
export GITHUB_REPO="nama-repo-build"
export GITHUB_WORKFLOW="build.yml"
# opsional Web2APK:
# export GITHUB_WORKFLOW_WEB="web2apk.yml"
# export GITHUB_REF="main"
```

Token GitHub (classic) butuh scope: **repo** + **workflow**.

```bash
npm start
```

## Workflow yang dibutuhkan di repo

### `build.yml` (Flutter)

```yaml
name: Build Flutter
on:
  workflow_dispatch:
    inputs:
      tag:
        description: Release tag yang berisi source ZIP
        required: true
      build_type:
        description: release atau debug
        default: release

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.32.8'
          channel: stable

      - name: Download source ZIP from release
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          mkdir -p source
          gh release download "${{ inputs.tag }}" --repo ${{ github.repository }} --pattern "*.zip" --dir source
          cd source && unzip -o *.zip
          # sesuaikan kalau struktur folder beda
          if [ -f pubspec.yaml ]; then ROOT=.; else ROOT=$(find . -name pubspec.yaml | head -1 | xargs dirname); fi
          echo "PROJECT_ROOT=$ROOT" >> $GITHUB_ENV

      - name: Flutter build
        run: |
          cd "$PROJECT_ROOT"
          flutter pub get
          if [ "${{ inputs.build_type }}" = "debug" ]; then
            flutter build apk --debug
          else
            flutter build apk --release
          fi

      - uses: actions/upload-artifact@v4
        with:
          name: app-release
          path: "**/build/app/outputs/flutter-apk/*.apk"
```

### `web2apk.yml` (opsional)

Sesuaikan inputs: `url`, `app_name`, `icon_url`.

## Catatan
- `config.js` sudah ada `OUTPUT_DIR` & `LOGS_DIR` (fix error path undefined).
- `db.js` obfuscate lama **tidak dipakai** (index.js pakai DB sendiri).
- Jangan share token ke publik.



### Colored Telegram Buttons

This build uses Teleproto 1.228.5 because it exposes Telegram's native KeyboardButtonStyle (primary/blue, success/green, danger/red) required by the colored MTProto button implementation.



## ✨ Animasi Button + Emoji/Sticker Start

Menu utama sekarang mendukung efek **kelap-kelip warna native Telegram** dan dekorasi start yang dapat diatur dari `config.js`. Animasi dibuat terbatas agar tidak mengganggu callback button atau build.

Atur bagian `BUTTON_BLINK`:
- `enabled`: `true/false`
- `intervalMs`: jeda pergantian warna (minimal aman 1200 ms)
- `cycles`: jumlah kedipan
- `palette`: `success`, `primary`, `danger`

Atur `START_DECORATION`:
- `type`: `emoji`, `sticker`, `both`, atau `none`
- `emojis`: daftar emoji yang dipakai
- `sticker`: path file sticker atau URL yang bisa dibaca Teleproto
- `stickerDelayMs`: jeda setelah sticker dikirim

Contoh:
```js
const START_DECORATION = {
  type: "both",
  emojis: ["✨", "🚀", "🔥", "⚡"],
  sticker: "./assets/start.webp",
  stickerDelayMs: 300,
};
```

## 🎨 Ganti Warna Background Button

Warna button sekarang bisa diubah dengan mudah dari `config.js`, pada bagian `BUTTON_COLORS`.

Pilihan:
- `success` = hijau
- `primary` = biru
- `danger` = merah

Contoh:
```js
BUTTON_COLORS: {
  build: "danger",
  web2apk: "success",
  status: "primary",
}
```

Kunci seperti `build`, `web2apk`, `status`, dll. adalah `data` callback button. Tidak perlu mengubah `index.js`.

## 🎨 Copy Tampilan (OpenAI Vision)

Fitur **Copy Tampilan** berada di menu **Tools**. User mengirim screenshot, bot menganalisisnya memakai OpenAI Vision, lalu mengirim hasil sebagai file `copy.js`.

Isi `OPENAI_API_KEY` di `config.js` dan restart bot. Model default: `gpt-5.6-luna`.

## 🎨 Copy Tampilan — Gemini Vision

Isi `GEMINI_API_KEY` di `config.js`. Tekan **🎨 Copy Tampilan** di Tools, kirim screenshot, lalu bot menganalisisnya dengan Gemini dan mengirim hasil sebagai `copy.js`.


### Gemini 404 fix
Copy Tampilan mencoba endpoint Gemini v1 lalu v1beta dan mengirim API key melalui header `x-goog-api-key`. Model default `gemini-2.5-flash`.

## 🎨 Copy Tampilan — Screenshot + Dart
Alur Copy Tampilan sekarang:
1. Tekan **🎨 Copy Tampilan** di Tools.
2. Kirim 1 screenshot tampilan.
3. Bot membalas **📸 Screenshot diterima! Sekarang kirim file .dart-nya (file .dart apa aja).**
4. Kirim file `.dart` apa pun (mis. `main.dart`, `home.dart`, dll.).
5. Gemini memakai screenshot sebagai target visual dan file Dart sebagai source, lalu bot mengirim hasil sebagai `copy.js`.


## 🎨 Copy Tampilan flow
Screenshot → file `.dart` → pilih output **HTML** atau **Dart**. Output Dart dikirim sebagai `copy.dart`; output HTML sebagai `copy.html`.

## 🛒 ORDER / JUALAN + QRIS

Menu utama sekarang memiliki tombol **🛒 Order** di luar Tools. Tombol **📋 Antrian Build** dan **⚙️ Status Bot** dipindahkan ke **🛠️ Tools**.

### Owner — tambah jualan
1. Buka **🛒 Order**.
2. Pilih **⚙️ Kelola Produk** → **➕ Tambah Jualan**.
3. Kirim nama produk, contoh `SC Build APK`.
4. Kirim harga, contoh `1000` untuk Rp1.000.
5. Kirim file yang dijual.
6. Produk muncul di katalog Order dan bisa dihapus Owner kapan saja.

### User — beli
1. Buka **🛒 Order**.
2. Pilih produk.
3. Bot membuat transaksi QRIS dinamis.
4. User scan QRIS.
5. Bot memantau status pembayaran.
6. Setelah status **settlement**, file produk dikirim otomatis.

### QRIS Midtrans
Isi di `config.js`:

```js
MIDTRANS_SERVER_KEY: "ISI_SERVER_KEY_MIDTRANS",
MIDTRANS_ENV: "sandbox", // ubah ke production untuk transaksi asli
MIDTRANS_QRIS_ACQUIRER: "gopay",
```

Untuk transaksi asli, gunakan Server Key dari akun Midtrans production. Jangan membagikan Server Key.
