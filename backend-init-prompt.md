# Backend Initialization Prompt — BPR Sahabat Sejati (Sistem Kehadiran Karyawan)

> Prompt ini digunakan untuk menginisiasi project backend NestJS dari nol hingga siap dikembangkan.

---

## 1. Tech Stack

| Layer              | Technology                                                  |
| ------------------ | ----------------------------------------------------------- |
| **Runtime**        | Node.js v22 LTS                                             |
| **Framework**      | NestJS v11 (latest)                                         |
| **Language**       | TypeScript (strict mode)                                    |
| **ORM**            | Prisma ORM v6 (latest)                                      |
| **Database**       | PostgreSQL 16                                               |
| **Authentication** | JWT (`@nestjs/jwt` + `passport-jwt`)                        |
| **Validation**     | `class-validator` + `class-transformer`                     |
| **Hashing**        | `bcrypt`                                                    |
| **File Upload**    | `@aws-sdk/client-s3` (IDCloudHost S3-Compatible)            |
| **Scheduler**      | `@nestjs/schedule` (cron job Alpha records)                 |
| **Config**         | `@nestjs/config` (environment-based)                        |
| **API Docs**       | `@nestjs/swagger` (OpenAPI 3.0)                             |
| **Rate Limiting**  | `@nestjs/throttler`                                         |
| **Logging**        | NestJS built-in Logger (console) + custom `LoggerModule`    |
| **Testing**        | Jest (unit) + Supertest (e2e)                               |
| **Linting**        | ESLint + Prettier (NestJS default config)                   |

---

## 2. Arsitektur & Prinsip

### 2.1 Clean Modular Architecture

Setiap domain bisnis adalah **1 NestJS Module** yang mandiri dan terisolasi. Tidak ada logic bisnis di controller — semua ada di service. Gunakan prinsip:

- **Single Responsibility**: 1 module = 1 domain bisnis
- **Dependency Injection**: Semua dependency lewat constructor injection
- **DTO Pattern**: Request validation pakai class-validator DTO
- **Repository Pattern**: Prisma service sebagai data access layer terpusat
- **Guard Pattern**: Auth & role-based access control pakai custom guards
- **Interceptor Pattern**: Response transformation konsisten
- **Filter Pattern**: Global exception handling terpusat

### 2.2 Konvensi Penamaan

| Item            | Convention         | Contoh                              |
| --------------- | ------------------ | ----------------------------------- |
| File            | kebab-case         | `employee.service.ts`               |
| Class           | PascalCase         | `EmployeeService`                   |
| Method          | camelCase          | `findById()`                        |
| Variable        | camelCase          | `employeeId`                        |
| Constant        | UPPER_SNAKE_CASE   | `MAX_FILE_SIZE`                     |
| Enum            | PascalCase         | `AttendanceStatus`                  |
| DTO             | PascalCase + Dto   | `CreateEmployeeDto`                 |
| Interface       | PascalCase         | `JwtPayload`                        |
| Guard           | PascalCase + Guard | `JwtAuthGuard`                      |
| Folder          | kebab-case         | `leave-request/`                    |
| DB Table (map)  | snake_case         | `leave_requests` (Prisma @@map)     |

---

## 3. Folder Structure

```
bpr-ams-backend/
├── prisma/
│   ├── schema.prisma              # Schema database (copy dari dokumentasi)
│   ├── migrations/                # Auto-generated migrations
│   └── seed.ts                    # Seeder: admin default, branch default, app_settings
│
├── src/
│   ├── main.ts                    # Bootstrap: global pipes, filters, interceptors, swagger
│   ├── app.module.ts              # Root module: import semua feature modules
│   │
│   ├── common/                    # === SHARED / CROSS-CUTTING ===
│   │   ├── constants/
│   │   │   └── index.ts           # App-wide constants (POINT_RULES, MAX_FILE_SIZE, dll)
│   │   │
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts    # @CurrentUser() parameter decorator
│   │   │   ├── roles.decorator.ts           # @Roles('SUPER_ADMIN','ADMIN') decorator
│   │   │   └── public.decorator.ts          # @Public() bypass JWT auth
│   │   │
│   │   ├── dto/
│   │   │   ├── pagination-query.dto.ts      # page, limit, search (reusable)
│   │   │   └── api-response.dto.ts          # { success, data, message, error } wrapper
│   │   │
│   │   ├── filters/
│   │   │   ├── http-exception.filter.ts         # Catch HttpException → format standar
│   │   │   └── prisma-exception.filter.ts       # Catch Prisma errors (unique, not found)
│   │   │
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts            # Extends AuthGuard('jwt'), handle @Public()
│   │   │   ├── roles.guard.ts               # Check admin role from JWT payload
│   │   │   └── device-binding.guard.ts      # Verify X-Device-Id header match
│   │   │
│   │   ├── interceptors/
│   │   │   ├── response-transform.interceptor.ts  # Wrap semua response → { success: true, data }
│   │   │   └── logging.interceptor.ts             # Log request method, url, duration
│   │   │
│   │   ├── interfaces/
│   │   │   ├── jwt-payload.interface.ts     # { sub, email, type: 'employee'|'admin', role? }
│   │   │   └── request-with-user.interface.ts  # Express Request + user property
│   │   │
│   │   ├── pipes/
│   │   │   └── parse-date.pipe.ts           # Parse & validate date string → Date
│   │   │
│   │   └── utils/
│   │       ├── pagination.util.ts           # Helper: buildPaginationMeta(total, page, limit)
│   │       ├── date.util.ts                 # Helper: isWorkday(), getWibNow(), formatDuration()
│   │       └── geofence.util.ts             # Helper: calculateDistance(lat1,lng1,lat2,lng2)
│   │
│   ├── config/                    # === CONFIGURATION MODULE ===
│   │   ├── config.module.ts       # ConfigModule.forRoot() dengan validasi
│   │   ├── app.config.ts          # registerAs('app') — PORT, NODE_ENV
│   │   ├── database.config.ts     # registerAs('database') — DATABASE_URL
│   │   ├── jwt.config.ts          # registerAs('jwt') — JWT_SECRET, JWT_EXPIRY
│   │   ├── s3.config.ts           # registerAs('s3') — S3_ENDPOINT, S3_BUCKET, S3_KEY, S3_SECRET
│   │   └── env.validation.ts      # Joi / class-validator schema untuk .env
│   │
│   ├── prisma/                    # === PRISMA MODULE ===
│   │   ├── prisma.module.ts       # Global module, exports PrismaService
│   │   └── prisma.service.ts      # extends PrismaClient, onModuleInit/onModuleDestroy
│   │
│   ├── auth/                      # === AUTH MODULE ===
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts     # POST /auth/login, /auth/admin/login, /auth/logout, /auth/refresh
│   │   ├── auth.service.ts        # Login logic, JWT sign/verify, device binding, refresh token
│   │   ├── strategies/
│   │   │   └── jwt.strategy.ts    # PassportStrategy(Strategy) — validate JWT payload
│   │   └── dto/
│   │       ├── login-employee.dto.ts    # email, password, deviceId, deviceModel, deviceOs
│   │       ├── login-admin.dto.ts       # email, password
│   │       └── refresh-token.dto.ts     # refreshToken
│   │
│   ├── employee/                  # === EMPLOYEE MODULE ===
│   │   ├── employee.module.ts
│   │   ├── employee.controller.ts       # CRUD + GET /employees/me + PATCH reset-device
│   │   ├── employee.service.ts
│   │   └── dto/
│   │       ├── create-employee.dto.ts   # nik, name, email, password, phone?, role, branchId
│   │       ├── update-employee.dto.ts   # PartialType(CreateEmployeeDto) tanpa password
│   │       └── query-employee.dto.ts    # extends PaginationQueryDto + branch?, device?
│   │
│   ├── branch/                    # === BRANCH MODULE ===
│   │   ├── branch.module.ts
│   │   ├── branch.controller.ts         # CRUD /branches
│   │   ├── branch.service.ts
│   │   └── dto/
│   │       ├── create-branch.dto.ts     # name, address, latitude, longitude, radius?
│   │       └── update-branch.dto.ts     # PartialType(CreateBranchDto)
│   │
│   ├── attendance/                # === ATTENDANCE MODULE ===
│   │   ├── attendance.module.ts
│   │   ├── attendance.controller.ts     # check-in, check-out, today, history, list (dashboard)
│   │   ├── attendance.service.ts        # Logika poin, geofence check, durasi
│   │   └── dto/
│   │       ├── check-in.dto.ts          # latitude, longitude, photo (base64), deviceId
│   │       ├── check-out.dto.ts         # latitude, longitude
│   │       ├── query-attendance.dto.ts  # extends PaginationQueryDto + date?, branch?, status?
│   │       └── query-history.dto.ts     # month?, year?
│   │
│   ├── leave-request/             # === LEAVE REQUEST MODULE ===
│   │   ├── leave-request.module.ts
│   │   ├── leave-request.controller.ts  # POST /leave, GET /leave, PATCH approve/reject, GET /leave/me
│   │   ├── leave-request.service.ts     # Submit, approve, reject, auto-update attendance
│   │   └── dto/
│   │       ├── create-leave.dto.ts      # type, startDate, endDate, reason, attachment?
│   │       ├── update-leave.dto.ts      # PartialType(CreateLeaveDto)
│   │       ├── reject-leave.dto.ts      # reason
│   │       └── query-leave.dto.ts       # extends PaginationQueryDto + status?, type?
│   │
│   ├── point/                     # === POINT MODULE ===
│   │   ├── point.module.ts
│   │   ├── point.controller.ts          # GET /points/me, GET /attendance/points-summary
│   │   └── point.service.ts             # Kalkulasi poin, rekap bulanan
│   │
│   ├── admin/                     # === ADMIN MODULE ===
│   │   ├── admin.module.ts
│   │   ├── admin.controller.ts          # CRUD /admins + toggle-status + dashboard summary
│   │   ├── admin.service.ts
│   │   └── dto/
│   │       ├── create-admin.dto.ts      # name, email, password, role
│   │       └── update-admin.dto.ts      # PartialType tanpa password
│   │
│   ├── notification/              # === NOTIFICATION MODULE ===
│   │   ├── notification.module.ts
│   │   ├── notification.controller.ts   # GET /notifications, PATCH read, PATCH read-all
│   │   ├── notification.service.ts      # Create, mark read, broadcast
│   │   └── dto/
│   │       └── query-notification.dto.ts
│   │
│   ├── report/                    # === REPORT MODULE ===
│   │   ├── report.module.ts
│   │   ├── report.controller.ts         # GET /reports/attendance, GET /reports/attendance/export
│   │   ├── report.service.ts            # Aggregate data, generate CSV/PDF
│   │   └── dto/
│   │       └── query-report.dto.ts      # startDate, endDate, branch?, format?
│   │
│   ├── dashboard/                 # === DASHBOARD MODULE ===
│   │   ├── dashboard.module.ts
│   │   ├── dashboard.controller.ts      # GET /dashboard/summary
│   │   └── dashboard.service.ts         # Stats, chart data, recent check-ins
│   │
│   ├── settings/                  # === SETTINGS MODULE ===
│   │   ├── settings.module.ts
│   │   ├── settings.controller.ts       # GET /settings, PUT /settings
│   │   ├── settings.service.ts
│   │   └── dto/
│   │       └── update-settings.dto.ts   # defaultCheckIn?, halfPointEnd?, defaultRadius?, companyName?
│   │
│   ├── upload/                    # === UPLOAD MODULE ===
│   │   ├── upload.module.ts
│   │   ├── upload.controller.ts         # POST /upload (multipart/form-data)
│   │   └── upload.service.ts            # S3 upload, resize image, validate file
│   │
│   ├── scheduler/                 # === SCHEDULER MODULE ===
│   │   ├── scheduler.module.ts
│   │   ├── alpha-cron.service.ts        # @Cron('0 23 * * 1-6') generate Alpha records
│   │   └── cron-monitor.controller.ts   # GET /internal/cron/alpha-status (internal)
│   │
│   └── audit-log/                 # === AUDIT LOG MODULE ===
│       ├── audit-log.module.ts
│       ├── audit-log.service.ts         # Log create/update/delete actions
│       └── audit-log.interceptor.ts     # Auto-log via interceptor (optional)
│
├── test/
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
│
├── .env.example                   # Template environment variables
├── .env                           # Local environment (gitignored)
├── .gitignore
├── .prettierrc
├── .eslintrc.js
├── nest-cli.json
├── tsconfig.json
├── tsconfig.build.json
├── package.json
└── README.md
```

---

## 4. Prisma Schema

Copy isi file `documentation/schema.prisma` ke `prisma/schema.prisma` tanpa perubahan. Schema sudah final.

---

## 5. Environment Variables

Buat file `.env.example`:

```env
# === Application ===
NODE_ENV=development
PORT=3000
API_PREFIX=v1

# === Database ===
DATABASE_URL=postgresql://postgres:password@localhost:5432/bpr_ams?schema=public

# === JWT ===
JWT_SECRET=super-secret-key-change-in-production
JWT_ACCESS_EXPIRY=24h
JWT_REFRESH_SECRET=another-secret-key-change-in-production
JWT_REFRESH_EXPIRY=7d

# === S3 (IDCloudHost) ===
S3_ENDPOINT=https://is3.cloudhost.id
S3_REGION=us-east-1
S3_BUCKET=bpr-ams
S3_ACCESS_KEY=your-access-key
S3_SECRET_KEY=your-secret-key

# === Rate Limiting ===
THROTTLE_TTL=60000
THROTTLE_LIMIT=60

# === Timezone ===
TZ=Asia/Jakarta
```

---

## 6. Global Setup (`main.ts`)

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global prefix: /v1
  app.setGlobalPrefix('v1');

  // CORS
  app.enableCors();

  // Global Validation Pipe
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      transformOptions: { enableImplicitConversion: true },
    }),
  );

  // Global Exception Filter
  app.useGlobalFilters(
    new PrismaExceptionFilter(),
    new HttpExceptionFilter(),
  );

  // Global Response Interceptor
  app.useGlobalInterceptors(
    new LoggingInterceptor(),
    new ResponseTransformInterceptor(),
  );

  // Swagger
  const config = new DocumentBuilder()
    .setTitle('BPR Sahabat Sejati - API')
    .setDescription('Sistem Kehadiran Karyawan API')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document);

  await app.listen(process.env.PORT || 3000);
}
```

---

## 7. Routing Map (Controller → Endpoint)

Semua endpoint sesuai API contract. Prefix global: `/v1`.

### AuthController (`/auth`)

| Method | Path               | Handler          | Auth     | Deskripsi           |
| ------ | ------------------ | ---------------- | -------- | ------------------- |
| POST   | `/auth/login`      | `loginEmployee`  | Public   | Login karyawan      |
| POST   | `/auth/admin/login`| `loginAdmin`     | Public   | Login admin         |
| POST   | `/auth/logout`     | `logout`         | JWT      | Logout              |
| POST   | `/auth/refresh`    | `refreshToken`   | Public   | Refresh JWT token   |

### EmployeeController (`/employees`)

| Method | Path                            | Handler        | Auth         | Deskripsi            |
| ------ | ------------------------------- | -------------- | ------------ | -------------------- |
| GET    | `/employees`                    | `findAll`      | Admin        | Daftar karyawan      |
| GET    | `/employees/me`                 | `getProfile`   | Employee JWT | Profil sendiri       |
| GET    | `/employees/:id`                | `findOne`      | Admin        | Detail karyawan      |
| POST   | `/employees`                    | `create`       | Admin        | Tambah karyawan      |
| PUT    | `/employees/:id`                | `update`       | Admin        | Update karyawan      |
| DELETE | `/employees/:id`                | `remove`       | Admin        | Hapus karyawan       |
| PATCH  | `/employees/:id/reset-device`   | `resetDevice`  | Admin        | Reset device binding |

### BranchController (`/branches`)

| Method | Path             | Handler    | Auth  | Deskripsi       |
| ------ | ---------------- | ---------- | ----- | --------------- |
| GET    | `/branches`      | `findAll`  | JWT   | Daftar cabang   |
| POST   | `/branches`      | `create`   | Admin | Tambah cabang   |
| PUT    | `/branches/:id`  | `update`   | Admin | Update cabang   |
| DELETE | `/branches/:id`  | `remove`   | Admin | Hapus cabang    |

### AttendanceController (`/attendance`)

| Method | Path                          | Handler           | Auth         | Deskripsi                |
| ------ | ----------------------------- | ------------------ | ------------ | ------------------------ |
| POST   | `/attendance/check-in`        | `checkIn`          | Employee JWT | Check-in harian          |
| POST   | `/attendance/check-out`       | `checkOut`         | Employee JWT | Check-out harian         |
| GET    | `/attendance/today`           | `getToday`         | Employee JWT | Status absensi hari ini  |
| GET    | `/attendance/history`         | `getHistory`       | Employee JWT | Riwayat absensi bulanan  |
| GET    | `/attendance`                 | `findAll`          | Admin        | Daftar absensi dashboard |
| GET    | `/attendance/points-summary`  | `getPointsSummary` | Admin        | Ringkasan poin bulanan   |

### LeaveRequestController (`/leave`)

| Method | Path                    | Handler    | Auth         | Deskripsi             |
| ------ | ----------------------- | ---------- | ------------ | --------------------- |
| POST   | `/leave`                | `create`   | Employee JWT | Ajukan izin           |
| GET    | `/leave/me`             | `getMyLeaves` | Employee JWT | Riwayat izin sendiri |
| GET    | `/leave`                | `findAll`  | Admin        | Daftar permohonan     |
| PUT    | `/leave/:id`            | `update`   | Admin        | Edit izin             |
| PATCH  | `/leave/:id/approve`    | `approve`  | Admin        | Setujui izin          |
| PATCH  | `/leave/:id/reject`     | `reject`   | Admin        | Tolak izin            |

### PointController (`/points`)

| Method | Path          | Handler   | Auth         | Deskripsi            |
| ------ | ------------- | --------- | ------------ | -------------------- |
| GET    | `/points/me`  | `getMyPoints` | Employee JWT | Poin kehadiran saya |

### AdminController (`/admins`)

| Method | Path                          | Handler        | Auth        | Deskripsi        |
| ------ | ----------------------------- | -------------- | ----------- | ---------------- |
| GET    | `/admins`                     | `findAll`      | Super Admin | Daftar admin     |
| POST   | `/admins`                     | `create`       | Super Admin | Tambah admin     |
| PUT    | `/admins/:id`                 | `update`       | Super Admin | Update admin     |
| DELETE | `/admins/:id`                 | `remove`       | Super Admin | Hapus admin      |
| PATCH  | `/admins/:id/toggle-status`   | `toggleStatus` | Super Admin | Toggle aktif     |

### DashboardController (`/dashboard`)

| Method | Path                 | Handler      | Auth  | Deskripsi         |
| ------ | -------------------- | ------------ | ----- | ----------------- |
| GET    | `/dashboard/summary` | `getSummary` | Admin | Ringkasan dashboard |

### NotificationController (`/notifications`)

| Method | Path                         | Handler       | Auth         | Deskripsi            |
| ------ | ---------------------------- | ------------- | ------------ | -------------------- |
| GET    | `/notifications`             | `findAll`     | JWT          | Daftar notifikasi    |
| PATCH  | `/notifications/:id/read`    | `markAsRead`  | JWT          | Tandai dibaca        |
| PATCH  | `/notifications/read-all`    | `markAllRead` | JWT          | Tandai semua dibaca  |

### ReportController (`/reports`)

| Method | Path                              | Handler    | Auth  | Deskripsi          |
| ------ | --------------------------------- | ---------- | ----- | ------------------ |
| GET    | `/reports/attendance`             | `getReport`| Admin | Laporan kehadiran  |
| GET    | `/reports/attendance/export`      | `export`   | Admin | Ekspor CSV/PDF     |

### SettingsController (`/settings`)

| Method | Path         | Handler    | Auth        | Deskripsi            |
| ------ | ------------ | ---------- | ----------- | -------------------- |
| GET    | `/settings`  | `get`      | Admin       | Ambil pengaturan     |
| PUT    | `/settings`  | `update`   | Super Admin | Perbarui pengaturan  |

### UploadController (`/upload`)

| Method | Path       | Handler  | Auth | Deskripsi     |
| ------ | ---------- | -------- | ---- | ------------- |
| POST   | `/upload`  | `upload` | JWT  | Upload file   |

### CronMonitorController (`/internal/cron`) — Internal Only

| Method | Path                             | Handler        | Auth     | Deskripsi            |
| ------ | -------------------------------- | -------------- | -------- | -------------------- |
| GET    | `/internal/cron/alpha-status`    | `getAlphaStatus` | Internal | Status cron Alpha  |

---

## 8. Aturan Bisnis Penting

### 8.1 Sistem Poin Kehadiran

```
Check-in ≤ 08:00 WIB   → status: HADIR,     points: 1
Check-in 08:01 – 08:30 → status: TERLAMBAT,  points: 0.5
Check-in > 08:30 WIB   → status: TERLAMBAT,  points: 0
Izin / Cuti             → status: IZIN_*,     points: 0
Tanpa keterangan        → status: ALPHA,      points: 0
```

Batas waktu (`08:00`, `08:30`) diambil dari tabel `app_settings`, bukan hardcoded.

### 8.2 Geofencing

- Hitung jarak antara koordinat karyawan dan koordinat cabang menggunakan **Haversine formula**
- Jika jarak > radius cabang → tolak check-in (`OUTSIDE_GEOFENCE`)
- Simpan `checkInInsideRadius` sebagai boolean

### 8.3 Device Binding

- Login pertama kali → otomatis bind `deviceId` ke employee
- Login berikutnya harus dari device yang sama
- Admin bisa reset device via `PATCH /employees/:id/reset-device`

### 8.4 Auto Alpha (Cron Job)

- Jalan setiap hari **Senin–Sabtu jam 23:00 WIB**
- Cek semua karyawan aktif yang TIDAK punya Attendance record DAN TIDAK punya LeaveRequest approved di tanggal tersebut
- Generate Attendance (`status: ALPHA, isAutoGenerated: true`) + PointRecord (`type: alpha, points: 0`)
- Gunakan unique constraint `(employeeId, date)` untuk prevent duplicate

### 8.5 JWT Dual Token

- **Access Token**: short-lived (24h), dipakai di header `Authorization: Bearer {token}`
- **Refresh Token**: long-lived (7d), dipakai untuk minta access token baru
- Payload: `{ sub: userId, email, type: 'employee' | 'admin', role?: AdminRole }`

---

## 9. Response Format Standar

### Success Response

```json
{
  "success": true,
  "data": { ... },
  "message": "Operasi berhasil."
}
```

### Error Response

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

### Paginated Response

```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "page": 1,
      "limit": 10,
      "totalItems": 45,
      "totalPages": 5
    }
  }
}
```

Implementasi via `ResponseTransformInterceptor` — controller cukup return data biasa, interceptor yang membungkusnya.

---

## 10. Guard & Decorator Access Control

### Guards (diaplikasikan bertingkat)

1. **JwtAuthGuard** (global) — Semua endpoint butuh JWT kecuali yang di-mark `@Public()`
2. **RolesGuard** — Cek `AdminRole` dari JWT payload. Digunakan bersama `@Roles('SUPER_ADMIN', 'ADMIN')`
3. **DeviceBindingGuard** — Validasi `X-Device-Id` header match dengan `deviceId` di DB (hanya endpoint mobile)

### Custom Decorators

```typescript
@Public()                    // Bypass JWT auth
@Roles('SUPER_ADMIN')        // Hanya SUPER_ADMIN
@Roles('SUPER_ADMIN','ADMIN')// SUPER_ADMIN atau ADMIN
@CurrentUser()               // Extract user dari request (employee atau admin)
```

---

## 11. Prisma Seed Data

File `prisma/seed.ts` harus generate data berikut saat development:

```typescript
// 1. AppSettings default
{
  id: 'default',
  defaultCheckIn: '08:00',
  halfPointEnd: '08:30',
  defaultRadius: 50,
  companyName: 'BPR Sahabat Sejati',
  timezone: 'Asia/Jakarta'
}

// 2. Super Admin default
{
  name: 'Super Admin',
  email: 'admin@bpr.co.id',
  password: bcrypt.hash('admin123', 10),
  role: 'SUPER_ADMIN',
  status: 'ACTIVE'
}

// 3. Branch default (5 cabang)
[
  { name: 'Kantor Pusat', address: 'Jl. Jend. Sudirman Kav. 52-53, Jakarta Selatan', latitude: -6.225, longitude: 106.800, radius: 50 },
  { name: 'Cabang Sudirman', address: 'Gedung Artha Graha Lt. 3, SCBD, Jakarta Selatan', latitude: -6.228, longitude: 106.805, radius: 30 },
  { name: 'Cabang Gatot Subroto', address: 'Menara Mulia, Jl. Gatot Subroto, Jakarta Selatan', latitude: -6.230, longitude: 106.810, radius: 30 },
  { name: 'Kas Pasar Minggu', address: 'Ruko Pasar Minggu Blok A12, Jakarta Selatan', latitude: -6.260, longitude: 106.845, radius: 25 },
  { name: 'Kas Kemang', address: 'Jl. Kemang Raya No. 8, Jakarta Selatan', latitude: -6.258, longitude: 106.817, radius: 25 }
]

// 4. Sample Employees (5 karyawan untuk testing)
// 5. Sample Attendance records (7 hari terakhir)
```

---

## 12. Rate Limiting

Sesuai API contract:

| Endpoint             | Limit        | Implementasi                          |
| -------------------- | ------------ | ------------------------------------- |
| `POST /auth/login`   | 5 req/menit  | `@Throttle({ default: { ttl: 60000, limit: 5 } })` |
| `POST /attendance/*` | 2 req/menit  | `@Throttle({ default: { ttl: 60000, limit: 2 } })` |
| Lainnya              | 60 req/menit | Global default via `ThrottlerModule`  |

---

## 13. Instruksi Eksekusi

### Step 1: Inisiasi Project

```bash
npx @nestjs/cli new bpr-ams-backend --package-manager npm --strict
cd bpr-ams-backend
```

### Step 2: Install Dependencies

```bash
# Core
npm install @nestjs/config @nestjs/swagger @nestjs/jwt @nestjs/passport @nestjs/schedule @nestjs/throttler

# Passport & Auth
npm install passport passport-jwt bcrypt
npm install -D @types/passport-jwt @types/bcrypt

# Prisma
npm install @prisma/client
npm install -D prisma

# Validation
npm install class-validator class-transformer

# S3 Upload
npm install @aws-sdk/client-s3

# Utility
npm install dayjs
```

### Step 3: Setup Prisma

```bash
npx prisma init
# Copy schema.prisma dari documentation/
# Edit .env dengan DATABASE_URL
npx prisma migrate dev --name init
npx prisma generate
```

### Step 4: Buat Semua Module

Buat setiap module sesuai folder structure di atas. Setiap module harus:

1. Memiliki `.module.ts` yang register controller & service
2. Memiliki `.controller.ts` dengan routing sesuai tabel routing di atas
3. Memiliki `.service.ts` dengan business logic
4. Memiliki `dto/` folder dengan class-validator DTO
5. Di-import di `app.module.ts`

### Step 5: Register di `app.module.ts`

```typescript
@Module({
  imports: [
    // Config
    ConfigModule.forRoot({ isGlobal: true, load: [appConfig, databaseConfig, jwtConfig, s3Config] }),

    // Rate limiting
    ThrottlerModule.forRoot([{ ttl: 60000, limit: 60 }]),

    // Scheduler
    ScheduleModule.forRoot(),

    // Core modules
    PrismaModule,
    AuthModule,

    // Feature modules
    EmployeeModule,
    BranchModule,
    AttendanceModule,
    LeaveRequestModule,
    PointModule,
    AdminModule,
    DashboardModule,
    NotificationModule,
    ReportModule,
    SettingsModule,
    UploadModule,
    SchedulerModule,
    AuditLogModule,
  ],
  providers: [
    { provide: APP_GUARD, useClass: JwtAuthGuard },
    { provide: APP_GUARD, useClass: ThrottlerGuard },
  ],
})
export class AppModule {}
```

### Step 6: Seed & Run

```bash
npx prisma db seed
npm run start:dev
```

---

## 14. Perintah Git

Setelah project selesai di-generate:

```bash
git init
git add .
git commit -m "Initial commit: NestJS backend with full modular architecture"
git remote add origin https://github.com/bimantaratirta/BPR-AMS-BE.git
git branch -M main
git push -u origin main
```

---

## 15. Catatan Tambahan

- **Semua pesan error dalam Bahasa Indonesia** sesuai API contract
- **Timezone selalu `Asia/Jakarta` (WIB, UTC+7)** — gunakan `dayjs` dengan timezone plugin
- **Prisma schema sudah final** — tidak perlu modifikasi, langsung copy
- **S3 storage menggunakan IDCloudHost** — kompatibel AWS S3 SDK
- **Jangan hardcode business rules** (batas waktu poin, radius) — ambil dari `app_settings` table
- **Audit log** untuk setiap operasi CRUD penting (create/update/delete employee, approve/reject leave, reset device)
- **API versioning** via global prefix `/v1`
- **File upload max 5MB** — validasi di controller level
