<div align="center">
  <h1>Contributing Guide</h1>
  <p><em>Hướng dẫn đóng góp cho dự án Houston123 Classrooms Management</em></p>
</div>

---

## 🔄 Bản song ngữ / Bilingual
Phần VI tiếng Việt ở trên mỗi mục – English follows after each Vietnamese section.

## 1. Triết lý dự án (Project Philosophy)
Chất lượng mã rõ ràng, type-safe, dễ mở rộng cho các module quản lý lớp học, điểm số, giáo viên, học sinh và ghi hình. Ưu tiên: Tính đúng – Dễ bảo trì – Hiệu năng – UX.

Code must be clear, type-safe and modular for classroom, scoring, attendance, recording domains. Priorities: Correctness – Maintainability – Performance – UX.

## 2. Quy trình nhánh (Git Branch Strategy)
- `main`: ổn định, đã qua review.
- `develop` (nếu tạo thêm): tích hợp trước khi merge vào `main`.
- Feature: `feat/<scope>-<short-desc>`
- Bugfix: `fix/<scope>-<issue#>`
- Hotfix: `hotfix/<critical-desc>`

Branch naming examples.

## 3. Commit Convention (Conventional Commits)
Định dạng:
```
<type>(<scope>): <message>
```
Types phổ biến:
- feat: thêm tính năng
- fix: sửa lỗi
- refactor: tái cấu trúc không đổi hành vi
- docs: tài liệu
- style: định dạng / không ảnh hưởng logic
- chore: việc phụ trợ (config, build script)
- perf: tối ưu hiệu năng
- test: thêm/chỉnh test

Ví dụ:
```
feat(attendance): add bulk import from CSV
fix(score-sheet): correct average calculation for empty set
refactor(api): extract classroom client
```

## 4. Mở Pull Request (Opening a PR)
Checklist trước khi mở PR:
- [ ] Đã chạy `npm run lint`
- [ ] Không còn console.log thừa (trừ debug hợp lý)
- [ ] Đã cập nhật UI screenshot nếu thay đổi giao diện
- [ ] Đã cập nhật README / docs nếu ảnh hưởng
- [ ] Không làm vỡ type (TS compile ok)
- [ ] Thử flow chính (login → chọn company/branch → xem bảng điểm / attendance)

When opening a PR include:
- Mô tả ngắn gọn mục tiêu / Brief goal
- Ticket / Issue liên quan / Related issue
- Ảnh chụp màn hình (nếu UI) / Screenshots
- Hướng dẫn test thủ công / Manual test steps

## 5. Review Guidelines
Reviewer kiểm tra:
1. Logic đúng & có test cơ bản (nếu thêm utils)
2. Không duplicate code → cân nhắc tách component/hook
3. API calls dùng đúng abstraction trong `lib/api`
4. UI đáp ứng (responsive) & không phá vỡ layout dashboard
5. i18n: text mới phải dùng key trong `translation.json`
6. Security: không log token, không expose secret

## 6. Style & Patterns
- Component trong `components/ui` đóng vai trò design system – hạn chế sửa phá vỡ backward compatibility.
- Domain component đặt trong thư mục con (e.g., `components/classroom/ScoreSheetTable.tsx`).
- Tách business logic phức tạp ra khỏi JSX (custom hooks hoặc utils).
- Dùng `zod` + `react-hook-form` cho validation form.
- Tránh hardcode string đa ngôn ngữ → dùng i18n key.

## 7. API Layer
- Tất cả gọi HTTP nên thông qua axios instance `lib/api/axios.ts` để tự động gắn headers (`Authorization`, `x-company`, `x-branch`).
- Nếu thêm domain mới tạo file `lib/api/<domain>.ts` export hàm thuần trả về `Promise<T>`.
- Xử lý lỗi: log gọn (đã có interceptor). Return error rõ ràng cho UI.

## 8. Quản lý State
- Ưu tiên state cục bộ (component state) → context chỉ cho cross-cutting (auth, theme, staff).
- Tránh lạm dụng context cho data tạm thời.

## 9. Hiệu năng
- Dùng `React.memo` nếu component render nhiều với props ổn định.
- Lazy load modal/dialog nếu nặng.
- Tránh fetch lặp – cân nhắc cache tạm (e.g., localStorage hoặc in‑memory singleton) cho danh sách ít thay đổi (company/branch list).

## 10. Bảo mật
- Không commit secret (.env *)
- Không log token hoặc payload nhạy cảm
- Kiểm tra expiry JWT (`authService.isAuthenticated`)

## 11. Thêm i18n key
1. Thêm vào `public/locales/en/translation.json` & `vi/translation.json`
2. Dùng: `t('module.key')`
3. Tránh trùng key; nhóm theo domain (`attendance.*`, `score.*`)

## 12. Roadmap Gợi ý / Suggested Roadmap
- Thêm Playwright E2E
- Thêm Vitest/Jest cho utils
- Thêm CI (GitHub Actions: lint + type check + build)
- Chuyển token hoàn toàn sang cookie HttpOnly

## 13. Liên hệ / Contact
Mở issue hoặc ping nhóm duy trì nội bộ.

---
Happy building! 🚀
