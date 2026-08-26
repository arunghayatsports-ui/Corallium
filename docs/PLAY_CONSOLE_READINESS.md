# Corallium Flow — Kesiapan Google Play Console

> Semakan pada **26 Ogos 2026**.
>
> **Nota penting:** dokumen ini ialah peta keperluan Play Console, **bukan** audit projek.
> Ia ditulis dalam sesi yang berjalan pada klon repositori ini, yang setakat ini
> mengandungi `README.md` sahaja — tiada kod Android. Tiada satu pun item di bawah
> disahkan terhadap kod Corallium Flow yang sebenar.

## Jawapan ringkas

**Tidak — kerja kod tidak meliputi semua yang ada dalam Play Console.**

Play Console ada dua lapisan berasingan:

1. **Lapisan aplikasi** — `targetSdk`, AAB, penandatanganan, pustaka bil, kebenaran,
   kestabilan. Inilah yang disentuh oleh kerja kod.
2. **Lapisan Console** — borang dasar, deklarasi, aset kedai, persediaan monetisasi,
   trek ujian. **Tiada satu pun boleh diselesaikan dengan menulis kod.**

Bagi kebanyakan aplikasi, lapisan kedua inilah yang melambatkan pelancaran.

---

## Tiga sekatan teknikal

| Keperluan | Tarikh kuat kuasa | Lanjutan |
|---|---|---|
| Sasar Android 16 (API 36) — aplikasi baharu dan kemas kini | **31 Ogos 2026** | 1 Nov 2026 |
| Play Billing Library 8.0.0+ | **31 Ogos 2026** | 1 Nov 2026 |
| Sokongan saiz halaman memori 16 KB (aplikasi sasar Android 15+) | 1 Nov 2025 | sudah berkuat kuasa |

Lanjutan bagi dua item pertama dipohon melalui borang dalam Play Console.

---

## Peta jurang

Legenda: **[Kod]** selesai dari sisi aplikasi · **[Kod+Console]** perlu kedua-duanya dan
mesti sepadan · **[Console]** kod tidak boleh sentuh langsung.

### A. Lapisan aplikasi

- **[Kod]** Format AAB dan Play App Signing
- **[Kod]** Integrasi Play Billing dalam aplikasi (aliran beli, sahkan resit, pulih pembelian)
- **[Kod]** Kestabilan, ProGuard/R8, kebenaran minimum

### B. App content — borang dasar (semua wajib)

- **[Console]** Dasar privasi — URL awam, boleh dicapai tanpa log masuk
- **[Kod+Console]** **Borang Data safety** — punca penolakan paling kerap. Mesti sepadan
  dengan kelakuan sebenar aplikasi, termasuk data yang dikumpul oleh SDK pihak ketiga
- **[Console]** Penilaian kandungan (soal selidik IARC)
- **[Console]** Target audience and content — jika ada kumpulan kanak-kanak, dasar Families terpakai
- **[Console]** App access — akaun ujian yang berfungsi jika aplikasi perlu log masuk
- **[Console]** Deklarasi iklan; kebenaran `AD_ID` jika `targetSdk` 33+
- **[Kod+Console]** Penghapusan akaun — laluan dalam aplikasi **dan** URL web
- **[Kod+Console]** Deklarasi bersyarat: jenis foreground service (Android 14+), akses semua
  fail, kebenaran foto/video, `QUERY_ALL_PACKAGES`, ciri kewangan, kesihatan, berita

### C. Penyenaraian kedai

- **[Console]** Nama (30 aksara), penerangan ringkas (80), penerangan penuh (4000); `ms-MY` + `en-US`
- **[Console]** Ikon 512×512 PNG, grafik ciri 1024×500, 2–8 tangkapan skrin telefon,
  tangkapan skrin tablet 7" dan 10"
- **[Console]** Kategori, tag, e-mel sokongan (dipaparkan awam), tapak web

### D. Monetisasi — sisi Console

- **[Console]** Profil pembayaran pedagang — pengesahan mengambil masa berhari-hari, mulakan awal
- **[Console]** Tetapan cukai dan pematuhan (termasuk layanan cukai Malaysia)
- **[Kod+Console]** Produk dalam aplikasi dan langganan — setiap ID dalam kod mesti wujud
  di Console **dan** berstatus aktif; base plan, tawaran, tempoh tangguh, account hold
- **[Console]** Harga bagi setiap wilayah yang diedarkan
- **[Console]** Penguji lesen — satu-satunya cara menguji pembelian tanpa dicaj
- **[Kod+Console]** Pemberitahuan pembangun masa nyata (topik Pub/Sub)

### E. Ujian dan pelancaran

- **[Console]** **Ujian tertutup: 12 penguji, 14 hari berterusan.** Terpakai kepada akaun
  peribadi yang dibuka selepas 13 Nov 2023. Jam berulang semula jika penguji jatuh bawah 12.
  Sejak 2026 Google turut menyemak penggunaan sebenar, bukan sekadar pendaftaran
- **[Console]** Permohonan akses produksi — borang berasingan selepas syarat 12/14 dipenuhi
- **[Kod+Console]** Pre-launch report — semak amaran ranap, kebolehcapaian, keselamatan
- **[Kod+Console]** Android vitals — ambang kelakuan buruk: ranap 1.09%, ANR 0.47%

---

## Susunan kerja

Jam 14 hari ialah item terpanjang, jadi ia perlu bermula seawal mungkin — tetapi ia hanya
boleh bermula selepas AAB yang sah berjaya dimuat naik.

1. **Hari ini** — Sahkan dua nombor: `targetSdk` dan versi Play Billing Library.
   Jika tidak sempat sebelum 31 Ogos, mohon lanjutan sekarang. Buka juga **Policy status**
   dalam Console; ia menyenaraikan setiap isu terbuka secara terus.
2. **Minggu ini** — Naik taraf ke API 36 dan PBL 8. Migrasi PBL 7→8 mengubah beberapa API
   langganan, jadi uji semula aliran pembelian sepenuhnya, bukan hanya kompil.
3. **Selari** — Mulakan profil pembayaran pedagang. Pengesahan tidak boleh dipercepatkan.
4. **Selepas (2)** — Muat naik AAB ke trek ujian tertutup dan mulakan jam 14 hari.
   Ambil 15–16 penguji, bukan 12 tepat. Galakkan penggunaan sebenar.
5. **Semasa 14 hari** — Habiskan semua borang App content. Beri masa paling banyak kepada
   Data safety: senaraikan setiap SDK dahulu, kemudian isytihar data setiap satunya.
6. **Semasa 14 hari** — Siapkan aset kedai; cipta dan aktifkan setiap produk; tambah penguji
   lesen dan lakukan satu pembelian ujian hujung ke hujung bagi setiap produk.
7. **Selepas 14 hari** — Mohon akses produksi. Jawab dengan terperinci: apa yang diuji,
   maklum balas apa yang diterima, apa yang berubah hasilnya.
8. **Pelancaran** — Berperingkat pada 10–20%, perhati Android vitals, kemudian naikkan.

---

## Untuk semakan sebenar terhadap kod

Salah satu daripada ini diperlukan:

- Tolak kod Corallium Flow ke repositori GitHub — kemudian `build.gradle`,
  `AndroidManifest.xml` dan integrasi bil boleh dibaca terus.
- Atau sambung semula sesi Claude Code di komputer tempatan tempat kod itu berada.
- Untuk jurang sisi Console: tangkapan skrin halaman **Policy status** dan **Dashboard**
  sudah memadai — kedua-duanya menyenaraikan item tertunggak secara eksplisit.

---

## Rujukan

- [Keperluan target API level](https://developer.android.com/google/play/requirements/target-sdk)
- [Penamatan versi Play Billing Library](https://developer.android.com/google/play/billing/deprecation-faq)
- [Sokongan saiz halaman 16 KB](https://developer.android.com/guide/practices/page-sizes)
- [Keperluan ujian akaun peribadi](https://support.google.com/googleplay/android-developer/answer/14151465)
