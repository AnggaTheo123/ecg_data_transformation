# Evaluasi Metrik Digitalisasi EKG terhadap Standar

Dokumen ini merangkum nilai metrik hasil digitalisasi untuk data yang ditinjau,
beserta penilaiannya terhadap standar/target yang ditetapkan. Mencakup tiga
kelompok data: 6-lead vektor (AliveCor/Kardia), 12-lead vektor (EDAN SE-301,
Export), dan citra 12-lead 3x4 (dataset publik Mendeley, alat EDAN SE-3).

## 1. Standar dan Target Acuan

| Kode | Kriteria | Ambang | Acuan |
|------|----------|--------|-------|
| S1 | Toleransi posisi temporal kompleks QRS | <= +/- 150 ms | ANSI/AAMI EC38/EC57 |
| S2 | Toleransi amplitudo kalibrasi | <= +/- 5% | IEC 60601-2-25 |
| S3 | Korelasi Pearson terhadap sinyal acuan | >= 0.94 | PMC9572306 |
| S4 | Signal-to-Noise Ratio (SNR) | > 12 dB | arXiv 2604.00396 |

## 2. Hasil per-Record

### 2.1 6-lead vektor (AliveCor/Kardia)

| Record | SNR (dB) | PRD (%) | Pearson r | WDD (%) |
|--------|----------|---------|-----------|---------|
| DH_6L-0425 | 40.9 | 0.97 | 1.0000 | 0.55 |
| PY_6L-20260525 | 44.7 | 0.66 | 1.0000 | 0.37 |
| RR_6L-20260525 | 43.9 | 0.69 | 1.0000 | 1.17 |
| Rata-rata | 43.2 | 0.77 | 1.0000 | 0.70 |

### 2.2 12-lead vektor (EDAN SE-301, Export)

| Record | SNR (dB) | PRD (%) | Pearson r | WDD (%) |
|--------|----------|---------|-----------|---------|
| 20251029-153616-0001 | 29.2 | 3.62 | 0.9993 | 2.07 |
| 20251029-154814-0002 | 14.7 | 19.84 | 0.9765 | 3.20 |
| 20251118-142345-0003 | 23.2 | 7.16 | 0.9973 | 1.15 |
| Rata-rata (n=11 set penuh) | 23.5 | 6.24 | 0.9954 | 1.6 |

Catatan: record 0002 merupakan rekaman limb-only (V1-V6 datar pada cetakan asli),
sehingga PRD relatif tinggi karena amplitudo limb-lead kecil; korelasi dan SNR
tetap memenuhi ambang.

### 2.3 Citra 12-lead 3x4 (Mendeley, EDAN SE-3)

Dataset citra hanya menyediakan gambar dan label diagnosis, tanpa sinyal digital
acuan. Validasi dilakukan terhadap nilai tercetak alat dan konsistensi fisiologis.

| Record | HR cetak (bpm) | HR digital (bpm) | Selisih HR | Konsistensi Einthoven (RMSE, mV) |
|--------|----------------|------------------|------------|----------------------------------|
| Normal(100) | 81 | 81 | 0 | 0.026 |
| HB(100) | 98 | 98 | 0 | 0.030 |
| MI(100) | 62 | 62 | 0 | 0.017 |

## 3. Penilaian terhadap Standar

| Standar | 6-lead vektor | 12-lead vektor | Citra 3x4 |
|---------|---------------|----------------|-----------|
| S1 Temporal QRS +/-150 ms | Memenuhi (pembacaan koordinat, galat ~0 ms) | Memenuhi (galat ~0 ms) | Memenuhi (HR digital = HR cetak, selisih 0 bpm) |
| S2 Amplitudo +/-5% | Memenuhi (pulsa kalibrasi 1 mV = 28.346 pt, galat 0%) | Memenuhi (galat 0%) | Bersyarat (kalibrasi dari pulsa 1 mV ~1-2%; dari lebar kolom ~6%) |
| S3 Pearson r >= 0.94 | Memenuhi (r = 1.0000) | Memenuhi (r = 0.9954; minimum 0.9765) | Tidak dapat dihitung (tidak ada sinyal acuan) |
| S4 SNR > 12 dB | Memenuhi (43.2 dB; minimum 40.9 dB) | Memenuhi (23.5 dB; minimum 14.7 dB) | Tidak dapat dihitung (tidak ada sinyal acuan) |

## 4. Kesimpulan

1. Data utama (6-lead dan 12-lead vektor) memenuhi keempat standar (S1-S4),
   dengan margin yang lebar di atas ambang. Korelasi mendekati sempurna
   (r >= 0.976), SNR di atas 12 dB pada seluruh record, kalibrasi amplitudo
   dan posisi temporal eksak karena sinyal dibaca langsung dari koordinat vektor.

2. Citra 12-lead 3x4 memenuhi standar temporal (S1) melalui kecocokan denyut
   jantung digital dengan nilai tercetak (selisih 0 bpm pada tiga record).
   Standar amplitudo (S2) tercapai bila kalibrasi diambil dari pulsa kalibrasi
   1 mV. Standar S3 dan S4 tidak dapat dihitung karena dataset citra tidak
   menyertakan sinyal digital acuan; validasi citra bersandar pada nilai tercetak
   alat dan konsistensi hukum Einthoven (RMSE <= 0.030 mV).

## 5. Catatan Metodologi

- SNR (dB) = 10 log10( Sum(r - mean)^2 / Sum(r - s)^2 ); r = sinyal acuan, s = sinyal uji.
- PRD (%) = 100 * sqrt( Sum(r - s)^2 / Sum(r - mean)^2 ).
- WDD = distorsi fitur diagnostik (HR dan amplitudo-R) terhadap beat tercocokkan.
- Untuk jalur vektor, sinyal acuan adalah trace vektor asli (di-render pada
  resolusi tinggi); sinyal uji adalah keluaran pipeline pada 250 Hz.
- Untuk citra, sinyal digital acuan tidak tersedia pada dataset; metrik per-sampel
  (S3, S4) karena itu tidak dapat dihitung dan ditandai sebagaimana adanya.
- Acuan amplitudo dan waktu pada citra mengikuti footer tercetak alat
  (25 mm/s, 10 mm/mV) dan denyut jantung tercetak.
