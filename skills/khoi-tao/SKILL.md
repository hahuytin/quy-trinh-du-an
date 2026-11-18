---
name: khoi-tao
description: Dùng khi người dùng mở một dự án mới, muốn đổi nền của dự án (sang giai đoạn mới, đổi stack, đổi mục tiêu hay luật làm việc), làm tiếp lần khởi tạo còn dở, hoặc chuyển một dự án có sẵn sang quy trình quy-trinh-du-an.
argument-hint: "[ý tưởng dự án | điều muốn thay đổi] (bỏ trống = Claude xem thư mục rồi hỏi)"
disable-model-invocation: true
---

# Khởi tạo dự án

Người dùng gõ kèm: $ARGUMENTS

Bạn đang đặt **nền** cho cả dự án. Mọi buổi làm việc sau (`/bat-dau`, `/giao-codex`, `/ket-thuc`) đều đọc bộ file
bạn viết ở đây; nền sai thì buổi nào cũng sai theo. Vì vậy bước này **chậm mà đúng**: bàn kỹ, không bịa, không vội viết file.

Người dùng thường hiểu nghiệp vụ nhưng ít kinh nghiệm lập trình. Nói tiếng Việt dễ hiểu; gặp thuật ngữ thì giải thích
bằng vai trò của nó trong dự án này (ví dụ: "CSDL - nơi lưu dữ liệu nhập vào app").

**Đích đến:** một bộ file đủ để một người (hoặc một Claude) mới vào dự án đọc xong là bắt đầu được việc số 1,
không phải hỏi lại điều quan trọng nào.

## Chặng 0: Xem trước, chọn chế độ

1. Đọc thư mục trước khi hỏi bất cứ gì: danh sách file, `README*`, file cấu hình gói (`package.json`, `composer.json`,
   `pyproject.toml`, `go.mod`...), `CLAUDE.md`, `AGENTS.md`, `docs/`, `.claude/commands/`, `git status`, `git log --oneline -10`.
2. Chạy `codex --version` (qua Bash) để biết máy có Codex không. Ghi nhớ kết quả cho chặng 5.
3. Chọn chế độ:

| Thấy gì | Chế độ | Làm gì |
|---|---|---|
| Có `docs/khoi-tao-nhap.md` | **Làm tiếp** | Đọc nháp, báo đã xong chặng nào, làm tiếp từ chặng dở. Không hỏi lại điều đã có trong nháp |
| `CLAUDE.md` có dòng `<!-- quy-trinh-du-an: v1 -->` | **Cập nhật** | Xem mục "Chế độ cập nhật" cuối file |
| Có `CLAUDE.md` / BACKLOG / context kiểu khác (không có dấu phiên bản) | **Chuyển đổi** | Đi đủ 5 chặng nhưng chỉ hỏi phần còn thiếu; nội dung cũ được ánh xạ sang mục mới, không bỏ |
| Không có các file trên | **Mới** | Đi đủ 5 chặng |

4. Báo người dùng ngắn (≤ 6 dòng): đã thấy gì trong thư mục, chế độ nào, sẽ đi qua những chặng nào.
   Rồi hỏi luôn câu đầu tiên của chặng 1 (hoặc chặng đang dở).

Dự án đã có code: những gì code trả lời được (stack, lệnh chạy, cấu trúc thư mục) thì **đọc ra và đọc lại cho người dùng xác nhận**,
không hỏi như thể chưa biết.

## Cách bàn (dùng cho mọi chặng)

- **Một chủ đề mỗi lượt.** Hỏi xong thì dừng lượt, chờ trả lời. Được gộp các chi tiết của cùng một chủ đề
  ("ai dùng, khoảng bao nhiêu người, trên máy tính hay điện thoại"); không gộp hai chủ đề khác nhau vào một lượt.
  Nhiều chủ đề một lượt làm người dùng trả lời qua loa.
- **Chưa rõ ý thì hỏi mở**, không đưa sẵn đáp án để chọn: lời của chính người dùng mang ý định mà phương án của bạn không có.
  Được kèm một ví dụ ngắn để người dùng hiểu câu hỏi hỏi gì.
- **Khi lựa chọn đặt tên được** (stack, chia module, cách lưu file...): đưa 2-3 phương án cụ thể, mỗi phương án được gì, mất gì,
  phương án bạn chọn và vì sao. Có công cụ hỏi lựa chọn thì dùng để người dùng bấm, phương án đề xuất đặt đầu.
- **Câu hỏi bám vào câu trả lời trước** hoặc điều bạn đọc được, giúp người dùng nghĩ tới hệ quả họ chưa tính
  ("Kho có 2 thủ kho cùng nhập. Hai người sửa cùng một phiếu thì sao?").
- **Người dùng nói "Claude tự chọn" / "tùy bạn"**: chọn, nói một dòng lý do, ghi là *lựa chọn của Claude* để họ còn sửa.
  Ngoại lệ: điều là **sự thật bên ngoài** hoặc **người khác quyết** (máy chủ có gì, mạng công ty cho truy cập từ ngoài không,
  ai có quyền gì, quy định công ty) thì không được tự chọn: ghi `[CẦN LÀM RÕ]` kèm người quyết.
- **Chỗ chưa ai trả lời được**: ghi `[CẦN LÀM RÕ] <câu hỏi> - người quyết: <ai> - khi nào: <mốc>`. Không bịa cho đủ mục.
- **Người dùng muốn bỏ qua chặng** ("để sau"): ghi "chưa chốt" kèm người quyết, sang chặng sau.
- **Hết mỗi chặng: đọc lại** điều đã hiểu (≤ 200 chữ, không tính bảng danh sách việc / dữ liệu). Phần người dùng nói ghi là của họ;
  phần bạn điền ghi là đề xuất của bạn. Hỏi "sai hay thiếu gì không?". Người dùng xác nhận mới sang chặng sau.
  Chế độ chuyển đổi / cập nhật: các chặng không có gì đổi được gộp vào một lần đọc lại.
- **Hết mỗi chặng: ghi nháp** vào `docs/khoi-tao-nhap.md` (tạo nếu chưa có): chặng đã xong, quyết định, lựa chọn của Claude,
  `[CẦN LÀM RÕ]`. Phiên có đầy hay bị `/clear` thì lần sau làm tiếp được.

Mỗi chặng dưới đây là **vùng cần phủ**, không phải kịch bản đọc to. Bỏ câu mà thư mục hoặc người dùng đã trả lời.

## Chặng 1: Tầm nhìn → `docs/dinh-huong.md`

Cần hiểu được: vấn đề đang gặp là gì và đang giải quyết bằng cách nào (Excel, giấy, app khác); ai dùng, bao nhiêu người,
dùng trên máy tính hay điện thoại; vì sao làm bây giờ; **thành công trông ra sao** (đo được thì tốt: "thủ kho tra tồn trong 10 giây");
**điều gì làm dự án thất bại dù app vẫn chạy** (ví dụ: "nhập lâu hơn ghi sổ thì không ai dùng"); đã thấy sản phẩm nào thích / ghét.

Câu mở đầu nên là câu hỏi về **vấn đề**, không phải về tính năng.

## Chặng 2: Phạm vi → `docs/dinh-huong.md` + BACKLOG

Cần chốt: **giai đoạn 1 tối thiểu** - những gì phải có để người dùng thật dùng được, không hơn; việc để giai đoạn sau;
**điều không làm** (nói rõ để sau này không ai làm thừa).

Chia giai đoạn 1 thành việc **nghiệp vụ**: mỗi việc 1 dòng, làm xong trong 1-2 buổi, có kết quả nhìn thấy được. Đọc lại danh sách việc.
Việc kỹ thuật (dựng khung code, đăng nhập...) chưa biết được ở đây: thêm sau chặng 3-4. Danh sách việc chốt lần cuối khi viết bộ file.

Bạn muốn thêm việc người dùng chưa nêu (nhập dữ liệu cũ, thẻ kho...): chỉ đưa vào giai đoạn 1 khi thiếu nó thì giai đoạn 1
không dùng được; ghi rõ là *đề xuất thêm của Claude*. Còn lại đưa vào "Giai đoạn sau".

## Chặng 3: Kỹ thuật → `CLAUDE.md` mục `## Stack`, `## Lệnh`

Hỏi **ràng buộc trước**, phương án sau: ai sẽ bảo trì; đã có máy chủ / hosting / CSDL nào của công ty chưa; ngân sách;
dữ liệu có nhạy cảm không; có cần chạy khi mất mạng không; cần đăng nhập không, dùng tài khoản ở đâu.

Rồi đưa 2-3 phương án stack hợp ràng buộc, kèm đánh đổi và phương án bạn chọn. Dự án đã có code: stack là cái đang dùng,
chỉ hỏi nếu người dùng muốn đổi.

`## Lệnh` cần: chạy local, test một file, test cả module / dự án, build (nếu có), kiểm cú pháp. Chốt stack xong thì **kiểm công cụ
trên máy** (`php --version`, `node --version`, `python --version`, `where <lệnh>`...): máy có nhiều bản hoặc lệnh trên PATH sai bản
thì ghi đủ đường dẫn bản đúng. Có code thì chạy thử các lệnh vừa ghi. Chưa có code thì ghi lệnh dự kiến, đánh dấu "kiểm lại sau việc đầu tiên".

## Chặng 4: Cấu trúc → `CLAUDE.md` mục `## Module` + `docs/context/`

Cần chốt: chia module theo **nghiệp vụ** (dự án nhỏ: một module `tong-quan` là đủ, đừng chia cho có); khóa ngắn của mỗi module;
cách đặt tên (thư mục code, bảng, đường dẫn trang) nếu stack cần; **dữ liệu chính** nói bằng lời (ví dụ: "phiếu nhập có nhiều dòng vật tư,
mỗi vật tư thuộc một kho"); module nào dùng chung dữ liệu của module nào.

## Chặng 5: Cách làm việc → `## Git`, `## Câu đồng ý`, `## Codex`, `## Cấm`, `## Nguyên tắc`, `AGENTS.md`

Cần chốt:
- **Codex**: máy có Codex không (kết quả chặng 0). Có thì có dùng không; người dùng nói không dùng thì ghi "Không dùng",
  dù máy có cài. Dùng thì giữ mặc định trong khung `mau/CLAUDE.md`
  (giao việc nhiều file kiểu lặp lại; Claude tự làm việc 1-3 file và phần lõi khó) trừ khi người dùng muốn khác.
  Nói rõ: sau mỗi `/bat-dau`, Claude luôn dừng hỏi từng việc "giao Codex hay Claude tự code".
- **Git**: nhánh chính (mặc định `main`); đã có kho từ xa (remote) chưa, ở đâu, công khai hay riêng tư; push sau mỗi commit không;
  mẫu commit message. Bản này chỉ đưa vào nhánh chính bằng merge tại máy.
- **Câu đồng ý**: mặc định "đồng ý đưa vào main" (gộp nhánh) và "đồng ý chạy" (lệnh ghi CSDL như migration / seed).
  Hỏi có thao tác khó lùi nào khác cần câu riêng không (xóa dữ liệu, gửi email thật...).
- **Ai đưa lên server**: mặc định người dùng tự làm, Claude không làm.
- **Điều cấm**: giữ danh sách mặc định trong khung, hỏi thêm điều riêng của dự án.
- **Nguyên tắc không đổi**: 3-6 điều xuyên suốt dự án (ví dụ: "ít file, ít tầng", "chữ hiển thị tiếng Việt có dấu").
  Đề xuất từ những gì đã bàn, người dùng sửa.

## Viết bộ file

Chỉ viết sau khi đã đọc lại xong chặng 5. Dùng khung trong thư mục `mau/` cạnh file này, điền nội dung đã bàn,
**xóa mọi dòng hướng dẫn `<...>` của khung**.

| File | Khung | Ghi chú |
|---|---|---|
| `CLAUDE.md` | `mau/CLAUDE.md` | Giữ dòng `<!-- quy-trinh-du-an: v1 -->` ở đầu và đủ 10 mục tên cố định. Ngắn (≤ ~120 dòng) vì nạp vào mọi phiên |
| `docs/dinh-huong.md` | `mau/dinh-huong.md` | ≤ 1 trang |
| `docs/tasks/BACKLOG.md` | `mau/BACKLOG.md` | Việc giai đoạn 1 từ số 1, trạng thái `Chưa làm`; "Số kế tiếp" đúng |
| `docs/context/<khóa>.md` | `mau/context.md` | Một file cho mỗi module; quyết định đã chốt kèm lý do, hướng đã bác |
| `AGENTS.md` | `mau/AGENTS.md` | Chỉ khi dùng Codex. Mục "Bắt buộc theo dự án" lấy từ `## Nguyên tắc`, `## Stack`, cách đặt tên |
| `.gitignore` | - | Thêm dòng còn thiếu, không lặp: `.env`; dùng Codex thì thêm `.codex-out/` |

**Chế độ chuyển đổi:**
- Ánh xạ từng phần của file cũ vào mục tên cố định; phần không khớp mục nào thì giữ thành mục riêng ở cuối `CLAUDE.md`.
  Không xóa nội dung cũ nào khi chưa hỏi.
- File context cũ giữ nguyên tên và nội dung; cột "Ngữ cảnh" của `## Module` trỏ đúng file đó (các lệnh đọc theo cột này,
  không đoán tên theo khóa). Chỉ thêm mục "Hướng đã bác" nếu file chưa có. Không sắp xếp lại theo khung.
- BACKLOG cũ giữ nguyên số việc, "Số kế tiếp", nhánh; chỉ thêm cột "Ai code" (`-`) nếu chưa có. Không đánh số lại từ 1.
- Dự án có lệnh riêng trùng tên (`.claude/commands/bat-dau.md`, `giao-codex.md`, `ket-thuc.md`): hỏi người dùng có xóa để dùng
  bản plugin không (nếu giữ, họ phải gõ `/quy-trinh-du-an:bat-dau`).

## Kiểm "người đọc mới"

Gọi một subagent mới (không có ngữ cảnh cuộc bàn) với nội dung:

> Bạn là lập trình viên giỏi vừa được giao dự án ở `<thư mục>`. Không hỏi được người viết. Đọc `CLAUDE.md`, `docs/dinh-huong.md`,
> `docs/tasks/BACKLOG.md`, `docs/context/`, `AGENTS.md` (nếu có); không đọc `docs/khoi-tao-nhap.md` và các file ghi chép hội thoại.
> Liệt kê những câu bạn còn phải hỏi trước khi bắt đầu việc <số và tên việc nên làm đầu tiên>:
> chỗ phải đoán mà đoán sai thì hỏng. Mỗi câu: câu hỏi + vì sao quan trọng. Xếp theo mức quan trọng. Bỏ qua chỗ bạn tự quyết hợp lý được
> và chỗ ghi "chưa chốt" / `[CẦN LÀM RÕ]` đã có người quyết. Không còn gì quan trọng thì nói "không còn gì quan trọng". Không sửa file.

Chia câu trả lời làm ba loại:
- **Đã bàn rồi**: bổ sung câu trả lời vào file.
- **Nhỏ** (người dựng hợp lý tự quyết được): tự quyết, ghi là lựa chọn của Claude.
- **Của người dùng** (câu trả lời đổi thứ sẽ làm): gom lại, hỏi trong **một tin nhắn** đánh số, quan trọng nhất trước
  (đây là ngoại lệ duy nhất của "một chủ đề mỗi lượt", để người dùng không bị hỏi lại nhiều lượt). Cập nhật file theo câu trả lời.
- **Người khác quyết** (bộ phận IT, cấp trên, quy định công ty): ghi `[CẦN LÀM RÕ]` kèm người quyết và mốc cần có câu trả lời.

Không có công cụ subagent: tự đọc lại bộ file với vai người mới, làm như trên.

## Duyệt và commit

Gửi người dùng bảng tóm tắt: mỗi file đã viết chứa gì (1 dòng); các **lựa chọn của Claude** xếp theo mức ảnh hưởng
(5 điều quan trọng nhất trước, nói rõ "đây là chỗ bạn nên đọc kỹ", nhất là khi bạn đã tự chọn nhiều); các mục "chưa chốt" /
`[CẦN LÀM RÕ]` còn lại kèm người quyết. Hỏi duyệt. Người dùng sửa thì sửa rồi hỏi lại.

Người dùng duyệt rồi:
1. Chưa có git: `git init -b main` (theo nhánh chính đã chốt).
2. Xóa `docs/khoi-tao-nhap.md`.
3. Commit bộ file. Message: `Khoi tao du an: <tên>` (chế độ chuyển đổi: `Chuyen sang quy trinh quy-trinh-du-an`;
   cập nhật: `Cap nhat nen du an: <điều đổi>`), trừ khi người dùng muốn kiểu khác.
4. Có remote và `## Git` bảo push thì push. **Tạo kho mới trên GitHub là việc ra bên ngoài**: chỉ làm khi người dùng bảo,
   mặc định riêng tư, hỏi lại trước khi tạo.

## Điều kiện "khởi tạo xong"

Chỉ báo xong khi đủ cả 6 điều:
1. Mọi mục bắt buộc của `CLAUDE.md` có nội dung, hoặc ghi "chưa chốt" kèm người quyết.
2. BACKLOG có việc giai đoạn 1 (hoặc việc cũ giữ nguyên khi chuyển đổi), đánh số, "Số kế tiếp" đúng.
3. Có `docs/dinh-huong.md` và ít nhất một file trong `docs/context/`.
4. Đã qua bước "người đọc mới" và xử lý xong câu hỏi của nó.
5. Người dùng nói duyệt.
6. Bộ file đã commit, nháp đã xóa.

Thông báo cuối: đã chốt gì (3-5 dòng), còn gì chưa chốt và ai quyết, rồi "Gõ `/bat-dau <số>` để làm việc <số>: <tên việc>"
với việc nên làm đầu tiên (dự án mới: việc số 1; chuyển đổi: việc dở hoặc việc chưa làm nên làm tiếp, không phải việc tạm dừng).

## Chế độ cập nhật (dự án đã khởi tạo, muốn đổi nền)

1. Đọc toàn bộ `CLAUDE.md`, `docs/dinh-huong.md`, BACKLOG, `docs/context/`.
2. Hỏi người dùng muốn đổi gì (nếu `$ARGUMENTS` chưa nói), xác định chặng bị ảnh hưởng. Chỉ bàn các chặng đó, cùng "Cách bàn" ở trên.
   - Sang giai đoạn mới: chặng 2 (và chặng 3-4 nếu cần). Thêm mục giai đoạn mới vào `dinh-huong.md`, thêm việc mới vào BACKLOG từ "Số kế tiếp".
   - Đổi stack: chặng 3; cập nhật `## Stack`, `## Lệnh`, `AGENTS.md`. Việc chuyển code cũ sang stack mới là việc code:
     thêm thành dòng BACKLOG, không làm trong `/khoi-tao`.
   - Đổi luật làm việc: chặng 5.
3. Không viết lại phần không đổi. Ghi quyết định mới kèm lý do: toàn dự án → `docs/dinh-huong.md` mục "Quyết định lớn";
   của một module → `docs/context/<khóa>.md`.
4. Kiểm "người đọc mới" (chỉ với phần đổi), duyệt, commit.

Thêm một module nhỏ trong định hướng đang có **không cần** chế độ này: `/bat-dau` tự làm.

## Dấu hiệu đang làm sai

| Ý nghĩ | Thực tế |
|---|---|
| "Hỏi gộp 4 câu cho nhanh" | Người dùng trả lời qua loa câu 3-4. Một câu mỗi lượt |
| "Ý tưởng rõ rồi, viết file luôn" | File viết trước khi bàn xong chặng 5 là file đoán. Đi đủ chặng |
| "Stack này hiển nhiên, khỏi hỏi" | Ghi là lựa chọn của Claude và nói ra; người dùng không sửa được điều họ không nghe thấy |
| "Mục này chưa ai nói, điền tạm cho đủ" | Ghi "chưa chốt" hoặc `[CẦN LÀM RÕ]`. Điền tạm là bịa |
| "Bộ file rõ rồi, bỏ bước người đọc mới" | Rõ với người đã bàn ≠ rõ với người mới. Luôn chạy |
| "CLAUDE.md cũ lộn xộn, viết lại từ đầu" | Ánh xạ, không bỏ. Hỏi trước khi xóa bất cứ dòng nào |
| "Người dùng biết thuật ngữ này" | Không chắc thì giải thích bằng vai trò trong dự án |
