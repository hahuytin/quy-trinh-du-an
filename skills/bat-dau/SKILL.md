---
name: bat-dau
description: Dùng khi người dùng mở đầu một buổi làm việc trong dự án theo quy trình quy-trinh-du-an - muốn biết dự án đang ở đâu, chọn việc của buổi, bàn rồi chốt danh sách việc.
argument-hint: "[số việc | mô tả việc] (bỏ trống = xem tổng quan rồi hỏi)"
disable-model-invocation: true
---

# Bắt đầu buổi làm việc

Người dùng gõ kèm: $ARGUMENTS

Một phiên = một buổi làm việc, các việc cùng một module. Trạng thái nằm trong file (`CLAUDE.md`, `docs/tasks/BACKLOG.md`,
`docs/context/`, phiếu việc), không nằm trong chat. Nói tiếng Việt dễ hiểu, giải thích thuật ngữ bằng vai trò của nó trong dự án.

## 1. Kiểm nền dự án

Đọc `CLAUDE.md`. Cần có dòng `<!-- quy-trinh-du-an: v1 -->` và đủ 10 mục: `## Dự án`, `## Nguyên tắc`, `## Stack`, `## Lệnh`,
`## Module`, `## Git`, `## Câu đồng ý`, `## Codex`, `## Cấm`, `## Quy trình`.

Thiếu: báo một dòng "Dự án chưa khởi tạo đủ (thiếu: <các mục>). Nên chạy `/khoi-tao` trước." Người dùng vẫn muốn làm thì làm tiếp,
dùng mặc định an toàn cho mục thiếu: nhánh chính `main`, câu đồng ý "đồng ý đưa vào main" / "đồng ý chạy", không push, hỏi lệnh test khi cần.

## 2. Xem hiện trạng

Đọc `docs/tasks/BACKLOG.md`. Chạy `git status`, `git branch --show-current`, `git log --oneline -5`, `git worktree list`.
Có thay đổi chưa commit của việc khác, hoặc thư mục đang ở nhánh của phiên khác: báo, không đụng.

## 3. Báo ngắn

Việc đang dở (nhánh nào, bước tiếp), việc chờ người dùng quyết, việc chưa làm. Có kèm số việc / mô tả: đọc luôn dòng đó,
file ngữ cảnh của module (theo cột "Ngữ cảnh" ở `## Module`) và phiếu nếu có (phiếu nằm trên nhánh khác thì `git show <nhánh>:<file>`).

## 4. Bàn

Người dùng chưa nói làm gì thì hỏi. Nêu việc rồi thì: xác định module (một phiên một module; khác module thì khuyên mở phiên khác),
đọc file ngữ cảnh của module (theo cột "Ngữ cảnh" ở `## Module`), rồi **bàn**: hỏi lại chỗ chưa rõ về nghiệp vụ (một câu mỗi lượt), góp ý, **cảnh báo nếu việc đụng
"Hướng đã bác"** trong file context.

Việc cần **module mới** (nhỏ, trong định hướng hiện có): thêm một dòng vào bảng `## Module` của `CLAUDE.md` và tạo
`docs/context/<khóa>.md` theo kiểu các file context có sẵn. Việc đổi nền dự án (giai đoạn mới, đổi stack, đổi luật): khuyên chạy `/khoi-tao`.

Sửa lỗi: tìm gốc lỗi trước khi sửa. Máy có skill `systematic-debugging` (plugin superpowers) thì dùng nó.
Thiết kế của việc ghi vào **phiếu việc** (mục 7), không ghi vào thư mục của plugin khác (ví dụ `docs/superpowers/`).

## 5. Chốt bảng việc

Lập bảng: số việc (việc đã có trong BACKLOG giữ số cũ; việc mới lấy "Số kế tiếp"), việc gì, **đề xuất** ai code kèm lý do 1 dòng
(theo `## Codex`), file dùng chung, có thao tác cần câu đồng ý không (theo `## Câu đồng ý`). So "File dùng chung" với các việc
`Đang làm` ở phiên khác; trùng thì báo. Việc lớn (nhiều phần, nhiều file, đổi dữ liệu): dùng Plan mode cho người dùng duyệt kế hoạch.

Việc cần sửa `CLAUDE.md` (thêm module, sửa lệnh sai ở `## Lệnh`...): ghi rõ trong bảng, người dùng thấy và đồng ý cùng lúc chọn ai code.

Trước khi đưa lựa chọn "Giao Codex": chạy `codex --version` qua Bash. Lỗi / không có, hoặc `## Codex` ghi "Không dùng":
**không** đưa lựa chọn "Giao Codex" (người dùng sẽ chọn một thứ không chạy được).

## 6. ĐIỂM DỪNG BẮT BUỘC: hỏi ai code

Chốt bảng xong thì **dừng hẳn lượt** và hỏi người dùng, **từng việc một**: "Giao Codex" hay "Claude tự code".
Đặt đề xuất của bạn lên đầu, kèm lý do 1 dòng. Có công cụ hỏi lựa chọn (ví dụ AskUserQuestion) thì dùng nó để người dùng bấm;
không chỉ viết câu hỏi lẫn trong đoạn văn.

Chưa có câu trả lời thì **không sửa file nào** (kể cả BACKLOG), không tạo nhánh, không gọi Codex.

**Vì sao:** người dùng muốn tự quyết ai code mỗi lần. Giao Codex tiết kiệm token, còn Claude code thì tốn hơn. Trước đây Claude
hay bàn xong là code luôn, và người dùng mất quyền chọn đó.

Các trường hợp vẫn phải dừng hỏi:
- Việc rất nhỏ (sửa một chữ, một dòng).
- `## Codex` ghi "việc 1-3 file thì Claude làm": đó là **gợi ý để bạn đề xuất**, không phải câu trả lời của người dùng.
- Người dùng đã nói "ok làm đi", "làm luôn" **trong lúc bàn hoặc ngay trong lời gọi `/bat-dau`**: câu đó đồng ý với ý tưởng,
  chưa phải chọn ai code.
- Máy không có Codex, hoặc `## Codex` ghi "Không dùng": vẫn hỏi "Claude code việc này nhé?" (nút "Claude tự code" / "Để sau").

Chỉ được bỏ qua điểm dừng khi người dùng đã **nói rõ ai code cho đúng việc đó** ("việc này Claude code luôn đi", "giao Codex việc 25").

| Ý nghĩ | Thực tế |
|---|---|
| "Việc 1 file, hiển nhiên Claude làm" | Đề xuất "Claude tự code" rồi vẫn hỏi. Người dùng bấm chỉ mất 1 giây |
| "Luật trong CLAUDE.md đã quyết rồi" | Luật đó là gợi ý đề xuất, không thay câu trả lời của người dùng |
| "Người dùng bảo làm đi rồi" | Đồng ý ý tưởng ≠ chọn ai code. Hỏi |
| "Hỏi nữa thì phiền" | Người dùng đã yêu cầu được hỏi. Không hỏi mới là phiền |
| "Sửa BACKLOG trước, hỏi sau" | Chưa trả lời thì không sửa file nào |

## 7. Ghi BACKLOG, viết phiếu

Có câu trả lời rồi:
- Thêm / cập nhật dòng BACKLOG **trên nhánh chính**: trạng thái `Đang làm`, cột "Ai code" theo câu trả lời, cột "Nhánh" ghi tên nhánh
  sắp tạo (mục 8). "Số kế tiếp" chỉ tăng khi thêm dòng mới.
- Viết phiếu `docs/tasks/<số>-<slug>.md` khi việc giao Codex hoặc kéo dài hơn 1 phiên, commit **cùng lúc với BACKLOG trên nhánh chính**
  (để phiên khác thấy việc đang làm). Mọi cập nhật sau đó (nhật ký, đổi người code) làm trên nhánh làm việc; gộp nhánh sẽ mang về.
- Commit (mẫu message ở `## Git`), push nếu `## Git` bảo push.
- Điều bàn ra mà "để sau" nhưng là việc cụ thể: thêm dòng BACKLOG mới trạng thái `Chưa làm`, không chỉ ghi vào context.

Khung phiếu:

```
# <số>. <tên việc>
Module: <khóa> · Ai code: <Codex | Claude> · Nhánh: <nhánh>

## Mục tiêu
<nghiệp vụ 2-4 câu>

## File được sửa
- <đường dẫn đầy đủ> (tạo mới | sửa)

## File mẫu
- <file có sẵn để chép kiểu>

## Không làm
- <thứ dễ bị làm thừa với việc này>

## Tiêu chí xong
- <hành vi kiểm được>
- <test cần có>

## Cần làm rõ
- [CẦN LÀM RÕ] <câu hỏi> - người quyết: <ai> | (không)

## Nhật ký
```

## 8. Làm

Tạo nhánh `lam/<YYYY-MM-DD>-<chủ-đề>` từ nhánh chính (hoặc vào nhánh dở của buổi trước và chạy `git merge <nhánh chính>` ngay;
xung đột thì dừng, báo). Rồi làm một mạch theo câu trả lời ở mục 6:
- **Giao Codex**: đọc `../giao-codex/SKILL.md` (cùng plugin, cạnh thư mục skill này) và làm theo.
- **Claude tự code**: code, chạy test liên quan theo `## Lệnh` sau mỗi bước, commit nhỏ mỗi commit một ý (không cần hỏi), push nếu `## Git` bảo.

Chỉ dừng hỏi ở: duyệt kế hoạch (việc lớn), câu đồng ý (`## Câu đồng ý`), và khi giữa chừng thấy nên đổi người code
(việc Claude làm hóa ra nên giao Codex, hoặc ngược lại): dừng, hỏi lại như mục 6.

Xong các việc của buổi: nhắc người dùng gõ `/ket-thuc`.
