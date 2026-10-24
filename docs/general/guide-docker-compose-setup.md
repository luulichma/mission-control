# Hướng dẫn cài Docker và setup Docker Compose

Hướng dẫn chung cho mọi dự án trong mission-control chạy bằng Docker Compose
(ví dụ DevLake). Gồm 2 phần:

1. [Cài Docker và Docker Compose](#phần-1-cài-docker-và-docker-compose) trên Windows, macOS, Linux.
2. [Setup file compose cho một dự án](#phần-2-setup-file-compose-cho-một-dự-án): thư mục chạy,
   `docker-compose.yml`, `.env`, file override.

Hướng dẫn riêng của từng dự án nằm trong `docs/projects/<tên-dự-án>/`.

## Yêu cầu

- Docker **v19.03.10+** và Docker Compose **v2.2.3+**. Đây là mức tối thiểu của DevLake;
  nên dùng bản mới nhất.
- RAM: nên có từ 8 GB trở lên khi chạy DevLake (MySQL + Grafana + backend).
- Dùng plugin `docker compose` (v2, có dấu cách), không dùng `docker-compose` (v1) đã ngừng
  hỗ trợ.

## Phần 1: Cài Docker và Docker Compose

### Windows 10/11

Docker trên Windows chạy qua WSL 2. Docker Desktop đã kèm sẵn Compose plugin.

1. **Kiểm tra ảo hóa đã bật:** mở Task Manager → Performance → CPU, dòng *Virtualization*
   phải là `Enabled`. Nếu `Disabled`, vào BIOS/UEFI bật Intel VT-x hoặc AMD-V.
2. **Cài WSL 2:** mở PowerShell bằng quyền Administrator:

   ```powershell
   wsl --install
   ```

   Khởi động lại máy, sau đó kiểm tra:

   ```powershell
   wsl --version
   ```

3. **Cài Docker Desktop:** tải bộ cài tại https://docs.docker.com/desktop/setup/install/windows-install/
   hoặc dùng winget:

   ```powershell
   winget install -e --id Docker.DockerDesktop
   ```

   Khi cài, giữ tùy chọn **Use WSL 2 instead of Hyper-V**.
4. **Mở Docker Desktop** và chấp nhận điều khoản. Đợi biểu tượng cá voi ở khay hệ thống báo
   *Engine running*. Kiểm tra Settings → General đã bật **Use the WSL 2 based engine**.
5. **Cấp RAM cho Docker:** với WSL 2, giới hạn RAM đặt trong file `%UserProfile%\.wslconfig`
   (không phải trong Settings → Resources của Docker Desktop). Tạo hoặc sửa file đó:

   ```ini
   [wsl2]
   memory=8GB
   ```

   Rồi chạy `wsl --shutdown` và mở lại Docker Desktop để áp dụng.

### macOS

1. Tải Docker Desktop đúng loại chip (Apple Silicon hoặc Intel) tại
   https://docs.docker.com/desktop/setup/install/mac-install/, hoặc dùng Homebrew:

   ```bash
   brew install --cask docker
   ```

2. Mở Docker từ Applications, chấp nhận điều khoản, đợi *Engine running*.
3. Vào Settings → Resources, đặt Memory tối thiểu 8 GB.

### Linux (Ubuntu/Debian)

Cài từ repository chính thức của Docker:

```bash
# 1. Gỡ bản cũ (nếu có)
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove -y $pkg
done

# 2. Thêm repository của Docker
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Cài Docker Engine + Compose plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4. Bật Docker khởi động cùng máy
sudo systemctl enable --now docker

# 5. Chạy docker không cần sudo (đăng xuất rồi đăng nhập lại sau lệnh này)
sudo usermod -aG docker $USER
```

Với Debian, thay `ubuntu` bằng `debian` trong 2 URL trên. Các distro khác: xem
https://docs.docker.com/engine/install/.

### Kiểm tra sau khi cài

Chạy được trên cả PowerShell, Terminal macOS và Linux:

```bash
docker --version          # >= 19.03.10
docker compose version    # >= v2.2.3
docker run --rm hello-world
```

Thấy dòng `Hello from Docker!` là cài thành công.

## Phần 2: Setup file compose cho một dự án

### Bước 1: Tạo thư mục chạy riêng

Mỗi dự án có **một thư mục chạy riêng, nằm ngoài repo mission-control**, để không lỡ commit
file `.env` chứa secret và để các dự án không lẫn vào nhau. Ví dụ:

```text
~/devlake/                       # thư mục chạy của dự án DevLake
├── docker-compose.yml           # định nghĩa service (thường tải từ upstream)
├── docker-compose.override.yml  # tùy chỉnh riêng của mình (không bắt buộc)
├── .env                         # biến môi trường + secret, KHÔNG commit
└── env.example                  # file mẫu của .env, được phép commit/chia sẻ
```

> Windows: nếu dự án mount thư mục từ máy vào container (bind mount), đặt thư mục chạy trong
> filesystem của WSL (ví dụ `\\wsl$\Ubuntu\home\<user>\devlake`) sẽ nhanh hơn nhiều so với ổ `C:`/`F:`.

**Lưu ý tên thư mục:** Compose lấy tên thư mục làm *project name*, và tên volume được ghép
tiền tố theo tên đó (ví dụ `devlake_mysql-data`). **Đổi tên hoặc chuyển thư mục là Compose tạo
volume mới, rỗng**, nhìn như mất dữ liệu. Để cố định, khai báo `name:` ở đầu file compose
(xem ví dụ bên dưới).

### Bước 2: Viết hoặc tải `docker-compose.yml`

Nếu dự án đã có file compose chính thức (như DevLake), tải về theo hướng dẫn của dự án.
Nếu tự viết, dùng cấu trúc sau:

```yaml
name: my-project                  # cố định project name (xem lưu ý ở Bước 1)

services:
  app:
    image: nginx:1.27             # luôn ghi rõ phiên bản, tránh dùng :latest
    restart: unless-stopped       # tự chạy lại khi lỗi hoặc khi khởi động lại Docker
    ports:
      - "127.0.0.1:8081:80"       # <cổng máy>:<cổng container>; 127.0.0.1 = chỉ truy cập từ máy này
    environment:
      TZ: Asia/Ho_Chi_Minh
      DB_PASSWORD: ${DB_PASSWORD} # lấy giá trị từ file .env
    volumes:
      - app-data:/data            # named volume: dữ liệu giữ lại khi down/up
    depends_on:
      - db

  db:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql

volumes:
  app-data:
  db-data:
```

Quy ước khi viết:

- **Không dùng khóa `version:`** ở đầu file. Compose v2 đã bỏ khóa này và sẽ cảnh báo.
- **Không ghi secret trực tiếp** vào file compose. Dùng `${TÊN_BIẾN}` và đặt giá trị trong `.env`.
- **Ghi rõ phiên bản image** để lần chạy sau không tự nhảy lên bản mới.
- **Dùng named volume** cho dữ liệu cần giữ (database...). Chỉ dùng bind mount
  (`./config:/etc/app`) khi cần sửa file cấu hình từ máy.
- **Bind cổng vào `127.0.0.1`** nếu service không cần truy cập từ máy khác.

### Bước 3: Tạo file `.env`

Compose **tự đọc file `.env` nằm cùng thư mục** với `docker-compose.yml` và thay vào các chỗ
`${TÊN_BIẾN}`:

```bash
cp env.example .env       # Windows PowerShell: Copy-Item env.example .env
```

Rồi điền giá trị:

```ini
# .env
DB_PASSWORD=<mật khẩu mạnh>
```

- `.env` chứa secret, **không commit, không gửi qua chat**. Chỉ chia sẻ `env.example`
  (cùng tên biến, giá trị để trống hoặc là ví dụ).
- Phân biệt: `${VAR}` trong file compose lấy từ `.env` để **thay vào file compose**.
  Nếu muốn đưa nguyên cả file biến vào **bên trong container**, dùng thêm `env_file: .env`
  trong service.

### Bước 4: Tùy chỉnh bằng `docker-compose.override.yml` (khuyến nghị)

Compose **tự gộp** `docker-compose.override.yml` vào `docker-compose.yml` khi chạy. Nên để
các thay đổi của mình (cổng, múi giờ, mật khẩu...) trong file này thay vì sửa trực tiếp file
upstream. Khi nâng cấp, chỉ cần thay file `docker-compose.yml` mới mà không mất tùy chỉnh.

```yaml
# docker-compose.override.yml: chỉ ghi phần muốn đổi
services:
  app:
    ports:
      - "127.0.0.1:9090:80"
    environment:
      TZ: Asia/Ho_Chi_Minh
```

### Bước 5: Kiểm tra cấu hình trước khi chạy

```bash
docker compose config            # in ra cấu hình cuối cùng (đã gộp override và thay biến .env)
docker compose config --services # liệt kê các service
```

Lệnh báo lỗi là file sai cú pháp YAML. Nếu thấy cảnh báo
`The "XYZ" variable is not set` thì `.env` đang thiếu biến `XYZ`.

> `docker compose config` in cả secret ra màn hình, không copy kết quả đi chỗ khác.

### Bước 6: Khởi động và kiểm tra

```bash
docker compose up -d
docker compose ps                # tất cả service phải ở trạng thái running / healthy
docker compose logs -f <service> # xem log nếu có service lỗi
```

### Checklist

- [ ] Thư mục chạy nằm ngoài repo mission-control.
- [ ] Có `name:` trong file compose (nếu tự viết).
- [ ] `.env` đã điền đủ biến, không bị commit.
- [ ] Tùy chỉnh riêng nằm trong `docker-compose.override.yml`.
- [ ] `docker compose config` chạy không lỗi.
- [ ] `docker compose ps` thấy tất cả service đang chạy.

## Lệnh thường dùng

Chạy trong thư mục chứa `docker-compose.yml`:

| Lệnh | Tác dụng |
|---|---|
| `docker compose up -d` | Khởi động toàn bộ service ở chế độ nền |
| `docker compose ps` | Xem trạng thái các service |
| `docker compose logs -f <service>` | Xem log của một service |
| `docker compose restart <service>` | Khởi động lại một service |
| `docker compose config` | Kiểm tra và xem cấu hình cuối cùng |
| `docker compose pull` | Tải image mới theo file compose |
| `docker compose down` | Dừng và xóa container (giữ volume, tức giữ dữ liệu) |
| `docker compose down -v` | Dừng và **xóa cả volume**. Mất dữ liệu, cẩn thận |

## Quy ước chung trong mission-control

- **Không commit file `.env`** hoặc bất kỳ file nào chứa token, mật khẩu, secret.
  Chỉ commit file mẫu (ví dụ `env.example`).
- Mỗi dự án có thư mục chạy riêng, nằm ngoài repo, để `docker compose` của các dự án không lẫn vào nhau.
- Tùy chỉnh riêng đặt trong `docker-compose.override.yml`, hạn chế sửa file compose upstream.
- Trước khi nâng cấp phiên bản, đọc phần "Nâng cấp" trong hướng dẫn của dự án và backup dữ liệu.

## Xử lý sự cố thường gặp

| Hiện tượng | Cách xử lý |
|---|---|
| Windows: `WSL 2 installation is incomplete` | Chạy `wsl --update` trong PowerShell (Administrator), rồi khởi động lại Docker Desktop |
| Windows: `Virtualization support not detected` | Bật Intel VT-x / AMD-V trong BIOS/UEFI |
| Windows: `user is not in the docker-users group` | Chạy `net localgroup docker-users <tên-user> /add` (Administrator), rồi đăng xuất và đăng nhập lại |
| `Cannot connect to the Docker daemon` | Docker Desktop chưa chạy (Windows/macOS) hoặc chạy `sudo systemctl start docker` (Linux) |
| `permission denied ... docker.sock` | Linux: chưa vào nhóm `docker`: chạy `sudo usermod -aG docker $USER` rồi đăng nhập lại |
| `port is already allocated` | Cổng bị chiếm: tìm bằng `sudo lsof -i :<cổng>` (Linux/macOS) hoặc `netstat -ano \| findstr :<cổng>` (Windows), tắt tiến trình đó hoặc đổi cổng trong file override |
| `docker-compose: command not found` | Dùng `docker compose` (v2), hoặc cài `docker-compose-plugin` |
| `The "XYZ" variable is not set` | File `.env` thiếu biến `XYZ`, hoặc `.env` không nằm cùng thư mục với file compose |
| Dữ liệu "biến mất" sau khi đổi tên thư mục | Compose tạo volume mới theo tên thư mục mới. Đổi lại tên cũ, hoặc khai báo `name:` bằng tên cũ |
| Container tự khởi động lại liên tục | Xem `docker compose logs <service>`; thường do thiếu RAM hoặc sai biến môi trường |
