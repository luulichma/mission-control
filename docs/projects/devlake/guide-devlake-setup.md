# Hướng dẫn cài DevLake bằng Docker Compose

Dựa theo tài liệu chính thức:
https://devlake.apache.org/docs/GettingStarted/DockerComposeSetup

## Điều kiện trước

- Đã cài Docker và Docker Compose theo
  [guide-docker-compose-setup.md](../../general/guide-docker-compose-setup.md).
- Có `openssl` (macOS/Linux có sẵn; Windows dùng Git Bash hoặc WSL).
- Các cổng `4000`, `3002`, `8080`, `3306` chưa bị chiếm.

## Bước 1: Tải file cài đặt

Tạo thư mục chạy riêng cho DevLake, **nằm ngoài repo mission-control** để không lỡ commit
file `.env` chứa secret:

```bash
mkdir -p ~/devlake && cd ~/devlake
```

Tải `docker-compose.yml` và `env.example` từ bản release mới nhất tại
https://github.com/apache/devlake/releases (mục *Assets*). Ví dụ với v1.0.3:

```bash
VERSION=v1.0.3
curl -fL -O https://github.com/apache/devlake/releases/download/$VERSION/docker-compose.yml
curl -fL -O https://github.com/apache/devlake/releases/download/$VERSION/env.example
```

> Nếu `curl` báo 404 thì tên file trong release đã đổi. Mở trang release và tải tay hai file trên.

## Bước 2: Tạo file `.env`

```bash
mv env.example .env
```

## Bước 3: Tạo khóa mã hóa `ENCRYPTION_SECRET`

DevLake dùng khóa này để mã hóa token và mật khẩu lưu trong database.

```bash
openssl rand -base64 2000 | tr -dc 'A-Z' | fold -w 128 | head -n 1
```

Copy chuỗi vừa tạo và gán vào `.env`:

```bash
ENCRYPTION_SECRET="<chuỗi vừa tạo>"
```

> **Quan trọng:** lưu `ENCRYPTION_SECRET` vào nơi an toàn, ví dụ trình quản lý mật khẩu.
> Mất khóa này thì không giải mã được token đã lưu, phải nhập lại toàn bộ kết nối.
> Nếu đặt biến này ở cả `docker-compose.yml` và `.env` thì giá trị trong `docker-compose.yml`
> được ưu tiên.

## Bước 4: Chỉnh múi giờ và bảo mật (khuyến nghị)

Mở `docker-compose.yml`:

- **Múi giờ:** mặc định là `UTC`. Đổi `TZ: UTC` thành `TZ: Asia/Ho_Chi_Minh` ở các service
  để thời gian trên dashboard khớp giờ Việt Nam.
- **Mật khẩu cho config-ui:** mặc định config-ui **không có đăng nhập**. Nếu máy chạy DevLake
  truy cập được từ mạng ngoài, bỏ comment và đặt giá trị cho 2 dòng sau trong service `config-ui`:

  ```yaml
  ADMIN_USER: <tên đăng nhập>
  ADMIN_PASS: <mật khẩu mạnh>
  ```

- **Mật khẩu Grafana:** mặc định là `admin` / `admin`. Đổi ngay sau lần đăng nhập đầu tiên.

## Bước 5: Khởi động

```bash
docker compose up -d
docker compose ps      # đợi tất cả service ở trạng thái running
```

Container `devlake` cần một lúc để khởi động xong (lần đầu chạy migration database).

## Bước 6: Kiểm tra

| Địa chỉ | Nội dung |
|---|---|
| http://localhost:4000 | Config UI: cấu hình kết nối và thu thập dữ liệu |
| http://localhost:3002 | Grafana: đăng nhập `admin` / `admin` lần đầu |

Nếu config-ui báo không kết nối được API: đợi vài giây rồi tải lại trang.
Vẫn lỗi thì xem log bằng `docker compose logs -f devlake`.

Bước tiếp theo: [kết nối GitHub](guide-github-connection.md).

## Nâng cấp phiên bản

1. **Backup dashboard Grafana** nếu đã tùy chỉnh (Export JSON), vì nâng cấp có thể ghi đè.
2. Dừng service:
   ```bash
   docker compose down
   ```
3. Tải `docker-compose.yml` và `env.example` của bản mới (như Bước 1).
4. So sánh file mới với file cũ. Giữ lại các giá trị đã tùy chỉnh (`ENCRYPTION_SECRET`, `TZ`,
   `ADMIN_USER`, `ADMIN_PASS`...).
5. Khởi động lại:
   ```bash
   docker compose up -d
   ```

> Dữ liệu nằm trong Docker volume nên `docker compose down` không làm mất dữ liệu.
> **Không** dùng `docker compose down -v` trừ khi muốn xóa sạch.

## Sự cố thường gặp

| Hiện tượng | Cách xử lý |
|---|---|
| config-ui báo "API unreachable" | Đợi container `devlake` khởi động xong; xem `docker compose logs -f devlake` |
| `devlake` khởi động lại liên tục | Kiểm tra `ENCRYPTION_SECRET` trong `.env` đã có giá trị chưa; kiểm tra RAM |
| Cổng `3306` bị chiếm (đã có MySQL trên máy) | Đổi `3306:3306` thành `3307:3306` ở service `mysql`, hoặc bỏ phần `ports` của `mysql` |
| Token không giải mã được sau khi cài lại | `ENCRYPTION_SECRET` đã đổi. Dùng lại khóa cũ hoặc nhập lại token cho từng kết nối |

Thêm: https://devlake.apache.org/docs/Troubleshooting/Installation
