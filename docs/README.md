# Tài liệu mission-control

mission-control là trung tâm quản lý chung cho mọi dự án. Tài liệu được chia làm 2 phần:

- `general/`: tài liệu chung cho toàn mission-control (quyết định kiến trúc, hướng dẫn
  dùng chung như cài Docker Compose).
- `projects/`: tài liệu riêng của từng dự án hoặc công cụ, mỗi dự án một thư mục con.

## Mục lục

### Chung (`general/`)

| File | Nội dung |
|---|---|
| [ADR-001-devlake-as-foundation.md](general/ADR-001-devlake-as-foundation.md) | **Quyết định nền tảng:** dùng Apache DevLake làm nền |
| [guide-docker-compose-setup.md](general/guide-docker-compose-setup.md) | Cài Docker và Docker Compose |

### Dự án (`projects/`)

| Dự án | Cửa vào |
|---|---|
| DevLake | [projects/devlake/README.md](projects/devlake/README.md) |

## Quy chuẩn đặt tên

### Cấu trúc thư mục

```
docs/
├── README.md                    # File này: mục lục + quy chuẩn
├── general/                     # Chung cho toàn mission-control
│   └── <loại>-<chủ-đề>.md
└── projects/                    # Riêng từng dự án/công cụ
    └── <tên-dự-án>/
        ├── README.md            # Bắt buộc: giới thiệu + mục lục của dự án
        └── <loại>-<chủ-đề>.md
```

### Quy tắc

1. **Tên file và thư mục:** tiếng Anh, không dấu, chữ thường, nối bằng dấu gạch ngang
   (kebab-case). Ví dụ: `guide-docker-compose-setup.md`. Nội dung viết tiếng Việt.
2. **Tên file có dạng `<loại>-<chủ-đề>.md`.** Tiền tố cho biết loại tài liệu:

   | Tiền tố | Ý nghĩa | Ví dụ |
   |---|---|---|
   | `ADR-NNN-` | Quyết định kiến trúc. Viết hoa, đánh số 3 chữ số tăng dần, không dùng lại số cũ | `ADR-001-devlake-as-foundation.md` |
   | `guide-` | Hướng dẫn cài đặt, làm theo từng bước | `guide-devlake-setup.md` |
   | `runbook-` | Vận hành, xử lý sự cố | `runbook-devlake-restart.md` |
   | `ref-` | Tài liệu tra cứu (cấu hình, biến môi trường...) | `ref-devlake-env-vars.md` |

3. **Chủ đề** đi từ chung đến cụ thể: `<đối-tượng>-<hành-động>`.
   Ví dụ: `guide-github-connection.md`, không phải `guide-connect-to-github-step-by-step.md`.
4. **ADR** nằm ở `general/` nếu ảnh hưởng toàn mission-control, hoặc ở
   `projects/<tên-dự-án>/` nếu chỉ ảnh hưởng một dự án. Số ADR đánh chung toàn repo.
5. **Thêm dự án mới:** tạo `projects/<tên-dự-án>/README.md`, rồi thêm một dòng vào
   bảng "Dự án" ở trên.
6. **Thêm hoặc đổi tên file:** cập nhật mục lục trong file này (và `README.md` của dự án
   nếu có).
