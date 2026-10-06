# Bài 3: Xử lý xung đột phức tạp trong quá trình Rebase

## Mục tiêu
* Hiểu được sự khác biệt giữa giải quyết xung đột khi Merge và khi Rebase.
* Áp dụng quy trình giải quyết xung đột từng bước (step-by-step conflict resolution) trong Rebase.
* Đưa lịch sử nhánh tính năng lên trên đầu nhánh chính một cách thẳng hàng.

## Yêu cầu
**Bối cảnh:** Nhánh `main` đã có thêm 2 commit mới sửa đổi tệp `config.json`. Nhánh `feature-api` cũng sửa đổi cùng các dòng trong tệp tin đó qua 2 commit khác nhau. Khi rebase nhánh `feature-api` lên `main`, xung đột sẽ xảy ra ở từng commit.
**Ràng buộc:** Phải giải quyết xung đột thủ công ở từng chặng, không được dùng merge và không được phá hỏng code của nhánh chính.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo mô phỏng và tạo nhánh
```bash
# Đứng ở main, tạo file config.json và commit
$ echo -e "{\n  \"port\": 8080,\n  \"debug\": false\n}" > config.json
$ git add config.json
$ git commit -m "init config"
[main 7a8b9c0] init config

# Tạo nhánh feature-api từ main
$ git checkout -b feature-api
Switched to a new branch 'feature-api'

# Trên nhánh feature-api: sửa port thành 9000 và commit
$ sed -i 's/8080/9000/' config.json
$ git commit -am "feat: change port"
[feature-api 8b9c0d1] feat: change port

# Trên nhánh feature-api: sửa debug thành true và commit
$ sed -i 's/false/true/' config.json
$ git commit -am "feat: enable debug"
[feature-api 9c0d1e2] feat: enable debug

# Quay lại main, tạo các commit mới (gây xung đột)
$ git checkout main
$ sed -i 's/8080/8081/' config.json
$ git commit -am "update port on main"
[main 1d2e3f4] update port on main

$ sed -i '/\"debug\": false/a \ \ \"env\": \"production\"' config.json
$ git commit -am "add env config"
[main 2e3f4g5] add env config
```

### Bước 2: Bắt đầu tiến trình Rebase và Xử lý xung đột chặng 1
```bash
$ git checkout feature-api
$ git rebase main
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply 8b9c0d1... feat: change port
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
```

**Sửa file `config.json` thủ công:**
*(Xóa các thẻ `<<<<<<<`, `=======`, `>>>>>>>` và kết hợp các thay đổi của cả main và feature-api)*
```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

**Xác nhận hết xung đột chặng 1 và tiếp tục rebase:**
```bash
$ git add config.json
$ git rebase --continue
[detached HEAD 3f4g5h6] feat: change port
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### Bước 3: Xử lý xung đột chặng 2
```bash
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply 9c0d1e2... feat: enable debug
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
```

**Sửa file `config.json` thủ công lần 2:**
```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

**Xác nhận và hoàn tất Rebase:**
```bash
$ git add config.json
$ git rebase --continue
[detached HEAD 4g5h6i7] feat: enable debug
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/feature-api.
```

### Bước 4: Kiểm tra kết quả sau cùng
```bash
$ git status
On branch feature-api
nothing to commit, working tree clean

$ git log --graph --oneline
* 4g5h6i7 (HEAD -> feature-api) feat: enable debug
* 3f4g5h6 feat: change port
* 2e3f4g5 (main) add env config
* 1d2e3f4 update port on main
* 7a8b9c0 init config
* 5e6f7g8 feat: hoan thien module authentication
...
```
*(Xác nhận: Tiến trình rebase thành công. Toàn bộ lịch sử các commit của nhánh `feature-api` đã được đắp lên trên cùng (ngay sau nhánh `main`) theo một đường thẳng tắp, không sinh ra merge commit)*
