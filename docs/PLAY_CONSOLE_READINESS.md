# Corallium Flow — audit kesiapan Play Console

Audit terhadap kod sebenar dalam `arunghayatsports-ui/corallium-test`,
pada `main` = `397a50b` (26 Ogos 2026, 11:17 +08).

Seni bina: Next.js + Capacitor, WebView jauh ke `https://www.corallium.my/dashboard/flow`.
`applicationId` `my.corallium.flow`, versionCode 4, versionName 1.3.

---

## Pembetulan kepada versi terdahulu dokumen ini

Versi pertama dokumen ini ditulis tanpa akses kepada kod, dan menyenaraikan dua
tarikh mati 31 Ogos 2026 sebagai mendesak. **Kedua-duanya tidak terpakai:**

- `targetSdkVersion = 36` sudah ditetapkan dalam `android/variables.gradle`.
- Tiada integrasi Play Billing langsung dalam repo — carian `billingclient`
  pulang kosong. Keperluan Play Billing Library 8 tidak berkenaan.

Dokumen `MIGRASI_31_OGOS_2026.md` telah dipadam atas sebab yang sama.

---

## Sudah disahkan siap

| Perkara | Bukti dalam kod |
|---|---|
| Target API 36 | `android/variables.gradle`: `compileSdkVersion = 36`, `targetSdkVersion = 36` |
| Saiz halaman 16 KB | AGP 8.13.0 + Gradle 8.14.3; tiada `.so` dikomit. Toolchain moden menjajarkan 16 KB secara lalai |
| Tiada Play Billing untuk dimigrasi | Tiada rujukan `billingclient` di seluruh repo |
| Gerbang penandatanganan | `android/app/build.gradle` — `gradle.taskGraph.whenReady` menggagalkan `bundleRelease` tanpa `key.properties`, dengan `signingConfig` tanpa syarat sebagai jaring kedua |
| Sandaran sesi ditutup | `allowBackup="false"` — menghalang kuki `app_webview/` dipulihkan dalam keadaan sudah log masuk |
| Halaman padam akaun | `app/padam-akaun/page.tsx` — memenuhi keperluan URL web Play |
| Dasar privasi | `app/privasi/page.tsx` + `lib/i18n/legal/privacy-content.ts` |
| Mitigasi dasar pembayaran | `components/flow/web-purchase-guard.tsx`, diguna pada `flow-pricing-page.tsx` dan `flow-plan-limit-notice.tsx` |
| Manifest bersih | Sumber `AndroidManifest.xml` mengisytihar `INTERNET` sahaja |

Penyekat dasar privasi yang dicatat dalam `docs/FLOW_ANDROID_RELEASE.md`
(placeholder `[ISI NAMA ENTITI]` dan `No. SSM [ISI]`) **sudah tiada** —
carian pada `lib/i18n/legal/` tidak menjumpai satu pun placeholder.
Catatan penyekat itu kini basi.

---

## Jurang A — dokumen keupayaan asli sudah basi, dan ia menyuap borang Data safety

`docs/FLOW_ANDROID_RELEASE.md` menyenaraikan keupayaan asli setakat
**versionCode 2 / versionName 1.1**, dan menandakan push notification sebagai
**TIADA** — "Memerlukan Firebase; belum dibina". Kesimpulannya:

> Jadi manifes kekal `INTERNET` sahaja, dan **tiada pengisytiharan kebenaran
> diperlukan dalam Play Console**.

Itu tidak lagi benar. Kod sekarang:

- `android/app/capacitor.build.gradle` mengkompil `implementation project(':capacitor-push-notifications')`
- `lib/flow/platform/push.ts` melaksanakan pendaftaran FCM sepenuhnya
- Aplikasi kini pada versionCode 4 / versionName 1.3

Kebenaran itu **tidak** datang daripada manifest plugin. Manifest
`@capacitor/push-notifications` sendiri hanya mengisytihar satu
`MessagingService` — tiada `uses-permission` langsung. Rantaian sebenar melalui
kebergantungan:

```
:capacitor-push-notifications
  └── com.google.firebase:firebase-messaging
        └── AndroidManifest.xml mengisytihar:
              ACCESS_NETWORK_STATE
              POST_NOTIFICATIONS
              WAKE_LOCK
```

Ketiga-tiganya bergabung ke dalam manifest akhir. Sumber `AndroidManifest.xml`
memang kekal `INTERNET` sahaja — tetapi artifak yang dimuat naik ke Play membawa
**empat** kebenaran.

Ia juga tidak bergantung pada `google-services.json`. Fail itu mengawal sama ada
plugin Gradle `com.google.gms.google-services` digunakan; kebergantungan
`firebase-messaging` masuk tanpa syarat melalui
`implementation project(':capacitor-push-notifications')`. Jadi kebenaran itu ada
dalam artifak walaupun push tidak pernah berfungsi.


Ini tepat kegagalan yang dokumen itu sendiri beri amaran — "Dokumen itu
mendahului kod, dan borang Play Console yang diisi daripadanya akan mengandungi
dakwaan yang artifak tidak sokong" — cuma kini arahnya terbalik: dokumen
**kurang** melaporkan apa yang artifak bawa.

**Tindakan:** sahkan manifest tergabung, bukan sumbernya, sebelum mengisi
sebarang borang:

```bash
cd android && ./gradlew :app:processReleaseManifest
# kemudian baca app/build/intermediates/merged_manifests/release/AndroidManifest.xml
```

Kemas kini jadual dalam `FLOW_ANDROID_RELEASE.md` supaya sepadan versionCode 4.

---

## Jurang B — push ada dalam artifak tetapi tidak dikonfigur

`google-services.json` tiada dalam repo. Tanpanya,
`com.google.gms.google-services` tidak digunakan dan pendaftaran FCM gagal —
`push.ts` menelan kegagalan itu secara senyap, dengan sengaja.

Kesan bersihnya: aplikasi menghantar kebenaran notifikasi untuk ciri yang tidak
pernah berfungsi. Pengguna melihat gesaan; tiada notifikasi pernah tiba.

**Putuskan sebelum penghantaran — dua pilihan sahaja:**

1. **Tambah `google-services.json`** dan hidupkan push. Maka token FCM ialah
   pengecam peranti, dan **mesti** diisytihar dalam borang Data safety
   (kategori: Device or other IDs), berserta perkongsian kepada Google.
2. **Buang plugin push** daripada binaan. `POST_NOTIFICATIONS` kemudian keluar
   daripada manifest tergabung, dan borang Data safety kekal lebih ringkas.

Menghantar dalam keadaan sekarang — plugin ada, konfigurasi tiada — ialah
pilihan paling teruk daripada kedua-duanya.

---

## Disahkan selamat — laluan bayar dalam bungkusan native

Versi terdahulu menyenaraikan ini sebagai satu jurang, dengan alasan
`allowNavigation` membenarkan `*.billplz.com` di peringkat hos dan kerana itu
tidak dapat membezakan kutipan pelanggan (dikecualikan) daripada bayaran
langganan Flow. Pengesahan terhadap kod menunjukkan kebimbangan itu tidak
berasas:

- **Tiada laluan checkout langganan Flow wujud.** Satu-satunya laluan checkout
  Flow ialah `app/api/flow/collect/checkout/route.ts` — pelanggan perniagaan
  membayar pemilik, iaitu perkhidmatan dunia sebenar yang dikecualikan daripada
  Play Billing.
- **CTA pelan Pro ialah WhatsApp dan e-mel**, bukan checkout. Dalam
  `flow-pricing-page.tsx` seluruh blok itu — badan *dan* butang — dibalut
  `WebPurchaseGuard`, bukan butang sahaja.
- **Gantian native tidak menyebut tempat mahupun cara membeli:** "Naik taraf
  tidak tersedia dalam aplikasi ini." Komen dalam
  `lib/i18n/flow-dashboard-modules/entitlement.ts` menyatakan itu disengajakan.
- Dasar privasi mengesahkan modelnya: yuran langganan dibayar kepada Corallium
  Tech **melalui invois** dengan pindahan bank atau FPX, bukan checkout layan
  diri.

Jadi `*.billplz.com` dalam `allowNavigation` hanya melayani aliran kutipan yang
dikecualikan. Tiada jemputan membeli, tiada URL, tiada kaedah dalam bungkusan
native.


## Jurang C — Minimum Functionality

`capacitor.config.ts` sendiri mencatat risiko ini:

> aplikasi yang hanya membungkus laman web tanpa keupayaan asli tertakluk kepada
> dasar "Minimum Functionality" Google Play dan berisiko ditolak

Mitigasinya sudah ada — kamera, share, dan push ialah keupayaan asli sebenar.
Sediakan nota kepada penyemak dalam medan **App access** yang menyebut secara
khusus di mana keupayaan asli itu digunakan (butang "Ambil foto" pada bukti
kerja, "Hantar pautan" pada sebut harga), supaya penyemak tidak melihatnya
sebagai pelayar semata-mata.

---

## Yang tinggal di sisi Console

Tiada satu pun boleh diselesaikan dengan kod:

- [ ] Borang **Data safety** — bergantung pada keputusan Jurang B
- [ ] **Penilaian kandungan** (soal selidik IARC)
- [ ] **Target audience and content**
- [ ] **App access** — akaun ujian berfungsi + nota keupayaan asli (Jurang C)
- [ ] **Aset kedai** — ikon 512×512, grafik ciri 1024×500, 2–8 tangkapan skrin telefon, tangkapan skrin tablet 7" dan 10"
- [ ] **Ujian tertutup** — 12 penguji, 14 hari berterusan (akaun peribadi selepas 13 Nov 2023); sejak 2026 Google turut menyemak penggunaan sebenar
- [ ] **Permohonan akses produksi** selepas syarat 12/14 dipenuhi
- [ ] **Pre-launch report** dan ambang Android vitals (ranap 1.09%, ANR 0.47%)

---

## Nota kecil

`minifyEnabled false` pada buildType release. Bukan penyekat Play, tetapi ia
bermakna tiada R8 — saiz artifak lebih besar dan tiada obfuskasi. Untuk shell
WebView yang hampir tiada logik Java, kesannya kecil.

Dasar privasi menamakan entiti dengan lengkap — Corallium Tech,
No. Pendaftaran SPA/2026/6433, berkuat kuasa 24 Ogos 2026, dwibahasa, dan
menyebut Corallium Flow secara khusus. `FLOW_TERMS_VERSION` (`"2026-08-24"`)
sepadan dengan tarikh berkuat kuasa itu.
