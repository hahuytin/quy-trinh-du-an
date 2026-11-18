# Thiết kế plugin `quy-trinh-du-an`

Ngày: 2026-11-18 · Trạng thái: đã duyệt, đã dựng bản 1.0.0 · Người đặt hàng: hahuytin

## 1. Mục đích

Đóng gói cách làm việc đang dùng ở một dự án thật của tác giả (bàn → chốt việc → Codex hoặc Claude code → kết thúc buổi)
thành **một plugin Claude Code dùng chung cho mọi dự án**, của mọi người, ở mọi tài khoản.
Cài một lần trên máy; mở bất kỳ thư mục / kho nào, gõ `/khoi-tao` là Claude biết phải làm gì,
và người dùng biết mình quản dự án ở đâu, bằng cách nào.

Người dùng chính: người hiểu nghiệp vụ nhưng ít kinh nghiệm lập trình (như tác giả), làm việc bằng tiếng Việt.
Bản 1 chỉ nhắm người dùng tiếng Việt: tên mục hợp đồng, câu đồng ý, nội dung file sinh ra đều tiếng Việt (lựa chọn của Claude, theo việc người dùng chọn tiếng Việt).

## 2. Mục tiêu, không làm, thất bại

**Mục tiêu**
- Trạng thái dự án nằm trong file của kho (BACKLOG, context, phiếu), không nằm trong chat: `/clear` hay đổi máy không mất gì.
- Bước `/khoi-tao` thật tốt: bàn từ A đến Z, ra bộ file đủ để một người (hoặc một Claude) mới vào hiểu và làm tiếp được ngay.
- Mỗi buổi làm việc có điểm vào (`/bat-dau`) và điểm ra (`/ket-thuc`) rõ ràng.
- Có Codex thì Claude điều phối, Codex code để tiết kiệm token; không có Codex thì Claude tự code, quy trình vẫn chạy.
- README đủ để chính chủ dự án đọc lại sau vài tháng vẫn nhớ cách dùng và chi tiết từng bước.

**Không làm**
- Không deploy / đưa lên server. Người dùng tự làm.
- Không chứa luật riêng của một dự án nào (đường dẫn PHP, tiền tố bảng...). Luật riêng nằm trong `CLAUDE.md` của dự án.
- Không bắt cài plugin khác. Có thì tận dụng, không có vẫn chạy.
- Không có bản cho claude.ai web (web không chạy được git / test / Codex trên máy).

**Thất bại dù vẫn chạy được**
- `/khoi-tao` hỏi qua loa rồi tự bịa định hướng; bộ file đẹp nhưng sai ý người dùng.
- Người dùng phải nhớ quy trình trong đầu (README không đủ, hoặc skill bắt gõ lệnh lạ).
- Claude tự gộp vào `main`, tự chạy migration, tự đẩy lên nơi lạ mà không có câu đồng ý.
- Lộ bí mật (khóa, mật khẩu) vào kho công khai của dự án.

## 3. Các cách đã cân nhắc

| Cách | Ưu | Nhược |
|---|---|---|
| **A. Một plugin, 4 skill riêng** (đã chọn) | Giữ đúng 4 bước quen; sửa kho là mọi người cập nhật | Trùng tên lệnh với dự án thì phải gõ đủ `/quy-trinh-du-an:bat-dau` |
| B. Một skill lớn chứa cả 4 bước | Một tên duy nhất | Mất điểm dừng rõ ràng, Claude dễ chọn nhầm bước, nạp nhiều chữ |
| C. Kho mẫu chép vào từng dự án | Đơn giản | Mỗi dự án một bản, sửa quy trình phải chép tay khắp nơi |

Chọn A vì người dùng muốn "cài một lần, dùng cho mọi dự án" và giữ các bước rõ ràng.

## 4. Phân phối

- Kho GitHub **công khai** `hahuytin/quy-trinh-du-an`, thư mục máy `D:\HSE\quy-trinh-du-an`.
- Kho vừa là plugin vừa là "chợ" (marketplace) chứa chính nó.
- Cài: `/plugin marketplace add hahuytin/quy-trinh-du-an` → `/plugin install quy-trinh-du-an@quy-trinh-du-an`.
- Cập nhật: `/plugin marketplace update quy-trinh-du-an`.
- Kho công khai nên **không chứa** tên dự án nội bộ, đường dẫn máy, email, luật nội bộ công ty.
- Đẩy kho bằng tài khoản `hahuytin` (máy có thể đăng nhập nhiều tài khoản GitHub; không đổi tài khoản mặc định).
  Cách làm (lựa chọn của Claude): không dùng `gh auth switch`; lấy token riêng từng lệnh bằng `gh auth token --user hahuytin`
  (cho `gh repo create` qua biến `GH_TOKEN`, cho `git push` qua credential helper chỉ đặt trong kho này).
- Tạo kho công khai là bước cuối, sau kiểm thử T0-T5; Claude hỏi người dùng một câu xác nhận ngay trước khi tạo.
- Giấy phép: MIT (lựa chọn của Claude, sửa được).

## 5. Luồng sử dụng

```
Một lần cho mỗi dự án (hoặc khi đổi hướng lớn):
  /khoi-tao → bàn A-Z → viết bộ file → người dùng duyệt → commit → "Khởi tạo xong, gõ /bat-dau"

Mỗi buổi làm việc:
  /bat-dau → bàn, chốt việc của buổi → DỪNG: người dùng chọn từng việc "Giao Codex" / "Claude tự code"
           → làm theo lựa chọn → /ket-thuc → /clear
```

Điểm dừng "chọn ai code" là bắt buộc, kể cả việc nhỏ (chi tiết mục 9.2 bước 7).

**Khi nào gọi lại `/khoi-tao`** — quy tắc: *đổi phần nền → `/khoi-tao`; thêm việc → `/bat-dau`*.

| Tình huống | Lệnh |
|---|---|
| Thêm tính năng trong định hướng hiện có | `/bat-dau` |
| Thêm một module nhỏ | `/bat-dau` (tự thêm dòng bản đồ module + file context) |
| Sang giai đoạn mới, đổi stack, đổi mục tiêu, đổi luật làm việc | `/khoi-tao` (chế độ cập nhật) |
| Khởi tạo dở dang từ phiên trước | `/khoi-tao` (chế độ làm tiếp) |
| Dự án đã có bộ file kiểu cũ (kiểu cũ) | `/khoi-tao` (chế độ chuyển đổi) |

`/bat-dau` tự kiểm hợp đồng `CLAUDE.md` (mục 7); thiếu thì nhắc chạy `/khoi-tao` và nói rõ thiếu mục nào. Không cần gọi `/khoi-tao` trước mỗi lệnh.

**Chỉ chạy khi người dùng gõ lệnh** (lựa chọn của Claude): cả 4 skill đặt `disable-model-invocation: true`.
Lý do: `/ket-thuc` gộp nhánh, `/khoi-tao` viết lại file, không được tự kích hoạt; và dự án gốc đang có 3 lệnh cùng tên,
Claude không được tự chọn nhầm bản plugin. Khi `/bat-dau` cần giao Codex, nó đọc thẳng `../giao-codex/SKILL.md`
(cùng plugin) rồi làm theo. Trùng tên với lệnh của dự án: gõ đủ `/quy-trinh-du-an:<lệnh>`.

## 6. Cấu trúc kho

```
quy-trinh-du-an/
├─ .claude-plugin/
│  ├─ plugin.json            tên, mô tả, phiên bản, tác giả "hahuytin" (không email)
│  └─ marketplace.json       chợ chứa chính plugin này (source "./")
├─ skills/
│  ├─ khoi-tao/
│  │  ├─ SKILL.md
│  │  └─ mau/                khung file sẽ viết vào dự án
│  │     ├─ CLAUDE.md
│  │     ├─ AGENTS.md
│  │     ├─ dinh-huong.md
│  │     ├─ BACKLOG.md
│  │     └─ context.md
│  ├─ bat-dau/SKILL.md       (khung phiếu việc viết thẳng trong file này)
│  ├─ giao-codex/SKILL.md
│  └─ ket-thuc/SKILL.md
├─ docs/thiet-ke.md          file này
├─ README.md                 hướng dẫn dùng, tiếng Việt (mục 10)
└─ LICENSE
```

Toàn bộ chữ trong skill và README: **tiếng Việt có dấu**. Claude trả lời theo ngôn ngữ người gõ.

## 7. Hợp đồng `CLAUDE.md` của dự án

Skill không chứa luật dự án; skill đọc các mục **có tên cố định** trong `CLAUDE.md` của dự án.
Dòng đầu file có dấu phiên bản: `<!-- quy-trinh-du-an: v1 -->` để skill biết file theo hợp đồng nào.

| Mục (tên cố định) | Chứa gì | Bắt buộc |
|---|---|---|
| `## Dự án` | 1-3 câu dự án là gì, cho ai; trỏ tới `docs/dinh-huong.md` | Có |
| `## Nguyên tắc` | Điều không đổi suốt dự án (vd "ít file, ít tầng", "chữ hiển thị tiếng Việt") | Có |
| `## Stack` | Ngôn ngữ, framework, CSDL, đường dẫn công cụ đặc biệt | Có |
| `## Lệnh` | Chạy local; test 1 file; test cả module/dự án; build; kiểm cú pháp | Có (chưa có code thì ghi "chưa chốt") |
| `## Module` | Bảng: module, khóa, file context, ghi chú. Không chia module: 1 dòng "tong-quan" | Có |
| `## Git` | Nhánh chính, remote, mẫu commit message, push sau mỗi commit hay không. Bản 1 chỉ hỗ trợ đưa vào main bằng `merge` tại máy; chế độ PR để bản sau | Có |
| `## Câu đồng ý` | Câu bắt buộc trước việc khó lùi (mặc định: "đồng ý đưa vào main", "đồng ý chạy") | Có |
| `## Codex` | Có dùng không; luật ở `AGENTS.md`; việc nào giao / không giao | Có |
| `## Cấm` | Điều tuyệt đối không làm (server thật, push force, in bí mật...) | Có |
| `## Quy trình` | 3 dòng: dùng plugin `quy-trinh-du-an`, vòng làm việc, nơi để trạng thái | Có |

Dự án được thêm mục riêng ngoài các mục trên (vd "Cách đặt tên"). Mục "chưa chốt" ghi rõ ai quyết và khi nào.

## 8. Các file trong dự án

| File | Chứa gì | Ai viết | Khi nào đổi |
|---|---|---|---|
| `CLAUDE.md` | Luật + lệnh theo hợp đồng mục 7. Ngắn (nạp mọi phiên) | `/khoi-tao` | Khi đổi nền |
| `AGENTS.md` | Luật cho Codex (chỉ khi dùng Codex): phạm vi, không làm thừa, bắt buộc theo dự án, cấm, mẫu báo cáo | `/khoi-tao`; `/ket-thuc` đề xuất thêm luật | Khi Codex lặp lỗi |
| `docs/dinh-huong.md` | 1 trang: vấn đề, người dùng, thành công, không làm, thất bại, giai đoạn 1/2/... | `/khoi-tao` | Khi đổi giai đoạn / mục tiêu |
| `docs/tasks/BACKLOG.md` | Mỗi việc 1 dòng: số, module, việc, trạng thái, nhánh, file dùng chung; số kế tiếp; "Xong gần đây" | `/khoi-tao` tạo; `/bat-dau`, `/ket-thuc` cập nhật | Mỗi buổi |
| `docs/context/<module>.md` | Quyết định đã chốt + lý do, hướng đã bác, việc dở. Hiện trạng, không ghi lịch sử | `/khoi-tao` tạo; `/ket-thuc` cập nhật | Khi có quyết định mới |
| `docs/tasks/<số>-<slug>.md` | Phiếu việc: mục tiêu, file được sửa, file mẫu, không làm, tiêu chí xong, `[CẦN LÀM RÕ]`, nhật ký. Xong thì xóa | `/bat-dau` | Việc giao Codex hoặc kéo dài hơn 1 phiên |
| `docs/khoi-tao-nhap.md` | Nháp khi khởi tạo chưa xong trong 1 phiên. Xóa khi xong | `/khoi-tao` | Tạm thời |
| `.gitignore` | Thêm `.codex-out/` (khi dùng Codex) | `/khoi-tao` | |

## 9. Từng skill

### 9.1 `/khoi-tao` (quan trọng nhất)

**Ba chế độ**, tự nhận ra ở chặng 0:
- **Mới**: chưa có `CLAUDE.md` theo hợp đồng.
- **Làm tiếp**: có `docs/khoi-tao-nhap.md` → đọc, báo đã xong chặng nào, làm tiếp, không hỏi lại.
- **Cập nhật / chuyển đổi**: đã có bộ file (theo hợp đồng, hoặc kiểu cũ) → đọc hết, chỉ bàn phần thay đổi
  hoặc phần thiếu, ánh xạ mục cũ sang mục hợp đồng, **không ghi đè** nội dung cũ mà chưa hỏi.

**Sáu chặng**

| Chặng | Bàn gì | Ghi vào |
|---|---|---|
| 0. Xem trước | Đọc thư mục: code, README, git, file có sẵn. Nói ngắn đã thấy gì | không ghi |
| 1. Tầm nhìn | Vấn đề cần giải, ai dùng, vì sao bây giờ, thành công trông ra sao, điều làm dự án thất bại dù chạy được | `docs/dinh-huong.md` |
| 2. Phạm vi | Giai đoạn 1 tối thiểu; việc để giai đoạn sau; điều không làm | `dinh-huong.md` + BACKLOG |
| 3. Kỹ thuật | Stack (Claude đưa 2-3 phương án + chọn), CSDL, nơi chạy, ràng buộc máy / chi phí / người bảo trì | `CLAUDE.md` `## Stack`, `## Lệnh` |
| 4. Cấu trúc | Module, đặt tên, dữ liệu chính, ranh giới module | `## Module`, `docs/context/` |
| 5. Cách làm việc | Codex có không (kiểm `codex --version`), kho GitHub, merge hay PR, câu đồng ý, ai lên server, điều cấm | `## Git`, `## Câu đồng ý`, `## Codex`, `## Cấm`, `AGENTS.md` |

**Cách bàn trong mỗi chặng** (học từ superpowers brainstorming):
- Đọc trước, hỏi sau: điều thư mục / người dùng đã trả lời thì không hỏi lại.
- Hỏi **từng câu một**, câu hỏi mở khi chưa rõ ý; khi lựa chọn đặt tên được thì đưa 2-3 phương án kèm đánh đổi và phương án Claude chọn.
- Hết chặng: **đọc lại** điều đã hiểu (≤ 200 chữ), người dùng sửa, rồi mới sang chặng sau.
- Người dùng nói "Claude tự chọn": Claude chọn, ghi rõ là lựa chọn của Claude kèm lý do.
- Chỗ mơ hồ chưa chốt: ghi `[CẦN LÀM RÕ] <câu hỏi> — người quyết: ...` (học từ Spec Kit clarify), không bịa.
- Hết mỗi chặng ghi nháp `docs/khoi-tao-nhap.md` (để làm tiếp nếu phiên đầy).
- Người dùng có thể bỏ qua chặng ("để sau"): ghi "chưa chốt".

**Viết bộ file**: từ khung `mau/`, điền nội dung đã bàn. Ngắn: `CLAUDE.md` mục tiêu ≤ ~120 dòng.

**Điều kiện "khởi tạo xong"** (đủ mới báo xong):
1. Mọi mục bắt buộc của hợp đồng có nội dung hoặc "chưa chốt" kèm người quyết.
2. BACKLOG có việc giai đoạn 1, đánh số, số kế tiếp đúng.
3. Có `docs/dinh-huong.md` và ít nhất một file context.
4. **Người đọc mới**: gọi một subagent chưa biết gì cuộc bàn, đưa bộ file, hỏi "dựa vào đây bắt đầu việc số 1 được không, còn chỗ nào phải đoán mà đoán sai thì hỏng?".
   Câu nhỏ: Claude tự điền, ghi là lựa chọn của Claude. Câu cần người dùng: gom hỏi **một lần**.
   Không có công cụ subagent: Claude tự đọc lại với vai người mới.
5. Người dùng nói duyệt.
6. Git: chưa có thì `git init`; commit bộ file; xóa `docs/khoi-tao-nhap.md`. Có remote thì push.
   **Tạo kho GitHub mới** là việc ra bên ngoài: hỏi người dùng trước, mặc định riêng tư.

Kết thúc bằng: tóm tắt đã chốt gì, còn gì "chưa chốt", và "gõ `/bat-dau` để làm việc số 1".

### 9.2 `/bat-dau`

Giữ nguyên logic của `bat-dau` của dự án gốc, bỏ phần riêng dự án:
1. Kiểm hợp đồng `CLAUDE.md` (dấu phiên bản + các mục bắt buộc). Thiếu → nhắc `/khoi-tao`, nói thiếu mục nào; người dùng vẫn muốn làm thì làm tiếp.
2. Xem hiện trạng: BACKLOG, `git status`, nhánh, 5 commit gần nhất, worktree. Thay đổi chưa commit của việc khác: báo, không đụng.
3. Báo ngắn: việc dở (nhánh, bước tiếp), việc chờ quyết, việc chưa làm.
4. Hỏi buổi này làm gì → xác định module (một phiên một module) → đọc file context → bàn, cảnh báo điều đã bị bác.
5. Việc cần module mới: thêm dòng `## Module` + tạo file context (không cần `/khoi-tao`).
6. Chốt bảng việc: số, việc, **đề xuất** ai code (Codex/Claude theo `## Codex`, kèm lý do 1 dòng), file dùng chung, có thao tác cần câu đồng ý không. Việc lớn: Plan mode.
7. **ĐIỂM DỪNG BẮT BUỘC: hỏi ai code.** Bàn xong, chốt bảng xong thì **dừng hẳn lượt**, hỏi người dùng từng việc:
   "Giao Codex" hay "Claude tự code" (đề xuất của Claude đặt đầu). Chưa có câu trả lời thì **không viết một dòng code nào**,
   không tạo nhánh, không giao Codex — kể cả khi việc nhỏ, kể cả khi Claude "chắc chắn" tự làm được, kể cả khi người dùng đã nói "làm đi"
   lúc đang bàn (câu đó là đồng ý ý tưởng, chưa phải chọn ai code). Máy không có Codex: vẫn dừng hỏi "Claude code việc này nhé?".
   Lý do: người dùng muốn tự quyết từng lần, vì giao Codex tiết kiệm token còn Claude code thì tốn hơn; trước đây Claude hay bàn xong là code luôn.
   Hỏi bằng công cụ hỏi lựa chọn (nếu có) để người dùng bấm, không chỉ viết câu hỏi trong đoạn văn.
8. Có câu trả lời rồi: cập nhật BACKLOG trên nhánh chính (cột "Ai code" theo câu trả lời), commit, push (nếu `## Git` bảo push).
   Viết phiếu cho việc giao Codex hoặc kéo dài hơn 1 phiên (khung phiếu trong skill; có mục `[CẦN LÀM RÕ]`).
9. Tạo nhánh `lam/<YYYY-MM-DD>-<chủ-đề>` (hoặc vào nhánh dở + merge nhánh chính). Làm một mạch theo câu trả lời,
   chỉ dừng ở: duyệt kế hoạch, câu đồng ý. Việc Claude định tự code mà giữa chừng thấy nên giao Codex (hoặc ngược lại): dừng, hỏi lại.
- Sửa lỗi: nếu máy có skill `systematic-debugging` (superpowers) thì dùng; không có thì tìm gốc lỗi trước khi sửa.
- Thiết kế việc ghi vào **phiếu việc**, không ghi vào thư mục của plugin khác (vd `docs/superpowers/`).

### 9.3 `/giao-codex <số>`

Giữ logic `giao-codex` của dự án gốc (phiếu → ra lệnh → review → phản hồi tối đa 3 vòng → chuyển Claude), bổ sung:
- Đầu tiên kiểm `codex --version`. Không có / lỗi → báo một dòng, Claude tự code, ghi vào BACKLOG.
- Lệnh `codex exec` chạy qua **Bash** (Git Bash trên Windows), có `< /dev/null`, chạy nền, đợi thông báo.
- Review đọc `## Nguyên tắc`, `## Stack` của `CLAUDE.md` và `AGENTS.md` thay cho danh sách luật cứng của dự án gốc.
- Lỗi Codex mắc ghi vào nhật ký phiếu (để `/ket-thuc` gom).

### 9.4 `/ket-thuc`

Giữ logic `ket-thuc` của dự án gốc, bổ sung:
- **Có bằng chứng mới nói xong** (học từ verification-before-completion): chạy test theo `## Lệnh`, dán kết quả đạt/tổng.
- **Soát độc lập** việc do Claude tự code trước khi đưa vào main: dùng `/code-review` nếu có, không có thì gọi subagent review diff. Việc Codex code đã được Claude review thì bỏ qua.
- **Dò bí mật** trong diff nhánh so với nhánh chính: `.env`, chuỗi giống khóa API / mật khẩu / token. Có → dừng, báo.
- **Đưa vào main**: chờ câu đồng ý rồi `git merge --no-ff` tại máy, push theo `## Git` (bản 1 không có chế độ PR).
- **Học từ lỗi Codex**: lỗi cùng loại lặp ≥ 2 lần (trong nhật ký phiếu / BACKLOG) → đề xuất câu luật thêm vào `AGENTS.md`, người dùng đồng ý mới ghi.
- Cập nhật BACKLOG, chép quyết định sang `docs/context/<module>.md`, xóa phiếu và nhánh đã gộp, nhắc `/clear`.

## 10. README (yêu cầu riêng của người dùng)

Người đọc (kể cả chủ dự án sau vài tháng) đọc README là hiểu cách dùng và chi tiết, không phải nhớ. Các mục:
1. Plugin là gì (3 câu) và dành cho ai.
2. Cài đặt, cập nhật, gỡ (lệnh chép được).
3. Luồng một hình (khối chữ như mục 5).
4. Bảng 4 lệnh: lệnh · khi nào gọi · làm gì · kết quả để lại. Nói rõ các điểm Claude luôn dừng hỏi: chọn ai code, duyệt kế hoạch, câu đồng ý.
5. `/khoi-tao` chi tiết: 3 chế độ, 6 chặng, file tạo ra, điều kiện xong, khi nào gọi lại.
6. Một buổi làm việc mẫu (đoạn hội thoại ngắn từ `/bat-dau` đến `/ket-thuc`).
7. Các file trong dự án, mỗi file chứa gì (bảng mục 8).
8. Hợp đồng `CLAUDE.md` (bảng mục 7).
9. Câu đồng ý và vì sao có.
10. Codex: cài thế nào, không có thì sao.
11. Câu hỏi thường gặp: quên đang ở đâu; muốn đổi stack; trùng tên lệnh; dùng cùng superpowers; phiên đầy; nhiều người cùng dự án.
12. Thay đổi theo phiên bản.

## 11. Quan hệ với plugin / tính năng khác

| Nguồn | Học gì | Đặt ở |
|---|---|---|
| superpowers brainstorming | Hỏi từng câu, đọc lại, phương án kèm đề xuất, người đọc mới soát | `/khoi-tao` |
| superpowers verification-before-completion | Có bằng chứng mới nói xong | `/ket-thuc` |
| superpowers systematic-debugging | Tìm gốc lỗi trước khi sửa | `/bat-dau` (dùng nếu có) |
| Spec Kit constitution / clarify | Mục `## Nguyên tắc`; đánh dấu `[CẦN LÀM RÕ]` | hợp đồng, `/khoi-tao`, phiếu |
| BMAD product brief | Định hướng gói trong 1 trang | `docs/dinh-huong.md` |
| Claude Code `/code-review`, Plan mode | Soát độc lập; duyệt kế hoạch việc lớn | `/ket-thuc`, `/bat-dau` |

Chỉ học ý tưởng, không chép mã, không bắt cài kèm.

## 12. Kiểm thử trước khi công bố

Chạy bằng subagent trong thư mục nháp (không đụng dự án thật). Kịch bản `/khoi-tao` là hội thoại nên
mỗi kịch bản kèm **câu trả lời soạn sẵn của người dùng**; subagent đóng vai Claude dùng skill.

| # | Kịch bản | Đạt khi |
|---|---|---|
| T1 | `/khoi-tao` thư mục trống, ý tưởng "app quản lý kho vật tư nhỏ" | Đủ điều kiện xong (mục 9.1); có hỏi từng câu, có đọc lại; không bịa chỗ người dùng chưa nói |
| T2 | `/khoi-tao` trên dự án có sẵn code nhỏ | Đọc code trước, không hỏi điều code đã trả lời; `## Lệnh` đúng với code |
| T3 | `/khoi-tao` chuyển đổi bộ file kiểu cũ (bản chép, đã bỏ thông tin nội bộ) | Ánh xạ đúng mục, không mất luật cũ, không ghi đè khi chưa hỏi |
| T4 | `/bat-dau` → một việc nhỏ → `/ket-thuc` trên kết quả T1, máy không Codex | BACKLOG, nhánh, commit đúng; dừng ở câu đồng ý; có bằng chứng test |
| T6 | `/bat-dau` với việc rất nhỏ (sửa 1 dòng chữ), người dùng nói "ok làm đi" ngay lúc bàn | Vẫn dừng hỏi "Giao Codex / Claude tự code" trước khi sửa file nào. Chạy cả bản không có bước 7 (đối chứng) để thấy lỗi cũ |
| T5 | `/giao-codex` khi không có Codex | Báo một dòng, Claude tự làm, ghi BACKLOG |
| T0 | T1 nhưng **không có skill** (đối chứng) | Ghi lại khác biệt để biết skill có giá trị gì |

Mỗi lỗi tìm được → sửa skill → chạy lại kịch bản đó.

## 13. Bản 1 và bản sau

**Bản 1**: 4 skill, khung `mau/`, hợp đồng v1, README đầy đủ, kiểm thử T0-T5, công bố kho.

**Bản sau**: chế độ đưa vào main bằng PR (nhánh chính bị khóa, ai gộp, khi nào coi là xong); người dùng không nói tiếng Việt; `/don-dep` hằng tuần; hook khi mở phiên hiện tóm tắt BACKLOG hoặc "chưa khởi tạo";
đánh số việc theo người / module cho nhóm nhiều người; chuyển dự án gốc sang dùng plugin (bỏ 3 lệnh riêng trong `.claude/commands`).

## 14. Để người dựng tự quyết

- Câu chữ cụ thể trong SKILL.md và khung `mau/`, miễn giữ đúng nội dung các mục trên.
- Mô tả (`description`) của từng skill để Claude gọi đúng lúc.
- Chi tiết mẫu dò bí mật.
- Chỗ để bản chép dự án gốc cho T3 (thư mục nháp, không nằm trong kho plugin); bằng chứng test khi `## Lệnh` còn "chưa chốt"
  (ghi rõ "chưa có lệnh test" thay cho kết quả); tăng `version` trong `plugin.json` mỗi lần đổi để `/plugin marketplace update` nhận.
