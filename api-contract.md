# 📋 API Contract — BPR Sahabat Sejati (Sistem Kehadiran Karyawan)

> **Base URL:** `https://api.bpr-sahabatsejati.co.id/v1`  
> **Content-Type:** `application/json`  
> **Authentication:** Bearer Token (JWT)  
> **Timezone:** `Asia/Jakarta (WIB, UTC+7)`

---

## 📑 Daftar Isi

1. [Autentikasi](#1-autentikasi)
2. [Karyawan (Employees)](#2-karyawan-employees)
3. [Kantor Cabang (Branches)](#3-kantor-cabang-branches)
4. [Absensi (Attendance)](#4-absensi-attendance)
5. [Izin / Cuti (Leave)](#5-izin--cuti-leave)
6. [Poin Kehadiran (Points)](#6-poin-kehadiran-points)
7. [Admin Dashboard](#7-admin-dashboard)
8. [Notifikasi](#8-notifikasi)
9. [Laporan (Reports)](#9-laporan-reports)
10. [Pengaturan (Settings)](#10-pengaturan-settings)
11. [Device Management](#11-device-management)
12. [Upload File (S3)](#12-upload-file-s3)
13. [Error Responses](#13-error-responses)

---

## 📐 Aturan Sistem Poin

| Waktu Check-in | Status        | Poin    |
| -------------- | ------------- | ------- |
| ≤ 08:00        | **Hadir**     | **1**   |
| 08:01 – 08:30  | **Terlambat** | **0.5** |
| > 08:30        | **Terlambat** | **0**   |
| Izin / Cuti    | Izin          | **0**   |

---

## 1. Autentikasi

### 1.1 Login Karyawan (Mobile)

```
POST /auth/login
```

**Request Body:**

```json
{
  "email": "ahmad.fauzi@bpr.co.id",
  "password": "password123",
  "deviceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "deviceModel": "Samsung Galaxy A54",
  "deviceOs": "Android 14"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresAt": "2026-02-15T22:25:58+07:00",
    "user": {
      "id": "clx1a2b3c4d5e6f7g8h9i0",
      "nik": "2023007",
      "name": "Ahmad Fauzi",
      "email": "ahmad.fauzi@bpr.co.id",
      "role": "Teller",
      "phone": "081234567890",
      "avatar": null,
      "branch": {
        "id": "clx9z8y7x6w5v4u3t2s1r0",
        "name": "Cabang Sudirman",
        "latitude": -6.228,
        "longitude": 106.805,
        "radius": 30
      },
      "deviceBound": true
    }
  }
}
```

**Response (401 Unauthorized — Device tidak terdaftar):**

```json
{
  "success": false,
  "error": {
    "code": "DEVICE_NOT_BOUND",
    "message": "Perangkat ini belum terdaftar. Silakan hubungi admin."
  }
}
```

### 1.2 Login Admin (Dashboard)

```
POST /auth/admin/login
```

**Request Body:**

```json
{
  "email": "admin@bpr.co.id",
  "password": "admin123"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresAt": "2026-02-15T22:25:58+07:00",
    "admin": {
      "id": "clxadmin001",
      "name": "Admin User",
      "email": "admin@bpr.co.id",
      "role": "SUPER_ADMIN",
      "status": "ACTIVE"
    }
  }
}
```

### 1.3 Logout

```
POST /auth/logout
```

**Headers:** `Authorization: Bearer {token}`

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Berhasil keluar."
}
```

### 1.4 Refresh Token

```
POST /auth/refresh
```

**Request Body:**

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresAt": "2026-02-16T22:25:58+07:00"
  }
}
```

---

## 2. Karyawan (Employees)

### 2.1 Daftar Karyawan

```
GET /employees?page=1&limit=10&search=ahmad&branch=Cabang+Sudirman
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | -------- | ------- | --------- |
| `page` | integer | 1 | Halaman |
| `limit` | integer | 10 | Item per halaman |
| `search` | string | - | Cari berdasarkan nama atau NIK |
| `branch` | string | - | Filter berdasarkan cabang |
| `device` | string | - | Filter status device: `bound`, `unbound` |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "employees": [
      {
        "id": "clx1a2b3c4d5e6f7g8h9i0",
        "nik": "2023007",
        "name": "Ahmad Fauzi",
        "email": "ahmad.fauzi@bpr.co.id",
        "phone": "081234567890",
        "role": "Teller",
        "branch": {
          "id": "clx9z8y7x6w5v4u3t2s1r0",
          "name": "Cabang Sudirman"
        },
        "deviceBound": true,
        "isActive": true,
        "createdAt": "2023-01-15T08:00:00+07:00"
      },
      {
        "id": "clx2b3c4d5e6f7g8h9i0j1",
        "nik": "2023001",
        "name": "Andi Pratama",
        "email": "andi.pratama@bpr.co.id",
        "phone": "081298765432",
        "role": "IT Support",
        "branch": {
          "id": "clxbranch001",
          "name": "Kantor Pusat"
        },
        "deviceBound": true,
        "isActive": true,
        "createdAt": "2023-01-10T08:00:00+07:00"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "totalItems": 45,
      "totalPages": 5
    }
  }
}
```

### 2.2 Detail Karyawan

```
GET /employees/:id
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clx1a2b3c4d5e6f7g8h9i0",
    "nik": "2023007",
    "name": "Ahmad Fauzi",
    "email": "ahmad.fauzi@bpr.co.id",
    "phone": "081234567890",
    "role": "Teller",
    "avatar": null,
    "branch": {
      "id": "clx9z8y7x6w5v4u3t2s1r0",
      "name": "Cabang Sudirman",
      "address": "Gedung Artha Graha Lt. 3, SCBD, Jakarta Selatan"
    },
    "deviceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "deviceModel": "Samsung Galaxy A54",
    "deviceOs": "Android 14",
    "isActive": true,
    "createdAt": "2023-01-15T08:00:00+07:00",
    "updatedAt": "2026-02-14T10:30:00+07:00"
  }
}
```

### 2.3 Tambah Karyawan

```
POST /employees
```

**Request Body:**

```json
{
  "nik": "2023011",
  "name": "Rina Wulandari",
  "email": "rina.wulandari@bpr.co.id",
  "password": "Temp@Pass123",
  "phone": "081355566677",
  "role": "Customer Service",
  "branchId": "clx9z8y7x6w5v4u3t2s1r0"
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "clxnew001",
    "nik": "2023011",
    "name": "Rina Wulandari",
    "email": "rina.wulandari@bpr.co.id",
    "phone": "081355566677",
    "role": "Customer Service",
    "branch": {
      "id": "clx9z8y7x6w5v4u3t2s1r0",
      "name": "Cabang Sudirman"
    },
    "deviceBound": false,
    "isActive": true,
    "createdAt": "2026-02-14T22:25:58+07:00"
  },
  "message": "Karyawan berhasil ditambahkan."
}
```

### 2.4 Update Karyawan

```
PUT /employees/:id
```

**Request Body:**

```json
{
  "name": "Ahmad Fauzi",
  "email": "ahmad.fauzi@bpr.co.id",
  "phone": "081234567890",
  "role": "Senior Teller",
  "branchId": "clxbranch001"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clx1a2b3c4d5e6f7g8h9i0",
    "nik": "2023007",
    "name": "Ahmad Fauzi",
    "role": "Senior Teller",
    "branch": {
      "id": "clxbranch001",
      "name": "Kantor Pusat"
    },
    "updatedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Data karyawan berhasil diperbarui."
}
```

### 2.5 Hapus Karyawan

```
DELETE /employees/:id
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Karyawan berhasil dihapus."
}
```

### 2.6 Profil Karyawan (Mobile — Self)

```
GET /employees/me
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clx1a2b3c4d5e6f7g8h9i0",
    "nik": "2023007",
    "name": "Ahmad Fauzi",
    "email": "ahmad.fauzi@bpr.co.id",
    "phone": "081234567890",
    "role": "Teller",
    "avatar": null,
    "branch": {
      "id": "clx9z8y7x6w5v4u3t2s1r0",
      "name": "Cabang Sudirman",
      "address": "Gedung Artha Graha Lt. 3, SCBD, Jakarta Selatan"
    },
    "deviceModel": "Samsung Galaxy A54",
    "deviceOs": "Android 14",
    "totalPoints": 15.5
  }
}
```

---

## 3. Kantor Cabang (Branches)

### 3.1 Daftar Cabang

```
GET /branches
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": [
    {
      "id": "clxbranch001",
      "name": "Kantor Pusat",
      "address": "Jl. Jend. Sudirman Kav. 52-53, Jakarta Selatan",
      "latitude": -6.225,
      "longitude": 106.8,
      "radius": 50,
      "employeeCount": 120,
      "isActive": true
    },
    {
      "id": "clx9z8y7x6w5v4u3t2s1r0",
      "name": "Cabang Sudirman",
      "address": "Gedung Artha Graha Lt. 3, SCBD, Jakarta Selatan",
      "latitude": -6.228,
      "longitude": 106.805,
      "radius": 30,
      "employeeCount": 45,
      "isActive": true
    },
    {
      "id": "clxbranch003",
      "name": "Cabang Gatot Subroto",
      "address": "Menara Mulia, Jl. Gatot Subroto, Jakarta Selatan",
      "latitude": -6.23,
      "longitude": 106.81,
      "radius": 30,
      "employeeCount": 38,
      "isActive": true
    },
    {
      "id": "clxbranch004",
      "name": "Kas Pasar Minggu",
      "address": "Ruko Pasar Minggu Blok A12, Jakarta Selatan",
      "latitude": -6.26,
      "longitude": 106.845,
      "radius": 25,
      "employeeCount": 12,
      "isActive": true
    },
    {
      "id": "clxbranch005",
      "name": "Kas Kemang",
      "address": "Jl. Kemang Raya No. 8, Jakarta Selatan",
      "latitude": -6.258,
      "longitude": 106.817,
      "radius": 25,
      "employeeCount": 10,
      "isActive": true
    }
  ]
}
```

### 3.2 Tambah Cabang

```
POST /branches
```

**Request Body:**

```json
{
  "name": "Cabang Kuningan",
  "address": "Jl. HR Rasuna Said Kav. C5, Jakarta Selatan",
  "latitude": -6.235,
  "longitude": 106.83,
  "radius": 35
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "clxbranch006",
    "name": "Cabang Kuningan",
    "address": "Jl. HR Rasuna Said Kav. C5, Jakarta Selatan",
    "latitude": -6.235,
    "longitude": 106.83,
    "radius": 35,
    "employeeCount": 0,
    "isActive": true,
    "createdAt": "2026-02-14T22:25:58+07:00"
  },
  "message": "Kantor cabang berhasil ditambahkan."
}
```

### 3.3 Update Cabang

```
PUT /branches/:id
```

**Request Body:**

```json
{
  "name": "Cabang Kuningan",
  "address": "Jl. HR Rasuna Said Kav. C5-C6, Jakarta Selatan",
  "latitude": -6.235,
  "longitude": 106.831,
  "radius": 40
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxbranch006",
    "name": "Cabang Kuningan",
    "address": "Jl. HR Rasuna Said Kav. C5-C6, Jakarta Selatan",
    "latitude": -6.235,
    "longitude": 106.831,
    "radius": 40,
    "updatedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Data cabang berhasil diperbarui."
}
```

### 3.4 Hapus Cabang

```
DELETE /branches/:id
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Kantor cabang berhasil dihapus."
}
```

**Response (409 Conflict — Masih ada karyawan):**

```json
{
  "success": false,
  "error": {
    "code": "BRANCH_HAS_EMPLOYEES",
    "message": "Tidak dapat menghapus cabang yang masih memiliki karyawan aktif."
  }
}
```

---

## 4. Absensi (Attendance)

### 4.1 Check-In (Mobile)

```
POST /attendance/check-in
```

**Request Body:**

```json
{
  "latitude": -6.2284,
  "longitude": 106.8053,
  "photo": "data:image/jpeg;base64,/9j/4AAQ...",
  "deviceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Response (200 OK — Hadir, ≤ 08:00):**

```json
{
  "success": true,
  "data": {
    "id": "clxatt001",
    "date": "2026-02-14",
    "checkInTime": "2026-02-14T07:55:12+07:00",
    "status": "HADIR",
    "points": 1,
    "branch": {
      "name": "Cabang Sudirman"
    },
    "insideRadius": true,
    "message": "Check In Berhasil!"
  }
}
```

**Response (200 OK — Terlambat, 08:01-08:30):**

```json
{
  "success": true,
  "data": {
    "id": "clxatt002",
    "date": "2026-02-14",
    "checkInTime": "2026-02-14T08:15:00+07:00",
    "status": "TERLAMBAT",
    "points": 0.5,
    "branch": {
      "name": "Cabang Sudirman"
    },
    "insideRadius": true,
    "message": "Check In Berhasil! (Terlambat - 0.5 poin)"
  }
}
```

**Response (200 OK — Terlambat, > 08:30):**

```json
{
  "success": true,
  "data": {
    "id": "clxatt003",
    "date": "2026-02-14",
    "checkInTime": "2026-02-14T08:45:00+07:00",
    "status": "TERLAMBAT",
    "points": 0,
    "branch": {
      "name": "Cabang Sudirman"
    },
    "insideRadius": true,
    "message": "Check In Berhasil! (Terlambat - 0 poin)"
  }
}
```

**Response (403 Forbidden — Diluar radius):**

```json
{
  "success": false,
  "error": {
    "code": "OUTSIDE_GEOFENCE",
    "message": "Anda berada di luar radius kantor. Jarak: 150m (batas: 30m)."
  }
}
```

**Response (409 Conflict — Sudah check-in):**

```json
{
  "success": false,
  "error": {
    "code": "ALREADY_CHECKED_IN",
    "message": "Anda sudah melakukan check-in hari ini pada pukul 07:55 WIB."
  }
}
```

**Response (403 Forbidden — Sedang izin/cuti):**

```json
{
  "success": false,
  "error": {
    "code": "ON_LEAVE",
    "message": "Anda sedang dalam masa Izin Sakit. Absensi dinonaktifkan."
  }
}
```

### 4.2 Check-Out (Mobile)

```
POST /attendance/check-out
```

**Request Body:**

```json
{
  "latitude": -6.2284,
  "longitude": 106.8053
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxatt001",
    "date": "2026-02-14",
    "checkInTime": "2026-02-14T07:55:12+07:00",
    "checkOutTime": "2026-02-14T17:02:45+07:00",
    "durationMinutes": 548,
    "durationFormatted": "9j 7m",
    "status": "HADIR",
    "points": 1,
    "message": "Check Out Berhasil!"
  }
}
```

### 4.3 Status Hari Ini (Mobile)

```
GET /attendance/today
```

**Response (200 OK — Sudah check-in):**

```json
{
  "success": true,
  "data": {
    "date": "2026-02-14",
    "status": "checked_in",
    "checkInTime": "2026-02-14T07:55:12+07:00",
    "checkOutTime": null,
    "attendanceStatus": "HADIR",
    "points": 1,
    "onLeave": false
  }
}
```

**Response (200 OK — Sedang cuti):**

```json
{
  "success": true,
  "data": {
    "date": "2026-02-14",
    "status": "on_leave",
    "checkInTime": null,
    "checkOutTime": null,
    "attendanceStatus": null,
    "points": 0,
    "onLeave": true,
    "leaveInfo": {
      "type": "IZIN_SAKIT",
      "dates": "14 Feb 2026"
    }
  }
}
```

### 4.4 Daftar Absensi (Dashboard)

```
GET /attendance?page=1&limit=10&date=2026-02-14&branch=Cabang+Sudirman&status=HADIR
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | -------- | ------------ | --------- |
| `page` | integer | 1 | Halaman |
| `limit` | integer | 10 | Item per halaman |
| `date` | string | today | Filter tanggal (YYYY-MM-DD) |
| `branch` | string | - | Filter cabang |
| `status` | string | - | Filter status |
| `search` | string | - | Cari nama/NIK |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "attendances": [
      {
        "id": "clxatt001",
        "date": "2026-02-14",
        "employee": {
          "nik": "2023007",
          "name": "Ahmad Fauzi",
          "branch": "Cabang Sudirman"
        },
        "checkInTime": "07:55",
        "checkOutTime": "17:02",
        "duration": "9j 7m",
        "status": "Hadir",
        "points": 1,
        "checkInPhoto": "https://is3.cloudhost.id/bpr-ams/selfie/clx1a2b3c/2026-02-14.jpg"
      },
      {
        "id": "clxatt002",
        "date": "2026-02-14",
        "employee": {
          "nik": "2023003",
          "name": "Budi Santoso",
          "branch": "Cabang Sudirman"
        },
        "checkInTime": "08:15",
        "checkOutTime": "17:20",
        "duration": "9j 5m",
        "status": "Terlambat",
        "points": 0.5
      },
      {
        "id": "clxatt003",
        "date": "2026-02-14",
        "employee": {
          "nik": "2023004",
          "name": "Dewi Lestari",
          "branch": "Cabang Sudirman"
        },
        "checkInTime": null,
        "checkOutTime": null,
        "duration": "-",
        "status": "Izin Sakit",
        "points": 0
      }
    ],
    "summary": {
      "totalEmployees": 45,
      "hadir": 38,
      "terlambat": 4,
      "izin": 2,
      "alpha": 1
    },
    "pagination": {
      "page": 1,
      "limit": 10,
      "totalItems": 45,
      "totalPages": 5
    }
  }
}
```

### 4.5 Riwayat Absensi Karyawan (Mobile)

```
GET /attendance/history?month=2&year=2026
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | ------- | ------------ | --------- |
| `month` | integer | bulan ini | Bulan (1-12) |
| `year` | integer | tahun ini | Tahun |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "month": "Februari",
    "year": 2026,
    "records": [
      {
        "date": "2026-02-14",
        "day": "Sabtu",
        "checkIn": "07:55:12",
        "checkOut": "17:02:45",
        "duration": "9j 7m",
        "status": "Hadir"
      },
      {
        "date": "2026-02-13",
        "day": "Jumat",
        "checkIn": "08:15:18",
        "checkOut": "17:10:33",
        "duration": "8j 55m",
        "status": "Terlambat"
      },
      {
        "date": "2026-02-12",
        "day": "Kamis",
        "checkIn": null,
        "checkOut": null,
        "duration": "-",
        "status": "Alpha"
      }
    ],
    "summary": {
      "hadir": 8,
      "terlambat": 3,
      "alpha": 1,
      "izin": 0
    }
  }
}
```

### 4.6 Ringkasan Poin Kehadiran (Dashboard)

```
GET /attendance/points-summary?month=2&year=2026&branch=Semua+Cabang
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | ------- | --------- | --------- |
| `month` | integer | bulan ini | Bulan (1-12) |
| `year` | integer | tahun ini | Tahun |
| `branch` | string | - | Filter cabang |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "month": "Februari",
    "year": 2026,
    "employees": [
      {
        "nik": "2023001",
        "name": "Andi Pratama",
        "branch": "Kantor Pusat",
        "tepatWaktu": 16,
        "setengahPoin": 4,
        "terlambat": 2,
        "totalPoin": 18
      },
      {
        "nik": "2023002",
        "name": "Siti Rahayu",
        "branch": "Kantor Pusat",
        "tepatWaktu": 19,
        "setengahPoin": 2,
        "terlambat": 1,
        "totalPoin": 20
      }
    ],
    "summary": {
      "totalPoin": 183.5,
      "avgPoin": 18.35,
      "totalTepatWaktu": 165,
      "totalTerlambat": 21
    },
    "rules": {
      "onTime": { "label": "Sampai 08:00", "points": 1 },
      "halfPoint": { "label": "08:01 – 08:30", "points": 0.5 },
      "late": { "label": "Setelah 08:30", "points": 0 }
    }
  }
}
```

---

## 5. Izin / Cuti (Leave)

### 5.1 Ajukan Izin (Mobile)

```
POST /leave
```

**Request Body:**

```json
{
  "type": "IZIN_SAKIT",
  "startDate": "2026-02-15",
  "endDate": "2026-02-16",
  "reason": "Demam tinggi dan perlu istirahat",
  "attachment": "data:image/jpeg;base64,/9j/4AAQ..."
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "clxleave001",
    "type": "IZIN_SAKIT",
    "startDate": "2026-02-15",
    "endDate": "2026-02-16",
    "reason": "Demam tinggi dan perlu istirahat",
    "attachment": "https://is3.cloudhost.id/bpr-ams/leave/clxleave001.jpg",
    "status": "PENDING",
    "createdAt": "2026-02-14T22:25:58+07:00"
  },
  "message": "Permohonan izin berhasil diajukan."
}
```

### 5.2 Daftar Permohonan Izin (Dashboard)

```
GET /leave?status=PENDING&page=1&limit=10
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | ------- | ------- | --------- |
| `page` | integer | 1 | Halaman |
| `limit` | integer | 10 | Item per halaman |
| `status` | string | - | Filter status (PENDING, APPROVED, REJECTED) |
| `type` | string | - | Filter tipe izin |
| `search` | string | - | Cari nama/NIK |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "requests": [
      {
        "id": "clxleave001",
        "employee": {
          "nik": "2023004",
          "name": "Dewi Lestari",
          "avatar": "DL",
          "branch": "Cabang Sudirman"
        },
        "type": "IZIN_SAKIT",
        "dates": "14 Feb 2026",
        "reason": "Demam tinggi dan perlu istirahat",
        "attachment": "https://is3.cloudhost.id/bpr-ams/leave/clxleave001.jpg",
        "status": "PENDING",
        "createdAt": "2026-02-13T20:30:00+07:00"
      },
      {
        "id": "clxleave002",
        "employee": {
          "nik": "2023006",
          "name": "Maya Sari",
          "avatar": "MS",
          "branch": "Cabang Gatot Subroto"
        },
        "type": "IZIN_CUTI",
        "dates": "14 - 16 Feb 2026",
        "reason": "Acara keluarga di luar kota",
        "attachment": null,
        "status": "PENDING",
        "createdAt": "2026-02-12T14:00:00+07:00"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

### 5.3 Approve Izin (Dashboard)

```
PATCH /leave/:id/approve
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxleave001",
    "status": "APPROVED",
    "approvedBy": {
      "name": "Admin User",
      "role": "SUPER_ADMIN"
    },
    "approvedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Izin berhasil disetujui."
}
```

### 5.4 Reject Izin (Dashboard)

```
PATCH /leave/:id/reject
```

**Request Body:**

```json
{
  "reason": "Dokumen pendukung tidak lengkap"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxleave001",
    "status": "REJECTED",
    "rejectReason": "Dokumen pendukung tidak lengkap",
    "approvedBy": {
      "name": "Admin User",
      "role": "SUPER_ADMIN"
    },
    "approvedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Izin berhasil ditolak."
}
```

### 5.5 Riwayat Izin Karyawan (Mobile)

```
GET /leave/me
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": [
    {
      "id": "clxleave010",
      "type": "IZIN_CUTI",
      "startDate": "2026-01-10",
      "endDate": "2026-01-12",
      "reason": "Acara keluarga",
      "status": "APPROVED",
      "createdAt": "2026-01-08T10:00:00+07:00"
    },
    {
      "id": "clxleave011",
      "type": "IZIN_SAKIT",
      "startDate": "2026-01-25",
      "endDate": "2026-01-25",
      "reason": "Flu berat",
      "attachment": "https://is3.cloudhost.id/bpr-ams/leave/clxleave011.jpg",
      "status": "APPROVED",
      "createdAt": "2026-01-25T07:00:00+07:00"
    },
    {
      "id": "clxleave012",
      "type": "IZIN_SETENGAH_HARI",
      "startDate": "2026-02-03",
      "endDate": "2026-02-03",
      "reason": "Keperluan ke bank",
      "status": "PENDING",
      "createdAt": "2026-02-02T18:00:00+07:00"
    }
  ]
}
```

### 5.6 Edit Izin (Dashboard)

```
PUT /leave/:id
```

**Request Body:**

```json
{
  "type": "IZIN_CUTI",
  "startDate": "2026-02-15",
  "endDate": "2026-02-17",
  "reason": "Perpanjangan cuti — acara keluarga"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxleave002",
    "type": "IZIN_CUTI",
    "startDate": "2026-02-15",
    "endDate": "2026-02-17",
    "reason": "Perpanjangan cuti — acara keluarga",
    "status": "PENDING",
    "updatedAt": "2026-02-14T22:35:00+07:00"
  },
  "message": "Data izin berhasil diperbarui."
}
```

---

## 6. Poin Kehadiran (Points)

### 6.1 Poin Karyawan (Mobile)

```
GET /points/me?month=2&year=2026
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | ------- | --------- | --------- |
| `month` | integer | bulan ini | Bulan (1-12) |
| `year` | integer | tahun ini | Tahun |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "totalPoints": 15.5,
    "monthPoints": 5.5,
    "daysPresent": 20,
    "month": "Februari",
    "year": 2026,
    "records": [
      {
        "label": "Absensi Tepat Waktu",
        "date": "2026-02-14",
        "time": "07:55",
        "points": 1,
        "type": "tepat"
      },
      {
        "label": "Absensi 08:01-08:30",
        "date": "2026-02-13",
        "time": "08:15",
        "points": 0.5,
        "type": "setengah"
      },
      {
        "label": "Absensi Tepat Waktu",
        "date": "2026-02-12",
        "time": "07:50",
        "points": 1,
        "type": "tepat"
      },
      {
        "label": "Absensi 08:01-08:30",
        "date": "2026-02-11",
        "time": "08:10",
        "points": 0.5,
        "type": "setengah"
      },
      {
        "label": "Absensi Tepat Waktu",
        "date": "2026-02-07",
        "time": "08:00",
        "points": 1,
        "type": "tepat"
      },
      {
        "label": "Tidak Hadir",
        "date": "2026-02-06",
        "time": null,
        "points": 0,
        "type": "alpha"
      },
      {
        "label": "Absensi Terlambat",
        "date": "2026-02-05",
        "time": "08:45",
        "points": 0,
        "type": "alpha"
      }
    ],
    "rules": {
      "onTime": { "label": "Sampai 08:00", "points": 1 },
      "halfPoint": { "label": "08:01 – 08:30", "points": 0.5 },
      "late": { "label": "Setelah 08:30", "points": 0 }
    }
  }
}
```

---

## 7. Admin Dashboard

### 7.1 Daftar Admin

```
GET /admins?search=admin
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": [
    {
      "id": "clxadmin001",
      "name": "Admin User",
      "email": "admin@bpr.co.id",
      "role": "SUPER_ADMIN",
      "status": "ACTIVE",
      "createdAt": "2025-01-01T08:00:00+07:00"
    },
    {
      "id": "clxadmin002",
      "name": "Sarah Admin",
      "email": "sarah@bpr.co.id",
      "role": "ADMIN",
      "status": "ACTIVE",
      "createdAt": "2025-06-15T08:00:00+07:00"
    },
    {
      "id": "clxadmin003",
      "name": "Budi Viewer",
      "email": "budi.viewer@bpr.co.id",
      "role": "VIEWER",
      "status": "INACTIVE",
      "createdAt": "2025-09-01T08:00:00+07:00"
    }
  ]
}
```

### 7.2 Tambah Admin

```
POST /admins
```

**Request Body:**

```json
{
  "name": "Rini Admin",
  "email": "rini@bpr.co.id",
  "password": "SecurePass@456",
  "role": "ADMIN"
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "clxadmin004",
    "name": "Rini Admin",
    "email": "rini@bpr.co.id",
    "role": "ADMIN",
    "status": "ACTIVE",
    "createdAt": "2026-02-14T22:25:58+07:00"
  },
  "message": "Admin berhasil ditambahkan."
}
```

### 7.3 Update Admin

```
PUT /admins/:id
```

**Request Body:**

```json
{
  "name": "Rini Admin",
  "email": "rini@bpr.co.id",
  "role": "SUPER_ADMIN"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxadmin004",
    "name": "Rini Admin",
    "email": "rini@bpr.co.id",
    "role": "SUPER_ADMIN",
    "status": "ACTIVE",
    "updatedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Data admin berhasil diperbarui."
}
```

### 7.4 Hapus Admin

```
DELETE /admins/:id
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Admin berhasil dihapus."
}
```

### 7.5 Toggle Status Admin

```
PATCH /admins/:id/toggle-status
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "clxadmin003",
    "status": "ACTIVE"
  },
  "message": "Status admin berhasil diubah."
}
```

### 7.6 Dashboard Summary

```
GET /dashboard/summary?comparePeriod=week
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| --------- | ------ | ------- | --------- |
| `comparePeriod` | string | `week` | Periode pembanding untuk kalkulasi persentase perubahan: `day` (vs kemarin), `week` (vs minggu lalu), `month` (vs bulan lalu) |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "stats": {
      "totalKaryawan": {
        "value": 248,
        "change": "+12%",
        "trend": "up"
      },
      "hadirHariIni": {
        "value": 219,
        "change": "+4.5%",
        "trend": "up"
      },
      "terlambat": {
        "value": 12,
        "change": "-2.1%",
        "trend": "down"
      },
      "alpha": {
        "value": 17,
        "change": "+1.2%",
        "trend": "up"
      }
    },
    "attendanceChart": {
      "labels": ["Sen", "Sel", "Rab", "Kam", "Jum", "Sab"],
      "hadir": [220, 235, 228, 242, 215, 200],
      "alpha": [28, 13, 20, 6, 33, 48]
    },
    "recentCheckins": [
      {
        "id": "clxatt001",
        "name": "Andi Pratama",
        "role": "IT Support",
        "time": "07:45",
        "status": "Hadir",
        "avatar": "AP"
      },
      {
        "id": "clxatt002",
        "name": "Siti Rahayu",
        "role": "HR Manager",
        "time": "07:50",
        "status": "Hadir",
        "avatar": "SR"
      },
      {
        "id": "clxatt003",
        "name": "Budi Santoso",
        "role": "Finance",
        "time": "08:12",
        "status": "Terlambat",
        "avatar": "BS"
      },
      {
        "id": "clxatt004",
        "name": "Dewi Lestari",
        "role": "Marketing",
        "time": "08:15",
        "status": "Terlambat",
        "avatar": "DL"
      },
      {
        "id": "clxatt005",
        "name": "Rizki Hidayat",
        "role": "Operations",
        "time": "08:32",
        "status": "Terlambat",
        "avatar": "RH"
      },
      {
        "id": "clxatt006",
        "name": "Ahmad Fauzi",
        "role": "Teller",
        "time": "07:55",
        "status": "Hadir",
        "avatar": "AF"
      }
    ]
  }
}
```

> **Catatan:**
>
> - `change` dihitung berdasarkan `comparePeriod` terhadap periode sebelumnya
> - `trend`: `"up"` artinya naik (hijau), `"down"` artinya turun (merah)
> - Untuk `terlambat`, trend `"down"` dianggap positif (lebih sedikit terlambat)
> - Data `attendanceChart` mengikuti hari kerja minggu ini
> - `recentCheckins` menampilkan 6 check-in terbaru hari ini

---

## 8. Notifikasi

### 8.1 Daftar Notifikasi

```
GET /notifications?page=1&limit=20
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "clxnotif001",
        "type": "LEAVE_APPROVED",
        "title": "Izin Disetujui",
        "message": "Permohonan Izin Cuti Anda (10-12 Jan 2026) telah disetujui.",
        "isRead": false,
        "createdAt": "2026-01-09T14:00:00+07:00"
      },
      {
        "id": "clxnotif002",
        "type": "POINT_MILESTONE",
        "title": "Pencapaian Poin",
        "message": "Selamat! Anda telah mengumpulkan 15 poin kehadiran bulan ini.",
        "isRead": true,
        "createdAt": "2026-01-31T17:00:00+07:00"
      },
      {
        "id": "clxnotif003",
        "type": "DEVICE_RESET",
        "title": "Perangkat Direset",
        "message": "Perangkat Anda telah direset oleh admin. Silakan login kembali.",
        "isRead": false,
        "createdAt": "2026-02-01T09:00:00+07:00"
      }
    ],
    "unreadCount": 2,
    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 3,
      "totalPages": 1
    }
  }
}
```

### 8.2 Tandai Notifikasi Dibaca

```
PATCH /notifications/:id/read
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Notifikasi ditandai telah dibaca."
}
```

### 8.3 Tandai Semua Dibaca

```
PATCH /notifications/read-all
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Semua notifikasi ditandai telah dibaca."
}
```

---

## 9. Laporan (Reports)

### 9.1 Laporan Kehadiran (Dashboard)

```
GET /reports/attendance?startDate=2026-02-01&endDate=2026-02-14&branch=Semua+Cabang
```

**Query Parameters:**
| Parameter | Tipe | Default | Deskripsi |
| ----------- | ------ | ------------- | --------- |
| `startDate` | string | awal bulan | Tanggal mulai (YYYY-MM-DD) |
| `endDate` | string | hari ini | Tanggal akhir (YYYY-MM-DD) |
| `branch` | string | Semua Cabang | Filter cabang |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "period": {
      "startDate": "2026-02-01",
      "endDate": "2026-02-14",
      "branch": "Semua Cabang"
    },
    "report": [
      {
        "nik": "2023001",
        "name": "Andi Pratama",
        "branch": "Kantor Pusat",
        "hadir": 20,
        "terlambat": 2,
        "izin": 1,
        "totalDurasi": "176j 30m",
        "poin": 18.5
      },
      {
        "nik": "2023002",
        "name": "Siti Rahayu",
        "branch": "Kantor Pusat",
        "hadir": 21,
        "terlambat": 1,
        "izin": 0,
        "totalDurasi": "185j 15m",
        "poin": 20.5
      },
      {
        "nik": "2023007",
        "name": "Ahmad Fauzi",
        "branch": "Cabang Sudirman",
        "hadir": 20,
        "terlambat": 2,
        "izin": 0,
        "totalDurasi": "178j 45m",
        "poin": 18.5
      }
    ],
    "pointRules": {
      "info": "Sampai 08:00 → 1 poin (Hadir) | 08:01–08:30 → 0.5 poin (Terlambat) | Setelah 08:30 → 0 poin (Terlambat)"
    }
  }
}
```

### 9.2 Ekspor Laporan (CSV)

```
GET /reports/attendance/export?format=csv&startDate=2026-02-01&endDate=2026-02-14
```

**Response:** Binary file download (CSV)

**Headers:**

```
Content-Type: text/csv
Content-Disposition: attachment; filename="laporan-kehadiran-2026-02-01-2026-02-14.csv"
```

### 9.3 Ekspor Laporan (PDF)

```
GET /reports/attendance/export?format=pdf&startDate=2026-02-01&endDate=2026-02-14
```

**Response:** Binary file download (PDF)

**Headers:**

```
Content-Type: application/pdf
Content-Disposition: attachment; filename="laporan-kehadiran-2026-02-01-2026-02-14.pdf"
```

---

## 10. Pengaturan (Settings)

### 10.1 Ambil Pengaturan Aplikasi

```
GET /settings
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "defaultCheckIn": "08:00",
    "halfPointEnd": "08:30",
    "defaultRadius": 50,
    "companyName": "BPR Sahabat Sejati",
    "companyLogo": null,
    "timezone": "Asia/Jakarta",
    "pointRules": {
      "onTime": {
        "maxTime": "08:00",
        "points": 1,
        "status": "HADIR"
      },
      "halfPoint": {
        "startTime": "08:01",
        "endTime": "08:30",
        "points": 0.5,
        "status": "TERLAMBAT"
      },
      "late": {
        "afterTime": "08:30",
        "points": 0,
        "status": "TERLAMBAT"
      },
      "leave": {
        "points": 0,
        "note": "Absensi dinonaktifkan"
      }
    }
  }
}
```

### 10.2 Perbarui Pengaturan

```
PUT /settings
```

**Request Body:**

```json
{
  "defaultCheckIn": "08:00",
  "halfPointEnd": "08:30",
  "defaultRadius": 50,
  "companyName": "BPR Sahabat Sejati"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "defaultCheckIn": "08:00",
    "halfPointEnd": "08:30",
    "defaultRadius": 50,
    "companyName": "BPR Sahabat Sejati",
    "updatedAt": "2026-02-14T22:30:00+07:00"
  },
  "message": "Pengaturan berhasil disimpan."
}
```

---

## 11. Device Management

### 11.1 Reset Device ID Karyawan

```
PATCH /employees/:id/reset-device
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "employeeId": "clx1a2b3c4d5e6f7g8h9i0",
    "name": "Ahmad Fauzi",
    "nik": "2023007",
    "deviceId": null,
    "deviceModel": null,
    "deviceOs": null,
    "previousDeviceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
  },
  "message": "Device ID berhasil direset. Karyawan dapat login dengan perangkat baru."
}
```

### 11.2 Bind Device (Otomatis saat login pertama)

Proses binding otomatis terjadi saat karyawan login untuk pertama kali setelah device direset. Dihandle oleh endpoint `POST /auth/login`.

---

## 12. Upload File (S3)

### 12.1 Upload Lampiran

```
POST /upload
Content-Type: multipart/form-data
```

**Form Data:**
| Field | Tipe | Deskripsi |
| -------- | ------ | --------- |
| `file` | File | File yang di-upload (max 5MB) |
| `type` | string | Tipe file: `selfie`, `leave_attachment`, `avatar` |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "url": "https://is3.cloudhost.id/bpr-ams/leave/clxfile001.jpg",
    "key": "leave/clxfile001.jpg",
    "size": 245678,
    "mimeType": "image/jpeg"
  }
}
```

**Response (413 — File terlalu besar):**

```json
{
  "success": false,
  "error": {
    "code": "FILE_TOO_LARGE",
    "message": "Ukuran file maksimal 5MB."
  }
}
```

---

## 13. Error Responses

### Format Error Standar

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Deskripsi error dalam bahasa Indonesia.",
    "details": {}
  }
}
```

### Daftar Error Code

| HTTP Code | Error Code             | Deskripsi                      |
| --------- | ---------------------- | ------------------------------ |
| 400       | `VALIDATION_ERROR`     | Data request tidak valid       |
| 401       | `UNAUTHORIZED`         | Token tidak valid atau expired |
| 401       | `INVALID_CREDENTIALS`  | Email atau password salah      |
| 401       | `DEVICE_NOT_BOUND`     | Perangkat belum terdaftar      |
| 403       | `FORBIDDEN`            | Tidak memiliki akses           |
| 403       | `OUTSIDE_GEOFENCE`     | Diluar radius geofencing       |
| 403       | `ON_LEAVE`             | Sedang dalam masa cuti/izin    |
| 403       | `ACCOUNT_INACTIVE`     | Akun dinonaktifkan             |
| 404       | `NOT_FOUND`            | Resource tidak ditemukan       |
| 409       | `ALREADY_CHECKED_IN`   | Sudah check-in hari ini        |
| 409       | `ALREADY_CHECKED_OUT`  | Sudah check-out hari ini       |
| 409       | `BRANCH_HAS_EMPLOYEES` | Cabang masih memiliki karyawan |
| 409       | `DUPLICATE_NIK`        | NIK sudah terdaftar            |
| 409       | `DUPLICATE_EMAIL`      | Email sudah terdaftar          |
| 413       | `FILE_TOO_LARGE`       | File melebihi batas ukuran     |
| 429       | `RATE_LIMIT`           | Terlalu banyak request         |
| 500       | `INTERNAL_ERROR`       | Kesalahan server internal      |

### Contoh Validation Error

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data tidak valid.",
    "details": {
      "email": "Format email tidak valid.",
      "password": "Password minimal 8 karakter.",
      "nik": "NIK harus diisi."
    }
  }
}
```

---

## 📌 Catatan Teknis

### Headers yang Diperlukan

```
Authorization: Bearer {jwt_token}
Content-Type: application/json
X-Device-Id: {device_uuid}          // Hanya mobile
X-App-Version: 1.0.0                // Hanya mobile
Accept-Language: id-ID
```

### Rate Limiting

| Endpoint                | Limit        |
| ----------------------- | ------------ |
| `POST /auth/login`      | 5 req/menit  |
| `POST /attendance/*`    | 2 req/menit  |
| Lainnya (authenticated) | 60 req/menit |

### Pagination Default

- Default `page`: 1
- Default `limit`: 10
- Maximum `limit`: 100

### Timezone

- Semua waktu menggunakan `Asia/Jakarta (UTC+7)`
- Format: ISO 8601 (`2026-02-14T22:25:58+07:00`)
- Tanggal saja: `YYYY-MM-DD`

### S3 Storage (IDCloudHost)

- **Endpoint:** `is3.cloudhost.id`
- **Bucket:** `bpr-ams`
- **Path Structure:**
  - Selfie: `selfie/{employeeId}/{date}.jpg`
  - Lampiran: `leave/{leaveRequestId}.jpg`
  - Avatar: `avatar/{employeeId}.jpg`
  - Logo: `company/logo.png`
