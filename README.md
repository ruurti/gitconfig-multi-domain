# Git Multi-Domain Config

Cấu hình git tự động chọn đúng user/email theo remote URL — không cần đổi tay.

## Cách hoạt động

Dùng tính năng `includeIf "hasconfig:remote.*.url:..."` để load config phụ dựa trên remote URL của repo.

```
~/.gitconfig               ← config mặc định + điều kiện include
~/.gitconfig-github        ← dùng khi remote là git@github.com
~/.gitconfig-company       ← dùng khi remote là git@git.xxx.com
```

## Cài đặt

**1. Copy file config về home:**

```bash
cp .gitconfig ~/.gitconfig
cp .gitconfig-github ~/.gitconfig-github
cp .gitconfig-company ~/.gitconfig-company
```

**2. Sửa thông tin cá nhân trong từng file:**

`~/.gitconfig-github`:
```ini
[user]
    name = Your Name
    email = your@users.noreply.github.com
```

`~/.gitconfig-company`:
```ini
[user]
    name = Your Name
    email = your@company.com
```

**3. Sửa domain công ty trong `~/.gitconfig`** (nếu khác `git.xxx.com`):

```ini
[includeIf "hasconfig:remote.*.url:git@git.yourcompany.com:**/**"]
    path = ~/.gitconfig-company
```

## Kiểm tra

```bash
# Trong repo GitHub
cd ~/projects/my-github-repo
git config user.email   # → your@users.noreply.github.com

# Trong repo công ty
cd ~/projects/company-repo
git config user.email   # → your@company.com
```

## Nâng cao

### Override thủ công cho một repo cụ thể (local)

Dùng khi repo đặc biệt cần dùng email khác với rule tự động:

```bash
cd ~/projects/special-repo
git config user.name "Your Name"
git config user.email "special@email.com"
```

Config local (`.git/config`) luôn ưu tiên hơn global — sẽ override `includeIf`.

### Set lại global default

```bash
git config --global user.name "Your Name"
git config --global user.email "default@email.com"
```

### Xem toàn bộ config đang active (theo thứ tự ưu tiên)

```bash
git config --list --show-origin    # xem từng giá trị đến từ file nào
git config --list --show-scope     # xem scope: system / global / local / worktree
```

---

## Debug

### Kiểm tra email đang được dùng trong repo hiện tại

```bash
git config user.email
git config user.name
```

### Xem tất cả config đang active và nguồn gốc

```bash
git config --list --show-origin
```

### Kiểm tra remote URL của repo (để biết rule nào sẽ match)

```bash
git remote -v
```

### Xem `includeIf` có được load không

```bash
git config --list --show-origin | grep includeIf
git config --list --show-origin | grep email
```

### Debug toàn bộ quá trình load config của git

```bash
GIT_CONFIG_NOSYSTEM=0 GIT_TRACE2=1 git config user.email 2>&1 | grep -E "config|include"
```

### Kiểm tra file config nào đang được áp dụng

```bash
# Xem config ở scope global
git config --global --list

# Xem config ở scope local (trong repo)
git config --local --list

# So sánh: giá trị nào thực sự thắng
git config user.email          # giá trị cuối cùng được dùng
git config --show-origin user.email   # + file nguồn
```

---

## Yêu cầu

Git >= 2.36 (hỗ trợ `hasconfig`).

```bash
git --version
```
