# MANUAL LENGKAP SETUP TAPAK ASAS & WEB APP
## Sistem E-Folio IPG Premium v15.0

Dokumen ini adalah panduan langkah-demi-langkah untuk menyediakan keseluruhan sistem dari kosong: pangkalan data (Google Sheets), pelayan (Apps Script), storan fail (Drive), dan penerbitan tapak (Vercel/GitHub).

---

## BAHAGIAN 0 — SENARAI SEMAK KESELURUHAN

- [ ] 1. Cipta Google Sheet (pangkalan data)
- [ ] 2. Sediakan struktur lembaran (Users, Data_Folio, Folio_Kursus, Subjek_Diajar)
- [ ] 3. Cipta projek Apps Script & tampal kod pelayan
- [ ] 4. Cipta Root Folder di Google Drive (untuk simpanan bahan tersusun)
- [ ] 5. Deploy Apps Script sebagai Web App & dapatkan URL `/exec`
- [ ] 6. Tampal URL tersebut ke dalam `index.html`
- [ ] 7. Naikkan `index.html` ke GitHub
- [ ] 8. Sambungkan repo GitHub ke Vercel & terbitkan
- [ ] 9. Cipta akaun Admin pertama & uji log masuk
- [ ] 10. Uji CRUD rekod, audit, dan salinan fail Drive

---

## BAHAGIAN 1 — SEDIAKAN GOOGLE SHEETS (PANGKALAN DATA)

### 1.1 Cipta Spreadsheet Baharu
1. Pergi ke [sheets.google.com](https://sheets.google.com) menggunakan akaun Google yang akan menjadi **akaun induk sistem** (cadangan: guna akaun rasmi jabatan, bukan akaun peribadi pentadbir individu).
2. Cipta spreadsheet baharu, namakan contohnya: `DB_EFOLIO_IPG_MASTER`.

### 1.2 Cipta Lembaran (Sheet Tabs) Berikut — Nama Mesti Tepat

| Nama Sheet | Kegunaan |
|---|---|
| `Users` | Senarai akaun pensyarah/admin |
| `Data_Folio` | Rekod Folio Pensyarah |
| `Folio_Kursus` | Rekod Folio Kursus |
| `Subjek_Diajar` | Rekod subjek yang diajar |

> Padam sheet default `Sheet1` selepas keempat-empat di atas dicipta, supaya tidak mengelirukan.

### 1.3 Header Lajur — Baris 1 Setiap Sheet

**Sheet `Users`** (baris 1, lajur A–H):
```
Username | Password | Nama_Penuh | No_Pekerja | Gred | Jabatan | Role | Foto_URL
```

**Sheet `Data_Folio`** (baris 1):
```
Username | Tahun | Kategori | Nama | Pautan | Keterangan
```

**Sheet `Folio_Kursus`** (baris 1):
```
Username | Tahun | Semester | Kategori | Nama_Kursus | Pautan | Keterangan
```

**Sheet `Subjek_Diajar`** (baris 1):
```
Username | Tahun | Semester | Kod_Subjek | Nama_Subjek | Kelas | Pautan
```

> **PENTING:** Nama lajur mesti sama persis (case-sensitive digunakan oleh kod semasa memadankan header). Jangan tambah ruang tersembunyi.

### 1.4 Cipta Akaun Admin Pertama Secara Manual
Dalam sheet `Users`, masukkan baris pertama secara manual (jangan tunggu auto-register untuk akaun admin utama):

| Username | Password | Nama_Penuh | No_Pekerja | Gred | Jabatan | Role | Foto_URL |
|---|---|---|---|---|---|---|---|
| admin@ipgm.edu.my | GOOGLE_AUTH | PENTADBIR SISTEM | - | - | UNIT ICT | Admin | (kosongkan) |

> Emel yang mengandungi perkataan `admin`, atau username `auditor`, akan **automatik** dikenali sebagai Admin oleh sistem semasa log masuk pertama — tetapi lebih selamat jika anda tetapkan baris ini secara manual dahulu.

---

## BAHAGIAN 2 — SEDIAKAN GOOGLE APPS SCRIPT (PELAYAN)

### 2.1 Buka Editor Apps Script
1. Dalam spreadsheet tadi, klik **Extensions (Sambungan) → Apps Script**.
2. Ini akan membuka projek Apps Script yang **terikat terus** kepada spreadsheet anda (penting — `SpreadsheetApp.getActiveSpreadsheet()` bergantung pada ikatan ini).

### 2.2 Tampal Kod
1. Padam kandungan default `Code.gs`.
2. Tampal keseluruhan kandungan fail `code.cs` yang telah anda sediakan (fungsi `doGet`, `doPost`, `buildRowFromPayload`, `getSheetDataAsJson`, `createJsonResponse`, dan tambahan fungsi salinan Drive jika digunakan).
3. Namakan semula fail projek kepada `EFolio_Server` (pilihan, untuk kekemasan).
4. Klik ikon **Simpan (💾)**.

### 2.3 Uji Dahulu Sebelum Deploy (Disyorkan)
1. Pilih fungsi `doGet` pada dropdown atas editor.
2. Klik **Run**.
3. Google akan minta **kebenaran (authorization)** — klik **Review permissions**, pilih akaun anda, klik **Advanced → Go to [nama projek] (unsafe)** (ini normal untuk skrip sendiri), klik **Allow**.
4. Jika tiada ralat merah dalam log (`Execution log`), skrip sedia untuk deploy.

---

## BAHAGIAN 3 — SEDIAKAN GOOGLE DRIVE (STORAN BAHAN TERSUSUN)

> Langkukan bahagian ini **hanya jika** anda menggunakan ciri "salin fail automatik ke folder tersusun" yang dibincangkan sebelum ini.

### 3.1 Cipta Root Folder
1. Buka [drive.google.com](https://drive.google.com) menggunakan **akaun yang sama** dengan pemilik Apps Script.
2. Cipta folder baharu, contoh: `EFOLIO_MASTER_STORAGE`.
3. Buka folder tersebut, salin ID daripada URL:
   ```
   https://drive.google.com/drive/folders/1AbCdEfGhIjKlMnOpQrStUvWxYz
                                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                          Ini ID Folder anda
   ```
4. Kembali ke Apps Script, tampal ID tersebut ke pemboleh ubah:
   ```javascript
   const ROOT_FOLDER_ID = 'TAMPAL_ID_DI_SINI';
   ```

### 3.2 Peraturan Perkongsian Fail Pensyarah
Fail yang ditaip oleh pensyarah di borang **mesti dikongsi sekurang-kurangnya sebagai "Viewer"** kepada akaun pemilik skrip, jika tidak sistem gagal menyalinnya secara senyap. Cadangan teks bantuan pada borang:

> *"Pastikan tetapan perkongsian fail Drive anda ditetapkan kepada 'Sesiapa sahaja yang mempunyai pautan' sebelum menampal pautan di sini."*

---

## BAHAGIAN 4 — DEPLOY SEBAGAI WEB APP

### 4.1 Deploy Pertama Kali
1. Dalam Apps Script, klik **Deploy → New deployment**.
2. Klik ikon gear ⚙️ di sebelah "Select type", pilih **Web app**.
3. Isikan:
   - **Description**: `EFolio v1`
   - **Execute as**: `Me (akaun anda)` — *(WAJIB — supaya semua pengguna kongsi 1 sumber data)*
   - **Who has access**: `Anyone` — *(WAJIB — supaya frontend awam boleh panggil API tanpa login Google-Apps-Script berasingan)*
4. Klik **Deploy**.
5. Sistem akan minta kebenaran sekali lagi — klik **Authorize access** dan ikuti langkah yang sama seperti 2.3.
6. Selepas berjaya, anda akan diberi **Web app URL** — formatnya:
   ```
   https://script.google.com/macros/s/AKfycb................................/exec
   ```
7. **SALIN URL INI** — ini adalah `API_URL` anda.

### 4.2 Kemaskini Kod Selepas Ini (PENTING)
Setiap kali anda **ubah** kod dalam `code.cs`/`Code.gs`, URL `/exec` **TIDAK berubah** — TETAPI anda mesti:
1. Klik **Deploy → Manage deployments**.
2. Klik ikon pensil ✏️ pada deployment sedia ada.
3. Pada "Version", pilih **New version**.
4. Klik **Deploy**.

> Jika anda hanya klik "Save" (💾) tanpa buat "New version", perubahan kod **TIDAK akan** dipakai oleh pengguna sedia ada — ini punca #1 ralat "kenapa kod saya tak jalan walaupun dah update".

---

## BAHAGIAN 5 — SAMBUNGKAN URL KE FRONTEND (index.html)

1. Buka fail `index.html`.
2. Cari baris:
   ```javascript
   const API_URL = "https://script.google.com/macros/s/AKfycbxpehf-tDk3.../exec";
   ```
3. Gantikan dengan URL Web App **baharu** anda daripada Bahagian 4.1.
4. Simpan fail.

---

## BAHAGIAN 6 — TERBITKAN TAPAK (GITHUB + VERCEL)

### 6.1 Naikkan ke GitHub
1. Cipta repositori baharu di [github.com](https://github.com), contoh: `efolio-ipg-v2`.
2. Muat naik (upload) fail `index.html` ke root repositori (guna butang **Add file → Upload files** di GitHub jika tidak biasa command line).
3. Commit perubahan.

### 6.2 Sambung ke Vercel
1. Log masuk ke [vercel.com](https://vercel.com) menggunakan akaun GitHub yang sama.
2. Klik **Add New → Project**.
3. Pilih repositori `efolio-ipg-v2` tadi.
4. Framework Preset: pilih **Other** (kerana ini HTML statik, bukan Next.js/React).
5. Biarkan tetapan Build & Output default kosong (fail `index.html` statik tidak perlukan proses "build").
6. Klik **Deploy**.
7. Selepas siap, Vercel akan beri URL awam, contoh: `https://efolio-ipg-v2.vercel.app`.

### 6.3 Kemaskini Automatik
Selepas sambungan ini disediakan sekali sahaja, **setiap kali** anda `commit` perubahan baharu pada `index.html` di GitHub, Vercel akan **auto-deploy** semula tapak dalam masa kurang seminit — tidak perlu ulang langkah 6.2.

---

## BAHAGIAN 7 — UJIAN PENERIMAAN (ACCEPTANCE TEST)

Lengkapkan senarai ini sebelum edarkan pautan kepada pengguna sebenar:

- [ ] Log masuk menggunakan emel baharu (bukan admin) → rekod baharu automatik tercipta dalam sheet `Users`.
- [ ] Kemaskini Profil → data berubah dalam sheet `Users` secara masa nyata.
- [ ] Tambah rekod dalam ketiga-tiga tab (Data_Folio, Folio_Kursus, Subjek_Diajar) → baris baharu muncul dalam sheet berkaitan.
- [ ] Edit rekod sedia ada → baris yang betul (`rowIndex`) dikemaskini, bukan baris baharu dicipta.
- [ ] Padam rekod → baris hilang daripada sheet.
- [ ] Log masuk sebagai Admin → butang "Urus Pensyarah" dan penapis "Pilih Pensyarah" kelihatan.
- [ ] Sebagai Admin, tukar penapis pensyarah → jadual papar rekod pensyarah tersebut sahaja.
- [ ] Klik "Audit Folio" pada tahun tertentu → kategori yang belum lengkap ditanda merah.
- [ ] *(Jika ciri Drive digunakan)* Tampal pautan fail yang dikongsi awam → semak Root Folder di Drive, fail tersalin ke `Root ▸ Pensyarah ▸ Tahun ▸ Kategori`.
- [ ] *(Jika ciri Drive digunakan)* Tampal pautan fail yang **tidak dikongsi** → sistem masih berjaya simpan rekod (tidak crash), guna pautan asal.

---

## BAHAGIAN 8 — RALAT LAZIM & PENYELESAIAN

| Gejala | Punca Berkemungkinan | Penyelesaian |
|---|---|---|
| `Ralat pelayan semasa...` pada semua tindakan | URL `API_URL` salah/lapuk | Semak semula Bahagian 5 |
| Kod baharu tidak berkesan langsung | Lupa buat "New version" semasa deploy | Ulang Bahagian 4.2 |
| Data tidak muncul untuk pensyarah tertentu | Nama header sheet tidak tepat/ada ruang | Semak semula Bahagian 1.3 |
| Fail Drive gagal disalin secara senyap | Fail asal tidak dikongsi ke akaun skrip | Maklumkan pensyarah tetapkan "Anyone with link" |
| Ralat "Exceeded maximum execution time" | Terlalu banyak baris data / panggilan serentak | Pertimbang jadualkan pembersihan data lama, atau naik taraf ke Sheets API + caching |
| Pengguna admin baharu tidak dapat akses ciri Admin | Emel tidak mengandungi "admin" & bukan "auditor" | Tetapkan lajur `Role` = `Admin` secara manual dalam sheet `Users` |

---

*Disediakan untuk: Sistem E-Folio IPG Premium v15.0*
*Rujukan fail berkaitan: `index.html`, `code.cs` (Apps Script)*
