# 🏫 DapodikSQ API — Official Documentation

[![API Status](https://img.shields.io/badge/API_Status-Online-brightgreen?style=flat-square)](https://hostgmkns.byethost32.com/dapodiksq/api.php)
[![Response Format](https://img.shields.io/badge/Format-JSON-blue?style=flat-square)](https://json.org)
[![CORS](https://img.shields.io/badge/CORS-Enabled_%28*%29-success?style=flat-square)](#)
[![Ecosystem](https://img.shields.io/badge/Ecosystem-SmarQuest_App-blueviolet?style=flat-square)](#)
[![Access](https://img.shields.io/badge/Access-Public_/_Open--Source-orange?style=flat-square)](#-etika-dan-tanggung-jawab-penggunaan)

**DapodikSQ API** adalah antarmuka *RESTful API* publik dan open-source untuk mengakses data pokok pendidikan (sekolah) se-Indonesia yang dinormalisasi dari Kemendikdasmen RI. 

Disediakan sebagai layanan fondasi bagi ekosistem **SmarQuest App** agar para pengembang frontend (Web, Mobile, Flutter, React, Vue, Android, iOS) dapat langsung menampilkan, memfilter, dan mencari data sekolah tanpa perlu mengelola server basis data sendiri.

---

## 📌 Informasi Dasar API

- **Endpoint URL**: `https://hostgmkns.byethost32.com/dapodiksq/api`
- **Metode HTTP**: `GET`
- **Format Output**: `application/json; charset=utf-8`
- **Autentikasi**: *Public Open Access* (Tanpa API Key)
- **Dukungan CORS**: Ya (`Access-Control-Allow-Origin: *`), dapat ditembak langsung dari browser klien tanpa kendala CORS.

---

## 🔍 Parameter Query URL (Request)

Anda dapat menggabungkan beberapa parameter query sekaligus untuk memfilter data sekolah secara presisi:

| Parameter | Tipe | Contoh Nilai | Keterangan |
| :--- | :--- | :--- | :--- |
| `q` / `keyword` | `string` | `Negeri 1` | Mencari kata kunci pada **Nama Sekolah**, **NPSN**, atau **Alamat**. |
| `npsn` | `string` | `20219141` | Pencarian spesifik sekolah berdasarkan 8 digit NPSN. |
| `bentuk` / `jenjang` | `string` | `SMK`, `SMA`, `SMP`, `SD` | Filter berdasarkan jenjang pendidikan sekolah. |
| `status` | `string` | `Negeri`, `Swasta` | Filter berdasarkan status kepemilikan sekolah. |
| `kab` / `kabupaten` | `string` | `Bandung` | Filter berdasarkan nama Kabupaten atau Kota. |
| `prov` / `provinsi` | `string` | `Jawa Barat` | Filter berdasarkan nama Provinsi. |
| `kecamatan` | `string` | `Katapang` | Filter berdasarkan nama Kecamatan. |
| `page` | `integer` | `1` *(Default: 1)* | Nomor halaman (*pagination*). |
| `limit` | `integer` | `25` *(Default: 25)* | Jumlah baris data per respons (Maks. 200, atau `all` untuk 500). |
| `delay` | `integer` | `500` *(ms)* atau `2` *(detik)* | Menambahkan jeda simulasi respons (Maks. 3000 ms) untuk tes UI *skeleton/loading*. |
| `raw` | `boolean` | `true` atau `1` | Menyertakan objek payload mentah asli dari Kemendikdasmen (`raw_data`). |

---

## 📥 Format Respons JSON

### Struktur Utama (Envelope)

```json
{
  "status": "success",
  "code": 200,
  "message": "Data sekolah berhasil diambil.",
  "pagination": {
    "page": 1,
    "limit": 15,
    "total_rows": 1,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  },
  "filters_applied": {
    "keyword": null,
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
      "kecamatan": "Kec. Katapang",
      "kabupaten_kota": "Kab. Bandung",
      "provinsi": "Prov. Jawa Barat",
      "updated_at": "2026-03-01 10:20:00"
    }
  ]
}
```

---

## 💡 Contoh Kasus Penggunaan (URL Request)

1. **Mengambil Daftar Sekolah Default (25 item pertama)**:
   ```text
   GET https://hostgmkns.byethost32.com/dapodiksq/api
   ```

2. **Mencari Semua SMK Negeri di Kabupaten Bandung (15 data per halaman)**:
   ```text
   GET https://hostgmkns.byethost32.com/dapodiksq/api?bentuk=SMK&status=Negeri&kab=Bandung&limit=15&page=1
   ```

3. **Mencari Sekolah Berdasarkan NPSN Tertentu**:
   ```text
   GET https://hostgmkns.byethost32.com/dapodiksq/api?npsn=20219141
   ```

4. **Pencarian Bebas Kata Kunci (Misal: "Sawah") di Provinsi Jawa Tengah**:
   ```text
   GET https://hostgmkns.byethost32.com/dapodiksq/api?q=Sawah&prov=Jawa Tengah
   ```

5. **Tes Efek Loading Skeleton pada UI (Simulasi Delay 600 ms)**:
   ```text
   GET https://hostgmkns.byethost32.com/dapodiksq/api?bentuk=SMA&delay=600
   ```

---

## 💻 Panduan Menghubungkan API ke Aplikasi

### 1. JavaScript (Fetch API / Browser / Node.js)
```javascript
const apiUrl = 'https://hostgmkns.byethost32.com/dapodiksq/api?bentuk=SMK&status=Negeri&kab=Bandung';

async function getSekolah() {
  try {
    const res = await fetch(apiUrl);
    const json = await res.json();
    
    if (json.status === 'success') {
      console.log(`Ditemukan ${json.pagination.total_rows} sekolah:`);
      json.data.forEach(item => {
        console.log(`- [${item.npsn}] ${item.nama_sekolah} (${item.kabupaten_kota})`);
      });
    }
  } catch (error) {
    console.error('Gagal mengambil data:', error);
  }
}

getSekolah();
```

### 2. Flutter / Dart
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<void> fetchSekolah() async {
  final url = Uri.parse('https://hostgmkns.byethost32.com/dapodiksq/api?bentuk=SMA&limit=10');
  final response = await http.get(url);

  if (response.statusCode == 200) {
    final Map<String, dynamic> body = json.decode(response.body);
    List sekolahList = body['data'];
    print('Total data diambil: ${sekolahList.length}');
  } else {
    throw Exception('Gagal memuat API DapodikSQ');
  }
}
```

### 3. Python (Requests)
```python
import requests

url = "https://hostgmkns.byethost32.com/dapodiksq/api"
params = {
    "bentuk": "SMK",
    "status": "Negeri",
    "limit": 10
}

response = requests.get(url, params=params)
if response.status_code == 200:
    res_data = response.json()
    for s in res_data.get("data", []):
        print(f"{s['npsn']} - {s['nama_sekolah']}")
```

### 4. PHP (cURL / Client)
```php
<?php
$apiUrl = 'https://hostgmkns.byethost32.com/dapodiksq/api?bentuk=SMP&limit=5';
$json = file_get_contents($apiUrl);
$data = json_decode($json, true);

if ($data['status'] === 'success') {
    foreach ($data['data'] as $sch) {
        echo $sch['nama_sekolah'] . " - " . $sch['kabupaten_kota'] . PHP_EOL;
    }
}
```

---

## ⚠️ Kode Status HTTP (Status Codes)

| Kode | Arti | Deskripsi |
| :--- | :--- | :--- |
| `200 OK` | Sukses | Permintaan berhasil diproses dan data dikembalikan. |
| `500 Internal Server Error` | Error Server | Terjadi kesalahan query basis data atau koneksi server gagal. |

---

## ⚖️ Etika dan Tanggung Jawab Penggunaan

API ini dibuka untuk publik sebagai bagian dari inisiatif open-source ekosistem **SmarQuest App**. 

Semua orang diperbolehkan menggunakan API ini untuk kebutuhan pendidikan, portofolio, penelitian, maupun aplikasi komersial dengan syarat **bertanggung jawab dan penuh kesadaran**:

1. **Hemat Trafik (Caching)**: Sangat disarankan menyimpan data statis (cache) di aplikasi Anda agar tidak melakukan panggilan request berulang untuk data yang sama.
2. **Anti-Spam / Rate Limiting**: Dilarang keras melakukan *flooding* atau *DDoS* ke endpoint ini demi kelangsungan akses bersama.
3. **Penggunaan Positif**: Bertujuan mendukung literasi digital dan transparansi informasi pendidikan Indonesia.

---

## 🤝 Ekosistem & Dukungan

Dikembangkan sebagai subsistem data terbuka untuk **SmarQuest App**. Apabila Anda menemukan kendala data atau ingin berpartisipasi mengembangkan ekosistem ini, silakan buat *issue* pada repositori resmi.
