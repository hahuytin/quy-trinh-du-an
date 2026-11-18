---
name: ket-thuc
description: Dùng khi người dùng kết thúc buổi làm việc trong dự án theo quy trình quy-trinh-du-an - chốt việc đã xong, đưa vào nhánh chính khi được đồng ý, cập nhật BACKLOG và context.
disable-model-invocation: true
---

# Kết thúc buổi làm việc

Mọi thông tin riêng của dự án (lệnh test, nhánh chính, câu đồng ý, có push không) lấy từ `CLAUDE.md`: `## Lệnh`, `## Git`, `## Câu đồng ý`.

## 1. Gom phần dở

Chạy `git status`, `git branch --show-current`, `git log --oneline <nhánh chính>..HEAD`. Commit nốt phần dở (mỗi commit một ý),
push nếu `## Git` bảo.

## 2. Từng việc của buổi

Theo bảng đã chốt lúc `/bat-dau`, xếp mỗi việc vào một loại:
- **Xong**: đủ tiêu chí trong phiếu (hoặc điều đã chốt), **có bằng chứng**: đã chạy test liên quan theo `## Lệnh` trong phiên này
  và thấy kết quả đạt. Ghi số đạt/tổng. `## Lệnh` chưa có lệnh test: ghi "chưa có lệnh test" và nói rõ đã kiểm bằng cách nào.
  Chưa chạy, hoặc chạy mà lỗi, thì **chưa xong**.
- **Chưa xong**: dòng BACKLOG = `Đang làm` + nhánh + bước tiếp 1 câu + hash mới nhất. Có phiếu thì thêm Nhật ký ngắn:
  `### <YYYY-MM-DD> <tóm tắt>` / `- Đã xong: ... (<hash>), test: <đạt>/<tổng>` / `- Ai code: Codex <n> vòng | Claude` /
  `- Bước tiếp: ...` / `- Quyết định mới: ...` / `- Chờ người dùng: ...` (dòng nào không có thì bỏ).

## 3. Mọi việc trên nhánh đã xong → chuẩn bị đưa vào nhánh chính

1. **Test cả module / dự án** theo `## Lệnh` (lâu thì chạy nền). Code lỗi thì sửa trước, không đưa vào nhánh chính.
   Chính lệnh trong `## Lệnh` sai (không chạy được): báo người dùng, đề xuất lệnh đúng, **hỏi trước** khi sửa `CLAUDE.md`.
2. **Soát độc lập** phần do Claude tự code (phần Codex code đã được Claude review thì bỏ qua):
   có lệnh `/code-review` thì dùng trên nhánh ở mức mặc định; không có thì gọi một subagent review `git diff <nhánh chính>...HEAD`
   (tìm lỗi đúng/sai, bỏ qua chuyện trình bày). Lỗi thật thì sửa, thêm test cho lỗi đó, commit, chạy lại test.
   Không cần soát lại lần hai trừ khi phần sửa lớn hơn phần ban đầu.
3. **Dò bí mật** trong `git diff <nhánh chính>...HEAD`: file `.env*`, chuỗi giống mật khẩu / khóa API / token
   (ví dụ `password=`, `secret`, `api_key`, `token`, `sk-`, `ghp_`, `AKIA`, khối `-----BEGIN ... PRIVATE KEY-----`).
   Thấy → **dừng**, báo người dùng tên file và loại thông tin (che giá trị), không đưa vào nhánh chính.
4. **Tóm tắt thay đổi bằng lời** cho người dùng và **chỉ cách thử cụ thể**: mở trang nào, bấm gì, thấy gì là đúng
   (lệnh chạy lấy ở `## Lệnh`). Thay đổi chưa có gì nhìn thấy trên màn hình: nói thẳng như vậy, đưa lệnh test để người dùng xem kết quả.
   Có lệnh ghi CSDL mới thì cần câu "đồng ý chạy" theo `## Câu đồng ý` trước khi người dùng thử.
5. Hỏi rõ: "Thử thấy ổn thì gõ đúng câu **<câu đồng ý gộp nhánh>**". Chỉ khi người dùng gõ đúng câu đó (mặc định **"đồng ý đưa vào main"**;
   "ổn", "ok" không tính): `git switch <nhánh chính>`, `git merge --no-ff <nhánh>`, push nếu `## Git` bảo.
6. Trên nhánh chính:
   - Chép quyết định đáng giữ: quyết định của một module → file ngữ cảnh của module, theo cột "Ngữ cảnh" ở `## Module` (kèm lý do; hướng bị bác vào "Hướng đã bác");
     quyết định toàn dự án (công nghệ, phạm vi, điều không làm) → `docs/dinh-huong.md` mục "Quyết định lớn".
   - Xóa phiếu đã xong.
   - BACKLOG: chuyển các việc sang "Xong gần đây" kèm hash merge.
   - Commit, push nếu `## Git` bảo. `git branch -d <nhánh>`, xóa nhánh trên remote nếu có.

Còn việc chưa xong: nhánh để lại, không gộp.

## 4. Học từ lỗi Codex

Đọc các dòng `Lỗi Codex: ...` trong Nhật ký phiếu của buổi này và trong BACKLOG. **Cùng một loại lỗi xuất hiện từ 2 lần trở lên**
mà `AGENTS.md` chưa có luật chặn: đề xuất một câu luật cụ thể (ví dụ "Không tạo class Service; logic để trong controller như file mẫu"),
nói nó thêm vào mục nào. Người dùng đồng ý mới ghi vào `AGENTS.md`, commit.

## 5. Báo người dùng

Buổi này xong gì (kèm bằng chứng test), còn gì, buổi sau gõ gì (`/bat-dau <số việc>`). Nhắc `/clear` (hoặc đóng phiên).
Phiên đã đầy mà còn việc: `/ket-thuc` → `/clear` → `/bat-dau` làm tiếp.
