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

*   **Công cụ ảo hóa:** Docker Engine và Docker Compose (Phiên bản 3.8). Toàn bộ dự án được triển khai tại thư mục Home của người dùng để tối ưu quyền đọc/ghi (~/my_server).
  
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
# Hướng dẫn tạo 2 website với 2 domain khác nhau bằng Nginx + Docker

## 1. Tạo cấu trúc thư mục `my-server`

Mở terminal (WSL) và chạy các lệnh sau để tạo thư mục ngay trong Home của bạn:

```bash
mkdir -p ~/my-server/nginx/conf.d
mkdir -p ~/my-server/nginx/html/hihi.mhanh.id.vn
mkdir -p ~/my-server/nginx/html/haha.mhanh.id.vn
cd ~/my-server
```

## 2. Tạo giao diện cho trang HIHI

Sử dụng lệnh `nano` để tạo file HTML cho web HIHI:

```bash
nano nginx/html/hihi.mhanh.id.vn/index.html
```

Dán bộ code giao diện phong cách **Xanh Tím Hiện Đại** vào:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HIHI Domain - Hệ thống IoT</title>
    <style>
        body { margin: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); height: 100vh; display: flex; align-items: center; justify-content: center; color: #333; }
        .container { background: rgba(255, 255, 255, 0.95); padding: 50px 40px; border-radius: 20px; box-shadow: 0 15px 35px rgba(0,0,0,0.2); text-align: center; max-width: 450px; width: 90%; backdrop-filter: blur(10px); }
        h1 { color: #2d3748; margin-bottom: 15px; font-size: 2.2em; }
        p { color: #718096; font-size: 1.1em; line-height: 1.6; margin-bottom: 25px; }
        .badge { display: inline-block; background: #ebf4ff; color: #4c51bf; padding: 5px 15px; border-radius: 20px; font-weight: bold; font-size: 0.9em; margin-bottom: 15px; }
        .btn { display: inline-block; padding: 12px 30px; background: linear-gradient(to right, #667eea, #764ba2); color: white; text-decoration: none; border-radius: 25px; transition: 0.3s; font-weight: bold; box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4); }
        .btn:hover { transform: translateY(-3px); box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6); }
    </style>
</head>
<body>
    <div class="container">
        <span class="badge">Nginx & Docker 🐳</span>
        <h1>Chào mừng đến với HIHI</h1>
        <p>Trang web này đang chạy trên domain <b>hihi.mhanh.id.vn</b>. Hệ thống đang hoạt động trơn tru!</p>
        <a href="#" class="btn">Khám phá hệ thống</a>
    </div>
</body>
</html>
```

Lưu lại (`Ctrl+O` → `Enter` → `Ctrl+X`).

## 3. Tạo giao diện cho trang HAHA

Tạo file HTML cho web HAHA:

```bash
nano nginx/html/haha.mhanh.id.vn/index.html
```

Dán bộ code giao diện phong cách **Cam Đào Ấm Áp** vào:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HAHA Domain - Quản trị</title>
    <style>
        body { margin: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%, #fecfef 100%); height: 100vh; display: flex; align-items: center; justify-content: center; color: #333; }
        .container { background: white; padding: 50px 40px; border-radius: 20px; box-shadow: 0 15px 35px rgba(255, 154, 158, 0.3); text-align: center; max-width: 450px; width: 90%; }
        h1 { color: #e53e3e; margin-bottom: 15px; font-size: 2.2em; }
        p { color: #718096; font-size: 1.1em; line-height: 1.6; margin-bottom: 25px; }
        .badge { display: inline-block; background: #fff5f5; color: #c53030; padding: 5px 15px; border-radius: 20px; font-weight: bold; font-size: 0.9em; margin-bottom: 15px; }
        .btn { display: inline-block; padding: 12px 30px; background: linear-gradient(to right, #ff758c 0%, #ff7eb3 100%); color: white; text-decoration: none; border-radius: 25px; transition: 0.3s; font-weight: bold; box-shadow: 0 4px 15px rgba(255, 117, 140, 0.4); }
        .btn:hover { transform: translateY(-3px); box-shadow: 0 6px 20px rgba(255, 117, 140, 0.6); }
    </style>
</head>
<body>
    <div class="container">
        <span class="badge">Node-RED Server ⚙️</span>
        <h1>Domain HAHA xin chào!</h1>
        <p>Bạn đã truy cập thành công vào <b>haha.mhanh.id.vn</b>. Web server đã được định tuyến chính xác.</p>
        <a href="#" class="btn">Chuyển đến Bảng điều khiển</a>
    </div>
</body>
</html>
```

## 4. Cấu hình Nginx

> **Điểm mấu chốt để chạy 2 website với 2 domain khác nhau:** Nginx dùng cơ chế **virtual host** — mỗi domain có một file `.conf` riêng trong thư mục `conf.d`, với `server_name` khác nhau và `root` trỏ đến thư mục HTML riêng. Khi request đến, Nginx dựa vào header `Host` (tên domain) để chọn đúng file cấu hình và trả về đúng nội dung tương ứng.

**Tạo cấu hình cho hihi:**

```bash
nano nginx/conf.d/hihi.conf
```

```nginx
server {
    listen 80;
    server_name hihi.mhanh.id.vn;

    location / {
        root /usr/share/nginx/html/hihi.mhanh.id.vn;
        index index.html;
    }
}
```

**Tạo cấu hình cho haha:**

```bash
nano nginx/conf.d/haha.conf
```

```nginx
server {
    listen 80;
    server_name haha.mhanh.id.vn;

    location / {
        root /usr/share/nginx/html/haha.mhanh.id.vn;
        index index.html;
    }
}
```

## 5. Khởi chạy hệ thống

Nếu bạn chưa tạo file `docker-compose.yml` tại thư mục này, hãy tạo nó:

```bash
nano docker-compose.yml
```

> Dán nội dung file `docker-compose` cũ của bạn vào, lưu ý phần `volumes` của nginx phải là:
> ```yaml
> - ./nginx/conf.d:/etc/nginx/conf.d
> - ./nginx/html:/usr/share/nginx/html
> ```

Cuối cùng, khởi động mọi thứ:

```bash
docker-compose down
docker-compose up -d
```
