# Hướng dẫn cài Docker và Docker Compose

Hướng dẫn chung cho mọi dự án trong mission-control chạy bằng Docker Compose
(ví dụ DevLake). Hướng dẫn riêng của từng dự án nằm trong `docs/projects/<tên-dự-án>/`.

## Yêu cầu

- Docker **v19.03.10+** và Docker Compose **v2.2.3+**. Đây là mức tối thiểu của DevLake;
  nên dùng bản mới nhất.
- RAM: nên có từ 8 GB trở lên khi chạy DevLake (MySQL + Grafana + backend).
- Dùng plugin `docker compose` (v2, có dấu cách), không dùng `docker-compose` (v1) đã ngừng
  hỗ trợ.

## Cài đặt

### macOS / Windows

Cài [Docker Desktop](https://docs.docker.com/get-docker/). Docker Desktop đã kèm sẵn
Compose plugin.

- Windows: bật WSL 2 khi được hỏi.
- Vào Settings → Resources, cấp tối thiểu 8 GB RAM cho Docker.

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

# 4. Chạy docker không cần sudo (đăng xuất rồi đăng nhập lại sau lệnh này)
sudo usermod -aG docker $USER
```

Với Debian, thay `ubuntu` bằng `debian` trong 2 URL trên. Các distro khác: xem
https://docs.docker.com/engine/install/.

## Kiểm tra

```bash
docker --version          # >= 19.03.10
docker compose version    # >= v2.2.3
docker run --rm hello-world
```

## Lệnh thường dùng

Chạy trong thư mục chứa `docker-compose.yml`:

| Lệnh | Tác dụng |
|---|---|
| `docker compose up -d` | Khởi động toàn bộ service ở chế độ nền |
| `docker compose ps` | Xem trạng thái các service |
| `docker compose logs -f <service>` | Xem log của một service |
| `docker compose restart <service>` | Khởi động lại một service |
| `docker compose pull` | Tải image mới theo file compose |
| `docker compose down` | Dừng và xóa container (giữ volume, tức giữ dữ liệu) |
| `docker compose down -v` | Dừng và **xóa cả volume**. Mất dữ liệu, cẩn thận |

## Quy ước chung trong mission-control

- **Không commit file `.env`** hoặc bất kỳ file nào chứa token, mật khẩu, secret.
  Chỉ commit file mẫu (ví dụ `env.example`).
- Mỗi dự án có thư mục chạy riêng, để `docker compose` của các dự án không lẫn vào nhau.
- Trước khi nâng cấp phiên bản, đọc phần "Nâng cấp" trong hướng dẫn của dự án và backup dữ liệu.

## Xử lý sự cố thường gặp

| Hiện tượng | Cách xử lý |
|---|---|
| `permission denied ... docker.sock` | Chưa vào nhóm `docker`: chạy `sudo usermod -aG docker $USER` rồi đăng nhập lại |
| `port is already allocated` | Cổng bị chiếm: tìm bằng `sudo lsof -i :<cổng>`, tắt tiến trình đó hoặc đổi cổng trong file compose |
| `docker-compose: command not found` | Dùng `docker compose` (v2), hoặc cài `docker-compose-plugin` |
| Container tự khởi động lại liên tục | Xem `docker compose logs <service>`; thường do thiếu RAM hoặc sai biến môi trường |
