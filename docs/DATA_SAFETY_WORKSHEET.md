# Borang Data safety — jawapan diterbitkan daripada kod

Diterbitkan daripada `arunghayatsports-ui/corallium-test` pada `main` = `397a50b`,
dan **disemak semula pada 25 Sept 2026** terhadap `main` = `d3d9a57`, skema
pangkalan data produksi, dan pembolehubah persekitaran produksi Vercel.
Setiap baris dirujuk kepada jadual, lajur atau fail sebenar.

Borang Data safety ialah punca penolakan Play yang paling kerap, kerana ia mesti
sepadan dengan kelakuan sebenar aplikasi. Dokumen ini menyediakan jawapannya
supaya borang itu diisi daripada kod, bukan daripada ingatan.

> **Kedua-dua suis sudah diputuskan oleh keadaan produksi** (25 Sept 2026):
> Device or other IDs = Tidak, dan tiada perkongsian kepada Anthropic. Lihat
> bahagian "Suis" di bawah — termasuk apa yang mesti dibuat SEBELUM mana-mana
> suis dihidupkan.

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
| User IDs | Ya | ID akaun Supabase Auth — setiap pengguna yang log masuk |

Tujuan: App functionality; tambah Account management bagi Name, Email address dan User IDs. Dikongsi: nilai mengikut bahagian Pihak ketiga di bawah — kemungkinan besar TIDAK.
Diperlukan atau pilihan: Name, Email address dan User IDs **diperlukan** —
pendaftaran tidak boleh berlaku tanpanya. Selebihnya pilihan.

### Financial info

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Purchase history | Ya | `flow_invoices`, `flow_quotes`, `flow_payments`, `flow_billing_events` |
| Other financial info | Ya | `flow_gateway_payments` — keadaan bil, jumlah, rujukan |
| User payment info | **Ya** | `flow_businesses.bank_name`, `bank_account_name`, `bank_account_number` |

**Nombor akaun bank DIKUMPUL.** Versi awal dokumen ini mengatakan sebaliknya,
dan itu salah. `supabase/migrations/202_flow_payment_settings.sql` menambah
lajur bank kepada `flow_businesses`; pemilik mengisinya dalam Tetapan
pembayaran (`components/flow/flow-payment-settings.tsx`), dan halaman kutipan
awam memaparkannya kepada pelanggan (`components/flow/flow-public-collect.tsx`)
supaya mereka boleh membuat pindahan.

Setakat 25 Sept 2026 tiada perniagaan yang mengisinya. Itu tidak mengubah
jawapan: Play bertanya tentang apa yang aplikasi **boleh** kumpul.

Paparan kepada pelanggan tidak menjadikannya "dikongsi" — pemilik sendiri
memilih untuk memaparkannya, jadi ia pemindahan yang dimulakan pengguna.

Nombor **kad** tetap tidak dikumpul — bayaran kad berlaku pada gerbang Billplz.

### Photos and videos

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Photos | Ya | `flow_job_evidence.storage_key` — foto bukti kerja |

Ditangkap melalui `@capacitor/camera`, dimampatkan 1600px pada peranti
(`lib/flow/platform/capture-photo.ts`), dimuat naik ke Supabase Storage.
`saveToGallery: false`, jadi tiada kebenaran storan diminta.

### Files and docs

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Files and docs | Ya | `flow_businesses.payment_qr_storage_key` — imej QR bayaran yang pemilik muat naik |

### App activity

| Jenis Play | Dikumpul | Sumber dalam kod |
|---|---|---|
| Other actions | Ya | `flow_events`, `flow_workflow_runs`, `flow_agent_runs` |

Ini log operasi dalam pangkalan data sendiri, bukan analitik pihak ketiga.
Ia ditulis setiap kali aplikasi digunakan, jadi tandakan **diperlukan**.

### Yang TIDAK dikumpul

Tandakan "No" dengan yakin untuk semua ini — tiada kod menyentuhnya:

- Location (kasar mahupun tepat)
- Contacts, Calendar, SMS, Call logs
- Health and fitness
- Advertising ID — tiada SDK iklan, jadi **jangan** isytihar kebenaran `AD_ID`
- Crash logs dan diagnostics — tiada SDK pelaporan ranap
- Installed apps
- Contacts — rekod pelanggan ditaip oleh pengguna; aplikasi tidak membaca
  senarai kenalan telefon

---

## Suis — kedua-duanya diputuskan oleh keadaan produksi

Disemak pada 25 Sept 2026. Senarai pembolehubah persekitaran produksi Vercel
untuk projek yang melayani `www.corallium.my` adalah lengkap (tiada yang
tersembunyi), dan tiada satu pun pembolehubah di bawah terdapat di dalamnya.

| Suis | Bukti | Jawapan |
|---|---|---|
| Push | `NEXT_PUBLIC_FLOW_PUSH_ENABLED` tidak ditetapkan, jadi `isPushSupported()` palsu dan dialog kebenaran tidak pernah dipaparkan; tiada `google-services.json`; jadual `flow_device_tokens` **tidak wujud** dalam produksi (migrasi 209 belum digunakan) | Device or other IDs = **Tidak** |
| Agent | `FLOW_AGENT_ENABLED` dan `ANTHROPIC_API_KEY` kedua-duanya tidak ditetapkan | **Tiada** aliran kepada Anthropic |

**Kedua-dua suis ini dihidupkan melalui Vercel, bukan melalui AAB.**
Menghidupkannya mengubah data yang dikumpul tanpa sebarang muat naik ke Play.
Kemas kini borang Data safety **sebelum** menghidupkan suis, bukan selepas.

Bahagian di bawah kekal sebagai rujukan untuk hari suis itu dihidupkan.

### 1. Device or other IDs — bergantung pada push

`supabase/migrations/209_flow_device_tokens.sql` mentakrifkan jadual itu dengan
lajur `token TEXT NOT NULL UNIQUE` dan `platform`, untuk token yang
`registerForPush()` dalam `lib/flow/platform/push.ts` pulangkan. Token FCM
ialah pengecam peranti.

- **Jika `google-services.json` ditambah** (push dihidupkan) → tandakan
  **Device or other IDs = Ya**, tujuan App functionality, dan isytihar
  perkongsian kepada Google.
- **Jika plugin push dibuang** → tandakan **Tidak**.

Selagi suis push mati, tiada token dikumpul, jadi jawapannya Tidak.

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

Setakat 25 Sept 2026 Agent dimatikan dalam produksi — lihat jadual suis di atas.

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

1. Sahkan kedua-dua suis masih mati di Vercel. Jika salah satu sudah
   dihidupkan, jawapan di atas tidak lagi terpakai.
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
Keadaan suis dan jadual produksi disemak pada 25 Sept 2026; ia boleh berubah
tanpa sebarang perubahan kod, jadi semak semula sebelum setiap penghantaran.
