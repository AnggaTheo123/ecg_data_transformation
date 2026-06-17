# EKG-BRIN — Digitalisasi EKG 12-Lead → Sinyal mV → FHIR

Mengubah **EKG 12-lead dari PDF** menjadi **sinyal digital (mV per-lead)**, lalu
**FHIR R4 Bundle** yang siap dikirim ke SatuSehat — plus **skrining AI** dan
**cek-mutu otomatis**. Dirancang agar bisa langsung dicoba: ada 11 PDF contoh di
dalam repo.

```
  INPUT                  PROSES                          OUTPUT
 ┌──────────┐   ┌────────────────────────────┐   ┌────────────────────────┐
 │ PDF EKG  │ → │ 1. Baca vektor PDF (eksak)  │ → │ <nama>_digital.json    │ sinyal mV + AI + mutu
 │ 12-lead  │   │ 2. Decode mV per-lead       │   │ <nama>_fhir.json       │ FHIR R4 Bundle
 │          │   │ 3. Skrining AI (ECGNet)     │   │ <nama>_compare.png      │ ASLI vs DIGITAL
 │          │   │ 4. Cek-mutu Einthoven       │   └────────────────────────┘
 └──────────┘   └────────────────────────────┘
```

Ada **dua jalur** digitalisasi dalam repo ini:

| Jalur | Entry point | Kapan dipakai | Butuh model? |
|---|---|---|---|
| **Vektor (utama)** | `proses_ekg.py` | PDF EKG asli (grafik = kurva vektor) | tidak |
| U-Net (cadangan) | `training/run_ecg.py` | PDF/foto hasil scan (grafik = piksel) | ya (`unet_best.pt`) |

Jalur **vektor** membaca koordinat kurva langsung dari PDF, jadi sangat akurat dan
tidak butuh GPU. Jalur **U-Net** untuk kasus gambar/scan yang tidak punya data vektor.

---

## 🚀 Cara tercepat mencoba (3 langkah)

```bash
# 1) install dependency
pip install -r requirements.txt

# 2) proses SATU PDF contoh (auto-deteksi 12-lead)
python proses_ekg.py --input 12-lead/20260604-153533/20251029-153616-0001.pdf

# 3) atau proses SEMUA 11 PDF contoh sekaligus
python proses_ekg.py --folder 12-lead
```

Hasil masuk ke `hasil/12-lead/`:

| File | Isi |
|---|---|
| `<nama>_digital.json` | sinyal mV 12-lead + skrining AI + ukuran klinis device + flag mutu |
| `<nama>_fhir.json` | **FHIR R4 Bundle** siap-SatuSehat |
| `<nama>_compare.png` | gambar **PDF asli vs hasil digital** untuk cek visual |

> **PyTorch opsional.** Tanpa `torch`, digitalisasi & FHIR tetap jalan — hanya
> skrining AI yang dilewati (ada pesan `[INFO]`). Pasang `torch` untuk mengaktifkannya.

---

## 📁 Struktur folder

```
EKG-BRIN-public/
├── proses_ekg.py          ⭐ ENTRY UTAMA — PDF 12-lead → digital + FHIR + compare
├── run_any.py             pipeline bersama (decode mV, AI, cek-mutu)
├── render_any.py          util: render JSON hasil jadi gambar
├── requirements.txt
│
├── 12-lead/               11 PDF EKG contoh (de-identified: ID urut saja)
├── Data_Siap_FHIR/        contoh JSON hasil + panduan pemetaan field → FHIR R4
│
├── input/                 parser & router input
│   └── parsers/           vector_pdf.py (baca vektor), parse_pdf/png/csv/xml/wfdb, dll.
├── output/                fhir_converter.py (JSON universal → FHIR Bundle)
│
└── training/              jalur U-Net (digitalisasi gambar/scan + pelatihan)
    ├── run_ecg.py         entry U-Net
    ├── digitize_real.py, decode.py, unet.py, train.py, synth_generator.py
    ├── checkpoints/unet_best.pt   (model segmentasi trace, ~30 MB)
    └── classify/          ECGNet skrining 5-superclass (ecgnet_best.pt + norm_stats.npz)
```

---

## 🧠 Jalur U-Net (untuk PDF/foto hasil scan)

Dipakai bila grafik EKG berupa **piksel** (scan/foto), bukan vektor PDF.

```bash
cd training
python run_ecg.py --pdf "C:/path/ekg_scan.pdf"     # satu file
python run_ecg.py --folder "C:/folder_pdf"          # banyak file
```

Melatih ulang model perlu dataset **PTB-XL** (~3 GB, dari
[PhysioNet](https://physionet.org/content/ptb-xl/)) — tidak ikut di repo, lihat
`.gitignore`. Pembuatan dataset sintetik: `training/synth_generator.py`.

---

## 📊 Hasil validasi (11 EKG contoh)

- **Digitalisasi (vektor):** Einthoven RMSE ~0.004 mV (limb-lead konsisten secara fisika)
- **Digitalisasi (U-Net, sintetik ground-truth):** Pearson r **0.955**
- **Heart rate vs device:** MAE **1.0 bpm**
- **Konsistensi ritme vs diagnosis device:** **100%** (10/10)
- **Skrining AI (ECGNet, PTB-XL test):** macro-AUC **0.925**

Detail metrik: lihat [EVALUASI_METRIK.md](EVALUASI_METRIK.md).

---

## 🧾 Output untuk tim FHIR

Folder [`Data_Siap_FHIR/`](Data_Siap_FHIR/) berisi contoh JSON + `README.md` berisi
**panduan pemetaan field → FHIR R4**. Tim FHIR cukup melakukan mapping; tidak perlu
menyentuh kode engine. Skema JSON digital: `ekg-brin/processed-ecg/v1`
(patient, recording, leads{mV, confidence}, printed_measurements, quality, ai_screening).

---

## ⚙️ Detail teknis

- **Kalibrasi:** 25 mm/s, 10 mm/mV (standar EKG). Output disampling **250 Hz**, durasi ~7.4 s/lead.
- **Highpass 0.5 Hz** untuk menghilangkan baseline wander.
- **Cek-mutu Einthoven/Goldberger:** III=II−I, aVR=−(I+II)/2, dst. RMSE besar ⇒ flag "perlu ditinjau".
- **Skrining AI:** ECGNet 1D-ResNet multi-label 5 superclass (NORM/MI/STTC/CD/HYP).

---

## ⚠️ Status & batasan

Prototipe **tervalidasi pada data uji** — untuk **penelitian / pengembangan /
integrasi FHIR**, **bukan untuk diagnosis klinis ke pasien**. Skrining AI =
**skrining eksperimental**, bukan diagnosis final. Penggunaan klinis masih perlu:
validasi lebih luas (banyak merek device), review kardiolog, dan jalur regulasi.

PDF contoh sudah **de-identified** (hanya ID urut + jenis kelamin).

## Lisensi
MIT — lihat [LICENSE](LICENSE).
