# Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

## Mục tiêu
* Sử dụng công cụ tương tác Interactive Rebase để chỉnh sửa lịch sử cục bộ.
* Thực hiện gộp nhiều commit nhỏ lẻ (squash) thành một commit duy nhất có ý nghĩa.
* Đổi tên thông điệp commit (reword) và xóa bỏ một commit không cần thiết (drop).

## Yêu cầu
**Bối cảnh:** Nhánh tính năng trước khi đẩy lên remote chứa nhiều commit thử nghiệm vụn vặt và thông điệp không rõ ràng.
**Ràng buộc:**
* Gộp 3 commit nhỏ cuối cùng thành 1 commit duy nhất.
* Thay đổi thông điệp commit sau khi gộp thành: `feat: hoan thien module authentication`.
* Xóa bỏ hoàn toàn một commit chứa file rác thử nghiệm (`temp.txt`).

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Tạo môi trường mô phỏng (4 commit nhỏ lẻ)
```bash
# Tạo commit 1
$ echo "module auth" > auth.js
$ git add auth.js
$ git commit -m "feat: khoi tao module auth"
[main 1a2b3c4] feat: khoi tao module auth

# Tạo commit 2
$ echo "fix auth error" >> auth.js
$ git commit -am "fix typo"
[main 2b3c4d5] fix typo

# Tạo commit 3
$ echo "add util" >> auth.js
$ git commit -am "adds utility functions"
[main 3c4d5e6] adds utility functions

# Tạo commit 4
$ echo "debug info" > temp.txt
$ git add temp.txt
$ git commit -m "add temp file for debug"
[main 4d5e6f7] add temp file for debug
```

**Lịch sử Git trước khi rebase:**
```bash
$ git log --oneline
4d5e6f7 (HEAD -> main) add temp file for debug
3c4d5e6 adds utility functions
2b3c4d5 fix typo
1a2b3c4 feat: khoi tao module auth
a2d7db5 Thêm bài tập 1: Khôi phục commit đã mất bằng Git Reflog
```

### Bước 2: Chạy lệnh Interactive Rebase
```bash
$ git rebase -i HEAD~4
```

### Bước 3: Giao diện soạn thảo Interactive Rebase (Mô phỏng ảnh chụp)
Khi trình soạn thảo mở lên, tiến hành cấu hình lại các lệnh (từ `pick` đổi thành `squash` hoặc `drop`):
```text
pick 1a2b3c4 feat: khoi tao module auth
squash 2b3c4d5 fix typo
squash 3c4d5e6 adds utility functions
drop 4d5e6f7 add temp file for debug

# Rebase a2d7db5..4d5e6f7 onto a2d7db5 (4 commands)
# ...
```
Sau đó lưu và đóng trình soạn thảo. Git tự động chuyển tiếp qua màn hình gộp nội dung commit. Xóa bỏ các message cũ và nhập thông điệp mới:
```text
feat: hoan thien module authentication

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
```

### Bước 4: Kiểm tra kết quả sau khi rebase
```bash
$ git log --oneline
5e6f7g8 (HEAD -> main) feat: hoan thien module authentication
a2d7db5 Thêm bài tập 1: Khôi phục commit đã mất bằng Git Reflog
```
*(Xác nhận: Lịch sử commit hiện tại đã trở nên gọn gàng sạch sẽ. Các commit lỗi typo đã bị gộp và commit chứa file rác đã bị hủy drop)*
