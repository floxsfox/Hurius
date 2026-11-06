# BeatDrop (Godot 4.6) – versi APK dari `index.html`

Game ritme: pilih lagu MP3/OGG dari file manager, game membuat not otomatis (generator V1 Beat / V2 Bass / V3 Melody),
lalu dimainkan 5 lajur. Semua dibuat lewat kode, tanpa scene rumit, supaya mudah di-build dari ponsel.

## Cara build APK dari ponsel (GitHub)

**Cara termudah – 3 langkah di browser ponsel:**

1. Buat repo baru di GitHub → **Add file → Create new file**.
   Ketik nama file `.github/workflows/android.yml` (garis miring otomatis membuat folder), tempel isi `android.yml`
   (file ini ada di dalam zip, juga dikirim terpisah), lalu **Commit**.
2. **Add file → Upload files** → pilih `BeatDrop_Godot.zip` (jangan di-extract) → **Commit**.
   Workflow otomatis membuka zip itu, mengimpor proyek, lalu meng-export APK.
3. Tab **Actions** → tunggu centang hijau (±6–10 menit pertama) → buka run-nya → unduh **BeatDrop-apk**
   (atau ambil `BeatDrop.apk` langsung di halaman **Releases → latest**).

Kalau kamu memakai git/Termux, cukup extract zip dan push semua isinya – tidak perlu langkah unzip.

Catatan build
- Versi Godot diatur di `android.yml` (`GODOT_VERSION`, default `4.6.2`). Dialog file manager Android (SAF) butuh **Godot ≥ 4.6**.
- APK ditandatangani dengan kunci yang dibuat otomatis lalu disimpan di cache GitHub, sehingga APK berikutnya bisa
  **menimpa** yang lama tanpa uninstall (lagu & skor tetap). Kalau cache GitHub terhapus (tidak dipakai >7 hari),
  kunci baru dibuat dan Android akan meminta uninstall dulu.
- Template export **release** dipakai (lebih cepat daripada debug).

## Yang sudah dibuat sesuai permintaan

| Permintaan | Implementasi |
|---|---|
| Support file manager | `DisplayServer.file_dialog_show` (SAF Android, tanpa izin storage). Lagu **disalin** ke penyimpanan game (`user://beatdrop/songs`), jadi tetap ada walau file asli dipindah. Pilih banyak lagu sekaligus. Cadangan: `FileDialog` bawaan. |
| FPS stabil, maks 60 | `application/run/max_fps=60` + `Engine.max_fps=60` + VSync. Layer statis digambar sekali (cache), hanya not/partikel digambar ulang. Tanpa alokasi per-frame di jalur panas. |
| Resolusi HD tapi lancar | Game dirender ke `SubViewport` berukuran **tetap 1280×720 (HD)**, apa pun resolusi layar. Pilihan di Settings → Graphics: Saver 540p / **HD 720p** / Sharp 900p / Full HD. **Adaptive quality**: bila FPS turun, resolusi render diturunkan otomatis (maks −30%) lalu dinaikkan lagi kalau stabil. Menu & HUD tetap tajam (resolusi layar asli). |
| Hapus paksa landscape | Overlay "putar layar" dihapus. APK dikunci **sensor-landscape** lewat `project.godot` (bisa dibalik 180°). Notch/gesture bar dihormati (area aman). |
| Penyimpanan | Skor terbaik, statistik, prestasi, warna, pengaturan, library & sampul lagu → `user://beatdrop/*.json`. Disimpan otomatis dan saat app di-pause. |
| Cache | Hasil analisis audio disimpan per lagu (`cache/*.bin`) – analisis hanya sekali per lagu & generator. Musik menu diunduh sekali lalu dicache (`cache/bgm.mp3`). Settings → Graphics → Storage: lihat ukuran & hapus cache. |
| Offline & online | Semua fitur jalan offline. Online hanya dipakai untuk mengunduh musik menu (otomatis, diulang tiap 14 hari). Tanpa internet → pakai cache; belum ada cache → menu tanpa musik. |

## Struktur

```
project.godot / export_presets.cfg / main.tscn
scripts/
  main.gd        orkestrasi: layar, input sentuh, file manager, analisis, siklus hidup Android
  game.gd        logika game: jam audio, judge, skor, HP, partikel, kualitas adaptif
  layers.gd      gambar lajur/not/partikel (proyeksi kamera miring tanpa buffer offscreen)
  hud.gd         skor, akurasi, HP, kombo, hitung mundur
  analyzer.gd    decode audio + generator not V1/V2/V3 (multi-thread + cache)
  store.gd       (autoload) penyimpanan, library, cache, statistik, prestasi, ID3 + sampul
  sfx.gd         (autoload) efek suara hasil sintesis, musik menu online+cache, bus audio
  s_*.gd, w_*.gd, ui.gd   layar & widget UI (dibuat lewat kode)
assets/ui/*.png  ikon      assets/icon_*.png  ikon adaptif Android
```

## Perbedaan dibanding versi web (jujur)

- **Analisis lagu** didekode native oleh Godot lalu dianalisis di thread latar dengan sample-rate diturunkan
  (4×). Pertama kali per lagu bisa 5–40 detik tergantung HP dan generator (V3 paling lama); setelah itu instan dari cache.
  Proses berjalan di thread, jadi UI tetap 60 FPS saat loading.
- Format lagu: **MP3 dan OGG**. M4A/AAC/FLAC tidak didukung Godot.
- Menu 3D-carousel disederhanakan menjadi kartu; efek blur latar tidak ada (lebih ringan).
- Intro: animasi logo. Mau video? Konversi ke **Ogg Theora** dan simpan sebagai `assets/intro.ogv` (otomatis dipakai).
- Musik menu: bila mau selalu ada offline sejak instalasi pertama, taruh `bgm_menu.mp3` di `assets/audio/`.
- Sinkron audio: memakai `AudioServer.get_output_latency()`; masih terasa meleset? Atur **Audio sync** di layar lagu (ms).
- Tombol Back Android: di game = jeda; di menu = tekan dua kali untuk keluar.

## Kontrol

Sentuh 5 lajur di bagian bawah. Uji di PC: `D F Space J K`, `Esc` = jeda.

## Kalau build gagal

Buka run di tab **Actions** → langkah yang merah → salin 30 baris terakhir lognya.
Kegagalan paling umum: nama versi Godot di `GODOT_VERSION` belum ada di godot-builds (ganti ke versi 4.6.x yang ada),
atau ada error skrip (log akan menyebut file dan nomor baris).
