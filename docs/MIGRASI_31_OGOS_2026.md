# Migrasi 31 Ogos 2026 — API 36 + Play Billing Library 8

Dua tarikh mati jatuh pada hari yang sama, dan kedua-duanya menyekat muat naik AAB:

- `targetSdk` mesti **36** (Android 16)
- Play Billing Library mesti **8.0.0 ke atas**

Lanjutan boleh dipohon melalui borang dalam Play Console, sah sehingga **1 November 2026**.
Jika migrasi tidak sempat, mohon lanjutan **sebelum** 31 Ogos — bukan selepas.

> Butiran di bawah diambil daripada dokumentasi rasmi Android pada 26 Ogos 2026.
> Ia belum disemak terhadap kod Corallium Flow — repositori ini tidak mengandungi kod Android.

---

## Bahagian 1 — Play Billing Library 8

### Kebergantungan

```gradle
dependencies {
    implementation "com.android.billingclient:billing:8.0.0"
}
```

### API yang dibuang dan penggantinya

| Dibuang | Ganti dengan |
|---|---|
| `querySkuDetailsAsync` | `queryProductDetailsAsync` |
| `queryPurchasesAsync(String, PurchasesResponseListener)` | `queryPurchasesAsync(QueryPurchasesParams, PurchasesResponseListener)` |
| `queryPurchaseHistoryAsync` | **Tiada pengganti langsung** — guna Play Developer API di pelayan |
| `enablePendingPurchases()` tanpa argumen | `enablePendingPurchases(PendingPurchasesParams)` |
| `setOldSkuPurchaseToken` | `setOldPurchaseToken` |
| `setReplaceProrationMode` / `setReplaceSkusProrationMode` | `setSubscriptionReplacementMode` |
| `BillingClient.Builder.enableAlternativeBilling` | `enableUserChoiceBilling(UserChoiceBillingListener)` |
| `AlternativeBillingListener` | `UserChoiceBillingListener` |
| `AlternativeChoiceDetails` | `UserChoiceDetails` |

Tandatangan `ProductDetailsResponseListener.onProductDetailsResponse` **juga berubah**, jadi
setiap pemanggil `queryProductDetailsAsync` perlu dikemas kini walaupun ia sudah guna API baharu.

### Perubahan kod

**Pembinaan BillingClient** — ini yang paling kerap terlepas pandang:

```kotlin
// PBL 7 dan ke bawah
billingClient = BillingClient.newBuilder(context)
    .setListener(purchasesUpdatedListener)
    .enablePendingPurchases()
    .build()

// PBL 8
billingClient = BillingClient.newBuilder(context)
    .setListener(purchasesUpdatedListener)
    .enablePendingPurchases(
        PendingPurchasesParams.newBuilder()
            .enableOneTimeProducts()
            .build()
    )
    .build()
```

**Pertanyaan pembelian:**

```kotlin
// Dibuang
billingClient.queryPurchasesAsync(BillingClient.SkuType.SUBS, listener)

// PBL 8
billingClient.queryPurchasesAsync(
    QueryPurchasesParams.newBuilder()
        .setProductType(BillingClient.ProductType.SUBS)
        .build(),
    listener
)
```

**Naik taraf / turun taraf langganan:**

```kotlin
// Dibuang
BillingFlowParams.SubscriptionUpdateParams.newBuilder()
    .setOldSkuPurchaseToken(token)
    .setReplaceProrationMode(mode)
    .build()

// PBL 8
BillingFlowParams.SubscriptionUpdateParams.newBuilder()
    .setOldPurchaseToken(token)
    .setSubscriptionReplacementMode(mode)
    .build()
```

### Selepas migrasi — uji, jangan sekadar kompil

Migrasi ini mengubah kelakuan langganan, bukan hanya nama kaedah. Uji sepenuhnya dengan
penguji lesen:

- [ ] Pembelian produk sekali beli, hingga resit disahkan
- [ ] Pembelian langganan baharu
- [ ] Naik taraf dan turun taraf langganan (setiap mod penggantian yang digunakan)
- [ ] Pemulihan pembelian selepas pasang semula
- [ ] Pembelian tertangguh (pending) — inilah yang `PendingPurchasesParams` kawal
- [ ] Pembatalan dan tamat tempoh

---

## Bahagian 2 — targetSdk 36 (Android 16)

### Risiko tertinggi: penguatkuasaan edge-to-edge

**Tiada opt-out.** Atribut `windows OptOutEdgeToEdgeEnforcement` kini dinyahaktifkan pada
peranti Android 16. Setiap aplikasi yang menyasar API 36 mesti mengendalikan window insets
dengan betul, jika tidak antara muka akan terpotong di bawah bar status dan bar navigasi.

Ini biasanya kerja UI paling besar dalam keseluruhan migrasi. Semak setiap skrin pada
peranti sebenar, bukan hanya pratonton reka letak.

### Predictive back

`onBackPressed()` **tidak lagi dipanggil**, dan `KeyEvent.KEYCODE_BACK` tidak lagi dihantar.
Mana-mana kod yang memintas butang kembali dengan API lama akan gagal secara senyap.

Penyelesaian sebenar ialah berpindah ke `OnBackPressedCallback` / `OnBackInvokedCallback`.
Opt-out sementara jika masa tidak mengizinkan:

```xml
<application android:enableOnBackInvokedCallback="false">
```

### Sekatan orientasi diabaikan pada skrin besar

Pada paparan `sw600dp` ke atas, semua ini **diabaikan**: `android:screenOrientation`,
`android:resizableActivity="false"`, `android:minAspectRatio`, `android:maxAspectRatio`,
serta `setRequestedOrientation()`.

Aplikasi yang direka portrait sahaja kini akan berputar pada tablet. Opt-out sementara:

```xml
<application>
    <property
        android:name="android.window.PROPERTY_COMPAT_ALLOW_RESTRICTED_RESIZABILITY"
        android:value="true" />
</application>
```

**Opt-out ini tidak berfungsi lagi pada API 37**, jadi ia menangguh kerja, bukan
menghapuskannya.

### Perubahan lain

| Perubahan | Kesan | Opt-out |
|---|---|---|
| `ScheduledExecutorService.scheduleAtFixedRate` | Hanya **satu** pelaksanaan terlepas dijalankan semula, bukan semua | Tiada |
| `android:elegantTextHeight` | Diabaikan — jejas teks Arab, Thai, Tamil, Telugu dan lain-lain | Tiada |
| `MediaStore#getVersion()` | Kini unik setiap aplikasi | Tiada |
| `BODY_SENSORS` → kebenaran kesihatan berbutir | `READ_HEART_RATE`, `READ_HEALTH_DATA_IN_BACKGROUND`; perlu aktiviti dasar privasi | Tiada |
| Padanan intent lebih ketat | Sedia ada sebagai opt-in; akan jadi lalai kemudian | Opt-in |
| Kebenaran rangkaian tempatan | Pratonton opt-in; penguatkuasaan kemudian | Opt-in |

Tiga baris terakhir hanya berkenaan jika aplikasi menggunakan ciri berkenaan.

---

## Susunan yang disyorkan

1. Naik taraf PBL ke 8 dahulu dan pulihkan aliran bil sehingga hijau. Ini terpencil
   daripada kerja UI dan boleh disahkan dengan pembelian ujian.
2. Naikkan `compileSdk` ke 36 dan betulkan ralat kompilasi, **tanpa** menukar `targetSdk`.
3. Barulah naikkan `targetSdk` ke 36 dan uji setiap skrin untuk insets dan navigasi kembali.
4. Muat naik ke trek dalaman dahulu, semak pre-launch report, kemudian barulah ke trek tertutup.

Jika langkah 3 tidak sempat sebelum 31 Ogos: mohon lanjutan sekarang, dan hantar naik taraf
PBL 8 sahaja. Kedua-dua keperluan mempunyai borang lanjutan yang berasingan.

---

## Rujukan

- [Migrasi ke Play Billing Library 8](https://developer.android.com/google/play/billing/migrate-gpblv8)
- [Perubahan kelakuan Android 16](https://developer.android.com/about/versions/16/behavior-changes-16)
- [Keperluan target API level](https://developer.android.com/google/play/requirements/target-sdk)
