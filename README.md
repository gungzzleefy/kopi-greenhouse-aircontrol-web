# 🌿 Kopi Greenhouse Aircontrol Web

> **Repository Description (for GitHub):**
> A Laravel-based IoT web platform for real-time monitoring and automated control of coffee greenhouse environments. Integrated with ESP32/Arduino sensors for temperature & humidity tracking, automated water pump control, and AI-powered coffee leaf disease diagnosis using FastAPI + Google Gemini, with a complementary mobile app via RESTful API.

---

## 📋 Table of Contents
- [About The Project](#about-the-project)
- [System Architecture](#system-architecture)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Project Flow](#project-flow)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [License](#license)
- [Indonesian Version / Versi Bahasa Indonesia](#-versi-bahasa-indonesia)

---

## About The Project

**Kopi Greenhouse Aircontrol** is an IoT-based web application built with Laravel 10 that serves as the central control and monitoring hub for a coffee plant greenhouse. The system integrates hardware sensors (temperature & humidity), automated relay control for water pumps, and an AI-powered coffee leaf disease diagnosis engine.

This platform acts as both:
- **A web dashboard** for administrators and staff to monitor live sensor data, manage greenhouse automation settings, view diagnosis history, and manage blog content.
- **A RESTful API backend** for the companion mobile application (Flutter), enabling remote access to all features from a smartphone.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     IoT Hardware Layer                       │
│   ESP32 / Arduino  ──►  DHT Sensor (Temp & Humidity)         │
│       └──► Relay Module (Water Pump Control)                 │
└──────────────────────┬───────────────────────────────────────┘
                       │ HTTP POST /api/senddata
                       ▼
┌──────────────────────────────────────────────────────────────┐
│              Laravel 10 Web Application (This Repo)          │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │  Web Routes │  │  API Routes  │  │  Sanctum Auth (API) │ │
│  │  (Blade UI) │  │  (JSON REST) │  └─────────────────────┘ │
│  └─────────────┘  └──────┬───────┘                          │
│         │                │                                   │
│  ┌──────▼────────────────▼──────────────────────────────┐   │
│  │              Controllers Layer                        │   │
│  │  Dashboard │ Auth │ Blog │ Karyawan │ RekamdataCtrl  │   │
│  │  ApiGetDataalatCtrl │ PredicCtrl │ SettingOtomatis   │   │
│  └────────────────────────┬─────────────────────────────┘   │
│                           │                                   │
│  ┌────────────────────────▼─────────────────────────────┐   │
│  │               MySQL Database                          │   │
│  │  users │ penggunas │ alats │ monicontrollings         │   │
│  │  settingotomatis │ otomatis │ diagnosapenyakitdauns   │   │
│  │  blogs │ reset_pasword_otps                           │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────┬────────────────────┬─────────────────────────┘
               │                    │
               ▼                    ▼
┌──────────────────┐   ┌────────────────────────────┐
│  FastAPI Server  │   │  Flutter Mobile App        │
│  (ML Model -     │   │  (Companion App)            │
│  Leaf Disease    │   │  Consumes REST API          │
│  Detection)      │   └────────────────────────────┘
└──────────────────┘
               │
               ▼
┌──────────────────────────┐
│  Google Gemini API       │
│  (AI Disease Description)│
└──────────────────────────┘
```

---

## Key Features

### 🖥️ Web Dashboard
- **Real-time Monitoring** — Live display of temperature and humidity data streamed from IoT sensors
- **Historical Data Charts** — View sensor data charts by date range with export to Excel/PDF
- **Automated Pump Control** — Toggle water pump relay manually or configure automatic triggers
- **Automatic Settings (2 modes)**:
  - *By Threshold* — Pump activates when temperature/humidity goes outside defined ranges
  - *By Time Schedule* — Pump activates during configured time windows (up to 2 time slots)
- **AI Leaf Disease Diagnosis** — Upload a coffee leaf photo; the system predicts the disease using a FastAPI ML model and generates a detailed description via Google Gemini API
- **Diagnosis History** — View and manage all past leaf diagnoses with predicted disease class and confidence score
- **Blog Management** — Create, edit, delete, and view blog articles for information sharing
- **Employee Management** — CRUD for greenhouse staff/employee data
- **User Profile** — Update personal info and profile photo
- **OTP-based Password Reset** — Forgot password flow using phone number + OTP verification

### 📱 Mobile API (Laravel Sanctum)
- Token-based authentication (login/logout)
- Get latest sensor readings per device
- Control water pump relay
- Get sensor chart data (weekly or by date range)
- Submit leaf disease diagnosis
- View diagnosis history (filter: latest, last hour, by disease type)
- Update profile photo and user data
- Change password

---

## Tech Stack

| Component | Technology |
|---|---|
| Backend Framework | Laravel 10 (PHP 8.2) |
| Authentication | Laravel Sanctum |
| Database | MySQL |
| Frontend Template | Blade (Laravel Templating Engine) |
| UI Alerts | SweetAlert2 (realrashid/sweet-alert) |
| Data Export | Maatwebsite Excel + PhpSpreadsheet |
| HTTP Client | Guzzle HTTP |
| AI Disease Diagnosis (ML) | FastAPI (Python) — external service |
| AI Text Generation | Google Gemini API (gemini-pro) |
| IoT Hardware | ESP32 / Arduino + DHT Sensor |
| Mobile App Backend | Laravel Sanctum REST API |
| Build Tool | Vite |

---

## Database Schema

| Table | Description |
|---|---|
| `users` | Authentication table (phone number + password login) |
| `penggunas` | Extended user profile (name, address, photo, description) |
| `alats` | IoT device registry with relay status flag |
| `monicontrollings` | Time-series log of sensor readings (temperature & humidity) per device |
| `settingotomatis` | Automation rules — by temperature/humidity threshold or by time schedule |
| `otomatis` | Current automation on/off state |
| `diagnosapenyakitdauns` | Leaf disease diagnosis records (image file, predicted class, confidence, AI description) |
| `blogs` | Blog/article content |
| `penggunas` | Staff/employee records |
| `reset_pasword_otps` | OTP tokens for password reset via phone number |

### Detected Disease Classes
| Class | Description |
|---|---|
| `nodisease` | Healthy leaf — no disease detected |
| `miner` | Leaf miner disease |
| `phoma` | Phoma leaf spot |
| `rust` | Coffee leaf rust (Hemileia vastatrix) |
| `NotFound` | Image not recognized as a coffee leaf |

---

## API Endpoints

### Public Endpoints
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/login` | Login and get Sanctum token |
| POST | `/api/senddata` | Receive sensor data from IoT device |
| GET | `/api/relay` | Get current relay/pump status (for ESP32 polling) |
| POST | `/api/diagnosa/{user_id}` | Submit leaf image for disease diagnosis |
| POST | `/api/lupa-password` | Initiate password reset via phone |
| POST | `/api/lupa-password/verifikasi-otp/{phone}` | Verify OTP |
| POST | `/api/lupa-password/reset-password/{phone}` | Set new password |
| POST | `/api/lupa-password/kirim-ulang-otp/{phone}` | Resend OTP |

### Protected Endpoints (Sanctum Token Required)
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/logout` | Logout and revoke token |
| POST | `/api/check-token` | Validate active token |
| GET | `/api/getdataalat/{device_id}` | Get latest sensor reading for a device |
| GET | `/api/get-pengguna/{user_id}` | Get user profile |
| POST | `/api/aturpompa` | Toggle water pump on/off |
| GET | `/api/chart` | Get weekly sensor data chart |
| GET | `/api/chartdaritanggal/{start}/{end}` | Get sensor chart by date range |
| POST | `/api/updatefoto/{user_id}` | Update profile photo |
| POST | `/api/update-data-pengguna-without-photo/{id}` | Update user data without changing photo |
| POST | `/api/change-password/{user_id}` | Change user password |
| GET | `/api/data-diagnosa/{params}` | Get diagnoses (all/recent/by disease type) |
| GET | `/api/data-diagnosa-detail/{id}` | Get single diagnosis detail |
| DELETE | `/api/data-diagnosa/{id}` | Delete a diagnosis record |

---

## Project Flow

### 1. IoT Sensor Data Flow
```
ESP32 reads DHT sensor
  → POST /api/senddata (temperature, humidity, device_id)
    → Saved to monicontrollings table
      → Web dashboard polls /fetch-data
        → Real-time display updates
```

### 2. Pump Automation Flow
```
Admin sets automation rule (threshold or time-based)
  → Saved to settingotomatis table
    → ESP32 polls GET /api/relay
      → If relay status = 1, ESP32 activates pump
        → Relay status toggled by web dashboard or app
```

### 3. Leaf Disease Diagnosis Flow
```
User uploads coffee leaf image (web or mobile)
  → Image forwarded to FastAPI ML server at :8585/predict/
    → ML model returns predicted_class + confidence score
      → Description generated via Google Gemini API
        → Result saved to diagnosapenyakitdauns table
          → User sees diagnosis + AI explanation
```

### 4. Authentication Flow (Web)
```
User visits /login → enters phone + password
  → Session-based auth (Laravel Auth)
    → Redirected to /dashboard (protected by auth middleware)
```

### 5. Authentication Flow (Mobile API)
```
Mobile app POST /api/login → returns Sanctum token
  → All subsequent requests include Bearer token
    → Laravel Sanctum validates token on protected routes
```

---

## Getting Started

### Prerequisites
- PHP >= 8.2
- Composer
- MySQL
- Node.js & NPM
- FastAPI ML server running (for leaf disease feature)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/agungkurniawanid/kopi-greenhouse-aircontrol-web.git
cd kopi-greenhouse-aircontrol-web

# 2. Install PHP dependencies
composer install

# 3. Install JS dependencies
npm install

# 4. Copy environment file
cp .env.example .env

# 5. Generate application key
php artisan key:generate

# 6. Configure .env (see Environment Variables section)

# 7. Run database migrations
php artisan migrate

# 8. Build frontend assets
npm run dev

# 9. Start the development server
php artisan serve
```

---

## Environment Variables

Edit `.env` with the following values:

```env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=             # Generated by php artisan key:generate
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=kopi-greenhouse-apps
DB_USERNAME=root
DB_PASSWORD=

MAIL_MAILER=log
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

> **Note:** The FastAPI ML server URL and Google Gemini API key are currently hardcoded in `ApiGetDataalatController.php` and `PredicController.php`. Move these to `.env` for production use.

---

## License

This project is open-sourced software licensed under the [MIT License](https://opensource.org/licenses/MIT).

---
---

# 🇮🇩 Versi Bahasa Indonesia

---

## Tentang Proyek Ini

**Kopi Greenhouse Aircontrol** adalah aplikasi web berbasis IoT yang dibangun menggunakan Laravel 10, berfungsi sebagai pusat kontrol dan pemantauan untuk greenhouse (rumah kaca) tanaman kopi. Sistem ini mengintegrasikan sensor hardware (suhu & kelembaban), kontrol relay otomatis untuk pompa air, serta mesin diagnosa penyakit daun kopi bertenaga AI.

Platform ini berfungsi sebagai:
- **Dashboard web** bagi admin dan staf untuk memantau data sensor secara langsung, mengatur otomasi greenhouse, melihat riwayat diagnosa, dan mengelola konten blog.
- **Backend API RESTful** untuk aplikasi mobile Flutter pendamping, memungkinkan akses jarak jauh ke semua fitur dari smartphone.

---

## Arsitektur Sistem

Lihat diagram arsitektur di bagian English di atas — arsitektur berlaku sama.

---

## Fitur Utama

### 🖥️ Dashboard Web
- **Pemantauan Real-time** — Tampilan langsung data suhu dan kelembaban dari sensor IoT
- **Grafik Data Historis** — Lihat grafik sensor berdasarkan rentang tanggal, ekspor ke Excel/PDF
- **Kontrol Pompa Otomatis** — Nyalakan/matikan relay pompa secara manual atau dengan pengaturan otomatis
- **Pengaturan Otomatis (2 mode)**:
  - *Berdasarkan Ambang Batas* — Pompa aktif jika suhu/kelembaban di luar rentang yang ditentukan
  - *Berdasarkan Jadwal Waktu* — Pompa aktif pada jam tertentu (hingga 2 slot waktu)
- **Diagnosa Penyakit Daun AI** — Upload foto daun kopi; sistem memprediksi penyakit menggunakan model ML FastAPI dan menghasilkan deskripsi detail via Google Gemini API
- **Riwayat Diagnosa** — Lihat dan kelola semua riwayat diagnosa daun beserta kelas penyakit dan skor kepercayaan
- **Manajemen Blog** — Buat, edit, hapus, dan lihat artikel blog untuk berbagi informasi
- **Manajemen Karyawan** — CRUD data staf/karyawan greenhouse
- **Profil Pengguna** — Perbarui informasi pribadi dan foto profil
- **Reset Password via OTP** — Alur lupa password menggunakan nomor telepon + verifikasi OTP

### 📱 API Mobile (Laravel Sanctum)
- Autentikasi berbasis token (login/logout)
- Ambil pembacaan sensor terbaru per perangkat
- Kontrol relay/pompa air
- Data grafik sensor (mingguan atau berdasarkan tanggal)
- Kirim diagnosa penyakit daun
- Lihat riwayat diagnosa (filter: terbaru, sejam lalu, per jenis penyakit)
- Perbarui foto profil dan data pengguna
- Ganti password

---

## Tech Stack

| Komponen | Teknologi |
|---|---|
| Framework Backend | Laravel 10 (PHP 8.2) |
| Autentikasi | Laravel Sanctum |
| Database | MySQL |
| Template Frontend | Blade (Laravel) |
| Alert UI | SweetAlert2 (realrashid/sweet-alert) |
| Ekspor Data | Maatwebsite Excel + PhpSpreadsheet |
| HTTP Client | Guzzle HTTP |
| Diagnosa Penyakit (ML) | FastAPI (Python) — layanan eksternal |
| Generasi Teks AI | Google Gemini API (gemini-pro) |
| Hardware IoT | ESP32 / Arduino + Sensor DHT |
| Backend Aplikasi Mobile | Laravel Sanctum REST API |
| Build Tool | Vite |

---

## Skema Database

| Tabel | Keterangan |
|---|---|
| `users` | Tabel autentikasi (login dengan nomor telepon + password) |
| `penggunas` | Profil pengguna diperluas (nama, alamat, foto, deskripsi) |
| `alats` | Registri perangkat IoT beserta status relay |
| `monicontrollings` | Log time-series pembacaan sensor (suhu & kelembaban) per perangkat |
| `settingotomatis` | Aturan otomasi — berdasarkan ambang suhu/kelembaban atau jadwal waktu |
| `otomatis` | Status on/off otomasi saat ini |
| `diagnosapenyakitdauns` | Catatan diagnosa penyakit daun (file gambar, kelas prediksi, kepercayaan, deskripsi AI) |
| `blogs` | Konten blog/artikel |
| `reset_pasword_otps` | Token OTP untuk reset password via nomor telepon |

### Kelas Penyakit yang Terdeteksi
| Kelas | Keterangan |
|---|---|
| `nodisease` | Daun sehat — tidak ada penyakit |
| `miner` | Penyakit penambang daun (Leaf Miner) |
| `phoma` | Bercak daun Phoma |
| `rust` | Karat daun kopi (Hemileia vastatrix) |
| `NotFound` | Gambar tidak dikenali sebagai daun kopi |

---

## Alur Proyek

### 1. Alur Data Sensor IoT
```
ESP32 membaca sensor DHT
  → POST /api/senddata (temperature, humidity, id_alat)
    → Disimpan ke tabel monicontrollings
      → Dashboard web polling /fetch-data
        → Tampilan real-time diperbarui
```

### 2. Alur Otomasi Pompa
```
Admin mengatur aturan otomasi (ambang batas atau jadwal waktu)
  → Disimpan ke tabel settingotomatis
    → ESP32 polling GET /api/relay
      → Jika status relay = 1, ESP32 mengaktifkan pompa
        → Status relay dikontrol dari dashboard web atau aplikasi
```

### 3. Alur Diagnosa Penyakit Daun
```
Pengguna upload foto daun kopi (web atau mobile)
  → Gambar diteruskan ke server FastAPI ML di :8585/predict/
    → Model ML mengembalikan predicted_class + skor confidence
      → Deskripsi dihasilkan via Google Gemini API
        → Hasil disimpan ke tabel diagnosapenyakitdauns
          → Pengguna melihat hasil diagnosa + penjelasan AI
```

### 4. Alur Autentikasi Web
```
Pengguna buka /login → masukkan nomor telepon + password
  → Autentikasi berbasis sesi (Laravel Auth)
    → Redirect ke /dashboard (dilindungi middleware auth)
```

### 5. Alur Autentikasi Mobile API
```
Aplikasi mobile POST /api/login → menerima token Sanctum
  → Semua permintaan berikutnya menyertakan Bearer token
    → Laravel Sanctum memvalidasi token di route yang dilindungi
```

---

## Cara Instalasi

### Prasyarat
- PHP >= 8.2
- Composer
- MySQL
- Node.js & NPM
- Server FastAPI ML yang berjalan (untuk fitur diagnosa daun)

### Langkah Instalasi

```bash
# 1. Clone repositori
git clone https://github.com/agungkurniawanid/kopi-greenhouse-aircontrol-web.git
cd kopi-greenhouse-aircontrol-web

# 2. Install dependensi PHP
composer install

# 3. Install dependensi JS
npm install

# 4. Salin file environment
cp .env.example .env

# 5. Generate application key
php artisan key:generate

# 6. Konfigurasi .env (lihat bagian Environment Variables)

# 7. Jalankan migrasi database
php artisan migrate

# 8. Build aset frontend
npm run dev

# 9. Jalankan server pengembangan
php artisan serve
```

---

## Variabel Lingkungan (.env)

Edit file `.env` dengan nilai berikut:

```env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=             # Di-generate oleh php artisan key:generate
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=kopi-greenhouse-apps
DB_USERNAME=root
DB_PASSWORD=
```

> **Catatan:** URL server FastAPI ML dan API key Google Gemini saat ini masih hardcoded di `ApiGetDataalatController.php` dan `PredicController.php`. Pindahkan ke `.env` untuk penggunaan produksi.

---

## Lisensi

Proyek ini adalah perangkat lunak open-source yang dilisensikan di bawah [MIT License](https://opensource.org/licenses/MIT).
