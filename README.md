# 🏫 DapodikSQ API — Official Documentation

[![API Status](https://img.shields.io/badge/API_Status-Online-brightgreen?style=flat-square)](https://hostgmkns.byethost32.com/dapodiksq/api)
[![Response Format](https://img.shields.io/badge/Format-JSON-blue?style=flat-square)](https://json.org)
[![CORS](https://img.shields.io/badge/CORS-Enabled_%28*%29-success?style=flat-square)](#)
[![Ecosystem](https://img.shields.io/badge/Ecosystem-SmarQuest_App_%26_GMKNS-blueviolet?style=flat-square)](#)
[![Integration](https://img.shields.io/badge/Integrated_With-SQEXAM-orange?style=flat-square)](#-integrasi-ekosistem-gmkns--sqexam)

**DapodikSQ API** adalah antarmuka *RESTful API* publik dan open-source untuk mengakses direktori data pokok pendidikan sekolah se-Indonesia yang dinormalisasi dari Kemendikdasmen RI. 

API ini dirancang khusus sebagai penyedia data fondasi (*upstream data service*) bagi ekosistem **SmarQuest App** serta modul pencarian sekolah pada aplikasi **SQEXAM (GMKNS Ecosystem)**.

---

## 📌 Informasi Dasar API

- **Endpoint URL**: `https://hostgmkns.byethost32.com/dapodiksq/api` *(atau `/dapodiksq/api.php`)*
- **Metode HTTP**: `GET`
- **Format Output**: `application/json; charset=utf-8`
- **Autentikasi**: *Public Open Access* (Tanpa API Key)
- **CORS Support**: Aktif (`Access-Control-Allow-Origin: *`), dapat diakses langsung dari browser klien, Node.js, Vercel Serverless, maupun Mobile Apps.
- **Micro-Caching**: Terintegrasi cache 30 detik untuk menjamin latensi respons tinggi di bawah 200 ms.

---

## 🔍 Parameter Query URL (Request)

Anda dapat menggabungkan beberapa parameter query sekaligus untuk memfilter data sekolah secara fleksibel:

| Parameter | Tipe | Contoh Nilai | Keterangan |
| :--- | :--- | :--- | :--- |
| `q` / `keyword` | `string` | `SMK 1 Katapang` | Pencarian fleksibel (*tokenized search*) pada **Nama Sekolah**, **NPSN**, atau **Alamat**. |
| `npsn` | `string` | `20219141` | Pencarian spesifik sekolah berdasarkan 8 digit kode NPSN. |
| `bentuk` / `jenjang` | `string` | `SMK`, `SMA`, `SMP`, `SD` | Filter berdasarkan bentuk pendidikan / jenjang. |
| `status` | `string` | `Negeri`, `Swasta` | Filter status kepemilikan sekolah (*whitelist*: Negeri / Swasta). |
| `kab` / `kabupaten` | `string` | `Bandung` | Filter nama Kabupaten atau Kota. |
| `prov` / `provinsi` | `string` | `Jawa Barat` | Filter nama Provinsi. |
| `kecamatan` | `string` | `Katapang` | Filter nama Kecamatan. |
| `page` | `integer` | `1` *(Default: 1)* | Nomor halaman (*pagination*). |
| `limit` | `integer` | `10` *(Default: 25, Maks: 100)* | Batas jumlah data yang ditampilkan per permintaan. |
| `format` | `string` | `sqexam` | Mengembalikan output khusus yang sudah dipetakan langsung untuk skema **SQEXAM**. |
| `delay` | `integer` | `100` *(ms)* | Menambahkan jeda simulasi (Maks. 300 ms) untuk pengujian UI *skeleton/loading*. |
| `raw` | `boolean` | `true` atau `1` | Menyertakan payload mentah asli Kemendikdasmen (`raw_data`). |

---

## 📥 Format Respons JSON

### 1. Format Standar (DapodikSQ Envelope)
Format bawaan yang dikonsumsi oleh *upstream handler* seperti Vercel Serverless Function:

```json
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
```

### 2. Format SQEXAM Langsung (`?format=sqexam`)
Digunakan jika frontend ingin mengambil skema instan yang cocok dengan model data aplikasi SQEXAM:

```json
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
```

---

## 🔗 Integrasi Ekosistem GMKNS / SQEXAM

Endpoint ini **didukung penuh** oleh arsitektur pencarian sekolah aplikasi **SQEXAM**. Handler Vercel Node.js (`schools-search.ts`) dapat langsung memanggil API DapodikSQ sebagai *upstream*:

### Contoh Implementasi Handler Vercel (`schools-search.ts`):

```typescript
import type { VercelRequest, VercelResponse } from '@vercel/node'

const SCHOOL_API_URL = 'https://hostgmkns.byethost32.com/dapodiksq/api'

export default async function handler(req: VercelRequest, res: VercelResponse) {
  const q = typeof req.query.q === 'string' ? req.query.q.trim().slice(0, 60) : ''
  const npsn = typeof req.query.npsn === 'string' ? req.query.npsn.trim() : ''
  const limit = Math.min(Number(req.query.limit) || 10, 20)

  const url = new URL(SCHOOL_API_URL)
  if (q) url.searchParams.set('q', q)
  if (npsn) url.searchParams.set('npsn', npsn)
  url.searchParams.set('limit', String(limit))

  try {
    const upstream = await fetch(url, {
      headers: { Accept: 'application/json' },
      signal: AbortSignal.timeout(10_000),
    })

    const payload = await upstream.json()

    if (!upstream.ok || payload.status !== 'success' || !Array.isArray(payload.data)) {
      return res.status(502).json({ success: false, message: 'Invalid Upstream Response' })
    }

    // Pemetaan langsung ke model data SQEXAM
    const schools = payload.data
      .map((school: any) => ({
        id: String(school.sekolah_id || school.npsn),
        npsn: String(school.npsn),
        name: String(school.nama_sekolah),
        school_type: String(school.bentuk_pendidikan),
      }))
      .filter((s: any) => s.id && s.name)

    return res.status(200).json({
      success: true,
      data: schools,
      meta: { count: schools.length },
    })
  } catch (error) {
    return res.status(502).json({ success: false, message: 'School Data Service Unavailable' })
  }
}
```

---

## 💻 Contoh Penggunaan di Berbagai Bahasa

### 1. JavaScript / TypeScript (Frontend Web)
```javascript
async function cariSekolah(keyword) {
  const url = `https://hostgmkns.byethost32.com/dapodiksq/api?q=${encodeURIComponent(keyword)}&limit=10`;
  const res = await fetch(url);
  const json = await res.json();
  
  if (json.status === 'success') {
    json.data.forEach(sch => {
      console.log(`[${sch.npsn}] ${sch.nama_sekolah} - ${sch.kabupaten_kota}`);
    });
  }
}

cariSekolah('SMK 1 Katapang');
```

### 2. Flutter / Dart (Mobile Apps)
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<List<dynamic>> searchSchool(String keyword) async {
  final url = Uri.parse('https://hostgmkns.byethost32.com/dapodiksq/api?q=$keyword&limit=10');
  final response = await http.get(url, headers: {'Accept': 'application/json'});

  if (response.statusCode == 200) {
    final Map<String, dynamic> body = json.decode(response.body);
    return body['data'] ?? [];
  }
  return [];
}
```

### 3. Python
```python
import requests

url = "https://hostgmkns.byethost32.com/dapodiksq/api"
params = {"q": "Negeri 1", "bentuk": "SMK", "limit": 10}

res = requests.get(url, params=params)
if res.status_code == 200:
    for school in res.json().get("data", []):
        print(f"{school['npsn']} - {school['nama_sekolah']}")
```

---

## ⚠️ Kode Status HTTP (HTTP Status Codes)

| Kode | Keterangan | Penjelasan |
| :--- | :--- | :--- |
| `200 OK` | Sukses | Permintaan berhasil diproses dan data sekolah dikembalikan. |
| `405 Method Not Allowed` | Metode Salah | Permintaan dikirim selain menggunakan metode `GET` atau `OPTIONS`. |
| `429 Too Many Requests` | Limit Terlampaui | Alamat IP Anda melebihi batas proteksi rate-limit (maks. 80 request/menit). |
| `500 Internal Server Error` | Gangguan Server | Terjadi kendala saat membaca basis data MySQL. |

---

## ⚖️ Etika dan Tanggung Jawab Penggunaan

Layanan API ini terbuka untuk umum dengan komitmen menjaga ketersediaan layanan bersama:
1. **Pemanfaatan Edukatif**: Ditujukan untuk memajukan integrasi aplikasi pendidikan (seperti SQEXAM), administrasi ujian, dan transparansi data sekolah.
2. **Caching Sisi Klien**: Disarankan melakukan *caching* lokal pada aplikasi klien untuk kata kunci sekolah yang sering dicari.
3. **Anti-Flooding**: Harap tidak melakukan *scraping massal* secara agresif demi menjaga stabilitas server bersama.

---

## 🤝 Ekosistem & Kontak

Bagian dari ekosistem **SmarQuest App** & **GMKNS Platform**. Kontribusi dan pelaporan kendala dapat dilakukan melalui menu *Issues* pada repositori GitHub resmi.
