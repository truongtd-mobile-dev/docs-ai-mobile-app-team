# AI Coding — Bản đúc kết kiến thức

> Ghi chú nội bộ cho lead, không phải bản team-facing. Mục đích: gom toàn bộ thứ gom được (6 bài viết + 3 script video + review của tôi) vào một chỗ để **bạn đọc rồi tự quyết** cái nào đưa vào `rn-ai-standard.html`.
> Giọng thẳng, có chính kiến và có đánh dấu độ chắc. Tool-agnostic (Claude Code / Codex / Gemini) — khi nói tên file/lệnh tôi ghi cả 3 nếu khác nhau.
> Cuối file là **bảng quyết định**: từng khái niệm → đã có trong deck chưa → nên thêm không → nguồn & độ chắc.

---

## 0. Cách đọc & độ tin cậy

Quy ước độ chắc, để bạn khỏi phải tin tôi mù:

- **[Đồng thuận]** — ≥3 nguồn độc lập nói cùng một thứ. Coi như chuẩn.
- **[1 nguồn]** — chỉ 1 bài/video nói. Hay nhưng cân nhắc, chưa phải luật.
- **[Tôi verify]** — tôi tự kiểm trong repo của team, không phải nghe nói.
- **[Ý kiến]** — quan điểm của tôi với tư cách người dùng, có thể cãi.

**Nguồn không lấy được:** Reddit (bị chặn) + 2 bài Medium (lỗi cert). Nội dung của chúng trùng nặng với 6 nguồn còn lại nên không nghĩ là thiếu mảng nào, nhưng cứ ghi ra cho minh bạch.

---

## 1. Mô hình tư duy gốc (nắm cái này thì phần còn lại tự suy ra được)

**AI = một junior dev cực nhanh, kiên nhẫn vô hạn, nhưng không có trí nhớ dài hạn và không có phán đoán (judgment).** Mọi best-practice bên dưới đều quy về việc xử lý đúng 3 điểm yếu đó:

- Không trí nhớ → **context engineering** (phần 2): nạp đúng thứ cần, dọn khi xong.
- Không phán đoán → **safety & review** (phần 5): không tin output mù, người vẫn chịu trách nhiệm.
- Nhanh nhưng dễ lạc → **workflow có kiểm soát** (phần 3): plan trước, từng bước nhỏ, verify.

**Tool-agnostic:** Claude / Codex / Gemini khác nhau ở UI và tên lệnh, **giống nhau ở 3 nguyên lý trên**. Đừng dạy team "cách bấm Claude", dạy "cách làm việc với một AI agent". [Ý kiến] — đây nên là thông điệp mở đầu của deck.

**Ràng buộc nên nằm trong file/config, không nằm trong prompt.** [1 nguồn — shanraisshan/12-patterns] Khác biệt giữa "vibe coding" (gõ prompt → sửa → lặp) và làm việc bài bản là: thay vì nhắc lại "chỉ đọc, đừng sửa, theo convention X" mỗi lượt, ta đẩy ràng buộc vào nơi *luôn có hiệu lực* — context file, agent có giới hạn tool cứng, hook, permission allowlist. Config enforce thì không phụ thuộc hôm nay bạn nhớ nhắc hay không. Đây là khung tư duy gốc cho phần 5–6 bên dưới.

**"AI không thay thế dev. Nó thay thế dev không dùng AI."** [1 nguồn — video 8 levels] Câu này hơi khẩu hiệu nhưng đúng tinh thần: chênh lệch năng suất giờ nằm ở chỗ ai biết điều phối AI. Dùng được làm câu chốt deck, đừng lạm dụng.

**8 mức "AI developer"** [1 nguồn — video]: từ (1) chưa dùng → (2) hỏi đáp như chatbot → (3) để AI sửa file → (4) tắt bớt permission, để chạy dài → ... → (8) điều phối nhiều agent song song. Không cần bê nguyên thang đo này, nhưng cái insight đáng giữ: **đi từ "kèm sát từng bước" sang "giao việc + review kết quả" là một quá trình, không nhảy cóc được.** Team mới nên ở mức kèm sát.

---

## 2. Context engineering — kỹ năng trung tâm

Đây là phần mọi nguồn nói nhiều nhất và cũng là phần deck đang mỏng nhất. Nếu chỉ thêm được 1 thứ vào deck, thêm ở đây.

### 2.1 Vì sao context là ràng buộc số 1
Cửa sổ context có hạn, và **càng đầy thì chất lượng càng tụt** — gọi là *context rot*. [Đồng thuận] Không phải "còn chỗ là còn tốt"; nhồi nhiều file rác vào làm AI lú đi. Một video mô tả chất lượng bắt đầu rơi từ **khoảng nửa cửa sổ**. [1 nguồn]

### 2.2 Kỷ luật ngân sách context (con số cụ thể)
- Giữ context **thấp**; dân pro nhắm **<30%**, lằn ranh khó chịu là **>40%**. [Đồng thuận, dinanjana + skill video]
- **Mở session mới khi vượt ~50%** hoặc khi chuyển sang task khác. [1 nguồn — Beto] "Một task = một session" là heuristic tốt.
- **Rewind/checkpoint rẻ hơn sửa chồng.** Khi AI đi sai, `/rewind` hoặc bỏ session làm lại — đừng để bản-sửa-lỗi nằm lại trong context vì nó vừa tốn token vừa làm nhiễu các lượt sau. [Ý kiến, củng cố bởi Beto]

→ **Deck hiện chỉ nói "reset khi xong / compact khi dài" — vague. Thêm 3 con số trên là nâng cấp lớn nhất, rẻ nhất.**

### 2.3 Đừng dán code — để AI tự đọc
`@mention` file/thư mục, hoặc để AI tự search. Dán nguyên file khổng lồ vào prompt là phản mẫu kinh điển: tốn context, lại còn dễ lệch bản. [Đồng thuận] — deck đã có, giữ.

### 2.4 Các lệnh dọn context
- `/clear` — xoá sạch, bắt đầu lại (giữa các task không liên quan).
- `/compact` — nén lịch sử lại, **kèm gợi ý** ("giữ phần X, bỏ phần Y") để nén có định hướng. [Đồng thuận] — deck đã có note compact, ổn.
- `/rewind` — quay lại checkpoint trước (xem 2.2).

### 2.5 Subagent để giữ context sạch — **mục đang thiếu hẳn trong deck**
Việc đào sâu (điều tra bug, đọc 20 file để hiểu một flow, research) → **giao cho subagent**. Subagent chạy trong context riêng, trả về **chỉ kết luận**, main session không bị ngập file rác. [Đồng thuận — official docs, ranthebuilder, dinanjana, viblo, video GSD]

Đây là cách *scale vượt một cửa sổ context*. Video "GSD" và "8 levels" coi đây là bước ngoặt giữa người dùng thường và power user. [Ý kiến] **Phải có trong deck** — và nó là "phần 2" tự nhiên của chương Context, không phải nội dung "nâng cao" tách rời.

> **Cảnh báo myth:** một bài blog lan truyền nói skill có frontmatter `context: fork` để tự chạy isolated — **sai** (tôi đã verify với docs chính thức). Skill KHÔNG tự isolate; cơ chế isolate context **luôn là subagent**. Đừng đi tìm `context: fork`.

### 2.6 Ghi plan/quyết định ra file để sống qua /clear
Plan và các quyết định kiến trúc → viết ra `plan.md` / `decisions.md` / `architecture.md`. [1 nguồn — dinanjana] Lý do: chúng là *context bền*, không bay mất khi clear session, và là chỗ AI đọc lại ở session sau. Khớp thẳng với 2.2 (session mới mà không mất mạch).

### 2.7 Context file — một nguồn, nhiều tool
`CLAUDE.md` (Claude) / `AGENTS.md` (Codex) / `GEMINI.md` (Gemini) đều là cùng một thứ: file mô tả dự án nạp tự động. [Đồng thuận]
- **Giữ ngắn — nhắm <200 dòng.** [Đồng thuận — ranthebuilder + docs chính thức] Docs nói rõ: **không có ngưỡng "bị bỏ qua"** — file dài hơn chỉ *giảm độ tuân thủ* và ngốn context. (Con số "60 dòng tối ưu / 200 là trần / 200+ bị ignore" trong blog 12-patterns là phóng tác — tôi đã verify, sai.)
- **Scope rule tới đúng nơi cần** [Tôi verify — docs chính thức]: hai cơ chế thật:
  - **Nested:** đặt context file trong thư mục con (vd `payments/CLAUDE.md`) → nạp *on-demand* khi AI đọc file trong thư mục đó. Để rule cạnh code nó áp dụng, đừng dồn hết vào root.
  - **Path-specific:** rule chỉ áp cho path khớp glob, khai báo qua frontmatter `paths:` (KHÔNG phải syntax `<important if="language=go">` mà blog viết — cái đó không tồn tại).
- **Auto-memory:** công cụ tự ghi lại thứ học được qua các session. [1 nguồn — official] Biết là có, đừng dựa dẫm — nó bổ sung chứ không thay file context bạn tự viết.
- **Một nguồn duy nhất:** nếu repo dùng nhiều tool, symlink/@import về cùng một file gốc thay vì maintain 3 bản lệch nhau. [Ý kiến] — deck đã có ý này (slide multi-AI), tốt.

---

## 3. Workflow chuẩn: Explore → Plan → Code → Commit (+ verify)

Khung này lặp lại gần như nguyên văn ở mọi nguồn. [Đồng thuận] Deck đang có hầu hết, dưới đây là các điểm tinh chỉnh.

### 3.1 Explore / Hiểu task trước
- Đọc task kỹ, **hỏi lại khi mơ hồ** thay vì đoán. [Đồng thuận] — `bss-jira-task-summary` của bạn chính là cụ thể hoá bước này.
- **Chốt data model sớm.** [1 nguồn — Beto, nhưng đáng giá] Đổi shape của data về sau = sửa cả DB + toàn bộ UI đụng tới nó. Với app build nhanh, sai data model là khoản nợ đắt nhất. RN/mobile rất hợp đưa vào (kèm chương Read-task).

### 3.2 Plan trước khi code
- **Plan Mode** (Claude: Shift+Tab; Codex/Gemini có cơ chế tương đương). [Đồng thuận]
- **Đọc plan, sửa plan, rồi mới confirm.** [Đồng thuận] Phản mẫu lớn: *accept plan mù*. Plan sai mà gật đầu thì code ra sai theo.
- Persist plan ra file (xem 2.6).
- **Spec-driven cho việc lớn, plan-mode cho việc nhỏ.** [1 nguồn — ranthebuilder] Đừng bắt mọi task nhỏ phải qua quy trình spec nặng.

### 3.3 Code theo lát mỏng (vertical slice)
- Làm **từng phần nhỏ, verify được**, không phải một cú đổ 800 dòng. [Đồng thuận]
- **Vertical slice / tracer bullet:** một lát cắt xuyên hết các tầng (UI → logic → data) chạy được, hơn là làm xong hết tầng này mới sang tầng kia. [1 nguồn — ranthebuilder] [Ý kiến] đáng thêm 1 dòng vào chương Implement.
- Giữ vai **reviewer chủ động**, không auto-accept; thấy sai thì **ESC ngắt ngay**. [Đồng thuận] — deck đã có ở chương Debug.

### 3.4 Verify trước khi nói "xong" — **deck đang nhẹ chỗ này**
- **"Prove it works."** Bắt AI chạy lệnh + đưa bằng chứng (test pass, output, screenshot) *trước khi* claim done. [Đồng thuận — ranthebuilder + video ultra-review] AI rất hay tuyên bố "đã xong" khi chưa chạy gì.
- **Bug phải reproduce trước khi báo/sửa.** [1 nguồn — video ultra-review] Không repro được thì chưa hiểu bug.
- Self-test (deck đã có) + verify-before-done là hai mặt của một thứ.

### 3.5 Code review
- **Review mọi code AI viết trước khi merge.** [Đồng thuận] Không self-merge code chưa đọc hiểu.
- **Cross-model / second-opinion review:** cho một AI khác (hoặc /review, /ultra-review với nhiều reviewer song song) soi lại code của AI thứ nhất. [1 nguồn — video] Mạnh vì mỗi model mù theo kiểu khác nhau.
- Có thể gắn review vào CI (GitHub Actions / pipeline tự review PR). [1 nguồn — official]

### 3.6 Debug có hệ thống
- Công thức 5 thành phần (deck đã có) + **reproduce trước, đoán sau.** [Đồng thuận]
- RN signals: Metro log / Reactotron / phân biệt lỗi JS vs native / device log.

### 3.7 Commit như checkpoint
- **Commit thường xuyên = revert rẻ.** [1 nguồn — Beto] Trước khi cho AI làm bước rủi ro, commit cái đã chạy được; sai thì `git revert`/`reset` về điểm sạch thay vì gỡ rối.

---

## 4. Model, thinking, effort

Bạn đã bỏ slide model. [Ý kiến] Tôi nghĩ giữ **1 dòng** là đủ, không cần slide riêng:

- **Chọn model theo độ khó:** Opus (kiến trúc/debug khó) · Sonnet (impl thường) · Haiku (việc cơ học, rẻ) · Fable 5. [Đồng thuận — ranthebuilder, shanraisshan] **Đừng mặc định model to nhất** cho mọi việc — tốn tiền vô ích.
- **Extended thinking:** `ultrathink` là keyword thật để yêu cầu suy nghĩ sâu nhất cho lượt đó [Tôi verify — docs chính thức]. Chuỗi `think` < `think hard` < `think harder` thì 2 blog (dinanjana, 12-patterns) mô tả như tầng NL trigger, **nhưng docs hiện tại không xác nhận** — tầng thấp bật qua UI/menu. → Trong deck cứ nói "có chế độ suy nghĩ sâu; dùng cho việc khó", đừng hardcode chuỗi keyword vì nó không chắc và đổi theo version.
- **Reasoning effort / fast mode** (tuỳ tool). 1M context có nhưng đắt, dùng khi thật cần.
- Model thay đổi nhanh — đừng hardcode tên model vào deck quá chi tiết, sẽ lỗi thời. [Ý kiến]

---

## 5. An toàn & độ tin — **chủ đề được nhấn mạnh nhất, deck đang yếu nhất**

Đây là chỗ tôi phản biện mạnh quyết định trước của bạn (cắt Anti-patterns + Advanced). Mọi nguồn — kể cả video khẩu hiệu nhất — đều quy về: **đừng tin AI mù.** Deck mất gần hết mảng này khi bỏ 2 chương kia.

- **Người vẫn chịu trách nhiệm cuối.** AI là đòn bẩy, không phải người gác cổng. [Đồng thuận]
- **Permission allowlist** (`settings.json` allow/deny) — cho phép lệnh đọc/an toàn, chặn lệnh nguy hiểm. [Đồng thuận — shanraisshan, viblo, official]
- **Secrets:** không commit `.env`; lấy từ Passbolt. [Tôi verify — đúng với v551/v560ns]
- **MCP / device / DB không trỏ production.** [Ý kiến — quan trọng với mobile]
- **Prompt injection / nội dung không tin cậy:** web page, issue, file lạ có thể chứa lệnh ẩn. AI đọc phải coi đó là dữ liệu, không phải lệnh. [1 nguồn] Đáng 1 dòng.
- **AI hay bỏ sót những thứ "không thấy ngay":** SEO, analytics, performance, security hardening, accessibility, edge case. [1 nguồn — ranthebuilder] Người phải chủ động hỏi tới.
- **Agent có giới hạn tool cứng:** định nghĩa subagent (vd `.claude/agents/reviewer.md` với `allowed_tools` chỉ Read) → nó *vật lý không thể* sửa file, bất kể prompt nói gì. [Tôi verify — docs chính thức] Đây chính là "config enforce, không phải prompt" — chắc chắn hơn câu "đừng sửa file nhé".
- **Hooks làm guardrail tự động:** `PreToolUse` / `PostToolUse` / `Stop` — chạy *ngoài* token-loop nên không tốn token của AI. [Tôi verify] Dùng: auto-format/lint sau khi sửa, chặn commit `.env`, chặn sửa `android/ios` (CNG), chạy script kiểm tra khi AI báo "xong". *Lưu ý:* `Stop` hook có thể chặn/inject context nhưng **không có cơ chế "tự đẩy agent quay lại làm tiếp"** sẵn — blog nói có là sai. — đây là nội dung "Advanced" cũ, đáng kéo lại.

→ **Đề xuất mạnh:** gộp lại **1 slide "Dùng AI an toàn + review"** (không bê lại bản dài cũ). Theo tôi cái này đáng hơn mọi mục lẻ ở phần 2–4, vì nó là thông điệp trung tâm của cả bộ tài liệu.

---

## 6. Mở rộng AI: skills, commands, hooks, MCP

- **Skills** — đóng gói quy trình lặp lại. Nguyên tắc: **tự xây + verify trước khi chuẩn hoá**, 1 skill = 1 mục đích, output có cấu trúc. [Đồng thuận] — deck đã đóng khung đúng (không bê skill chưa thử thành chuẩn team). `bss-feature-et` vẫn ở diện **chưa verify**, đừng nâng thành chuẩn.
- **Phần signal cao nhất của một skill là "Gotchas".** [1 nguồn — Thariq, qua 12-patterns] Không phải overview hay ví dụ, mà là **các kiểu lỗi AI thực sự gặp** khi chạy skill đó — ghi lại mỗi lần fail, tích luỹ dần. Trùng đúng cách bạn viết skill (`bss-jira-task-summary` có "Anti-Patterns" + "Red Flags") → đây là chỗ nên dặn team đầu tư khi viết skill.
- **Slash commands** (`.claude/commands/`) — shortcut cho flow lặp (`/et`, `/review`, `/pr`). [Đồng thuận]
- **Hooks** — tự động hoá + enforce (xem phần 5). [Đồng thuận]
- **MCP:**
  - Device automation (Argent / mobile-mcp) — chỉ staging/read-only.
  - **Docs MCP (Context7)** — kéo doc API mới nhất, tránh AI code theo kiến thức cũ. [Đồng thuận — shanraisshan, viblo] Đáng nhắc 1 dòng vì RN/Expo đổi API liên tục.
  - **Serena (LSP)** — truy cập code theo symbol, tiết kiệm token so với đọc cả file. [1 nguồn] Hơi niche, optional.
- **Skill > MCP khi làm được cả hai:** skill minh bạch, review được; MCP là hộp đen hơn. [1 nguồn — ranthebuilder] [Ý kiến] đồng ý.
- **Hệ sinh thái skill có sẵn** (superpowers, GSD, skill-creator, claude-mem...) [1 nguồn — video]: **đánh giá rồi mới adopt**, đừng cài cả mớ vì thấy người ta khen. Trùng tinh thần "verify trước khi chuẩn hoá".

---

## 7. Scale lên (nâng cao — optional cho deck team mới)

- **Git worktree** — nhiều nhánh/agent chạy song song không đạp nhau. [Đồng thuận]
- **Nhiều agent / agent team** — một lead điều phối, mỗi agent một việc. [Đồng thuận — official, video]
- **Background agents / scheduled / CI review** — giao việc chạy nền, hoặc tự review PR trong pipeline. [1 nguồn — official]
- **Token/cost monitoring (ccusage)** — theo dõi chi phí. [1 nguồn] Niche.

[Ý kiến] Phần này gói gọn ở **nửa dưới của slide "an toàn + nâng cao"**: chỉ cần worktree + multi-agent + subagent. Đừng làm cả chương — team chưa tới mức đó.

---

## 8. React Native / mobile — phần đặc thù

- **Data model sớm** (đã nói ở 3.1). Với app, đây là quyết định đắt nhất.
- **New Architecture là default** (RN mới); repo nào còn Old Arch phải **ghi lý do** trong context file. [Tôi verify — v551/v560ns đã New Arch]
- **React Compiler** (React 19 / RN 0.81+) auto-memo — giảm tay nghề tối ưu re-render. [Đồng thuận RN community] Verify version trước khi đưa con số vào deck.
- **Hermes / FlashList / TTI** — perf cơ bản; profile trước khi optimize. [Đồng thuận]
- **3 archetype** (đã trong deck/onboarding): CNG (đừng sửa native, prebuild + plugin-hash) · bare-RN (native committed, Fastlane, patch-package) · legacy (document cấu trúc lạ TRƯỚC). [Tôi verify across v567/v560ns/v551]
- **Device testing:** Argent (RN-aware, chính) / mobile-mcp (nhẹ); **Maestro cho E2E** (đã dùng ở v560ns/v578, không phải Detox). [Tôi verify]

---

## 9. Failure modes — gom một chỗ (đối chiếu khi review deck)

| Phản mẫu | Vì sao hỏng |
|---|---|
| Ask-and-pray (giao việc, không plan) | AI lạc hướng, ra một mớ phải bỏ |
| Accept plan mù | Plan sai → code sai theo, tốn hơn |
| Dán file khổng lồ vào prompt | Đốt context, dễ lệch bản — để AI tự `@`/đọc |
| Skip discovery (đoán toạ độ/đoán API) | Hallucinate; với device phải discovery trước tap |
| Ship code AI chưa review | Bug + lỗ hổng lọt; người vẫn chịu trách nhiệm |
| Để context phình (>50%) | Context rot, chất lượng tụt — clear/compact/subagent |
| Claim "done" chưa chạy gì | Phải prove-it-works trước khi đóng |
| Chống cấu trúc legacy thay vì document nó | Phí công; document cái lạ rồi để AI theo |

---

## 10. Bảng quyết định — bạn dùng cái này để chốt deck

Cột "Khuyến nghị": **THÊM** (đáng đưa vào deck) · **UPDATE** (đã có, làm rõ thêm) · **GIỮ** (đã ổn) · **OPT** (optional, tuỳ bạn).

| Khái niệm | Trong deck? | Khuyến nghị | Độ chắc |
|---|---|---|---|
| Subagent giữ context sạch | Không | **THÊM** (ưu tiên 1) | Đồng thuận |
| Slide "an toàn + review" gộp | Mất khi cắt | **THÊM** (ưu tiên 1) | Đồng thuận |
| Ngưỡng context cụ thể (<40%, session mới >50%, rewind) | Vague | **UPDATE** (ưu tiên 2) | Đồng thuận |
| CLAUDE.md ngắn (~<200 dòng) + scope theo tầng + auto-memory | Không | **UPDATE** (ưu tiên 2) | Đồng thuận |
| Persist plan ra file (plan.md/decisions.md) | Không | **THÊM** | 1 nguồn |
| Prove-it-works / reproduce trước khi sửa | Nhẹ | **UPDATE** | Đồng thuận |
| Cross-model / parallel review | Không | OPT | 1 nguồn |
| Model & effort (1 dòng) | Đã bỏ | **UPDATE** (1 dòng, không cần slide) | Đồng thuận |
| Vertical slice / tracer bullet | Không | OPT (1 dòng vào Implement) | 1 nguồn |
| Data model sớm | Không | **THÊM** (RN-relevant) | 1 nguồn |
| Hooks guardrail (format/lint/chặn .env/native) | Mất khi cắt | gộp vào slide an toàn | Đồng thuận |
| Agent giới hạn tool cứng (`allowed_tools`) | Không | OPT (1 dòng slide an toàn) | Tôi verify |
| Permission allowlist | Mất khi cắt | gộp vào slide an toàn | Đồng thuận |
| Prompt injection | Không | OPT (1 dòng slide an toàn) | 1 nguồn |
| Docs MCP (Context7) | Không | OPT (1 dòng) | Đồng thuận |
| Serena / ccusage / background agents | Không | OPT (bỏ qua được) | 1 nguồn |
| Git worktree + multi-agent | Mất khi cắt | nửa dưới slide nâng cao | Đồng thuận |
| @mention / không dán code | Có | GIỮ | Đồng thuận |
| /clear, /compact (có hint) | Có | GIỮ | Đồng thuận |
| Plan Mode → review → confirm → từng phần | Có | GIỮ | Đồng thuận |
| Debug 5 thành phần + ESC | Có | GIỮ | Đồng thuận |
| Skills: tự xây + verify trước chuẩn hoá | Có | GIỮ | Đồng thuận |
| Multi-AI một nguồn (CLAUDE/AGENTS/GEMINI) | Có | GIỮ | Đồng thuận |
| 3 archetype RN | Có (onboarding) | GIỮ | Tôi verify |

**Nếu chỉ làm "ưu tiên 1 + 2"** (5 việc): deck đi từ 14 → ~16 slide, lấy lại đúng phần xương sống bị mất, vẫn gọn. Đó là mức tôi khuyên.

---

## Phụ lục — nguồn & đóng góp của từng nguồn

**Bài viết (lấy được 6/8):**
1. `code.claude.com/docs` (official) — CLAUDE.md + memory, skills, hooks, MCP, subagents/agent teams, background agents, CLI scripting, scheduled, GitHub Actions review, plan review.
2. ranthebuilder — CLAUDE.md <200 dòng, guardrails, spec-driven vs plan-mode, Opus/Sonnet phân vai, skill > MCP, manual security review, AI bỏ sót SEO/analytics/perf/hardening, domain > tool.
3. shanraisshan (github) — settings.json permissions, Serena/Context7 MCP, subagents, extended thinking levels, custom commands, worktree+ccmanager, /clear /compact, Explore→Plan→Code→Commit, ccusage.
4. dinanjana (medium) — Plan Mode, review plans, small verifiable steps, clarifying Qs, .claude hierarchy, persist plans ra file, thinking depth, treat-as-junior, ESC, engaged reviewer, failure modes.
5. viblo — cụm trùng shanraisshan + SuperClaude, Vibe Kanban parallel agents.
6. "12 patterns" (Medium, do user cấp) — **là bản tóm tắt repo shanraisshan (= cùng nguồn #3), không phải nguồn độc lập** → không dùng để nâng lên [Đồng thuận]. Đóng góp chi tiết: nested/path-scoped CLAUDE.md, agent giới hạn tool, 3 hook point, Gotchas, khung "constraint trong file không trong prompt". Cảnh báo: bài trộn nhiều feature bịa (xem "Myths bị bác" dưới).

**Script video (3):**
7. Beto (RN/Expo build) — Expo Go, plan mode, be specific, **data model shape**, skills, context % (clear >50%, <30%), git commit checkpoint, model + 1M context, effort high/max, image trong prompt, @ reference, session mới mỗi task.
8. "6 skills" — skill-creator, superpowers, GSD (context engineering, fresh subagent mỗi task, quality gate), /review + /ultra-review (parallel reviewer, repro bug trước khi báo), context-mode, claude-mem, front-end design skill; context rot ~nửa cửa sổ.
9. "8 levels" — thang tiến hoá AI dev, điều phối fleet agent, "AI thay dev không dùng AI", permission on→off→multi-agent, review là kỹ năng sống còn.

**Trạng thái nguồn:** 2 bài Medium (dinanjana #4, "12 patterns" #6) ban đầu fetch lỗi cert — **user đã cấp text, giờ đã đọc tận mắt**. dinanjana không thêm gì mới so với bản tôi fetch được; "12 patterns" = tóm tắt shanraisshan (xem #6). Còn lại **Reddit vẫn chưa lấy được** (bị chặn) — chấp nhận thiếu.

---

### Myths bị bác (tôi verify với docs chính thức — để team khỏi tin nhầm)

Bài "12 patterns" lan truyền mạnh, trộn feature thật với bịa. Nếu ai trong team đọc nó, đính chính sẵn:

| Claim trong blog | Thực tế |
|---|---|
| `<important if="language=go">` — conditional rule trong CLAUDE.md | **Sai.** Path-scoped rule khai báo qua frontmatter `paths:` (glob), không phải HTML attribute. |
| `context: fork` — frontmatter cho skill tự chạy isolated | **Sai.** Skill không tự isolate; isolate là qua **subagent**. |
| `/sandbox` — lệnh giảm 84% permission prompt | **Sai.** Sandbox bật qua settings (`sandbox.enabled`), không có slash command. |
| `/loop 30m /code-review` — lịch chạy local | **Không xác nhận.** Chỉ có `/schedule` (routine trên cloud). |
| `--bare` — flag bỏ context discovery, nhanh 10x | **Sai.** SDK có `settingSources: []`, không có flag `--bare`. |
| CLAUDE.md "60 dòng tối ưu / 200 trần / 200+ bị ignore" | **Phóng tác.** Không có ngưỡng cứng; dài hơn chỉ *giảm tuân thủ*. |
| `Stop` hook fail → tự đẩy agent làm tiếp | **Một phần.** Hook chặn/inject được, nhưng không có cơ chế retry-loop sẵn. |
| think < think hard < think harder < ultrathink (NL trigger) | **Một phần.** Chỉ `ultrathink` là keyword xác nhận; tầng dưới bật qua UI. |

**Thật & dùng được** (đã verify): nested CLAUDE.md theo thư mục · path-scoped rule (`paths:`) · subagent giới hạn tool (`allowed_tools`) · hooks `PreToolUse/PostToolUse/Stop` chạy ngoài token-loop · `/btw` (hỏi xen không ngắt task) · `-w` / `--worktree` (chạy trong git worktree).
