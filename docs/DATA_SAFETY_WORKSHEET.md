# Borang Data safety — jawapan diterbitkan daripada kod

Diterbitkan daripada `arunghayatsports-ui/corallium-test` pada `main` = `397a50b`.
Setiap baris dirujuk kepada jadual, lajur atau fail sebenar.

Borang Data safety ialah punca penolakan Play yang paling kerap, kerana ia mesti
sepadan dengan kelakuan sebenar aplikasi. Dokumen ini menyediakan jawapannya
supaya borang itu diisi daripada kod, bukan daripada ingatan.

> **Dua jawapan bergantung pada keputusan yang belum diambil.** Lihat bahagian
> "Bergantung pada suis" di bawah sebelum mengisi.

---

## Penemuan utama: tiada SDK penjejak langsung

Senarai `dependencies` tidak mengandungi satu pun SDK analitik, iklan, atau
pelaporan ranap. Tiada Google Analytics, Firebase Analytics, Sentry, Mixpanel,
PostHog, atau rangkaian iklan.

Kesannya pada borang: **tiada** data dikumpul untuk pengiklanan atau analitik
pihak ketiga, dan **tiada** advertising ID. Itu memudahkan borang dengan
ketara — kebanyakan penolakan Data safety berpunca daripada SDK yang pembangun
sendiri lupa ia ada.

---

## Nota penting sebelum mengisi

Sebahagian besar data yang aplikasi ini simpan bukan tentang **pengguna**
aplikasi, tetapi tentang **pelanggan pengguna** — nama, telefon dan alamat orang
yang diberi sebut harga atau invois oleh pemilik perniagaan.

Play tetap mengira ini sebagai data yang dikumpul. Isytiharkannya. Jangan
tinggalkan kerana "itu data pelanggan dia, bukan data dia".

---

## Data dikumpul

### Personal info

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Name | Ya | `profiles.full_name` (pengguna aplikasi); `flow_customers.name` (pelanggan pengguna) |
| Email address | Ya | `profiles.email`; `flow_customers.email` |
| Phone number | Ya | `flow_customers.phone`, `flow_customers.phone_normalized` |
| Address | Ya | `flow_customers.address` |
| Other info | Ya | `flow_customers.company_name`, `flow_customers.notes` |

Tujuan: App functionality. Dikongsi: nilai mengikut bahagian Pihak ketiga di bawah — kemungkinan besar TIDAK.
Diperlukan atau pilihan: nama diperlukan (`name TEXT NOT NULL`), selebihnya
pilihan.

### Financial info

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Purchase history | Ya | `flow_invoices`, `flow_quotes`, `flow_payments`, `flow_billing_events` |
| Other financial info | Ya | `flow_gateway_payments` — keadaan bil, jumlah, rujukan |

**Nombor kad atau bank TIDAK dikumpul.** Bayaran berlaku pada gerbang Billplz;
aplikasi merekod status bil dan jumlah sahaja. Jangan tanda "Credit card,
debit card, or bank account number".

### Photos and videos

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Photos | Ya | `flow_job_evidence.storage_key` — foto bukti kerja |

Ditangkap melalui `@capacitor/camera`, dimampatkan 1600px pada peranti
(`lib/flow/platform/capture-photo.ts`), dimuat naik ke Supabase Storage.
`saveToGallery: false`, jadi tiada kebenaran storan diminta.

### App activity

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Other actions | Ya | `flow_events`, `flow_workflow_runs`, `flow_agent_runs` |

Ini log operasi dalam pangkalan data sendiri, bukan analitik pihak ketiga.

### Yang TIDAK dikumpul

Tandakan "No" dengan yakin untuk semua ini — tiada kod menyentuhnya:

- Location (kasar mahupun tepat)
- Contacts, Calendar, SMS, Call logs
- Health and fitness
- Advertising ID — tiada SDK iklan, jadi **jangan** isytihar kebenaran `AD_ID`
- Crash logs dan diagnostics — tiada SDK pelaporan ranap
- Installed apps, Files and docs (selain foto bukti di atas)

---

## Bergantung pada suis — dua jawapan belum boleh dimuktamadkan

### 1. Device or other IDs — bergantung pada push

`flow_device_tokens` sudah wujud dengan lajur `token TEXT NOT NULL UNIQUE` dan
`platform`, dan `lib/flow/platform/push.ts` menulis kepadanya. Token FCM ialah
pengecam peranti.

- **Jika `google-services.json` ditambah** (push dihidupkan) → tandakan
  **Device or other IDs = Ya**, tujuan App functionality, dan isytihar
  perkongsian kepada Google.
- **Jika plugin push dibuang** → tandakan **Tidak**.

Menghantar dalam keadaan sekarang mengelirukan: kod dan jadual wujud, tetapi
pendaftaran gagal senyap tanpa konfigurasi. Putuskan dahulu.

### 2. Perkongsian kepada Anthropic — bergantung pada Agent

`lib/flow/agent/provider.ts` hidup hanya apabila **kedua-dua**
`FLOW_AGENT_ENABLED=1` dan `ANTHROPIC_API_KEY` hadir. Apabila hidup, alat Agent
dalam `lib/flow/agent/tools.ts` boleh membaca — antaranya — `cari_pelanggan`,
`senarai_invois`, `butiran_invois`, `senarai_kerja` dan `kutipan_terbuka`.

Maknanya **maklumat peribadi pelanggan mengalir ke API Anthropic** apabila
Agent dihidupkan dalam produksi.

- **Jika Agent hidup dalam produksi** → sebut pemprosesan AI dalam dasar
  privasi, dan sahkan akaun Anthropic anda berada di bawah terma perniagaan
  yang mengehadkan penggunaan kepada memproses bagi pihak anda. Di bawah terma
  itu Anthropic ialah penyedia perkhidmatan, jadi laluan ini **tidak** menjadi
  "dikongsi" pada borang — lihat bahagian Pihak ketiga. Jika anda tidak dapat
  mengesahkan terma itu, tanda Personal info dan Financial info sebagai
  dikongsi.
- **Jika Agent dimatikan dalam produksi** → tiada aliran langsung kepada
  Anthropic untuk didedahkan langsung.

Semak nilai sebenar pembolehubah persekitaran produksi sebelum menjawab.

---

## Pihak ketiga

**Jangan tanda semua ini sebagai "dikongsi".** Play mentakrifkan *sharing*
sebagai pemindahan kepada pihak ketiga, tetapi ia mengecualikan beberapa jenis
pemindahan secara khusus — antaranya pemindahan kepada **penyedia perkhidmatan**
yang memproses data bagi pihak pembangun mengikut arahan, terma kontrak dan
dasar pembangun itu. Menanda pemproses sebagai "dikongsi" menghasilkan label
Data safety yang tidak tepat, dan label tidak tepat itulah punca penolakan
paling kerap.

| Pihak | Data | Peranan Play | Syarat |
|---|---|---|---|
| Supabase | Semua data aplikasi — auth, pangkalan data, storan foto | Penyedia perkhidmatan — hos data bagi pihak anda | Sentiasa |
| Vercel | Hosting dan log permintaan | Penyedia perkhidmatan — infrastruktur | Sentiasa |
| Google (FCM) | Token peranti | Penyedia perkhidmatan — penghantaran push | Hanya jika `google-services.json` ditambah |
| Anthropic | Kandungan yang alat Agent baca, termasuk PII pelanggan | Penyedia perkhidmatan — memproses mengikut arahan anda | Hanya jika Agent dihidupkan |
| Billplz | Jumlah bil, rujukan, maklumat pembayar | Gerbang bayaran; pemindahan dimulakan pengguna semasa membayar | Sentiasa (aliran kutipan) |

Kesimpulan bagi Flow seperti kod berdiri sekarang: **data DIKUMPUL, tetapi
tiada satu pun laluan di atas yang jelas menjadi "dikongsi"** — setiap penerima
bertindak sebagai pemproses bagi pihak anda, dan laluan Billplz dimulakan oleh
pengguna sendiri.

Dua syarat yang mesti anda sahkan sebelum bergantung pada pengecualian itu:

1. **Kontrak wujud dan mengehadkan penggunaan.** Pengecualian penyedia
   perkhidmatan bergantung pada terma yang mengikat penerima untuk memproses
   mengikut arahan anda sahaja. Terma perniagaan standard Supabase, Vercel,
   Google dan Anthropic memang begitu — tetapi ia terpakai kepada akaun anda,
   jadi sahkan anda berada di bawah terma tersebut, bukan pelan percuma dengan
   syarat berbeza.
2. **Penerima tidak menggunakan data untuk tujuannya sendiri.** Sebaik sahaja
   mana-mana pihak menggunakan data itu untuk tujuan sendiri, pengecualian
   gugur dan ia menjadi perkongsian sebenar.

Setiap satu tetap perlu muncul dalam dasar privasi — pengecualian *sharing*
dalam borang Play tidak membatalkan kewajipan pendedahan privasi. Dasar semasa
(`lib/i18n/legal/privacy-content.ts`) sudah menyebut aliran bayaran; semak sama
ada ia menyebut Anthropic jika Agent akan hidup.

---

## Amalan keselamatan

| Soalan Play | Jawapan | Asas |
|---|---|---|
| Data disulitkan semasa transit | **Ya** | HTTPS sahaja; `allowMixedContent: false` dalam `capacitor.config.ts` |
| Pengguna boleh minta data dipadam | **Ya** | `app/padam-akaun/page.tsx` — laluan web, memenuhi keperluan Play |
| Anda mengikut Families Policy | Bergantung | Hanya jika Target audience merangkumi kanak-kanak |

Nota tambahan yang menyokong borang: `allowBackup="false"` menghalang kuki sesi
WebView masuk ke sandaran awan Google — bukan soalan borang, tetapi ia
menguatkan jawapan keselamatan anda.

**Ia tidak meliputi pemindahan peranti-ke-peranti.** Dokumentasi Android
menyatakannya secara jelas untuk aplikasi yang mensasarkan Android 12 (API 31)
ke atas: pada peranti sesetengah pengeluar, `allowBackup="false"` mematikan
sandaran awan *tetapi tidak* mematikan pemindahan D2D. Flow mensasarkan API 36,
jadi ini terpakai. Untuk menutup laluan itu juga, gunakan
`android:dataExtractionRules` dengan bahagian `<device-transfer>` yang
mengecualikan direktori WebView — kemudian sahkan dengan ujian pemindahan
sebenar, bukan dengan membaca manifest.

---

## Susunan mengisi

1. Putuskan dua suis di atas — push dan Agent. Semuanya bergantung padanya.
2. Isi Data collected menggunakan jadual di atas.
3. Bagi setiap jenis: tujuan **App functionality** melainkan ada sebab lain.
4. Bagi setiap jenis: nilai *sharing* mengikut PERANAN setiap penerima dalam
   jadual pihak ketiga — bukan sekadar kehadirannya. Pemproses bagi pihak anda
   tidak dikira sebagai dikongsi.
5. Sahkan dasar privasi menyenaraikan setiap pihak ketiga yang anda tanda.
6. Sahkan manifest tergabung sepadan — lihat `PLAY_CONSOLE_READINESS.md` Jurang A.

---

## Batasan

Diterbitkan daripada skema dan kod, bukan daripada memerhati trafik masa jalan.
Dua perkara khususnya perlu disahkan terhadap persekitaran produksi sebenar,
bukan repo: nilai `FLOW_AGENT_ENABLED` dan kehadiran `google-services.json`
dalam binaan keluaran.
