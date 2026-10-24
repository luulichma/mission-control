# CLAUDE.md — mission-control

Hướng dẫn cho AI agent (Claude Code và các agent khác) khi làm việc trong repo này.

## Quy tắc làm việc

- **Chỉ làm việc trên nhánh `main`.** Commit và push trực tiếp lên `origin/main`.
  Không tạo nhánh feature, không tạo pull request trừ khi chủ repo yêu cầu rõ ràng.
- Trước khi làm việc: `git checkout main && git pull origin main`.

## Bối cảnh dự án

- **Tên:** mission-control
- **Mục tiêu (theo README):** "AI-powered developer control center" — trung tâm điều khiển
  cho developer, dùng AI để quản lý nhiều repository, workflow, task và pipeline CI/CD.
- **License:** MIT (Copyright 2026 nguyenhue).

## Trạng thái hiện tại (2026-10-24)

- Repo mới khởi tạo, chỉ có 1 commit ("Initial commit") với `README.md`, `LICENSE`, `.gitignore`.
- **Chưa có mã nguồn**, chưa có build/test/lint, chưa có CI.
- `.gitignore` là template Python chuẩn của GitHub → dự kiến stack chính là **Python**
  (chưa được xác nhận; chưa có `pyproject.toml`/`requirements.txt`).

## Ghi chú cho agent

- Khi thêm code/tooling đầu tiên, cập nhật file này: stack, cấu trúc thư mục,
  lệnh build/test/lint, và quy ước code.
- Giữ file này ngắn gọn và đúng với thực tế của repo.
