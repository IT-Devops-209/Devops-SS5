# Bài 1: Khôi phục commit đã mất bằng Git Reflog

## Mục tiêu
* Hiểu được cách Git lưu trữ lịch sử hoạt động cục bộ thông qua Reflog.
* Sử dụng thành thạo lệnh `git reflog` để truy vết các tham chiếu commit cũ.
* Khôi phục thành công một commit đã bị xóa mất khỏi nhánh làm việc sau khi thực thi lệnh reset hard.

## Yêu cầu
**Bối cảnh:** Vô tình chạy lệnh `git reset --hard HEAD~1` trên repository cục bộ, khiến cho commit quan trọng nhất chứa mã nguồn vừa chỉnh sửa bị biến mất và không hiển thị trên `git log`.
**Ràng buộc:** Không được viết lại code thủ công, bắt buộc phải dùng cơ chế Reflog của Git để kéo lại commit trạng thái cũ.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Tạo giả lập môi trường và commit tính năng
```bash
$ git init
Initialized empty Git repository in /home/devops/project/.git/

$ echo "Initial file" > init.txt
$ git add init.txt
$ git commit -m "Initial commit"
[main (root-commit) a1b2c3d] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 init.txt

$ echo "Day la tinh nang quan trong" > feature.txt
$ git add feature.txt
$ git commit -m "Them tinh nang quan trong"
[main e3a5b2c] Them tinh nang quan trong
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

### Bước 2: Giả lập lỗi làm mất commit bằng lệnh reset --hard
```bash
$ git reset --hard HEAD~1
HEAD is now at a1b2c3d Initial commit

$ git log --oneline
a1b2c3d (HEAD -> main) Initial commit
```
*(Xác nhận: File tính năng quan trọng đã biến mất khỏi lịch sử và working directory)*

### Bước 3: Tra cứu Reflog để tìm mã hash của commit đã mất
```bash
$ git reflog
a1b2c3d (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
e3a5b2c HEAD@{1}: commit: Them tinh nang quan trong
a1b2c3d (HEAD -> main) HEAD@{2}: commit (initial): Initial commit
```
*(Phân tích: Commit `Them tinh nang quan trong` có mã hash là `e3a5b2c` nằm ở vị trí `HEAD@{1}`)*

### Bước 4: Khôi phục lại commit về nhánh hiện tại
```bash
$ git reset --hard e3a5b2c
HEAD is now at e3a5b2c Them tinh nang quan trong

$ git log --oneline
e3a5b2c (HEAD -> main) Them tinh nang quan trong
a1b2c3d Initial commit
```
*(Kết quả: Commit đã được khôi phục thành công vào lại log hiện tại!)*
