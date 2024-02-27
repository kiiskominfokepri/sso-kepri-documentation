# OpenID Connect scopes

Selain cakupan otorisasi normal yang dijelaskan sebelumnya, SSO-KEPRI mendukung `scopes` OpenID Connect yang umum, diantaranya:

| OIDC scope | Deskripsi |
| --------- | ----------- |
| openid | Akan menyertakan `id_token` dalam respon token, dengan pengidentifikasi subjek (`sub` claim) |
| profile | Akses ke data: `name`, `family_name`, `given_name`, `middle_name`, `nickname`, `preferred_username`, `profile`, `website`, `gender`, `birthdate`, `zoneinfo`, `locale`, dan `updated_at` |
| email | Akses ke data: `email`, `email_verified` |
| address | Akses ke data `address` |
| phone | Akses ke data: `phone_number` dan `phone_number_verified` |
| groups | Akses ke daftar grup tempat pengguna berada |
| attributes | Akses ke atribut yang diberikan kepada pengguna oleh admin, disimpan sebagai pasangan nilai-kunci (key-value pairs) |
| offline_access | Akses ke token penyegaran jenis `Offline`, memungkinkan klien mendapatkan token akses baru tanpa memerlukan interaksi langsung |
