================================================================================
           DAPODIKSQ API - DOKUMENTASI RESMI (OFFICIAL DOCUMENTATION)
     Upstream Directory Service & School Search Provider (Kemendikdasmen)
       Ekosistem Terintegrasi: SmarQuest App & GMKNS Platform (SQEXAM)
================================================================================

[1. INFORMASI DASAR API]
--------------------------------------------------------------------------------
- Endpoint Utama    : https://hostgmkns.byethost32.com/dapodiksq/api
- Endpoint Alternatif: https://hostgmkns.byethost32.com/dapodiksq/api.php
- Metode HTTP       : GET
- Format Output     : application/json; charset=utf-8
- Autentikasi       : Public Open Access (Tanpa API Key)
- Dukungan CORS     : Aktif (Access-Control-Allow-Origin: *)
- Micro-Caching     : 30 Detik (Menjaga latensi respons cepat & anti-overload)
- Rate Limiting     : 80 request / menit per alamat IP


[2. DAFTAR PARAMETER QUERY URL (REQUEST)]
--------------------------------------------------------------------------------
Gunakan kombinasi parameter berikut pada query string URL:

1. q / keyword
   - Tipe        : String (Maks. 60 karakter)
   - Deskripsi   : Pencarian cerdas fleksibel (tokenized search) yang mencocokkan
                   nama sekolah, NPSN, atau alamat secara tidak kaku.
   - Contoh      : ?q=SMK+1+Katapang atau ?q=Negeri+1

2. npsn
   - Tipe        : String (8 digit angka)
   - Deskripsi   : Pencarian presisi spesifik berdasarkan 8 digit Nomor Pokok
                   Sekolah Nasional.
   - Contoh      : ?npsn=20219141

3. bentuk / jenjang
   - Tipe        : String
   - Deskripsi   : Memfilter berdasarkan bentuk pendidikan.
   - Pilihan     : SD, SMP, SMA, SMK, SLB, TK, PKBM
   - Contoh      : ?bentuk=SMK

4. status
   - Tipe        : String
   - Deskripsi   : Memfilter status kepemilikan sekolah.
   - Pilihan     : Negeri, Swasta
   - Contoh      : ?status=Negeri

5. kab / kabupaten / kabupaten_kota
   - Tipe        : String
   - Deskripsi   : Memfilter nama Kabupaten atau Kota.
   - Contoh      : ?kab=Bandung

6. prov / provinsi
   - Tipe        : String
   - Deskripsi   : Memfilter nama Provinsi.
   - Contoh      : ?prov=Jawa+Barat

7. kecamatan
   - Tipe        : String
   - Deskripsi   : Memfilter nama Kecamatan.
   - Contoh      : ?kecamatan=Katapang

8. page
   - Tipe        : Integer (Default: 1)
   - Deskripsi   : Nomor halaman data (pagination).
   - Contoh      : ?page=2

9. limit
   - Tipe        : Integer (Default: 25, Maksimal: 100)
   - Deskripsi   : Batas jumlah data yang dikembalikan dalam satu respons.
   - Contoh      : ?limit=10

10. format
    - Tipe       : String
    - Deskripsi  : Pilihan format output data. Isi dengan "sqexam" jika ingin
                   skema respons langsung dipetakan ke format aplikasi SQEXAM.
    - Contoh     : ?format=sqexam

11. delay
    - Tipe       : Integer (Maksimal: 300 ms)
    - Deskripsi  : Menambahkan jeda waktu simulasi respons untuk pengujian UI
                   efek skeleton / loading.
    - Contoh     : ?delay=150

12. raw
    - Tipe       : Boolean (true / 1)
    - Deskripsi  : Menyertakan seluruh payload objek JSON mentah asli dari
                   Kemendikdasmen (raw_data).
    - Contoh     : ?raw=true


[3. STRUKTUR FORMAT RESPONS JSON]
--------------------------------------------------------------------------------

A. FORMAT STANDAR (DapodikSQ Envelope)
   Format default yang dikonsumsi oleh upstream service Vercel SQEXAM:

   {
     "status": "success",
     "code": 200,
     "message": "Data sekolah berhasil diambil.",
     "pagination": {
       "page": 1,
       "limit": 10,
       "total_rows": 1,
       "total_pages": 1,
       "has_next": false,
       "has_prev": false
     },
     "filters_applied": {
       "keyword": "SMK 1",
       "npsn": null,
       "bentuk": "SMK",
       "status": "Negeri",
       "kabupaten": "Bandung",
       "provinsi": null,
       "kecamatan": null,
       "delay_ms": 0
     },
     "data": [
       {
         "id": 42,
         "sekolah_id": "9B0A3D2F-XXXX-XXXX-XXXX",
         "npsn": "20219141",
         "nama_sekolah": "SMKN 1 KATAPANG",
         "bentuk_pendidikan": "SMK",
         "status_sekolah": "Negeri",
         "alamat": "Jl. Ceuri BK No. 1, Katapang",
         "kecamatan": "Katapang",
         "kabupaten_kota": "Kab. Bandung",
         "provinsi": "Jawa Barat",
         "updated_at": "2026-03-01 10:20:00"
       }
     ]
   }

B. FORMAT SQEXAM LANGSUNG (?format=sqexam)
   Format siap pakai yang langsung cocok dengan model frontend SQEXAM:

   {
     "success": true,
     "data": [
       {
         "id": "9B0A3D2F-XXXX-XXXX-XXXX",
         "npsn": "20219141",
         "name": "SMKN 1 KATAPANG",
         "school_type": "SMK"
       }
     ],
     "meta": {
       "count": 1,
       "total_rows": 1
     }
   }


[4. INTEGRASI EKOSISTEM GMKNS / SQEXAM (schools-search.ts)]
--------------------------------------------------------------------------------
API DapodikSQ telah dirancang 100% kompatibel dengan handler Vercel Node.js
pada aplikasi ujian SQEXAM (schools-search.ts).

Alur Normalisasi Lapisan SQEXAM:
1. Handler memanggil URL upstream:
   https://hostgmkns.byethost32.com/dapodiksq/api?q=...&npsn=...&limit=...
2. Memverifikasi status === 'success' dan data berupa Array.
3. Melakukan pemetaan field:
   - school.sekolah_id     -> school.id (terdapat fallback otomatis ke NPSN/ID)
   - school.npsn           -> school.npsn
   - school.nama_sekolah   -> school.name
   - school.bentuk_pendidikan -> school.school_type
4. Mengembalikan respons final berstatus 200 JSON ke antarmuka aplikasi ujian.


[5. CONTOH KODE IMPLEMENTASI]
--------------------------------------------------------------------------------

A. JAVASCRIPT / TYPESCRIPT (Fetch API Browser / Node.js)
   -----------------------------------------------------
   async function cariSekolah(keyword) {
     const endpoint = `https://hostgmkns.byethost32.com/dapodiksq/api?q=${encodeURIComponent(keyword)}&limit=10`;
     const res = await fetch(endpoint, {
       headers: { 'Accept': 'application/json' }
     });
     const json = await res.json();
     
     if (json.status === 'success') {
       json.data.forEach(item => {
         console.log(`[${item.npsn}] ${item.nama_sekolah} (${item.kabupaten_kota})`);
       });
     }
   }
   cariSekolah('SMKN 1 Katapang');


B. DART / FLUTTER (Mobile Application)
   -----------------------------------
   import 'dart:convert';
   import 'package:http/http.dart' as http;

   Future<List<dynamic>> searchSchool(String keyword) async {
     final uri = Uri.parse('https://hostgmkns.byethost32.com/dapodiksq/api?q=$keyword&limit=10');
     final response = await http.get(uri, headers: {'Accept': 'application/json'});

     if (response.statusCode == 200) {
       final Map<String, dynamic> body = json.decode(response.body);
       return body['data'] ?? [];
     }
     return [];
   }


C. PYTHON
   ------
   import requests

   url = "https://hostgmkns.byethost32.com/dapodiksq/api"
   params = {"q": "Negeri 1", "bentuk": "SMK", "limit": 10}

   response = requests.get(url, params=params)
   if response.status_code == 200:
       sekolah_list = response.json().get("data", [])
       for sch in sekolah_list:
           print(f"{sch['npsn']} - {sch['nama_sekolah']}")


D. PHP (cURL Client)
   -----------------
   <?php
   $apiUrl = 'https://hostgmkns.byethost32.com/dapodiksq/api?q=SMK&limit=5';
   $response = file_get_contents($apiUrl);
   $result = json_decode($response, true);

   if ($result && $result['status'] === 'success') {
       foreach ($result['data'] as $row) {
           echo "{$row['npsn']} | {$row['nama_sekolah']}" . PHP_EOL;
       }
   }


[6. KODE STATUS HTTP (HTTP STATUS CODES)]
--------------------------------------------------------------------------------
- 200 OK                  : Permintaan berhasil diproses dan data dikembalikan.
- 400 Bad Request         : Parameter masukan tidak memenuhi format validasi.
- 405 Method Not Allowed  : Metode HTTP salah (hanya mengizinkan metode GET).
- 429 Too Many Requests   : Batas rate-limit terlampaui (> 80 request / menit).
- 500 Internal Server Err : Gangguan saat menghubungi basis data MySQL ByetHost.
- 502 Bad Gateway         : Layanan upstream gagal merespons tepat waktu.


[7. ETIKA & KETENTUAN PENGGUNAAN (FAIR USE)]
--------------------------------------------------------------------------------
Layanan API ini terbuka untuk kepentingan umum di bawah lisensi open-source
ekosistem SmarQuest App dengan mematuhi prinsip tanggung jawab:

1. Kemanfaatan Edukasi : Digunakan untuk mendukung sistem administrasi ujian
                          (SQEXAM), verifikasi sekolah, dan portofolio teknologi.
2. Caching Klien       : Disarankan menerapkan caching statis di sisi klien
                          untuk kata kunci yang berulang demi menjaga efisiensi.
3. Larangan Flooding   : Dilarang keras melakukan bot spamming atau DDoS yang
                          dapat merugikan pengguna lain di server bersama.


[8. DUKUNGAN EKOSISTEM & KONTAK]
--------------------------------------------------------------------------------
- Platform Ekosistem : SmarQuest App & GMKNS Platform
- Repositori Resmi   : GitHub (Farhan-A-F/DapodikSQ)
- Pemeliharaan Data  : Tersinkronisasi berkala via GitHub Actions & Dapodik Engine

================================================================================
          (c) 2026 DapodikSQ Ecosystem - All Rights Reserved.
================================================================================
