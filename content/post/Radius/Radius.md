+++
title = 'Cara Membuat Radius Server di Ubuntu 22.04'
date = 2024-08-30T21:31:43+07:00
draft = false
+++

# Tutorial Membangun Radius Server Menggunakan FreeRADIUS di Ubuntu 22.04

FreeRADIUS adalah server RADIUS yang memungkinkan Anda untuk membuat server otentikasi bagi jaringan Anda, seperti jaringan Wi-Fi, yang dapat menangani banyak pengguna. Tutorial ini akan membimbing Anda dalam menginstal FreeRADIUS pada Ubuntu Server 22.04.

## Memulai Instalasi

Langkah pertama adalah memastikan bahwa sistem Anda up-to-date. Anda dapat melakukannya dengan menjalankan perintah berikut di terminal:

```bash
sudo apt update
sudo apt upgrade -y
```

### Menginstal FreeRADIUS dan Dependencies

Berikut adalah beberapa paket yang perlu Anda instal jika belum ada dalam sistem Anda. Beberapa di antaranya adalah `php`, `mariadb`, dan `freeradius`.

```bash
sudo apt install php apache2 php-mbstring php-xml mariadb-server freeradius freeradius-mysql -y
```

Anda mungkin perlu mengaktifkan beberapa modul Apache. Anda dapat melakukannya dengan menjalankan perintah berikut:

```bash
sudo systemctl enable --now apache2
sudo systemctl enable --now mariadb
```

Setelah Apache diaktifkan, Anda dapat melanjutkan untuk mengkonfigurasi MariaDB (jika ini adalah instalasi pertama).

```bash
sudo mysql_secure_installation
```

Ikuti instruksi yang muncul di layar untuk mengamankan MariaDB Anda.

## Mengatur MariaDB

Setelah MariaDB terinstal, kita dapat mulai mengkonfigurasi database. Bergantung pada bagaimana Anda mengatur MariaDB, Anda mungkin perlu masuk sebagai root untuk menjalankan perintah SQL. Gunakan perintah berikut untuk masuk:

```bash
sudo mysql -u root -p
```

Setelah Anda masuk, Anda perlu melakukan beberapa langkah berikut:

1. Membuat database.
2. Membuat user untuk FreeRADIUS.
3. Memberikan hak akses pada user tersebut.

Untuk membuat database, jalankan perintah `CREATE DATABASE` berikut:

```sql
CREATE DATABASE radius;
```

Sekarang kita perlu membuat user untuk FreeRADIUS. Disarankan untuk menggunakan password yang aman.

```sql
CREATE USER 'radius'@'localhost' IDENTIFIED BY 'password_aman';
```

Untuk memberikan hak akses, jalankan perintah:

```sql
GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
```

Jalankan perintah berikut untuk memastikan hak akses diberikan secara penuh:

```sql
FLUSH PRIVILEGES;
exit;
```

## Mengonfigurasi FreeRADIUS untuk Menggunakan SQL

Sekarang setelah FreeRADIUS terinstal, kita perlu memberi tahu FreeRADIUS untuk menulis login ke SQL database yang telah kita buat.

Buka file konfigurasi:

```bash
sudo nano /etc/freeradius/3.0/mods-available/sql
```

Cari baris yang dimulai dengan `driver = "rlm_sql_mysql"` dan hilangkan tanda komentar (hapus tanda `#` di depannya). Juga, pastikan Anda mengonfigurasi `server`, `login`, dan `password` dengan informasi yang sesuai:

```text
server = "localhost"
login = "radius"
password = "password_aman"
```

Simpan perubahan dengan menekan `Ctrl + X`, kemudian `Y`, diikuti dengan `Enter`.

### Menyelesaikan Konfigurasi FreeRADIUS

Aktifkan modul SQL dengan menjalankan perintah berikut:

```bash
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
sudo systemctl restart freeradius
```

## Optional: Instalasi PhpMyAdmin

Langkah ini bersifat opsional. Namun, disarankan untuk menginstal PhpMyAdmin untuk memudahkan manajemen database FreeRADIUS melalui antarmuka web. Anda dapat menginstalnya dengan menjalankan perintah berikut:

```bash
sudo apt install phpmyadmin -y
```

## Penugasan VLAN Dinamis

Jika Anda memiliki kebutuhan jaringan yang lebih kompleks, Anda dapat mengaktifkan VLAN dinamis. Ini dilakukan dengan menambahkan beberapa baris konfigurasi ke file FreeRADIUS.

Buka file `/etc/freeradius/3.0/sites-enabled/default` dan tambahkan konfigurasi VLAN sesuai kebutuhan.

## Hal yang Perlu Diketahui

Pastikan untuk selalu restart FreeRADIUS setiap kali Anda melakukan perubahan konfigurasi. Ini dapat dilakukan dengan menjalankan perintah:

```bash
sudo systemctl restart freeradius
```

## Menghubungkan ke RADIUS Server

Terakhir, Anda perlu menghubungkan perangkat jaringan atau sistem otentikasi Anda ke server RADIUS yang baru saja Anda buat. Pastikan untuk menambahkan IP address dari perangkat Anda sebagai NAS (Network Access Server) di file konfigurasi FreeRADIUS.

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

Tambahkan informasi perangkat Anda di file tersebut, simpan, dan restart FreeRADIUS.

## Bonus: Migrasi Server RADIUS

Jika Anda memiliki server RADIUS lama dan ingin memigrasikan data ke server baru, Anda perlu menyalin file `.crt`, `.key`, dan file lainnya yang relevan ke server baru.

---

Semoga tutorial ini bermanfaat dalam membantu Anda membangun server RADIUS di Ubuntu 22.04!
