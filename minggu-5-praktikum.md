# Praktikum Minggu 5 --- CRUD Laravel API

## 1. Informasi Proyek

-   Project: `praktikum4`
-   Base URL lokal: `http://127.0.0.1:8000`
-   Pengujian: Postman

## 2. Endpoint API

  --------------------------------------------------------------------------
  Method            URL                 Fungsi            Status umum
  ----------------- ------------------- ----------------- ------------------
  GET               `/api/books`        Daftar buku       `200 OK`

  POST              `/api/books`        Tambah buku       `201 Created`

  GET               `/api/books/{id}`   Detail buku       `200 OK` 

  PUT/PATCH         `/api/books/{id}`   Perbarui buku     `200 OK`

  DELETE            `/api/books/{id}`   Hapus buku        `204 No Content`
                                                          
  --------------------------------------------------------------------------

Periksa route dengan `php artisan route:list --path=api/books`. Pastikan
URL tidak menjadi `/api/api/books`.

## 3. Validasi Request

### StoreBookRequest

Aturan yang perlu ada: - `title`: wajib, string, maksimal 200
karakter. - `isbn`: wajib, string, tepat 13 karakter, unik di tabel
`books`. - `available`: opsional; jika dikirim harus boolean.

Pesan validasi kustom yang terlihat pada implementasi: -
`Title wajib diisi.` - `Title maksimal 200 karakter.` -
`ISBN wajib diisi.` - `ISBN harus terdiri dari 13 karakter.` -
`ISBN sudah terdaftar.` - `Available harus berupa boolean.`

### UpdateBookRequest

Aturan ISBN harus mengabaikan ID buku yang sedang diperbarui agar ISBN
milik buku itu sendiri tidak dianggap duplikat. Untuk mendukung PATCH
sebagian, field yang tidak dikirim perlu memakai `sometimes`. Jika
`title` dan `isbn` tetap `required` tanpa `sometimes`, request yang
hanya berisi `available` akan menghasilkan `422`.

Contoh aturan inti untuk update sebagian:

``` php
'title' => ['sometimes', 'required', 'string', 'max:200'],
'isbn' => [
    'sometimes',
    'required',
    'string',
    'size:13',
    Rule::unique('books', 'isbn')->ignore($bookId),
],
'available' => ['sometimes', 'boolean'],
```

Pastikan `use Illuminate\\Validation\\Rule;` ada di bagian atas file
jika menggunakan `Rule::unique`.

## 4. Resource Response

Respons sukses yang memakai `BookResource` seharusnya konsisten,
biasanya dibungkus objek `data`. Berdasarkan screenshot Postman yang
sudah berhasil, respons create berisi:

``` json
{
  "data": {
    "id": 3,
    "title": "Belajar Pemrograman API",
    "isbn": "9786020301231",
    "available": true,
    "created_at": "2026-10-12T09:59:47.000000Z"
  }
}
```

Waktu tersebut adalah nilai yang terlihat di screenshot dan bisa berbeda
pada request lain. Pastikan `BookResource` menampilkan `updated_at` juga
jika itu bagian dari kontrak yang digunakan.

## 5. Postman Testing

Untuk request JSON, pilih **Body → raw → JSON**. Pastikan header
`Accept: application/json` dan `Content-Type: application/json` aktif.

### Test 1 --- Create Berhasil

-   Method: `POST`
-   URL: `http://127.0.0.1:8000/api/books`
-   Payload yang sudah terbukti berhasil:

``` json
{
  "title": "Belajar Pemrograman API",
  "isbn": "9786020301231",
  "available": true
}
```

Hasil yang terlihat: `201 Created`, dengan response body
    "data": {
        "id": 3,
        "title": "Belajar Pemrograman API",
        "isbn": "9786020301231",
        "available": true,
        "created_at": "2026-10-12T09:59:47.000000Z"
    }

### Test 2 --- Validation Gagal

-   Method: `POST`
-   URL: `http://127.0.0.1:8000/api/books`
-   Body:

``` json
{
  "title": "",
  "isbn": "123"
}
```

`422 Unprocessable Content
bentuk respons:

``` json
    "message": "Title wajib diisi. (and 1 more error)",
    "errors": {
        "title": [
            "Title wajib diisi."
        ],
        "isbn": [
            "ISBN wajib diisi."
        ]
    }
```

### Test 3 --- Update Sebagian

-   Method: `PATCH`
-   URL: `http://127.0.0.1:8000/api/books/ID_AKTUAL`
-   Body:

``` json
{
  "available": false
}
```
response
{
    "data": {
        "id": 3,
        "title": "Belajar Pemrograman API",
        "isbn": "9786020301231",
        "available": false,
        "created_at": "2026-10-12T09:59:47.000000Z"
    }
}
### Test 4 --- Not Found

-   Method: `GET`
-   URL: `http://127.0.0.1:8000/api/books/999999`
-   Hasil : `404 Not Found`.

``` json
{
  "message": "Book not found",
  "errors": null
}
```
### Test 5 --- Delete

Gunakan ID buku uji yang ada dan tidak perlu dipertahankan.

-   Method: `DELETE`
-   URL: `http://127.0.0.1:8000/api/books/ID_AKTUAL`
-   Hasil: `204 No Content`, tanpa body.

## 6. Deklarasi Penggunaan AI

AI digunakan sebagai alat bantu belajar untuk membantu memahami struktur
Laravel API, membaca pesan error, menelusuri kemungkinan penyebab
kesalahan, dan memahami cara menguji endpoint melalui Postman. Saran
yang diberikan diperiksa kembali dengan menjalankan perintah PHP/Laravel
dan menguji endpoint secara langsung. Pemeriksaan hasil dan bukti akhir
didasarkan pada hasil aktual dari proyek.

## 7. Referensi

-   [Laravel --- Validation](https://laravel.com/docs/validation)
-   [Laravel --- Eloquent
    Resources](https://laravel.com/docs/eloquent-resources)
-   [Laravel --- Controllers](https://laravel.com/docs/controllers)
-   [Postman --- Send
    requests](https://learning.postman.com/docs/sending-requests/requests/)
