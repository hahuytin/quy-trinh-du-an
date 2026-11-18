# quy-trinh-du-an

Plugin Claude Code giúp bạn **quản lý một dự án phần mềm cùng Claude theo từng buổi làm việc**, bằng tiếng Việt.
Bạn gõ `/khoi-tao` một lần để bàn định hướng và dựng bộ file của dự án. Sau đó, mỗi buổi bạn gõ `/bat-dau`, chọn ai code
(Codex hoặc Claude), làm, rồi gõ `/ket-thuc`. Mọi thứ đã bàn và đã chốt được ghi vào file trong kho, nên `/clear`, đổi máy
hay vài tháng sau quay lại cũng không mất gì.

Dành cho: người hiểu nghiệp vụ nhưng ít kinh nghiệm lập trình, muốn Claude làm phần kỹ thuật mà mình vẫn nắm và quyết được mọi việc.
Dùng được cho mọi dự án, mọi tài khoản Claude Code.

---

## Mục lục
0. [Từ ngữ cần biết](#0-từ-ngữ-cần-biết)
1. [Cài đặt](#1-cài-đặt)
2. [Luồng sử dụng](#2-luồng-sử-dụng) (có mục [Dự án đang làm dở](#dự-án-đang-làm-dở))
3. [Bốn lệnh](#3-bốn-lệnh)
4. [/khoi-tao chi tiết](#4-khoi-tao-chi-tiết)
5. [Một buổi làm việc mẫu](#5-một-buổi-làm-việc-mẫu)
6. [Các file trong dự án](#6-các-file-trong-dự-án)
7. [Hợp đồng CLAUDE.md](#7-hợp-đồng-claudemd)
8. [Câu đồng ý](#8-câu-đồng-ý)
9. [Codex](#9-codex)
10. [Câu hỏi thường gặp](#10-câu-hỏi-thường-gặp)
11. [Thay đổi theo phiên bản](#11-thay-đổi-theo-phiên-bản)

---

## 0. Từ ngữ cần biết

| Từ | Nghĩa trong plugin này |
|---|---|
| **Buổi làm việc** | Một lần ngồi làm, từ `/bat-dau` tới `/ket-thuc`. Thường là một phiên chat Claude |
| **Phiên** | Một cuộc chat với Claude. `/clear` xóa trí nhớ phiên, nhưng file trong kho vẫn còn |
| **Kho (repo)** | Thư mục dự án có `git` theo dõi lịch sử. `/khoi-tao` tự bật git nếu chưa có |
| **Commit** | Một lần lưu thay đổi vào lịch sử git, có lời ghi chú. Lùi lại được |
| **Nhánh (branch)** | Một bản làm việc riêng. Claude code trên nhánh `lam/<ngày>-<chủ-đề>`, bản chính (`main`) không bị đụng |
| **Gộp nhánh (merge) / đưa vào main** | Mang thay đổi từ nhánh làm việc vào bản chính. Chỉ làm khi bạn gõ câu đồng ý |
| **Remote / push** | Bản sao của kho trên mạng (thường là GitHub) / gửi commit lên đó. Claude chỉ push khi `CLAUDE.md` cho phép, và không tự tạo kho mới trên mạng |
| **CSDL, migration, seed** | Nơi lưu dữ liệu của app / lệnh đổi cấu trúc nơi lưu đó / lệnh nạp dữ liệu mẫu |
| **Deploy** | Đưa app lên máy chủ thật cho người dùng. Plugin không bao giờ làm việc này |
| **Module** | Một mảng nghiệp vụ của dự án (ví dụ "Kho", "Nhân sự"). Mỗi module có một **khóa** ngắn và một file ngữ cảnh |
| **Phiếu việc** | File mô tả một việc cho người code: mục tiêu, file được sửa, không làm gì, thế nào là xong |
| **Codex** | Công cụ code của OpenAI. Claude giao việc, Codex code, Claude kiểm |
| **Token / hạn mức** | Đơn vị tính chi phí dùng Claude / mức dùng tối đa trong một khoảng thời gian |
| **Subagent / "người đọc mới"** | Một Claude khác Claude gọi ra, không biết gì về cuộc bàn, để đọc soát với con mắt mới |
| **Soát độc lập / dò bí mật** | Nhờ một Claude khác tìm lỗi trong phần code mới / tìm mật khẩu, khóa lỡ ghi vào code |

Cách đọc lệnh: `<...>` là phần **bắt buộc** bạn thay bằng nội dung thật; `[...]` là phần **có thể bỏ trống**.
Ví dụ `/giao-codex <số việc>` → gõ `/giao-codex 3`; `/bat-dau [số việc]` → gõ `/bat-dau` hoặc `/bat-dau 3`.

---

## 1. Cài đặt

Cần: [Claude Code](https://claude.com/claude-code) (ứng dụng desktop hoặc dòng lệnh) và `git`. Codex không bắt buộc.
Không cần tài khoản GitHub để cài; cần khi bạn muốn sao lưu dự án lên GitHub.

Gõ trong ô chat của Claude Code:

```
/plugin marketplace add hahuytin/quy-trinh-du-an
```

```
/plugin install quy-trinh-du-an@quy-trinh-du-an
```

Khởi động lại phiên (ứng dụng desktop: mở phiên mới; dòng lệnh: thoát rồi chạy lại `claude`).
Gõ `/` là thấy các lệnh `/khoi-tao`, `/bat-dau`, `/giao-codex`, `/ket-thuc`.

**Cập nhật** khi có bản mới: chạy hai lệnh sau trong cửa sổ dòng lệnh (terminal), rồi khởi động lại phiên.

```bash
claude plugin marketplace update quy-trinh-du-an
```

```bash
claude plugin update quy-trinh-du-an@quy-trinh-du-an
```

**Gỡ:**

```
/plugin uninstall quy-trinh-du-an@quy-trinh-du-an
```

Plugin cài một lần trên máy, dùng cho mọi thư mục. Mỗi dự án chỉ cần chạy `/khoi-tao` một lần.

---

## 2. Luồng sử dụng

```
MỘT LẦN cho mỗi dự án (hoặc khi đổi hướng lớn):

  /khoi-tao ──► bàn từ A đến Z (5 chặng, từng câu một)
            ──► Claude viết bộ file ──► "người đọc mới" soát ──► bạn duyệt ──► commit
            ──► "Khởi tạo xong, gõ /bat-dau"

MỖI BUỔI làm việc:

  /bat-dau ──► Claude báo đang ở đâu ──► bạn nêu việc ──► bàn, chốt bảng việc
           ──► DỪNG: bạn chọn từng việc "Giao Codex" hay "Claude tự code"
           ──► làm (/giao-codex hoặc Claude code) trên nhánh riêng
  /ket-thuc ──► test, soát, dò bí mật ──► bạn bấm thử ──► "đồng ý đưa vào main"
           ──► cập nhật BACKLOG, context ──► /clear
```

**Quy tắc nhớ nhanh:** *đổi phần nền → `/khoi-tao`; thêm việc → `/bat-dau`.*

| Tình huống | Gõ lệnh |
|---|---|
| Dự án mới tinh, hoặc thư mục có code nhưng chưa có bộ file | `/khoi-tao` |
| Thêm tính năng trong định hướng đang có | `/bat-dau` |
| Thêm một module nhỏ | `/bat-dau` (Claude tự thêm vào bản đồ module) |
| Sang giai đoạn 2, đổi công nghệ, đổi mục tiêu, đổi luật làm việc | `/khoi-tao` (chế độ cập nhật) |
| Lần khởi tạo trước chưa xong | `/khoi-tao` (làm tiếp từ chỗ dở) |
| Quên đang làm gì | `/bat-dau` |

**Khi đang làm việc** (sau khi chốt việc ở `/bat-dau`), Claude chỉ dừng hỏi bạn ở các chỗ sau, còn lại làm một mạch:
- chọn ai code (luôn hỏi, kể cả việc nhỏ, kể cả máy không có Codex);
- duyệt kế hoạch, với việc lớn (nhiều phần, nhiều file, đổi dữ liệu);
- trước thao tác khó lùi (cần [câu đồng ý](#8-câu-đồng-ý));
- khi gặp điều phải đổi so với đã chốt (đổi người code, phải sửa `CLAUDE.md`, máy không có Codex...).

`/khoi-tao` và phần bàn của `/bat-dau` là hội thoại: Claude hỏi từng câu và đọc lại cho bạn sửa.

### Dự án đang làm dở

Cài plugin vào dự án đang làm thì **chưa có gì thay đổi**: các lệnh chỉ chạy khi bạn gõ.

| Dự án đang có | Gõ | Claude làm |
|---|---|---|
| Có code, chưa có bộ file quản lý | `/khoi-tao` | Đọc code trước (công nghệ, lệnh chạy, test), chỉ hỏi điều code không trả lời được, rồi viết bộ file |
| Đã có cách làm riêng (có `CLAUDE.md`, backlog, lệnh `/bat-dau` riêng...) | `/khoi-tao chuyển dự án sang plugin` | Ánh xạ nội dung cũ vào các mục chuẩn, giữ nguyên luật cũ, chỉ hỏi phần thiếu, không xóa gì khi chưa hỏi |

- Việc dở trong backlog giữ nguyên số, trạng thái và nhánh; làm tiếp bằng `/bat-dau <số việc>`.
- Nên chuyển khi không có việc đang code dở (ngay sau một lần `/ket-thuc`).
- Lệnh cũ trùng tên trong `.claude/commands/`: Claude hỏi bạn xóa hay giữ. Giữ thì gõ đủ `/quy-trinh-du-an:bat-dau` để dùng bản plugin.

---

## 3. Bốn lệnh

| Lệnh | Khi nào gọi | Claude làm gì | Để lại gì |
|---|---|---|---|
| `/khoi-tao [ý tưởng]` | Một lần đầu dự án; khi đổi nền | Bàn 5 chặng, viết bộ file, cho người đọc mới soát, chờ bạn duyệt, commit | `CLAUDE.md`, `docs/dinh-huong.md`, `docs/tasks/BACKLOG.md`, `docs/context/`, `AGENTS.md` (nếu dùng Codex) |
| `/bat-dau [số việc hoặc mô tả]` | Đầu mỗi buổi | Kiểm nền, báo hiện trạng, bàn việc, chốt bảng, **hỏi ai code**, ghi BACKLOG, viết phiếu, tạo nhánh, làm | Dòng BACKLOG, phiếu việc, nhánh `lam/<ngày>-<chủ-đề>`, các commit |
| `/giao-codex <số việc>` | Việc bạn đã chọn "Giao Codex" (thường `/bat-dau` tự gọi) | Kiểm Codex, ra lệnh Codex code theo phiếu, review, phản hồi tối đa 3 vòng, không đạt thì Claude làm tiếp | Code trên nhánh, nhật ký phiếu ghi số vòng và lỗi Codex |
| `/ket-thuc` | Cuối mỗi buổi | Chạy test có bằng chứng, soát độc lập, dò bí mật, chờ câu đồng ý rồi gộp nhánh, cập nhật BACKLOG / context, đề xuất luật mới cho Codex | BACKLOG cập nhật, quyết định ghi vào context, phiếu đã xong bị xóa |

Nếu dự án của bạn đã có lệnh riêng trùng tên (ví dụ `.claude/commands/bat-dau.md`), gõ đủ tên: `/quy-trinh-du-an:bat-dau`.

---

## 4. /khoi-tao chi tiết

Đây là bước quan trọng nhất: mọi buổi sau đều đọc bộ file tạo ra ở đây. Claude bàn chậm mà chắc, **không bịa**.

### Bốn chế độ (Claude tự nhận ra)

| Thư mục đang có | Chế độ | Claude làm |
|---|---|---|
| Chưa có bộ file | **Mới** | Đi đủ 5 chặng |
| Có `docs/khoi-tao-nhap.md` | **Làm tiếp** | Đọc nháp, làm tiếp từ chặng dở, không hỏi lại |
| `CLAUDE.md` đã theo plugin này | **Cập nhật** | Chỉ bàn phần bạn muốn đổi |
| Có `CLAUDE.md` / backlog kiểu khác | **Chuyển đổi** | Ánh xạ nội dung cũ sang mục mới, chỉ hỏi phần thiếu, không xóa gì khi chưa hỏi |

### Năm chặng bàn

Trước chặng 1 là chặng 0 "xem trước": Claude đọc thư mục (code, README, git, file có sẵn), kiểm máy có Codex không,
chọn chế độ, rồi báo ngắn đã thấy gì. Chặng 0 không hỏi bạn gì.

| Chặng | Bàn gì | Ghi vào |
|---|---|---|
| 1. Tầm nhìn | Vấn đề đang gặp, ai dùng, vì sao bây giờ, thành công trông ra sao, điều làm dự án thất bại dù app vẫn chạy | `docs/dinh-huong.md` |
| 2. Phạm vi | Giai đoạn 1 tối thiểu, việc để sau, điều không làm; chia thành việc | `dinh-huong.md`, BACKLOG |
| 3. Kỹ thuật | Ràng buộc (ai bảo trì, máy chủ, ngân sách, dữ liệu nhạy cảm), rồi 2-3 phương án công nghệ kèm đề xuất; lệnh chạy / test | `CLAUDE.md` `## Stack`, `## Lệnh` |
| 4. Cấu trúc | Chia module theo nghiệp vụ, đặt tên, dữ liệu chính | `## Module`, `docs/context/` |
| 5. Cách làm việc | Có dùng Codex không, kho GitHub, câu đồng ý, ai đưa lên server, điều cấm, nguyên tắc không đổi | `## Git`, `## Câu đồng ý`, `## Codex`, `## Cấm`, `## Nguyên tắc`, `AGENTS.md` |

Trong mỗi chặng:
- **Một câu hỏi mỗi lượt.** Chỗ cần chọn thì Claude đưa 2-3 phương án, nói được gì mất gì, phương án nào Claude chọn.
- Bạn nói **"Claude tự chọn"**: Claude chọn và ghi rõ là *lựa chọn của Claude* để bạn sửa sau.
- Chỗ chưa ai trả lời được: ghi `[CẦN LÀM RÕ] ... - người quyết: ...`, không điền bừa.
- Hết chặng: Claude **đọc lại** điều đã hiểu, bạn sửa rồi mới sang chặng sau.
- Hết chặng: Claude ghi **nháp** `docs/khoi-tao-nhap.md`. Phiên đầy cũng không mất; gõ lại `/khoi-tao` là làm tiếp.

### Khi nào khởi tạo xong

Claude chỉ báo xong khi đủ 6 điều:
1. Mọi mục bắt buộc của `CLAUDE.md` có nội dung, hoặc ghi "chưa chốt" kèm người quyết.
2. BACKLOG có các việc giai đoạn 1, đánh số.
3. Có `docs/dinh-huong.md` và file context.
4. Một **"người đọc mới"** (một Claude khác, không biết gì về cuộc bàn) đọc bộ file và không còn câu hỏi quan trọng.
   Câu nào nó hỏi mà chỉ bạn trả lời được, Claude gom lại hỏi bạn **một lần**.
5. Bạn nói duyệt.
6. Bộ file đã commit vào git.

Cuối cùng Claude nói: "Gõ `/bat-dau` để làm việc số 1: ...".

---

## 5. Một buổi làm việc mẫu

```
Bạn:    /bat-dau
Claude: Đang ở nhánh main, sạch. Việc dở: không. Chưa làm: 3 (Phiếu nhập kho), 4 (Báo cáo tồn).
        Buổi này làm gì?
Bạn:    làm việc 3
Claude: (đọc docs/context/kho.md) Phiếu nhập có cho nhập vật tư chưa có trong danh mục không?
Bạn:    không, phải tạo danh mục trước
Claude: Chốt: việc 3 - Phiếu nhập kho (chỉ chọn vật tư có trong danh mục). Đề xuất: Giao Codex
        (5 file kiểu form + danh sách lặp lại). Không cần câu đồng ý.
        [Giao Codex (đề xuất)]  [Claude tự code]
Bạn:    (bấm) Giao Codex
Claude: Đã ghi BACKLOG, viết phiếu docs/tasks/3-phieu-nhap.md, tạo nhánh lam/2026-11-20-phieu-nhap.
        Đang giao Codex... Codex xong vòng 2. Test 12/12 đạt. Gõ /ket-thuc khi muốn chốt buổi.
Bạn:    /ket-thuc
Claude: Test cả module 40/40 đạt. Không thấy bí mật trong thay đổi. Đã thêm màn hình Phiếu nhập.
        Bạn bấm thử trên máy; thấy ổn thì gõ "đồng ý đưa vào main".
Bạn:    đồng ý đưa vào main
Claude: Đã gộp, cập nhật BACKLOG (việc 3 → Xong), ghi quyết định vào docs/context/kho.md.
        Buổi sau: /bat-dau 4. Gõ /clear để dọn phiên.
```

---

## 6. Các file trong dự án

| File | Chứa gì | Ai viết | Khi nào đổi |
|---|---|---|---|
| `CLAUDE.md` | Luật và lệnh của dự án theo [hợp đồng](#7-hợp-đồng-claudemd). Ngắn, vì Claude đọc nó ở mọi phiên | `/khoi-tao`; `/bat-dau` thêm dòng module mới hoặc sửa lệnh sai (bạn thấy và đồng ý trước) | Khi đổi nền, thêm module |
| `AGENTS.md` | Luật cho Codex: chỉ sửa file trong phiếu, không làm thừa, không chạy git ghi, mẫu báo cáo | `/khoi-tao`; `/ket-thuc` đề xuất thêm luật | Khi Codex lặp lỗi |
| `docs/dinh-huong.md` | Một trang: vấn đề, người dùng, thành công, thất bại, giai đoạn, không làm, **quyết định lớn toàn dự án** kèm lý do | `/khoi-tao`; `/ket-thuc` thêm quyết định lớn | Khi đổi giai đoạn / mục tiêu / công nghệ |
| `docs/tasks/BACKLOG.md` | Mỗi việc một dòng: số, module, việc, trạng thái, ai code, nhánh, file dùng chung; "Số kế tiếp" | `/khoi-tao` tạo; `/bat-dau`, `/ket-thuc` cập nhật | Mỗi buổi |
| `docs/context/<module>.md` | Quyết định đã chốt kèm lý do, hướng đã bác, câu cần làm rõ, việc dở | `/khoi-tao` tạo; `/ket-thuc` cập nhật | Khi có quyết định mới |
| `docs/tasks/<số>-<slug>.md` | **Phiếu việc**: mục tiêu, file được sửa, file mẫu, không làm, tiêu chí xong, nhật ký. Xong thì xóa | `/bat-dau` | Việc giao Codex hoặc kéo dài hơn 1 buổi |
| `docs/khoi-tao-nhap.md` | Nháp khi khởi tạo chưa xong. Tự xóa khi xong | `/khoi-tao` | Tạm thời |
| `.codex-out/` | Báo cáo của Codex. Không commit (`/khoi-tao` thêm vào `.gitignore`) | `/giao-codex` | Mỗi lần giao |

Muốn biết dự án đang ở đâu mà không mở Claude: đọc `docs/tasks/BACKLOG.md`.
Muốn biết vì sao một điều được quyết như vậy:
- quyết định **toàn dự án** (công nghệ, phạm vi, điều không làm - ví dụ "không làm phần kế toán"): `docs/dinh-huong.md`, mục "Không làm" và "Quyết định lớn";
- quyết định **của một module** (cách tính, quy tắc nghiệp vụ, hướng đã bác): `docs/context/<module>.md`.

---

## 7. Hợp đồng CLAUDE.md

Plugin không chứa luật riêng của dự án nào. Các lệnh đọc luật từ `CLAUDE.md` của dự án, theo **các mục có tên cố định**.
Dòng đầu file là `<!-- quy-trinh-du-an: v1 -->` để lệnh biết file theo hợp đồng nào. Đừng đổi tên các mục này.

| Mục | Chứa gì |
|---|---|
| `## Dự án` | 1-3 câu dự án là gì, cho ai; trỏ tới `docs/dinh-huong.md` |
| `## Nguyên tắc` | 3-6 điều không đổi suốt dự án |
| `## Stack` | Ngôn ngữ, framework, CSDL, đường dẫn công cụ đặc biệt |
| `## Lệnh` | Chạy local, test một file, test cả module, build, kiểm cú pháp |
| `## Module` | Bảng module: tên, khóa, file ngữ cảnh, ghi chú |
| `## Git` | Nhánh chính, remote, push sau commit không, mẫu commit message |
| `## Câu đồng ý` | Câu bắt buộc trước thao tác khó lùi |
| `## Codex` | Có dùng không; gợi ý việc nào nên giao |
| `## Cấm` | Điều tuyệt đối không làm |
| `## Quy trình` | Ba dòng nhắc cách làm việc theo plugin |

Bạn được thêm mục riêng ở cuối (ví dụ "Cách đặt tên"). Mục chưa có câu trả lời ghi "chưa chốt - người quyết: ...".
Thiếu mục nào, `/bat-dau` sẽ báo và gợi ý chạy `/khoi-tao`.

---

## 8. Câu đồng ý

Có những thao tác khó hoặc không thể lùi lại. Claude chỉ làm khi bạn gõ **đúng câu**:

| Thao tác | Câu mặc định |
|---|---|
| Gộp nhánh làm việc vào nhánh chính | **"đồng ý đưa vào main"** |
| Lệnh ghi cơ sở dữ liệu trên máy (migration, seed, xóa dữ liệu) | **"đồng ý chạy"** |

Vì sao cần câu cố định: "ok", "được" dễ bị hiểu nhầm là đồng ý cho việc khác. Một câu riêng cho mỗi loại thao tác thì không nhầm được.
Gõ đúng chữ, có dấu, không cần dấu ngoặc kép; viết hoa hay thường đều được. Claude sẽ nhắc đúng câu khi cần.
Bạn đổi hoặc thêm câu trong mục `## Câu đồng ý` của `CLAUDE.md`.

Trước khi xin câu đồng ý gộp nhánh, `/ket-thuc` chỉ bạn cách **thử trên máy**: mở trang nào, bấm gì, thấy gì là đúng.
Thay đổi không có gì hiện trên màn hình thì Claude nói rõ và đưa lệnh test để bạn xem kết quả.

Plugin **không bao giờ** tự đưa code lên server (deploy). Việc đó bạn tự làm.

---

## 9. Codex

[Codex](https://developers.openai.com/codex) là công cụ code của OpenAI. Dùng nó để code các việc nhiều file kiểu lặp lại giúp
tiết kiệm token của Claude: Claude bàn, viết phiếu, ra lệnh, review, chạy test và commit; Codex chỉ code theo phiếu.

- **Cài**: theo hướng dẫn của OpenAI (cần tài khoản OpenAI), rồi kiểm bằng lệnh `codex --version` gõ trong **cửa sổ dòng lệnh**
  (không phải ô chat Claude). Lần đầu Claude gọi Codex, Claude Code sẽ hỏi quyền: chọn cho phép.
- **Không có Codex**: mọi thứ vẫn chạy. Claude tự kiểm, không đưa lựa chọn "Giao Codex"; vẫn dừng hỏi "Claude code việc này nhé?"
  với hai nút "Claude tự code" / "Để sau". Nếu bạn đã chọn Codex mà Codex hỏng, Claude dừng hỏi lại, không tự chuyển.
- **Giao hay không** là quyền của bạn: sau mỗi `/bat-dau`, Claude luôn hỏi từng việc. Gợi ý của Claude dựa trên mục `## Codex`
  (thường: việc nhiều file lặp lại thì giao Codex; việc 1-3 file hoặc phần lõi khó thì Claude làm).
- Codex tối đa **3 vòng** phản hồi. Hết hạn mức, lặp lỗi, hoặc sửa ngoài phạm vi thì Claude làm tiếp trên phần Codex đã làm.
- Codex mắc cùng một loại lỗi từ 2 lần, `/ket-thuc` đề xuất thêm một câu luật vào `AGENTS.md`; bạn đồng ý mới ghi.

---

## 10. Câu hỏi thường gặp

**Tôi quên dự án đang ở đâu.** Gõ `/bat-dau`, Claude đọc BACKLOG và git rồi báo. Hoặc tự mở `docs/tasks/BACKLOG.md`.

**Mỗi lệnh có phải gọi `/khoi-tao` trước không?** Không. `/khoi-tao` chỉ chạy một lần đầu dự án và khi đổi nền.
`/bat-dau` tự kiểm; thiếu gì nó nhắc.

**Muốn đổi công nghệ / sang giai đoạn mới.** Gõ `/khoi-tao <điều muốn đổi>`. Claude chỉ bàn phần đổi, cập nhật tài liệu,
ghi lý do vào `docs/dinh-huong.md` mục "Quyết định lớn". Việc chuyển code cũ sang công nghệ mới được thêm thành các dòng backlog,
làm dần bằng `/bat-dau`.

**Bàn xong Claude code luôn mà không hỏi.** Không đúng quy trình: sau khi chốt việc, Claude phải dừng hỏi ai code. Nhắc Claude
"dừng, hỏi tôi ai code". Phần đã sửa nằm trên nhánh làm việc, chưa vào `main`, nên bạn vẫn quyết được giữ hay bỏ.
Báo lỗi tại <https://github.com/hahuytin/quy-trinh-du-an/issues>.

**Lệnh trùng tên với lệnh có sẵn của dự án.** Gõ đủ `/quy-trinh-du-an:bat-dau`, hoặc xóa lệnh cũ trong `.claude/commands/`
(`/khoi-tao` chế độ chuyển đổi sẽ hỏi bạn việc này).

**Tôi cài thêm plugin superpowers thì sao?** Dùng chung được. Skill của superpowers hay ghi thiết kế vào `docs/superpowers/`;
với dự án theo plugin này, thiết kế của mỗi việc nằm trong phiếu `docs/tasks/<số>-<slug>.md`.

**Phiên đầy (Claude chậm, quên).** Gõ `/context` để xem phiên đã dùng bao nhiêu; quá khoảng 60% mà còn việc thì
`/ket-thuc` → `/clear` → `/bat-dau`. `/ket-thuc` không bắt buộc việc phải xong: việc chưa xong được ghi "Đang làm" kèm bước tiếp,
nhánh để nguyên, buổi sau làm tiếp. Mọi thứ đã nằm trong file nên không mất.

**Nhiều người cùng làm một dự án.** Bản 1 dành cho một người làm chính. Nhiều người thì mỗi người kéo code mới (`git pull`) trước
`/bat-dau`, và tránh hai người sửa BACKLOG cùng lúc. Bản sau sẽ hỗ trợ tốt hơn.

**Dự án dùng pull request (PR) để duyệt code.** Bản 1 chỉ gộp nhánh tại máy. Chế độ PR có trong kế hoạch bản sau.

**Claude có tự đẩy code lên server không?** Không. Plugin cấm deploy; bạn tự làm.

---

## 11. Thay đổi theo phiên bản

**1.0.0** (2026-11): bản đầu. 4 lệnh, khung file, hợp đồng `CLAUDE.md` v1, điểm dừng bắt buộc "chọn ai code".
Đã kiểm thử bằng các kịch bản: khởi tạo thư mục trống, dự án có sẵn code, chuyển đổi dự án kiểu cũ; một buổi `/bat-dau` → `/ket-thuc`;
không có Codex; người dùng nói "làm luôn" khi chưa chọn ai code.

**Dự kiến bản sau:** chế độ PR; lệnh `/don-dep` hằng tuần (xóa nhánh đã gộp, rút gọn BACKLOG); tự hiện tóm tắt BACKLOG khi mở phiên;
hỗ trợ nhiều người cùng dự án.

Thiết kế đầy đủ: [docs/thiet-ke.md](docs/thiet-ke.md).

## Giấy phép

MIT - xem [LICENSE](LICENSE).
