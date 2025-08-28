<div align="center">
	<img src="./public/houston123-logo.png" alt="Houston123" width="160" />
  
	<h1>Houston123 Classrooms Management</h1>
	<p><strong>Next.js 15 (App Router) • TypeScript • Tailwind CSS 4 • i18next • PM2 • Radix UI • React 19</strong></p>
	<p>Quản lý lớp học, điểm số, chuyên cần, ghi hình & báo cáo tiến độ cho hệ thống Houston123.</p>
	<p><em>Bilingual README: Vietnamese (🇻🇳) & English (🇺🇸)</em></p>
</div>

---

## Mục lục (VI)
1. Giới thiệu
2. Kiến trúc & Thư mục
3. Công nghệ chính
4. Yêu cầu hệ thống
5. Cài đặt & Chạy local
6. Biến môi trường
7. Quy ước mã nguồn
8. I18n / Đa ngôn ngữ
9. Build & Triển khai (PM2 / Vercel)
10. Bảo mật & Headers
11. Testing (định hướng)
12. Quy trình đóng góp
13. Giấy phép

## Table of Contents (EN)
1. Overview
2. Architecture & Structure
3. Tech Stack
4. Requirements
5. Local Setup
6. Environment Variables
7. Code Conventions
8. Internationalization
9. Build & Deployment
10. Security Notes
11. Testing (roadmap)
12. Contributing
13. License

---

## 1. Giới thiệu (Overview)
Ứng dụng quản lý lớp học nội bộ: điểm danh (attendance), bảng điểm (score sheet), cảnh báo điểm thấp, quản lý giáo viên – học sinh, ghi hình lớp học, biểu đồ tiến độ.

The internal classroom management platform: attendance, score sheets, low-score alerts, teacher/student management, class recordings, progress dashboards.

## 2. Kiến trúc & Thư mục (Architecture)
```
src/
	app/                # Next.js App Router (pages, layouts, API routes)
	context/            # React Context providers (auth, theme, staff)
	lib/                # Core libs: i18n, API clients, utils
		api/              # Axios service wrappers (auth, classroom, company...)
		utils/            # Pure utility modules (calendar, progress)
	types/              # TypeScript type definitions
	components/         # Reusable UI + domain components
		ui/               # Design system (Radix + Tailwind)
		classroom/        # Domain: score, attendance, recording dialogs
		calendar/         # Calendar view variants
public/locales/       # i18n resource JSON (en, vi)
ecosystem.config.cjs  # PM2 cluster config
```

### Luồng dữ liệu (Data Flow)
1. UI component -> gọi hàm trong `src/lib/api/*.ts`
2. Axios interceptor thêm headers: `Authorization`, `x-company`, `x-branch`
3. API route nội bộ (`/api/auth/token` ...) forward đến ERP external (`https://erp.houston123.edu.vn`)
4. Token lưu cả cookie HTTP-only (server) & localStorage (client) để sử dụng cho các request tiếp theo.

## 3. Công nghệ chính (Tech Stack)
- Next.js 15 (App Router, Server Actions experimental)
- React 19 + TypeScript strict
- Tailwind CSS v4 + Radix UI primitives + custom design system trong `components/ui`
- i18next + LanguageDetector (EN/VI)
- Axios với interceptors (auth + tenant context: company/branch)
- PM2 cluster mode cho production scaling
- date-fns, react-hook-form + zod validation

## 4. Yêu cầu hệ thống (Requirements)
- Node.js >= 18.x LTS
- npm (khuyên dùng) hoặc pnpm/yarn/bun
- PM2 (production) `npm i -g pm2` (hoặc cài cục bộ đã có trong devDependencies)

## 5. Cài đặt & Chạy local (Local Setup)
```bash
git clone <repo-url>
cd houston123-classrooms-management
npm install
npm run dev
```
Truy cập: http://localhost:3000

Build thử:
```bash
npm run build
npm start
```

## 6. Biến môi trường (Environment Variables)
Hiện mã nguồn sử dụng tối thiểu biến runtime. Bạn có thể mở rộng bằng cách tạo file `.env.local`.

Gợi ý thêm (tùy nhu cầu):
```
NEXT_PUBLIC_API_BASE=https://api.example.com
ERP_API_URL=https://erp.houston123.edu.vn
```
Trong sản xuất với PM2: chỉnh ở `ecosystem.config.cjs` phần `env` / `env_production`.

## 7. Quy ước mã nguồn (Code Conventions)
- Đường dẫn alias: `@/*` trỏ vào `src/*` (tsconfig paths)
- Strict TypeScript bật `strict: true`
- ESLint (Next.js config) + Tailwind utilities
- Component đặt tên PascalCase, hooks camelCase
- API client tách nhỏ theo domain (auth, classroom, company ...)

## 8. Quốc tế hoá (Internationalization)
- i18next + `react-i18next`
- Resources: `public/locales/{en,vi}/translation.json`
- Tự động phát hiện: localStorage -> navigator -> htmlTag
- Thay đổi ngôn ngữ: component `LanguageSwitcher.tsx`

## 9. Build & Triển khai (Deployment)
### 9.1 PM2 (Production Bare Metal / VM)
```bash
npm ci --only=prod # hoặc: npm install --production --omit=dev
npm run build
npm run start:pm2
```
Cluster config trong `ecosystem.config.cjs`:
- instances: `max` (theo số CPU core)
- exec_mode: `cluster`
- Tự restart nếu >512MB / process
- Ghi log tại `./logs/`

Các lệnh:
```bash
npm run reload:pm2   # Zero-downtime reload
npm run logs:pm2     # Xem log
npm run stop:pm2     # Dừng
pm2 monit            # Realtime metrics
pm2 scale web 4      # Scale thủ công
```

Triển khai cập nhật:
```bash
git pull
npm ci --only=prod
npm run build
npm run reload:pm2
```

### 9.2 Vercel
Chỉ cần connect repo -> Vercel tự build (Next.js hỗ trợ native). Loại bỏ PM2 (không cần). Logs & scaling do Vercel quản lý.

### 9.3 Reverse Proxy (Optional)
Nginx ví dụ:
```
location / {
	proxy_pass http://127.0.0.1:3000;
	proxy_http_version 1.1;
	proxy_set_header Host $host;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection 'upgrade';
}
```

## 10. Bảo mật (Security Notes)
- Token lưu ở cả cookie HTTP-only (an toàn XSS) và localStorage (tiện interceptor) – có thể refactor thành chỉ cookie + `Authorization` header qua API route.
- CORS (route auth) đáp ứng OPTIONS.
- Cân nhắc thêm Helmet headers ở reverse proxy.
- Kiểm tra hết hạn JWT trong `authService.isAuthenticated`.

## 11. Testing (Roadmap)
Chưa có test. Đề xuất:
- Unit: utils (calendar, progress)
- Integration: API routes (Next.js request mocks)
- E2E: Playwright (auth + attendance + score editing flow)

## 12. Đóng góp (Contributing)
Xem file `CONTRIBUTING.md` (sẽ mô tả Conventional Commits, quy trình PR, review checklist).

## 13. Giấy phép (License)
Private / Internal. (Cập nhật nếu open-source trong tương lai.)

---

## English Quick Start
```bash
git clone <repo-url>
cd houston123-classrooms-management
npm install
npm run dev
```
Prod (PM2):
```bash
npm ci --only=prod
npm run build
npm run start:pm2
```

## Maintainers
- Team Houston123

Issues & feature requests: please open via GitHub Issues using the provided templates.

---

> Last updated: 2025-08-28

