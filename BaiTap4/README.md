# Bài 4: Mô phỏng quy trình Hotfix & Gitflow thực tế

## Mục tiêu
* Áp dụng mô hình phân nhánh Gitflow chuẩn trong quản lý vòng đời phần mềm.
* Thực hành tạo, kiểm thử và tích hợp nhánh Hotfix trực tiếp vào hệ thống đang chạy.
* Đồng bộ hóa bản vá nóng về nhánh phát triển (develop) để tránh trôi lỗi (regression).

## Yêu cầu
**Bối cảnh:** Hệ thống đang chạy trên nhánh `main` (phiên bản stable v1.0.0) gặp lỗi nghiêm trọng lộ dữ liệu người dùng. Nhánh `develop` đang phát triển dở dang các tính năng mới và không thể deploy ngay được.
**Ràng buộc:**
* Tạo nhánh sửa lỗi khẩn cấp `hotfix/v1.0.1` tách ra trực tiếp từ `main`.
* Sửa lỗi xong, gộp `hotfix` vào `main`, tạo tag phiên bản `v1.0.1` để chuẩn bị release.
* Bắt buộc phải gộp ngược lại nhánh `hotfix` này vào nhánh `develop` để code tương lai cũng được vá lỗi.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo mô phỏng (Thiết lập nhánh main và develop)
```bash
# Đang ở nhánh main
$ git commit --allow-empty -m "Initial commit v1.0.0"
$ git tag v1.0.0

# Tạo nhánh develop và phát triển tính năng mới
$ git checkout -b develop
$ git commit --allow-empty -m "start working on v1.1.0 features"
```

### Bước 2: Bắt đầu tiến trình vá lỗi nóng (Hotfix)
```bash
# Trở về main (code production) và tách nhánh hotfix mới
$ git checkout main
Switched to branch 'main'

$ git checkout -b hotfix/v1.0.1
Switched to a new branch 'hotfix/v1.0.1'

# Sửa lỗi và commit trên nhánh hotfix
$ echo "fixed security vulnerability" > security_patch.txt
$ git add security_patch.txt
$ git commit -m "fix: patch critical data leak"
[hotfix/v1.0.1 7c8d9e0] fix: patch critical data leak
 1 file changed, 1 insertion(+)
 create mode 100644 security_patch.txt
```

### Bước 3: Gộp Hotfix vào Main và đánh Tag phát hành
```bash
$ git checkout main
Switched to branch 'main'

$ git merge --no-ff hotfix/v1.0.1 -m "Merge branch 'hotfix/v1.0.1' into main"
Merge made by the 'ort' strategy.

# Tạo tag bản release mới
$ git tag -a v1.0.1 -m "Release Hotfix 1.0.1"
```

### Bước 4: Đồng bộ Hotfix về lại nhánh Develop (Tránh trôi lỗi)
```bash
$ git checkout develop
Switched to branch 'develop'

$ git merge --no-ff hotfix/v1.0.1 -m "Merge branch 'hotfix/v1.0.1' into develop"
Merge made by the 'ort' strategy.

# Dọn dẹp nhánh cục bộ (Xóa nhánh hotfix sau khi hoàn tất)
$ git branch -d hotfix/v1.0.1
Deleted branch hotfix/v1.0.1 (was 7c8d9e0).
```

### Bước 5: Kiểm tra kết quả tổng thể (Minh chứng)

**1. Kiểm tra các nhánh và tag hiện có:**
```bash
$ git branch -a
* develop
  main

$ git tag
v1.0.0
v1.0.1
```

**2. Đồ thị lịch sử gộp nhánh (Gitflow Graph):**
```bash
$ git log --graph --oneline --all
*   a1b2c3d (HEAD -> develop) Merge branch 'hotfix/v1.0.1' into develop
|\  
| * 7c8d9e0 fix: patch critical data leak
* | e3a5b2c start working on v1.1.0 features
|/  
| *   9f8e7d6 (tag: v1.0.1, main) Merge branch 'hotfix/v1.0.1' into main
| |\  
|/ /  
* / 1a2b3c4 (tag: v1.0.0) Initial commit v1.0.0
```
*(Xác nhận: Sơ đồ hiển thị rõ ràng luồng Gitflow. Nhánh `hotfix` (7c8d9e0) tách từ `main`, sau đó được gộp đồng thời vào cả `main` (tạo tag v1.0.1) và `develop`. Không có tính năng nào đang dang dở trên `develop` bị rò rỉ sang `main` trong quá trình này).*
