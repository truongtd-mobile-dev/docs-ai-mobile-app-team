# Onboarding Checklist — đưa 1 repo RN vào chuẩn team

> Mục tiêu: mỗi repo React Native đạt một mức nền tảng chung để cả team + Claude Code làm việc nhanh, an toàn, chất lượng.
> Cách dùng: copy danh sách này vào PR/issue "Adopt Claude standard" của repo, tick từng mục. Mỗi mục kèm **cách verify**.
> Không phải làm hết một lần — theo lộ trình 3 phase ở cuối. Repo legacy: retrofit dần, không big-bang.

---

## Phase 1 — Quality gates (nền tảng kỹ thuật)

- [ ] **TypeScript cho code mới** — `tsconfig.json` có, `strict: true`.
  - *Verify:* `npx tsc --noEmit` chạy không lỗi (hoặc lỗi đã được track).
- [ ] **ESLint** cấu hình & chạy được.
  - *Verify:* `yarn lint` (hoặc `npx eslint .`) chạy ra kết quả, không crash config.
- [ ] **Prettier** thống nhất format.
  - *Verify:* có `.prettierrc` (hoặc trong eslint), `npx prettier --check .` chạy được.
- [ ] **Husky + lint-staged** pre-commit auto-fix.
  - *Verify:* có `.husky/pre-commit` gọi `lint-staged`; thử commit file bẩn → được fix/lint.
- [ ] **Jest** cấu hình; có ít nhất test cho logic mới.
  - *Verify:* `npx jest` chạy (kể cả 0 test cũng phải không lỗi config).
- [ ] **PR CI gate**: lint + typecheck + test phải xanh trước merge.
  - *Verify:* pipeline (Jenkins/Gitea CI) chặn merge khi 1 trong 3 đỏ.
- [ ] **Node pin**: `.nvmrc` hoặc `engines` trong `package.json`.
  - *Verify:* file tồn tại, khớp version team dùng.

> Repo thiếu (vd v551): bắt đầu từ **logic mới + bug-fix có test kèm**, thêm Prettier/Husky trước, mở rộng dần.

---

## Phase 2 — Context file & tài liệu

- [ ] **Context file** ở root, fork từ `docs-ai/CLAUDE.md.template`, đã điền hết `[FILL]`.
  - *Verify:* không còn `[FILL]`; phần Commands chạy đúng thật; archetype + New Arch on/off có lý do.
- [ ] **Đa công cụ:** repo có người dùng Codex/Gemini → có `AGENTS.md`/`GEMINI.md` trỏ về cùng một nguồn.
  - *Verify:* file tồn tại (symlink/@import), không phải bản lệch nhau.
- [ ] **Gotchas** ghi rõ bẫy thật của repo (repo legacy: cấu trúc lạ được document).
  - *Verify:* người mới đọc Gotchas là tránh được lỗi phổ biến.
- [ ] **Khởi tạo bằng tool** (Claude `/init`, …) rồi chỉnh tay — không để bản generic.
  - *Verify:* context file phản ánh đúng cấu trúc hiện tại.
- [ ] **`docs/`** có chỗ lưu quyết định + flow QA (khuyến nghị: `code-logic/`, `ai-testing/`).
  - *Verify:* thư mục tồn tại hoặc README trỏ tới nơi lưu.

---

## Phase 3 — Skill kit & MCP (tận dụng AI)

- [ ] **Skill kit (đã verify)** cài cho repo/máy dev:
  - [ ] `react-native-best-practices` (Callstack — perf)
  - [ ] Bộ discipline (vd `superpowers`) — *sau khi đã kiểm nguồn & thử thật*
  - [ ] Argent iOS/Android device suite (setup · interact · test-ui-flow · profiler)
  - *Verify:* skill xuất hiện khi mở tool ở repo; cài qua **plugin marketplace** để đồng bộ; cơ chế khác theo tool (Claude `.claude/skills/<tên>/SKILL.md`; Codex/Gemini tương đương).
- [ ] **Skill tự xây cho tác vụ lặp lại** (ET, mô tả PR, release checklist…) — viết → thử → verify trước khi chuẩn hoá.
  - *Verify:* skill có quy trình + output cấu trúc rõ; **KHÔNG** đưa skill chưa verify thành chuẩn team.
- [ ] **MCP device** cấu hình (chỉ staging / read-only).
  - *Verify:* `.mcp.json` có server device (mobile-mcp/Argent); KHÔNG có endpoint prod.
- [ ] **`.claude/settings.json`** baseline (permission allowlist hợp lý).
  - *Verify:* các lệnh đọc thường dùng được allow; không allow lệnh nguy hiểm.
- [ ] *(Tùy)* **Hooks guardrail**: chặn sửa `android/ios` (CNG), chặn commit `.env`.
  - *Verify:* thử thao tác bị chặn → hook chặn đúng.
- [ ] *(Tùy)* **`.claude/commands/`** slash command lặp lại (`/et`, `/debug`, `/review`, `/pr`).
  - *Verify:* gõ `/et` trong repo → chạy đúng template/skill.

---

## Bảo mật — bắt buộc mọi phase

- [ ] `.env*` **gitignored**, không secret nào trong git history gần đây.
  - *Verify:* `git ls-files | grep -E '\.env'` rỗng; secrets lấy từ Passbolt.
- [ ] MCP / device / DB **không trỏ production**.
  - *Verify:* review `.mcp.json` + script — chỉ staging/read-only.
- [ ] Mọi code AI viết **được review** trước khi merge.
  - *Verify:* PR có người duyệt; không self-merge code chưa đọc hiểu.

---

## Archetype playbook — đọc đúng loại

- [ ] Đã xác định archetype của repo: **CNG** / **bare-RN** / **legacy** (ghi trong CLAUDE.md).
- [ ] Đã đọc playbook tương ứng trong `rn-ai-standard.html`:
  - **CNG** → đừng sửa native; prebuild + plugin-hash; OTA cẩn trọng.
  - **bare-RN** → native committed; New Arch default; Fastlane signing; patch-package.
  - **legacy** → document cấu trúc lạ trước; code mới function+hooks; retrofit quality gate.

---

### Định nghĩa "đã onboard"

Repo coi như đạt chuẩn khi: **Phase 1 + Phase 2 hoàn tất**, bảo mật pass, và archetype playbook đã đọc.
Phase 3 (skill kit + MCP) nâng dần — ưu tiên cho repo hoạt động nhiều nhất trước.
