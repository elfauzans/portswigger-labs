# Lab: Username Enumeration via Different Responses

## Deskripsi

Lab ini menunjukkan bagaimana perbedaan respons dari server dapat digunakan untuk menentukan apakah sebuah username valid.

## Tingkat

Apprentice

## Kerentanan

Username Enumeration

## Tools

- Burp Suite Community
- Burp Intruder

## Langkah

1. Intercept POST /login.
2. Kirim request ke Intruder.
3. Brute-force daftar username.
4. Amati perbedaan response.
5. Identifikasi username valid.
6. Brute-force password untuk username tersebut.
7. Temukan password dari respons 302.
8. Login.

## Dampak

Penyerang dapat mengidentifikasi akun yang valid sehingga mempermudah serangan password spraying atau credential stuffing.

## Mitigasi

- Gunakan pesan error yang sama untuk username dan password yang salah.
- Terapkan rate limiting.
- Tambahkan account lockout.
- Gunakan MFA.

## Screenshot

- Login page
- Intruder username enumeration
- Intruder password brute force
- Login berhasil

username : adm
password : aaaaaa 