---
name: giao-codex
description: Dùng khi người dùng đã chọn "Giao Codex" cho một việc trong BACKLOG - Claude viết phiếu, ra lệnh Codex code, review và phản hồi tới khi đạt.
argument-hint: "<số việc> [phần cần giao]"
disable-model-invocation: true
---

# Giao việc cho Codex

Giao việc số: $ARGUMENTS

Claude giữ vai: viết phiếu, ra lệnh, review, chạy test, commit. **Codex chỉ code.** Luật của Codex nằm ở `AGENTS.md` của dự án
(Codex tự đọc). Điểm yếu đã biết của Codex: **băm nhỏ, làm thừa** - phải chặn từ phiếu và soát khi review.

## 0. Kiểm Codex

Chạy `codex --version` qua Bash. Lỗi / không có lệnh → **dừng, hỏi người dùng** (đây là đổi người code, người dùng phải quyết):
"Máy không có Codex. Việc <số>: Claude tự code hay để sau?" - nút "Claude tự code" / "Để sau". Chọn Claude: ghi cột "Ai code" của dòng
BACKLOG thành `Claude (không có Codex)`, Claude tự code theo phiếu, commit không có hậu tố Codex. Chọn để sau: trạng thái `Tạm dừng`,
bước tiếp "cài Codex hoặc chọn Claude code". Dự án không có `AGENTS.md`:
báo người dùng, đề nghị chạy `/khoi-tao` (chế độ cập nhật, chặng 5) để tạo; chưa có thì dùng mục "Không làm" của phiếu làm luật.

## 1. Chuẩn bị

- Đang ở nhánh làm việc (không phải nhánh chính); `git status` sạch (commit phần đang dở trước) để diff chỉ là của Codex.
- Phiếu `docs/tasks/<số>-<slug>.md` đủ các mục, viết cụ thể (Codex làm đúng chữ, không đoán ý):
  - **Mục tiêu**: nghiệp vụ 2-4 câu.
  - **File được sửa**: đường dẫn đầy đủ, ghi rõ file nào tạo mới. Ít nhất có thể.
  - **File mẫu**: file có sẵn để chép kiểu.
  - **Không làm**: những thứ dễ bị làm thừa với việc này (tầng mới, tùy chọn, refactor...).
  - **Tiêu chí xong**: hành vi kiểm được + test cần có.
  - **Cần làm rõ**: còn `[CẦN LÀM RÕ]` chưa có câu trả lời → hỏi người dùng trước, không giao.
- Cỡ mỗi lần giao: một phần trọn vẹn, khoảng ≤ 8 file. Việc to hơn: giao lần lượt từng phần.
- Thấy việc không hợp giao Codex (1-3 file, lõi nghiệp vụ khó, sửa phần dùng chung): nói với người dùng, hỏi có đổi sang Claude code không.

## 2. Ra lệnh

Chạy qua **Bash** (trên Windows là Git Bash), **chạy nền**, đợi thông báo xong. Trong lúc Codex chạy, Claude **không sửa file**.

```bash
mkdir -p .codex-out
codex exec -C "<thư mục repo>" -s workspace-write --json -o .codex-out/<số>-v1.md "Đọc AGENTS.md rồi docs/tasks/<file phiếu>. Làm phần: <phần>. Chỉ sửa các file trong mục File được sửa. Kết thúc bằng báo cáo theo mẫu AGENTS.md." < /dev/null > .codex-out/<số>-v1.jsonl 2>&1
```

- `< /dev/null` bắt buộc (không có thì Codex đứng chờ nhập). Prompt viết được tiếng Việt có dấu.
- Id phiên Codex ở dòng đầu `.codex-out/<số>-v1.jsonl`: `{"type":"thread.started","thread_id":"..."}`. Ghi vào `.codex-out/<số>.id`.
- Lần đầu Claude Code sẽ hỏi quyền chạy `codex`: nhờ người dùng chọn cho phép.
- `.codex-out/` phải nằm trong `.gitignore` (thêm nếu thiếu).

## 3. Review

1. Đọc báo cáo `.codex-out/<số>-v<n>.md`.
2. `git status`, `git diff --stat`: file ngoài danh sách → trả lại bằng `git restore <file>` (file mới ngoài danh sách: xóa đúng file đó), ghi thành lỗi.
3. **Soát băm nhỏ / làm thừa**: class, tầng, file mới không có trong phiếu; hàm chỉ gọi một lần bị tách riêng; tùy chọn, cấu hình,
   tham số không ai dùng; refactor chỗ không liên quan; số dòng lớn hơn rõ so với file mẫu cùng loại.
4. **Soát đúng luật dự án**: `## Nguyên tắc`, `## Stack` của `CLAUDE.md` và mục "Bắt buộc theo dự án" của `AGENTS.md`; test đủ tiêu chí phiếu.
5. **Claude tự chạy** kiểm cú pháp và test liên quan theo `## Lệnh`.

## 4. Phản hồi (tối đa 3 vòng)

```bash
codex exec resume <id> --json -o .codex-out/<số>-v<n+1>.md "<phản hồi>" < /dev/null > .codex-out/<số>-v<n+1>.jsonl 2>&1
```

- Phản hồi đánh số, mỗi ý một việc cụ thể: file nào, sai gì, sửa thành gì ("gộp `FooService` vào `FooController`, xóa file service";
  "bỏ tham số `$options`"; "test thiếu trường hợp người không có quyền gọi phải bị chặn").
- Không khen, không giải thích dài. Nhắc lại luật `AGENTS.md` nào bị vi phạm.

## 5. Chuyển sang Claude tự code khi

- Codex báo hết hạn mức / lỗi giới hạn, hoặc lệnh `codex` lỗi.
- Hết 3 vòng vẫn chưa đạt; hoặc 2 vòng liền lặp cùng một lỗi; hoặc Codex sửa file ngoài phạm vi lần thứ hai.
- Review thấy việc cần hiểu nghiệp vụ sâu hơn phiếu.

Báo người dùng một dòng lý do. Claude làm tiếp **trên phần Codex đã làm**, không làm lại từ đầu (trừ khi phần đó sai hướng).

## 6. Đạt

- Commit (Claude), message theo `## Git` kèm hậu tố `(Codex code, Claude review, <n> vòng)`; push nếu `## Git` bảo.
- Ghi vào Nhật ký phiếu và dòng BACKLOG: ai code, mấy vòng, **lỗi Codex đã mắc** theo dạng `Lỗi Codex: <loại lỗi ngắn>`
  (ví dụ `Lỗi Codex: tách service thừa`). `/ket-thuc` gom các dòng này để đề xuất luật mới cho `AGENTS.md`.
- Báo người dùng ngắn: phần nào xong, test bao nhiêu đạt, còn gì.
