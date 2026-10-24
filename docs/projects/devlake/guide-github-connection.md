# Hướng dẫn kết nối GitHub và thu thập dữ liệu

Dựa theo tài liệu chính thức: https://devlake.apache.org/docs/Configuration/GitHub

## Điều kiện trước

- DevLake đã chạy theo [guide-devlake-setup.md](guide-devlake-setup.md).
- Có quyền đọc các repo cần theo dõi.

## Bước 1: Tạo GitHub token

Có 2 loại token. **Khuyến nghị dùng Classic token**: chạy được GraphQL API nên thu thập
nhanh hơn hơn 10 lần.

### Cách A: Personal access token (classic), khuyến nghị

1. Vào GitHub → **Settings → Developer settings → Personal access tokens → Tokens (classic)**
   → **Generate new token (classic)**.
2. Chọn quyền (scope):

   | Trường hợp | Scope cần chọn |
   |---|---|
   | Chỉ repo public | `repo:status`, `repo_deployment`, `read:user`, `read:org` |
   | Có repo private | `repo` (toàn bộ), `read:user`, `read:org` |

3. Đặt hạn dùng (Expiration), tạo token và copy ngay (GitHub chỉ hiện một lần).

> Nếu repo thuộc organization: org phải cho phép dùng Personal Access Token.
> Xem https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/setting-a-personal-access-token-policy-for-your-organization

### Cách B: Fine-grained token (chỉ đọc, giới hạn repo)

Dùng khi không muốn cấp toàn quyền `repo`. Cấp quyền **Read-only** cho: `Actions`,
`Contents`, `Discussions`, `Issues`, `Metadata`, `Pull requests`.

> Fine-grained token **không hỗ trợ GraphQL**. Phải tắt *Use GraphQL APIs* ở Bước 2,
> và thu thập sẽ chậm hơn nhiều.

### Giới hạn tốc độ

Mỗi token được 5.000 request/giờ (GitHub Enterprise: 15.000). Muốn nhanh hơn thì nhập
nhiều token, nhưng các token phải thuộc **các tài khoản GitHub khác nhau**, vì token cùng
tài khoản dùng chung giới hạn. Nếu bật GraphQL thì thường một token là đủ.

## Bước 2: Tạo kết nối (Connection)

Mở http://localhost:4000 → **Connections** → chọn **GitHub** → tạo kết nối mới:

| Trường | Giá trị |
|---|---|
| Connection Name | Tên dễ nhận biết, ví dụ `github-<tên-org-hoặc-user>` |
| Endpoint URL | `https://api.github.com/` (phải có dấu `/` ở cuối) |
| Personal Access Token(s) | Token tạo ở Bước 1 |
| Use GraphQL APIs | **Bật** (tắt nếu dùng fine-grained token) |
| Proxy URL | Để trống, trừ khi mạng công ty bắt buộc dùng proxy |
| Fixed Rate Limit | Để trống (DevLake tự điều chỉnh) |

Bấm **Test Connection**. Thành công thì bấm **Save Connection**.

## Bước 3: Chọn repo cần theo dõi (Data Scope)

Trong kết nối vừa tạo, chọn các repo cần thu thập bằng cách duyệt theo org/user hoặc tìm
kiếm. Lưu ý: ô tìm kiếm chỉ thêm được repo public; repo private chọn bằng cách duyệt.

## Bước 4: Cấu hình cách đọc dữ liệu (Scope Config), khuyến nghị

Không bắt buộc, nhưng **nên cấu hình** để phân tích đầu việc chính xác. Mỗi repo có tối
đa một Scope Config, và có thể dùng chung một config cho nhiều repo cùng quy ước label.

**Dữ liệu thu thập:** giữ mặc định (Issue Tracking, Source Code Management, Code Review,
CI/CD, Cross Domain). Có thể bỏ bớt mục không cần để thu thập nhanh hơn.

**Issue Tracking**: phân loại issue dựa vào label (dùng regex):

| Trường | Ý nghĩa | Ví dụ regex |
|---|---|---|
| Type/Requirement | Issue là yêu cầu hoặc tính năng | `(feature\|enhancement)` |
| Type/Bug | Issue là bug. Cần có để dùng dashboard "Weekly Bug Retro" | `bug` |
| Type/Incident | Issue là sự cố | `incident` |
| Priority | Lấy độ ưu tiên từ label | label `priority/p0` → `priority/(.*)$` |
| Severity / Component | Tương tự Priority | label `severity/high` → `severity/(.*)$` |

**CI/CD**: xác định deployment, cần cho chỉ số DORA:

- Nếu deploy bằng GitHub Deployments: nhập regex khớp tên environment production, ví dụ `(?i)prod`.
- Nếu deploy bằng workflow GitHub Actions: bật tùy chọn chuyển workflow run thành deployment, rồi nhập:
  - Regex nhận diện deployment (khớp tên workflow hoặc job), ví dụ `(?i).*deploy.*`
  - Regex nhận diện production, ví dụ `(?i).*prod.*`. Để trống nghĩa là coi mọi deployment là production.

**Code Review**: phân loại PR theo label, ví dụ label `type/feature` → `type/(.*)$`.

Bấm **Save**.

> **Gợi ý cho mission-control:** thống nhất một bộ label chung cho mọi repo (ví dụ `type/bug`,
> `type/feature`, `priority/p0..p3`) để dùng chung một Scope Config và so sánh được giữa các repo.

## Bước 5: Tạo Project và thu thập dữ liệu

1. Vào **Projects** → **Create Project**. Gợi ý: một project trong DevLake ứng với một dự án
   cần theo dõi.
2. Thêm GitHub connection vào project và chọn các repo thuộc dự án đó.
3. Cấu hình **Sync Policy**:

   | Mục | Gợi ý |
   |---|---|
   | Data Time Range | Mặc định 6 tháng gần nhất. Lần đầu nên giữ ngắn để chạy nhanh |
   | Sync Frequency | Hằng ngày, hoặc nhập cron, ví dụ `0 1 * * *` (1 giờ sáng mỗi ngày) |
   | Skip Failed Tasks | Bật, để pipeline dài không phải chạy lại từ đầu khi lỗi vài task |

4. Bấm **Collect Data**. Theo dõi tiến độ ở tab **Status** của project.

## Bước 6: Xem dashboard

Mở http://localhost:3002 (hoặc bấm *View Dashboards* trong config-ui). Các dashboard nên
xem trước:

- **GitHub**: tổng quan issue, PR, commit theo repo.
- **DORA**: tần suất deploy, lead time, tỉ lệ lỗi. Cần cấu hình CI/CD ở Bước 4.
- **Weekly Bug Retro**: cần cấu hình Type/Bug ở Bước 4.

## Sự cố thường gặp

| Hiện tượng | Cách xử lý |
|---|---|
| Test Connection báo lỗi quyền | Kiểm tra scope token. Từ v0.18 DevLake tự kiểm tra và báo scope còn thiếu |
| Không thấy repo private khi chọn scope | Token thiếu scope `repo`, hoặc org chưa cho phép PAT |
| Thu thập rất chậm | Bật GraphQL (cần classic token), thêm token từ tài khoản khác, hoặc giảm Data Time Range |
| Dashboard trống | Đợi pipeline chạy xong; kiểm tra khoảng thời gian (time range) trên Grafana |
| Dashboard DORA / Bug Retro trống | Chưa cấu hình Scope Config ở Bước 4 |

Thêm: https://devlake.apache.org/docs/Troubleshooting/Configuration
