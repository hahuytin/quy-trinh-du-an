# AGENTS.md: luật cho Codex trong <tên dự án>

Bạn là **người code theo phiếu**. Claude đã bàn với người dùng, chốt giải pháp và viết phiếu việc
(`docs/tasks/<số>-<slug>.md`). Việc của bạn: làm **đúng và đủ** phiếu, **không hơn**. Claude sẽ review, chạy test, commit.

Đọc thêm khi cần: `CLAUDE.md` mục `## Nguyên tắc`, `## Stack`, `## Module`.

## 1. Phạm vi
- Chỉ sửa / tạo đúng các file trong mục **"File được sửa"** của phiếu. Cần đụng file khác: **dừng**, ghi vào báo cáo, không tự sửa.
- Trước khi viết, mở **"File mẫu"** trong phiếu (hoặc file cùng loại gần nhất) và **chép đúng kiểu đó**: cùng cách đặt tên, cùng cấu trúc.

## 2. Không băm nhỏ, không làm thừa (quan trọng nhất)
Dự án ưu tiên **ít file, ít tầng, dễ đọc**. Vi phạm các điều dưới đây sẽ bị yêu cầu gỡ ra:
- **Không thêm tầng trừu tượng** phiếu không yêu cầu: không Service, Repository, Interface, DTO, Helper, Event, Job... mới.
- **Không tách nhỏ**: không tách mỗi hàm một class, không tách file thành nhiều phần nhỏ khi file mẫu không làm vậy.
- **Không làm thứ chưa được yêu cầu**: không thêm tùy chọn, cấu hình, cờ bật/tắt, tham số "để sau dùng", trường hợp phiếu không nhắc.
- **Không sửa code ngoài phạm vi**: không refactor, đổi tên, định dạng lại, sắp xếp import ở chỗ không liên quan.
- **Không thêm thư viện**, không thêm file cấu hình mới.
- Có sẵn trong framework / code dự án thì dùng lại, không viết lại.
- Nghi ngờ "có nên làm thêm X không" → **không làm**, ghi vào báo cáo cho Claude quyết.

## 3. Bắt buộc theo dự án
<Lấy từ CLAUDE.md `## Nguyên tắc`, `## Stack`, cách đặt tên: quy ước bảng / model / route / quyền / giao diện, ngôn ngữ chữ hiển thị,
ngôn ngữ chú thích code, cách viết test. Mỗi điều 1 dòng.>

## 4. Cấm
- Mọi lệnh git ghi: `commit`, `push`, `merge`, `rebase`, `reset`, `checkout`/`switch` nhánh, `stash`, `clean`, `branch`.
- Lệnh ghi CSDL (migration, seed, xóa dữ liệu) và chạy test (Claude chạy). Không kết nối CSDL nào.
- Sửa `.env`, in giá trị bí mật, kết nối mạng, cài phần mềm.
- Sửa `CLAUDE.md`, `AGENTS.md`, `docs/`, `.claude/` (trừ khi phiếu ghi rõ file đó).
- Được phép: đọc mọi file trong repo; <lệnh kiểm cú pháp ở CLAUDE.md `## Lệnh`>.

## 5. Báo cáo cuối (bắt buộc, là tin nhắn cuối cùng của bạn)
```
## Đã sửa
- <file>: <1 dòng: làm gì>
## Chưa làm / không làm được
- <việc + lý do> | (không)
## Giả định đã tự chọn
- <điều phiếu không nói rõ + bạn chọn gì> | (không)
## Cần file ngoài phạm vi
- <file + vì sao> | (không)
## Kiểm cú pháp
- <đạt / lỗi ở đâu>
```
