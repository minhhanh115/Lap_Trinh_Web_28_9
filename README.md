# Bài tập Lập trình Web 28-9

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
```

## Phần 1: Triển khai hệ thống (Bài tập 1)

### 1. Môi trường triển khai
*   **Hệ điều hành:** Giả lập Linux sử dụng WSL (Windows Subsystem for Linux - Ubuntu).

<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/868b0b41-9852-4861-8369-2b7346f1fe48" />

*   **Công cụ ảo hóa: Docker Engine và Docker Compose (Phiên bản 3.8). Toàn bộ dự án được triển khai tại thư mục Home của người dùng để tối ưu quyền đọc/ghi (~/my_server).
  
### 2. Các dịch vụ (Services) được cấu hình

Hệ thống sử dụng file docker-compose.yml để khởi chạy đồng thời 5 dịch vụ lõi trên cùng một mạng nội bộ (server_network):
1.  **Nginx:** Đóng vai trò Web Server phục vụ file tĩnh (HTML/CSS/JS) và Reverse Proxy định tuyến API.

 ``` text
 nginx:
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
      - server_network
```
2.  **Node-RED:** Nền tảng lập trình luồng để thiết kế và xử lý API backend.

```text
nodered:
    image: nodered/node-red:latest
    container_name: nodered
    ports:
      - "1880:1880"
    volumes:
      - nodered-data:/data
    restart: unless-stopped
    networks:
      - server_network
```

3.  **MariaDB:** Hệ quản trị cơ sở dữ liệu quan hệ.

```text
mariadb:
    image: mariadb:10.11
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: hanh
      MYSQL_DATABASE: my_database
      MYSQL_USER: hanh_user
      MYSQL_PASSWORD: mhanh115
    volumes:
      - mariadb-data:/var/lib/mysql
    restart: unless-stopped
    networks:
      - server_network
```
4.  **phpMyAdmin:** Công cụ quản trị cơ sở dữ liệu trực quan trên nền web.

```text
phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: hanh
    depends_on:
      - mariadb
    restart: unless-stopped
    networks:
      - server_network
```
5.  **Cloudflared:** Sử dụng Cloudflare Tunnel để đưa các dịch vụ local ra môi trường Internet an toàn thông qua tên miền thật.

```text
 cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    command: tunnel --no-autoupdate run --token eyJhIjoiOTQ2MTQ0ZGRjZTYxYzlkMmVmMjk2YmVkYmY2YzExNzQiLCJ0IjoiNTE4NTFiNmYtY2ZmMi00MzE5LThjMDQtMzQzOGJkMzU2YzUxIiwicyI6IlkyUTVZemRrWW1ZdE9ETmlaQzAwTVdaaExUazNOVEl0TU>    restart: unless-stopped
    networks:
      - server_network

volumes:
  nodered-data:
  mariadb-data:

networks:
  server_network:
    driver: bridge
```

### 3. Hai website với 2 domain khác nhau.
Sử dụng tính năng Virtual Host để một container Nginx có thể phục vụ 2 tên miền độc lập.
