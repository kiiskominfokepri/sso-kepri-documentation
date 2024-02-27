# Client Registration

Pendaftaran aplikasi klien akan dilakukan oleh `Pengelola Utama` dari portal [SSO-KEPRI](`https://sso.kepriprov.go.id`).
Anda akan mendapatkan beberapa informasi, seperti:

1. `Client identifier` atau `client-id`, adalah id yang dibutuhkan untuk menyatakan identitas klien.
2. `Client secret`, adalah string rahasia yang dibutuhkan untuk memverifikasi validitas dari request klien. **Informasi ini harus dijaga agar tidak dapat diakses oleh pengguna tak berizin.*

Untuk mendapatkan akses ke OAuth milik SSO-KEPRI, Anda harus menyiapkan beberapa informasi berikut ini:
1. `Redirect URI`, kami membutuhkan daftar `URI` yang Anda siapkan sebagai `handler callback` dari kami. Kami akan meneruskan hasil dari autentikasi ke `URI` yang telah Anda daftarkan.
2. `Resource permission`, kami harus mengetahui resource yang akan Anda akses melalui OAuth. Kami akan membatasi apa yang *`boleh`* dan apa yang *`tidak boleh`* Anda akses dari portal SSO-KEPRI.