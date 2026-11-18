# Kế hoạch dựng plugin quy-trinh-du-an (bản 1)

Thiết kế: [thiet-ke.md](thiet-ke.md). Người thực hiện: Claude tự làm trong một phiên, kiểm thử bằng subagent.

## Ràng buộc chung
- Toàn bộ chữ trong skill, khung, README: tiếng Việt có dấu.
- Không chứa tên dự án nội bộ, đường dẫn máy, email (kho công khai).
- Cả 4 skill: `disable-model-invocation: true`.
- Hợp đồng `CLAUDE.md`: dòng đầu `<!-- quy-trinh-du-an: v1 -->`, 10 mục cố định (thiết kế mục 7).
- Không chế độ PR, không deploy.

## Việc

| # | Việc | File | Xong khi |
|---|---|---|---|
| 1 | Khung plugin | `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `LICENSE`, `.gitignore` | `claude plugin validate .` đạt |
| 2 | `/khoi-tao` + khung | `skills/khoi-tao/SKILL.md`, `skills/khoi-tao/mau/{CLAUDE,AGENTS,dinh-huong,BACKLOG,context}.md` | validate đạt; T1, T2, T3 đạt |
| 3 | `/bat-dau` | `skills/bat-dau/SKILL.md` (có khung phiếu việc, điểm dừng bước 7) | T4, T6 đạt |
| 4 | `/giao-codex` | `skills/giao-codex/SKILL.md` | T5 đạt |
| 5 | `/ket-thuc` | `skills/ket-thuc/SKILL.md` | T4 đạt |
| 6 | README | `README.md` (12 mục, thiết kế mục 10) | Người đọc mới (subagent) trả lời đúng 5 câu hỏi dùng thử chỉ từ README |
| 7 | Kiểm thử | thư mục nháp ngoài kho | T0 (đối chứng), T1-T6 theo thiết kế mục 12; lỗi → sửa skill → chạy lại |
| 8 | Công bố | kho `hahuytin/quy-trinh-du-an` công khai | Người dùng xác nhận ngay trước khi tạo; cài thử bằng `/plugin marketplace add` |

## Cách kiểm thử hội thoại
Subagent đóng vai Claude dùng skill trong thư mục nháp. Câu trả lời của người dùng soạn sẵn trong `tra-loi.md`;
subagent ghi mỗi câu hỏi nó đặt ra vào `hoi-thoai.md` rồi lấy câu trả lời kế tiếp. Chấm theo bảng "Đạt khi" ở thiết kế mục 12.
