# Qui chuẩn dùng AI để code — Team React Native

Bản đầy đủ đi kèm slide deck. Deck (`rn-ai-standard.html`) để trình bày và onboard nhanh; file này là chỗ giải thích kỹ — đọc khi cần biết *vì sao* và *làm thế nào*, không chỉ *cái gì*.

## Bộ tài liệu

| File | Dùng để |
|---|---|
| `rn-ai-standard.html` | Slide deck — overview, trình bày, onboard nhanh |
| `README.md` (file này) | Bản chuẩn đầy đủ — giải thích từng phần |
| `CLAUDE.md.template` | Khung context file cho mỗi repo fork về điền |
| `onboarding-checklist.md` | Các bước đưa một repo vào chuẩn, kèm cách verify |

## Tài liệu này giải quyết chuyện gì

Team có nhiều repo React Native và mỗi người dùng một trợ lý AI khác nhau — Claude Code, Codex, Gemini. Không ai làm giống ai, chất lượng output lệch nhau. Bản chuẩn này thống nhất cách làm việc với AI: nhanh hơn, code chắc hơn, không lộ secret.

Hai ràng buộc định hình toàn bộ tài liệu:

1. **Nhiều trợ lý AI.** Không phải ai cũng dùng Claude Code, nhiều người đang dùng Codex và Gemini. Nên chuẩn viết theo nguyên lý chung cho mọi agentic coding tool, kèm ghi chú riêng cho từng tool. Đây không phải tài liệu "chỉ Claude".
2. **Nhiều kiểu repo.** Expo CNG, bare-RN-trên-Expo-SDK, và codebase legacy/framework. Một bộ luật cứng không vừa cho cả ba → chuẩn chia tầng (mục dưới).

Tài liệu này không ghi "tiết kiệm X%". Không có số đo thực tế để ghi, nên ghi vào chỉ làm nó nghe kêu. Thay vào đó nói thẳng việc nào giao AI được, việc nào phải tự cầm.

### Việc nào giao AI được, việc nào phải tự cầm

Giao AI thoải mái: đọc và định vị trong codebase lớn hoặc lạ, viết boilerplate, viết test, refactor cơ học, debug khi đã có bước tái hiện, tra API/SDK, sinh checklist, convert/format dữ liệu.

Phải tự cầm, AI chỉ soạn nháp: quyết định kiến trúc, đánh đổi nghiệp vụ, code chạm tiền/thanh toán, secret và quyền truy cập dữ liệu, sửa vùng legacy nhạy cảm. Nói chung là chỗ nào sai sẽ tốn kém để rút lại thì người quyết, không phải AI.

## Chuẩn chia 3 tầng

Repo khác nhau, tool khác nhau, nên không thể áp một bộ luật cứng:

1. **Universal** — cách làm việc với AI, áp cho mọi repo và mọi tool. Phần lớn tài liệu này.
2. **Per-project** — thứ đặc thù từng repo (lệnh build, state, data layer, gotcha) không nằm trong chuẩn chung; nó nằm trong context file của repo đó, điền theo `CLAUDE.md.template`.
3. **Per-type playbook** — runbook ngắn cho 3 loại repo: Expo CNG, Bare RN, Legacy/framework.

Quy tắc đơn giản: hướng dẫn nào chỉ đúng với một repo thì nó thuộc context file của repo đó, đừng nhét vào chuẩn chung.

---

## 1. Trợ lý AI và context file

Mọi agentic coding tool bây giờ đều có chung ba thứ: một context file tự đọc đầu session, hỗ trợ MCP để gắn tool ngoài, và vòng lặp đọc-ngữ-cảnh → đề-xuất → thực-thi. Với team, khác biệt đáng quan tâm nhất chỉ là tên context file:

| Tool | Context file | Ghi chú |
|---|---|---|
| Claude Code | `CLAUDE.md` | Skills, MCP, Plan Mode, subagents, hooks, slash commands |
| Codex | `AGENTS.md` | Theo chuẩn mở [agents.md](https://agents.md); MCP; approval mode |
| Gemini CLI | `GEMINI.md` | MCP; extensions / cơ chế kích hoạt skill |

Đừng nuôi 3 bản context lệch nhau. Giữ một nguồn, để `AGENTS.md`/`GEMINI.md` trỏ về nó (symlink hoặc `@import`). Sửa một chỗ, cả ba tool thấy.

Context file cần có gì thì xem `CLAUDE.md.template`: overview + loại repo, tech stack, lệnh build/run/test thật, env và secret, native workflow, testing và CI, conventions, và phần gotcha ("đừng làm…"). Với repo legacy, phần gotcha và mô tả cấu trúc lạ là quan trọng nhất — nó ngăn AI hiểu sai rồi sửa loạn.

Mọi repo phải có context file. Tạo nháp bằng lệnh của tool (Claude là `/init`) rồi sửa tay cho khớp thực tế, đừng để bản generic.

---

## 2. Cách làm việc — phần Universal

### 2.1 Đọc task và lên plan trước

Đừng đoán rồi code luôn. Cho AI đọc task trước, đọc cả comment chứ không chỉ description — comment thường chứa quyết định của khách và phân tích của dev (đây cũng là lý do tồn tại skill `bss-jira-task-summary`). Bắt AI tóm tắt yêu cầu, nêu câu hỏi cần clarify, chỉ ra dependency và risk. Rồi xem plan, confirm, mới code.

Làm vậy thì chỗ mơ hồ lộ ra sớm — lúc sửa còn rẻ — thay vì code xong mới biết hiểu sai.

ET thì cho AI bóc task và nháp effort, nhưng người chốt. Buffer cho phần chưa chắc (technical debt, perf server, tích hợp bên thứ ba) theo phán đoán từng task, đừng dùng một con số cố định vì nó không phản ánh rủi ro thật.

### 2.2 Workflow implement

1. Tạo branch `V###APP-###`, cập nhật context file trước khi bắt đầu.
2. Lên plan trước, code sau. Claude có Plan Mode (Shift+Tab); Codex/Gemini thì yêu cầu nêu plan rồi bật approval mode để chờ duyệt. Xem plan → confirm → mới code.
3. Làm từng phần nhỏ, mỗi phần xong review và confirm rồi qua phần kế. Phần lớn và độc lập thì tách sub-task riêng cho gọn context.
4. Nhờ AI review lại code vừa viết, chạy test trước khi commit.
5. Commit `type(TICKET): summary`, lint-staged tự fix, mở PR trên Gitea.

### 2.3 Chọn model theo độ khó

Nguyên lý chung cho mọi tool: việc khó dùng tier mạnh, việc cơ học dùng tier nhanh. Tier mạnh đắt và chậm hơn, đừng dùng cho việc rename biến.

- Việc khó (kiến trúc, debug rối, refactor lớn, lên plan): tier mạnh nhất, để reasoning effort cao — ví dụ Claude Opus, Gemini Pro, Codex bản high-reasoning.
- Việc thường (implement, sửa vừa): tier cân bằng — Claude Sonnet, Gemini Flash.
- Việc cơ học (lặp lại, tóm tắt, sửa nhỏ): tier nhanh/rẻ — Claude Haiku.

Nhưng context sạch quan trọng hơn chọn model. Model mạnh mà nhồi context rác vẫn ra kết quả tệ.

### 2.4 Quản context (không dán code)

Agentic tool tự đọc và sửa file, nên không cần dán code vào prompt như chat web. Đây là chỗ nhiều người làm sai theo thói quen cũ và tốn token vô ích.

Nên làm: trỏ tới file (Claude là `@mention`) rồi để AI tự đọc/search; để context file và plan-file giữ ngữ cảnh; xong task thì reset context (Claude `/clear`), session dài thì nén lại (Claude `/compact`); việc lớn thì tách sub-task. Codex và Gemini có cơ chế tương đương, đọc doc của tool.

Đừng làm: dán cả file 500 dòng; mô tả lại thứ đã có trong context file; một session kéo qua nhiều task không liên quan.

### 2.5 Debug — đưa đủ 5 thành phần

Đừng quăng mỗi error message. Đưa đủ năm thứ để AI khỏi đoán: (1) error message + stack trace đầy đủ, (2) bước tái hiện, (3) kết quả mong đợi vs thực tế, (4) file/feature vừa sửa gần đây, (5) environment (RN version, platform, device/sim, build variant).

Bắt AI loại trừ giả thuyết có hệ thống, đừng vá theo triệu chứng. Với RN, nhớ phân biệt lỗi JS với lỗi native (native thường phải đọc Xcode/Logcat), và tận dụng Metro log, Reactotron (repo nào có), log từ device. AI đi sai hướng thì dừng lại (Claude là ESC), thu hẹp scope, đừng để nó đi tiếp cho tốn token.

### 2.6 Testing và cổng CI

Code AI viết cần có lưới đỡ. Mức tối thiểu:

- Unit/logic: Jest + `@testing-library/react-native` cho hook/util/logic. Đừng snapshot tràn lan, nó dễ vỡ mà ít giá trị.
- E2E: Maestro. Team đã chạy ở v560ns và v578 (`.maestro/*.yaml`), nên dùng chung cho flow quan trọng — login, giỏ hàng, thanh toán.
- PR gate: `tsc --noEmit` + ESLint + Jest phải xanh mới cho merge. Cấu hình CI (Jenkins/Gitea) chặn merge khi đỏ.
- TDD khi hợp: bug và feature có logic rõ thì viết test trước, code cho tới khi xanh.

Repo chưa có test (như v551) thì không cần phủ ngược toàn bộ. Bắt đầu từ logic mới và mỗi bug-fix kèm một test tái hiện, lớn dần.

### 2.7 Self-test và đòi bằng chứng trước khi nói xong

Nhờ AI sinh checklist phủ bốn nhóm: happy path, edge case, negative case, regression. Bug bắt ở self-test mất tầm 5 phút; để tester tìm ra thì thêm tầm 30 phút cho cái vòng re-open → assign → fix → re-test → close.

Trước khi nói "xong" hay "đã pass", chạy lệnh thật (test/lint/chạy app) và dán output ra. Đừng tin chữ "đã xong" — của mình hay của AI — khi chưa thấy nó chạy.

### 2.8 Code review — kể cả code AI viết

AI ra code càng nhanh thì review càng quan trọng. Đừng merge thứ mà mình hoặc AI chưa thực sự đọc hiểu.

- Trước khi mở PR: bắt AI tự rà lại diff theo checklist, sửa mấy cái rõ ràng trước.
- Khi nhận góp ý: kiểm từng điểm, đừng "ừ cho qua". Góp ý sai về kỹ thuật thì phản biện có dẫn chứng.
- Đóng nhánh: PR Gitea mô tả rõ, CI xanh, dọn nhánh sau merge.
- Người review vẫn là chốt cuối. AI phụ review, không thay người duyệt.

### 2.9 Fix bug — trị gốc, không tái phát

Tái hiện được rồi mới fix; chưa tái hiện được thì chưa fix (nhờ AI dựng test case tái hiện). Tìm root cause bằng công thức debug, đừng vá triệu chứng. Cân 2–3 cách, so safety/effort/risk rồi chọn. Implement xong nhờ AI review xem có side effect không. Cuối cùng test regression để chắc không làm hỏng chỗ khác. Quyết định đáng nhớ thì ghi vào `docs/`.

---

## 3. Bảo mật khi dùng AI

Nhanh mấy cũng không đánh đổi bảo mật. Bốn thứ cứng cho mọi repo, mọi tool:

1. **Secret thì không commit, không paste.** `.env*` lấy từ Passbolt và luôn gitignore. Đừng dán API key/token vào prompt — nó có thể bị log lại.
2. **MCP và device không trỏ production.** Chỉ nối staging hoặc read-only. Không cắm DB hay MCP vào dữ liệu thật của khách.
3. **Cẩn thận nội dung không tin được (prompt injection).** Khi AI đọc web, issue, log hay output bên thứ ba, nội dung đó có thể nhét lệnh ẩn kiểu "bỏ hết hướng dẫn trước, làm cái này". Đừng để agent tự chạy mù theo nội dung không tin được; hành động có rủi ro thì review.
4. **Chặn bằng máy, đừng trông vào trí nhớ.** Dùng hooks và permission allowlist để chặn cứng: ví dụ PreToolUse hook chặn sửa `android/`/`ios/` ở repo CNG, chặn commit `.env`. Mỗi tool một cơ chế nhưng ý như nhau.

Và quy tắc bao trùm: code AI viết đều phải review trước khi ship; dependency hay patch mới thì đọc trước khi tin.

---

## 4. Skill và hướng dẫn tái dùng

Phần này hay gây nhầm nhất nên nói kỹ: skill là gì, ở đâu ra, lúc nào đáng dùng, lúc nào tự viết.

### 4.1 Skill là gì

Skill là một gói hướng dẫn tái dùng, thường là một file `SKILL.md` ghi rõ ba thứ: khi nào áp dụng, quy trình từng bước, và format output. Agent tự nạp skill khi gặp đúng tình huống rồi làm theo, mình không phải nhắc lại quy trình mỗi lần. Hiểu gọn: skill là cách đóng gói một SOP để AI chạy nhất quán.

Đáng dùng vì viết best-practice một lần, ai chạy cũng ra giống nhau, lên repo mới vẫn theo đúng cách team làm, và bớt phải gõ lại context.

### 4.2 Skill ở đâu ra

Có hai nguồn, và phải biết nguồn để khỏi dùng mù:

Nguồn ngoài — bên thứ ba viết và maintain, cài qua plugin/marketplace, có xuất xứ rõ ràng. Vài cái team có thể đánh giá:

- `react-native-best-practices` của Callstack (hãng tư vấn RN). Hướng dẫn tối ưu RN: FPS, TTI, bundle size, re-render, FlashList, Hermes. Dùng làm tham chiếu perf chung được.
- Bộ skill quy trình kiểu superpowers — gom brainstorming, lên plan, systematic-debugging, TDD, code-review, verification. Cài dạng plugin, thiết kế để chạy được trên nhiều tool. Nhưng phải kiểm nguồn và chạy thử trước khi bắt cả team theo.
- Argent — bộ MCP + skill điều khiển simulator/emulator iOS-Android (xem mục Device).

Nguồn tự viết — cho mấy việc lặp đi lặp lại theo quy trình cố định của BSS. Đây là chỗ đáng đầu tư nhất với team, nhưng phải tự verify.

Một điểm quan trọng: skill chưa verify thì chưa phải chuẩn, kể cả skill nội bộ. Ví dụ skill ET đang dùng thử trong team thì vẫn chỉ là ứng viên cho tới khi kiểm chứng xong; đừng vì nó tiện mà coi như chuẩn.

### 4.3 Đa công cụ — sự thật về tính khả chuyển

Khái niệm skill thì chung, nhưng cơ chế khác nhau và port giữa tool có giới hạn:

- Claude Code: skill ở `.claude/skills/<tên>/SKILL.md` — chú ý `skills` số nhiều, mỗi skill là một thư mục chứa `SKILL.md`.
- Codex: hướng dẫn chủ yếu qua `AGENTS.md`, có cơ chế skill/prompt riêng.
- Gemini CLI: qua extensions / cơ chế kích hoạt skill riêng.

Một số bộ (như superpowers) kèm sẵn bản thích ứng cho từng tool, nhưng đừng cho rằng skill viết cho Claude chạy y nguyên trên Codex/Gemini — thử trước rồi tin.

### 4.4 Tự viết skill cho việc lặp lại

Việc nào team làm đi làm lại theo một quy trình cố định thì đóng thành skill. Ứng viên rõ ràng: ET/ước lượng, viết mô tả PR, checklist release, bóc task Jira, review checklist.

Skill tốt khác skill dở ở chỗ:

| Dở | Tốt |
|---|---|
| Mô tả chung chung ("hãy làm ET") | Quy trình cụ thể từng bước |
| Output mỗi lần một kiểu | Output có format cố định |
| Một skill ôm nhiều việc | Một skill một mục đích |
| Chưa thử đã bắt cả team dùng | Chạy thử kỹ rồi mới chuẩn hoá |

Cách làm: viết nháp → chạy thử trên vài task thật → sửa → ổn rồi mới commit vào repo hoặc đẩy lên marketplace cho cả team. Skill ET hiện có của team là một ví dụ đang ở giai đoạn "chạy thử", chưa nên áp cứng. (Có meta-skill như `writing-skills` hỗ trợ viết cho đúng cấu trúc.)

---

## 5. Chạy app trên iOS/Android qua AI

Qua MCP device (ví dụ Argent), trợ lý AI điều khiển được simulator/emulator để chạy, debug, QA và profile app thật — cả hai nền tảng. Đây là thứ mobile có mà chat AI thường không làm được.

- Setup và chạy: boot iOS simulator / Android emulator, lấy UDID/serial, cài và mở app đúng scheme/variant; start Metro, fix build, đọc log, reload bundle.
- Debug runtime: inspect component tree, eval JS, đọc console qua Metro (CDP).
- UI test: vòng interact → screenshot → verify. Một quy tắc cứng: luôn lấy toạ độ từ tool discovery, đừng đoán toạ độ từ screenshot.
- Record và replay flow: ghi lại một flow nhiều bước để chạy lại — phục vụ regression và so sánh sau khi sửa.
- Profiling: đo re-render, CPU hotspot, fiber tree, UI hang (mục Performance).

Chọn tool: Argent là bộ RN-aware đầy đủ (profiler, component tree, flows), nên chọn nó nếu team chuẩn hoá. mobile-mcp nhẹ hơn, đã có sẵn ở vài repo. Flow QA tái dùng thì lưu vào `docs/`.

Một điểm thực tế: mấy skill/MCP device này hiện chưa cài đồng đều cả team. Nên đưa "cài bộ device chung" vào lộ trình rollout (Phase 3), đừng giả định ai cũng đã có.

---

## 6. Performance và profiling

Đo trước, tối ưu sau. Tìm bottleneck bằng profiler, đừng tối ưu theo cảm giác.

- Profiler: RN profiler (re-render, CPU, fiber tree) + native profiler (UI hang, hotspot native).
- React Compiler (React 19 / RN 0.81+): tự memo hoá, bớt phải `useMemo`/`useCallback` tay. Đáng bật và đo cho code mới.
- New Architecture (Fabric + TurboModules): giờ là mặc định; Hermes cũng là engine mặc định. Theo dõi repo nào còn Old Arch và vì sao.
- List và TTI: list dài dùng FlashList; giảm bundle size; đo Time-To-Interactive cho startup.
- Skill tham chiếu: `react-native-best-practices` của Callstack.

---

## 7. Playbook theo loại repo

### A. Expo CNG (ví dụ v567app — Shopify)

Native sinh từ prebuild, không commit. Quy tắc:

- Đừng sửa `android/`/`ios/`. Native cần đổi thì sửa qua `app.config.js` / config plugin.
- Đổi plugin hay `app.config` rồi thì rebuild cache (plugin-hash), không thì cache stale.
- Release theo env variant. Nếu repo dùng OTA (đẩy bundle thẳng prod) thì test rất kỹ trước khi publish, vì lỗi là ra thẳng người dùng.
- Lệnh build cụ thể (fast-run, cache-envs, env:stg) là đặc thù repo — ghi ở context file của repo, không phải luật chung.

### B. Bare RN trên Expo SDK (ví dụ v560ns / v578 — Magento, MobX-State-Tree, Maestro)

Native được commit và quản trực tiếp. Quy tắc:

- Quản version Gradle/CocoaPods ngay trong `android/`/`ios/`; patch-package có kỷ luật.
- New Architecture là mặc định — repo nào còn Old Arch thì ghi rõ lý do trong context file.
- Signing và build qua Fastlane + schemes/flavors; `APP_ENV` chọn scheme/variant.
- Secret qua Passbolt + script setup-env; `.env.*` không commit.

### C. Legacy / framework (ví dụ v551 — SimiCart, JS-dominant, class component)

Codebase dựng trên framework, cấu trúc khác chuẩn. Quy tắc:

- Ghi cấu trúc lạ vào context file trước tiên: framework, layout/event registry, singleton, và cả mấy thứ phải bỏ qua (kiểu README template của Expo). CLAUDE.md của v551 đang làm tốt việc này — có branch note, cảnh báo README, mô tả hai data layer, native-base fork vendored, Redux generic fallback. Dùng nó làm mẫu.
- Component: code mới dùng function + hooks, đừng trộn bừa vào class component cũ.
- Data layer: nói rõ khi nào dùng REST (cũ) khi nào GraphQL (mới), tránh nhân đôi pattern.
- Quality gate thêm dần cho code mới (Prettier, Husky, test) — đừng làm một phát toàn repo.

---

## 8. Hiện trạng và lộ trình

Hiện trạng hôm nay, nói để biết nâng cái gì trước chứ không phải để chấm điểm repo:

- Đã có nền AI: repo Expo hiện đại như v567app — có context file, `.claude/`, `.mcp.json`. Lấy làm mẫu nhân ra.
- Quality gate đủ nhưng chưa cấu hình AI: bare-RN (v560ns/v578) — ESLint, Prettier, Husky, Jest, Maestro đủ cả, nhưng chưa có context file hay skill kit.
- Cần retrofit: legacy v551 — có ESLint, TS và một CLAUDE.md tốt, nhưng thiếu Prettier, Husky, test runner.

Lộ trình làm theo phase:

1. Quality gate + PR CI gate cho mọi repo: TS cho code mới, ESLint, Prettier, Husky + lint-staged, Jest; cổng CI chạy lint + typecheck + test.
2. Context file cho mọi repo: fork `CLAUDE.md.template` về điền; repo nào có người dùng Codex/Gemini thì thêm `AGENTS.md`/`GEMINI.md` trỏ về cùng nguồn; chạy `onboarding-checklist.md`.
3. Skill kit + MCP đã verify: đánh giá rồi cài bộ skill phù hợp và MCP device chung; tự viết skill cho mấy việc lặp lại của team.

Nên giao một người (hoặc nhóm nhỏ) làm owner của chuẩn: review template, đánh giá skill, cập nhật khi có tool mới, rà lại mỗi sprint. Đây là living document, không phải viết một lần xong để đó.

---

## 9. Mấy lỗi hay gặp

- Sửa thẳng `android/`/`ios/` ở repo CNG → đổi qua config plugin / theo playbook.
- Commit `.env` hoặc dán secret vào prompt → Passbolt + gitignore.
- Cắm MCP/DB vào production → chỉ staging, read-only.
- Tap không qua discovery tool (đoán toạ độ) → discovery, lấy toạ độ, rồi tap.
- Không confirm plan đã code → plan, review, confirm, rồi code.
- Ship code AI chưa review, hoặc tin skill chưa verify → người review, và verify skill trước khi áp.
- Đụng repo legacy mà không chịu document cấu trúc → ghi gotcha vào context file trước.

---

## 10. Nguồn để tự kiểm

Ghi nguồn ra để team kiểm trước khi tin, đừng dùng tool/skill mù:

- agents.md — chuẩn mở cho context file của agent (Codex và nhiều tool dùng): https://agents.md
- Model Context Protocol (MCP) — chuẩn nối tool ngoài, Claude Code / Codex / Gemini CLI đều hỗ trợ.
- react-native-best-practices — skill perf RN của Callstack; kiểm repo/marketplace nguồn trước khi cài.
- Bộ skill quy trình (superpowers) — cài dạng plugin; xác minh nguồn và chạy thử trước khi đưa vào chuẩn team.
- Argent — MCP/CLI điều khiển iOS simulator và Android emulator cho RN.
- Skill nội bộ (ET, Jira-summary…) — do team tự viết; chưa verify thì chưa phải chuẩn.

Tài liệu này là living document — cập nhật khi tool, convention, hoặc cấu trúc dự án đổi.
