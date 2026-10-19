# DevOps Final – Đề 003: Quản lý công việc

Bộ mã nguồn khởi đầu cho bài thi **DevOps Fundamentals – Cuối kỳ**.
Ứng dụng hiển thị **danh sách công việc** lấy từ MySQL thông qua Spring Boot, trả về **dạng JSON** qua Nginx (reverse proxy).

## Kiến trúc

```
Trình duyệt ── http://<IP_máy_chủ>:<APP_PORT>
    ▼
[nginx]  nginx:1.27-alpine        GET /*  → app:8080
    ▼
[app]    Spring Boot 4 (JRE 21)   GET /api/tasks → mysql:3306
    ▼
[mysql]  mysql:8.0                volume mysql_data, KHÔNG public cổng mysql
```

## Cấu trúc thư mục

Đổi tên thư mục/repository thành `devops-final-de003-<họ-tên>` (ví dụ `devops-final-de003-nguyenvana`).
Dấu ★ là phần sinh viên phải làm.

```
devops-final-de003-nguyenvana/     ← Mã nguồn Spring Boot 4 (Java 21, Gradle) – CÓ SẴN
├── src/main/java/com/ptit/devops/task/   ← Controller, Entity, Repository
├── src/main/resources/application.properties
├── build.gradle                   ← Build ra file build/libs/app.jar
├── settings.gradle
├── Dockerfile                     ← ★ FILE KHUNG: tự hoàn chỉnh phần còn thiếu
├── .dockerignore                  ← CÓ SẴN
├── nginx/default.conf             ← ★ FILE KHUNG: tự đổi PORT (tránh trùng nhau)
├── database/init.sql              ← Bảng tasks + 8 bản ghi mẫu – CÓ SẴN
├── docker-compose.yml             ← ★ FILE KHUNG: mysql có sẵn, điền TODO 1–5
├── .github/workflows/ci-cd.yml    ← ★ FILE KHUNG: điền TODO 1–6
├── student.env                    ← ★ ĐIỀN THÔNG TIN SINH VIÊN
├── screenshots/                   ← ★ Ảnh minh chứng (Phần 5)
├── submission-content/            ← ★ Toàn bộ tệp minh chứng khi nộp bài
├── .gitignore                     ← CÓ SẴN
└── README.md
```

Phần **sinh viên phải làm** (xem đề thi):

| File | Việc cần làm |
|------|--------------|
| `student.env` | Điền thông tin sinh viên và thông số triển khai |
| `Dockerfile` | **Hoàn chỉnh**: multi-stage build, stage `build` (Gradle + JDK 21) và stage runtime (JRE 21) |
| `docker-compose.yml` | **Điền các TODO 1–5** (service `mysql` đã có sẵn) |
| `.github/workflows/ci-cd.yml` | **Điền các TODO 1–6** của pipeline build → push image → deploy |
| `screenshots/` | `01-actions.png`, `02-ghcr.png`, `03-compose-ps.png`, `04-website.png` |

## API của ứng dụng

| Method | Đường dẫn | Mô tả |
|--------|-----------|-------|
| GET | `/api/tasks` | Danh sách công việc |

## Biến môi trường mà ứng dụng cần

| Biến | Ý nghĩa |
|------|---------|
| `DB_HOST`, `DB_PORT` | Địa chỉ MySQL (mặc định `mysql`, `3306`) |
| `DB_NAME`, `DB_USER`, `DB_PASSWORD` | Database và tài khoản kết nối |
| `APP_VERSION` | Phiên bản (truyền vào lúc build image, ví dụ commit SHA) |

## Chạy thử (khi đã hoàn thiện Dockerfile và docker-compose.yml)

```bash
docker compose --env-file student.env up -d --build
docker compose --env-file student.env ps
curl http://localhost:<APP_PORT>/api/tasks
docker compose --env-file student.env down
```

BƯỚC 1: Tạo Repository trên GitHub
Mở trình duyệt, truy cập GitHub cá nhân của bạn: https://github.com/new
Tại mục Repository name, nhập đúng tên theo yêu cầu:
text


devops-final-de003-buituangiang
Tại mục chọn chế độ, chọn Public (bắt buộc theo đề bài).
KHÔNG tích chọn Add a README file, .gitignore hay license (vì code đã có sẵn).
Bấm nút Create repository.
BƯỚC 2: Cấu hình SSH Key trên Server
Tôi đã tạo sẵn cặp khóa SSH deploy_key ngay trong thư mục máy bạn.

Đăng nhập SSH vào server bằng terminal của bạn:
bash


ssh azureuser@20.18.24.239
Sau khi đã vào terminal của server, mở file ~/.ssh/authorized_keys:
bash


nano ~/.ssh/authorized_keys
Copy nguyên vẹn dòng public key bên dưới và paste vào cuối file authorized_keys:
text


ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINePJNwm5CjAdpKGlq/kWu4tsE7xH1yrVzFvrSHbfdWO giang@MSI
Nhấn Ctrl + O rồi Enter để lưu, sau đó nhấn Ctrl + X để thoát.
Đảm bảo user azureuser đã thuộc nhóm docker (để chạy docker không cần sudo):
bash


docker ps
(Nếu lệnh docker ps chạy được bình thường là đạt).
BƯỚC 3: Cấu hình Secrets và Quyền trên GitHub
1. Cấp quyền ghi gói (GHCR) cho GitHub Actions:
Trên trang repo GitHub vừa tạo, vào: Settings → chọn Actions (ở thanh menu bên trái) → General.
Kéo xuống mục Workflow permissions:
Tích chọn: Read and write permissions.
Bấm nút Save.
2. Thêm 3 GitHub Secrets:
Vẫn ở menu Settings → nhìn bên trái chọn Secrets and variables → chọn Actions.

Bấm nút New repository secret màu xanh lá để thêm lần lượt 3 secret:

Secret 1:

Name: SERVER_HOST
Secret: 20.18.24.239
Bấm Add secret.
Secret 2:

Name: SERVER_USER
Secret: azureuser
Bấm Add secret.
Secret 3:

Name: SERVER_SSH_KEY
Secret: Copy toàn bộ nội dung private key bên dưới (kể cả dòng BEGIN và END):
text


-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDXjyTcJuQowHaShpav5FruLbBO8R9cq1cxb60h233VjgAAAJAwYqAWMGKg
FgAAAAtzc2gtZWQyNTUxOQAAACDXjyTcJuQowHaShpav5FruLbBO8R9cq1cxb60h233Vjg
AAAEDEHpPmi9JECIABQ/5XBWYpECpT95wOSmTP/qBCZ1RM9NePJNwm5CjAdpKGlq/kWu4t
sE7xH1yrVzFvrSHbfdWOAAAACWdpYW5nQE1TSQECAwQ=
-----END OPENSSH PRIVATE KEY-----
Bấm Add secret.
BƯỚC 4: Đẩy (Push) code lên GitHub
Mở terminal tại máy tính của bạn (ngay thư mục dự án này) và chạy 2 lệnh sau:

bash


git remote add origin https://github.com/Btg1603/devops-final-de003-buituangiang.git
git push -u origin main
BƯỚC 5: Theo dõi Pipeline CI/CD
Lên lại trang GitHub repo của bạn, bấm vào tab Actions.
Bạn sẽ thấy workflow CI/CD đang chạy:
Job Build & Test: Đọc student.env → Build Docker image → Chạy smoke test → Push image lên GitHub Packages (GHCR).
Job Deploy: SSH vào server 20.18.24.239 → Pull image mới về → Khởi động container bằng Docker Compose.
Đợi cả 2 job hiện dấu tích xanh ✅ Success.
BƯỚC 6: Chụp 4 ảnh minh chứng nộp bài (Phần 5)
Lưu 4 ảnh vào thư mục screenshots/ và copy vào submission-content/:

01-actions.png: Chụp màn hình tab Actions trên GitHub với workflow chạy thành công cả 2 job build và deploy.
02-ghcr.png: Vào trang chủ GitHub profile của bạn hoặc mục Packages của repo, chụp màn hình container package app có đủ 2 tag: latest và commit SHA.
03-compose-ps.png: Mở terminal SSH trên server (ssh azureuser@20.18.24.239), vào thư mục dự án và gõ:
bash


cd devops-final-de003-buituangiang
docker compose --env-file student.env ps
Chụp lại kết quả hiển thị cả 3 service app, nginx, mysql đều Up / healthy.
04-website.png: Mở trình duyệt truy cập:
http://20.18.24.239:8080/api/tasks
Chụp màn hình trình duyệt hiển thị dữ liệu JSON danh sách 8 công việc.
