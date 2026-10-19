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

1. Sửa lỗi permission denied của Docker trên server (BẮT BUỘC)

Tài khoản azureuser chưa có quyền chạy Docker không cần sudo, dẫn đến pipeline khi SSH vào server sẽ bị chặn quyền.

👉 Trên terminal server (azureuser@mayao2:~$), bạn chạy 2 lệnh sau:

bash
sudo usermod -aG docker azureuser
newgrp docker

Sau đó gõ lại:

bash
docker ps

Khi màn hình hiện ra dòng tiêu đề CONTAINER ID   IMAGE   COMMAND ... (không còn báo permission denied) là đã xong bước này!

2. Về lỗi No such file or directory và couldn't find env file

Lý do chưa có thư mục này là vì:

Job Deploy lên server ở lần chạy trước bị lỗi nên chưa kịp tự clone mã nguồn về.
Tên repository trên GitHub của bạn là final-de003-buituangiang (nên khi deploy thành công, thư mục tạo ra trên server sẽ tên là final-de003-buituangiang).
3. Kích hoạt lại Pipeline CI/CD
Đảm bảo bạn đã thêm đủ 3 Secret (SERVER_HOST, SERVER_USER, SERVER_SSH_KEY) tại: 👉 https://github.com/AIappliacation/final-de003-buituangiang/settings/secrets/actions
Trên giao diện GitHub Actions (ở ảnh bạn vừa chụp):
Bấm nút Re-run jobs ở góc trên bên phải → chọn Re-run all jobs (hoặc bấm vào nút Latest #2 bên cạnh để xem kết quả lần chạy thứ 2 nếu đã chạy).

Khi job Deploy lên server báo xanh ✅:

Trên server sẽ tự động có thư mục final-de003-buituangiang.
Bạn vào thư mục đó để kiểm tra container:
bash
cd ~/final-de003-buituangiang
docker compose --env-file student.env ps