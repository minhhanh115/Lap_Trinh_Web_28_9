# Báo cáo Bài tập Lập trình Web 28-9

**Sinh viên :** Nguyễn Minh Hạnh  
**Msv:** K235480106023  
**Lớp:** K59KMT

## Cấu trúc thư mục dự án (Directory Structure)
```text
my_server/
├── docker-compose.yml            # Tệp cấu hình gốc khởi tạo các dịch vụ
├── nginx/
│   ├── conf.d/                   # Chứa file cấu hình Virtual Host
│   │   ├── hihi.conf             # Cấu hình domain hihi.mhanh.id.vn
│   │   └── haha.conf             # Cấu hình domain haha.mhanh.id.vn
│   └── html/                     # Nơi chứa mã nguồn frontend
│       ├── hihi.mhanh.id.vn/
│       │   └── index.html        # Giao diện chính và mã gọi API (JS)
│       └── haha.mhanh.id.vn/
│           └── index.html        # Giao diện phụ
└── nodered/
    └── flows.json                # Tệp JSON sao lưu luồng API của Node-RED

## Phần 1: Triển khai hệ thống (Bài tập 1)

### 1. Môi trường triển khai
*   **Hệ điều hành:** Giả lập Linux sử dụng WSL (Windows Subsystem for Linux - Ubuntu).

<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/868b0b41-9852-4861-8369-2b7346f1fe48" />

### 2. Các dịch vụ (Services) được cấu hình
*   **Công cụ quản lý:** Docker và Docker Compose.
Hệ thống chạy 5 dịch vụ lõi thông qua file `docker-compose.yml`:
1.  **Nginx:** Đóng vai trò Web Server phục vụ file tĩnh (HTML/CSS/JS) và Reverse Proxy định tuyến API.

 ``` nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html
    restart: unless-stopped
    networks:
      - server_network```
2.  **Node-RED:** Nền tảng lập trình luồng để thiết kế và xử lý API backend.
3.  **MariaDB:** Hệ quản trị cơ sở dữ liệu quan hệ.
4.  **phpMyAdmin:** Công cụ quản trị cơ sở dữ liệu trực quan trên nền web.
5.  **Cloudflared:** Sử dụng Cloudflare Tunnel để đưa các dịch vụ local ra môi trường Internet an toàn thông qua tên miền thật.
