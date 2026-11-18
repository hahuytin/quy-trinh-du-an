<!-- quy-trinh-du-an: v1 -->
# <Tên dự án>: hướng dẫn cho Claude

<Dòng `<...>` là hướng dẫn của khung: điền nội dung rồi xóa. Giữ nguyên dòng đầu và tên 10 mục `##` bên dưới:
các lệnh /bat-dau, /giao-codex, /ket-thuc đọc đúng các tên mục này. Mục chưa có câu trả lời: ghi "chưa chốt - người quyết: ...".
Có thể thêm mục riêng của dự án ở cuối file.>

## Dự án
<1-3 câu: dự án là gì, cho ai, giải quyết vấn đề gì.> Định hướng đầy đủ: [docs/dinh-huong.md](docs/dinh-huong.md).
<Người dùng: ví dụ "hiểu nghiệp vụ, ít kinh nghiệm lập trình: trả lời tiếng Việt dễ hiểu, giải thích thuật ngữ bằng vai trò trong dự án".>

## Nguyên tắc
<3-6 điều không đổi suốt dự án, mỗi điều 1 dòng kèm lý do ngắn. Ví dụ:>
- <Ít file, ít tầng, dễ đọc: người bảo trì không chuyên.>
- <Chữ hiển thị tiếng Việt có dấu.>

## Stack
<Ngôn ngữ, framework, CSDL, thư viện giao diện. Đường dẫn công cụ đặc biệt trên máy nếu có.>

## Lệnh
| Việc | Lệnh |
|---|---|
| Chạy local | `<lệnh>` |
| Test một file | `<lệnh> <file>` |
| Test cả module / dự án | `<lệnh>` |
| Build | `<lệnh>` <hoặc "không có"> |
| Kiểm cú pháp | `<lệnh>` <hoặc "không có"> |
<Lưu ý khi chạy: ví dụ "bật CSDL trước", "lỗi kết nối CSDL = CSDL tắt: dừng, nhờ người dùng bật". Chưa có code: ghi lệnh dự kiến + "kiểm lại sau việc số 1".>

## Module
| Module | Khóa | Ngữ cảnh | Ghi chú |
|---|---|---|---|
| <Tên> | `<khóa>` | [docs/context/<khóa>.md](docs/context/<khóa>.md) | <thư mục code, tiền tố...> |
<Dự án không chia module: một dòng "Tổng quan | `tong-quan` | docs/context/tong-quan.md".>
Mỗi việc mới trong một module: đọc file ngữ cảnh của module đó trước. Một phiên làm việc = một module.

## Git
- Nhánh chính: `<main>`. Remote: `<origin = ... (riêng tư / công khai)>` <hoặc "chưa có">.
- Nhánh làm việc: `lam/<YYYY-MM-DD>-<chủ-đề>` tạo từ nhánh chính. Không commit code thẳng lên nhánh chính.
- Đưa vào nhánh chính: `git merge --no-ff` tại máy, chỉ sau câu đồng ý.
- Commit nhỏ, mỗi commit một ý, không cần hỏi. Push sau mỗi commit: <có / không>.
- Mẫu commit message: `<Module>: <việc đã làm>` <đổi nếu muốn, ví dụ tiếng Việt không dấu>.

## Câu đồng ý
Các thao tác sau chỉ làm khi người dùng gõ đúng câu:
| Thao tác | Câu |
|---|---|
| Gộp nhánh làm việc vào nhánh chính | **"đồng ý đưa vào main"** |
| Lệnh ghi CSDL local (migration, seed, xóa dữ liệu) | **"đồng ý chạy"** |
<Thêm dòng nếu dự án có thao tác khó lùi khác.>

## Codex
<Có dùng / Không dùng (máy không có Codex hoặc người dùng không muốn).>
<Nếu dùng:>
- Luật cho Codex: [AGENTS.md](AGENTS.md). Claude bàn, viết phiếu, ra lệnh, review, chạy test, commit; Codex chỉ code theo phiếu.
- Nên giao Codex: việc nhiều file kiểu lặp lại (danh mục, form, danh sách, test).
- Nên để Claude: việc 1-3 file, lõi nghiệp vụ khó, phần dùng chung.
- Đây chỉ là **đề xuất**: sau mỗi `/bat-dau`, Claude luôn dừng hỏi người dùng từng việc "giao Codex hay Claude tự code".
- Tối đa 3 vòng phản hồi; Codex hết hạn mức / lặp lỗi / sửa ngoài phạm vi → Claude làm tiếp trên phần đã có. Kết quả Codex ở `.codex-out/` (không commit).

## Cấm
- Push lên nơi khác ngoài remote đã ghi ở `## Git`; `push --force`; `rebase`, `reset --hard`, `clean`; đổi kho sang công khai.
- Kết nối CSDL / máy chủ thật (production), dùng thông tin kết nối trong mã nguồn.
- Chạy lệnh ghi dữ liệu trên máy chủ thật; deploy (người dùng tự làm).
- In bí mật (mật khẩu, khóa API, token): chỉ nói tên file và loại thông tin, che giá trị. Không commit file `.env`.
<Thêm điều cấm riêng của dự án.>

## Quy trình
Dự án dùng plugin `quy-trinh-du-an`: `/khoi-tao` (đặt / đổi nền) · mỗi buổi `/bat-dau` → chọn ai code → làm → `/ket-thuc` → `/clear`.
Trạng thái nằm trong file, không nằm trong chat: `docs/tasks/BACKLOG.md` (mỗi việc 1 dòng), `docs/context/<khóa>.md`
(quyết định đã chốt), `docs/tasks/<số>-<slug>.md` (phiếu việc, xóa khi xong). Lịch sử nằm trong git.
