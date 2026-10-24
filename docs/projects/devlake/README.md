# DevLake

[Apache DevLake](https://github.com/apache/devlake) là nền tảng open-source (Apache License
2.0). Nó thu thập dữ liệu từ GitHub, GitLab, Jira, CI/CD... về một nơi và hiển thị bằng
dashboard Grafana (tiến độ, PR, issue, chỉ số DORA).

mission-control dùng DevLake làm nền tảng theo dõi tiến độ các repo. Lý do xem
[ADR-001](../../general/ADR-001-devlake-as-foundation.md).

## Thành phần

| Service | Vai trò | Cổng mặc định |
|---|---|---|
| `config-ui` | Giao diện cấu hình: kết nối, project, đồng bộ dữ liệu | `4000` |
| `grafana` | Dashboard | `3002` |
| `devlake` | Backend API, chạy pipeline thu thập dữ liệu | `8080` |
| `mysql` | Cơ sở dữ liệu | `3306` |

## Tài liệu

Làm theo thứ tự:

1. [Cài Docker và setup Docker Compose](../../general/guide-docker-compose-setup.md) (hướng dẫn chung)
2. [guide-devlake-setup.md](guide-devlake-setup.md): cài và chạy DevLake
3. [guide-github-connection.md](guide-github-connection.md): kết nối GitHub và thu thập dữ liệu các repo

## Thông tin phiên bản

- Phiên bản mới nhất khi viết tài liệu (2026-10-24): **v1.0.3**.
- Trang release: https://github.com/apache/devlake/releases
- Tài liệu chính thức: https://devlake.apache.org/docs/Overview/Introduction
