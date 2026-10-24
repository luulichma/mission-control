# Quyết định 001: Dùng Apache DevLake làm nền tảng

- **Ngày:** 2026-10-24
- **Trạng thái:** Đã chấp nhận (giai đoạn dùng thử)

## Bối cảnh

mission-control cần theo dõi tiến độ của nhiều repository và phân tích các đầu việc
cần làm (issue, PR, CI/CD). Thay vì tự xây phần thu thập dữ liệu và dashboard từ đầu,
cần một nền tảng có sẵn để đi nhanh.

## Quyết định

1. **Dùng Apache DevLake làm nền tảng chính.** DevLake là dự án open-source
   (Apache License 2.0), đã có sẵn:
   - Kết nối GitHub, GitLab, Jira và các công cụ CI/CD.
   - Thu thập và chuẩn hóa dữ liệu của nhiều repo vào một nơi.
   - Dashboard Grafana cho tiến độ, PR, issue và chỉ số DORA.
2. **Thử dùng trước, cải tiến sau.** Trong thời gian dùng thử, vấn đề nào phát sinh
   sẽ được ghi lại và cải tiến dần trong mission-control (cấu hình, dashboard riêng,
   script bổ sung, phân tích đầu việc bằng AI...).

## Hệ quả

- Không tự xây lại phần thu thập dữ liệu và dashboard mà DevLake đã làm tốt.
- mission-control tập trung vào phần DevLake chưa có, đặc biệt là phân tích và gợi ý
  đầu việc.
- Phụ thuộc vào DevLake (cách triển khai, phiên bản, data model). Nếu DevLake không
  đáp ứng được thì xem xét lại quyết định này.

## Ghi nhận trong thời gian dùng thử

| Ngày | Vấn đề | Hướng cải tiến | Trạng thái |
|---|---|---|---|
| | | | |
