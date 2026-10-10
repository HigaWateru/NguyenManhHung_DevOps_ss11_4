# Bài 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

## 1. Thông tin bài tập
- **Khóa học:** DevOps
- **Buổi học:** Session 11 - CI/CD với GitHub Actions
- **Bài thực hành:** Bài 4 (GitHub Secrets & GHCR Authentication)
- **Thư mục bài tập:** `ss11_4`

---

## 2. Kịch bản GitHub Actions (`.github/workflows/ci.yml`)

```yaml
name: CI Java Gradle and GHCR Login

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  test-and-login:
    name: Build, Test & Registry Login
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập môi trường JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # Cố định phiên bản Gradle 8.5 để tương thích với Spring Boot 3
      - name: Thiết lập Gradle 8.5
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.5'

      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Chạy Unit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      # --- BƯỚC ĐĂNG NHẬP CONTAINER REGISTRY CỦA BÀI 4 ---
      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 3. Kết quả đạt được
1. **Bảo mật tuyệt đối:** Không lưu trữ mật khẩu hay token dạng plaintext trong repository. Toàn bộ thông tin nhạy cảm được mã hóa qua cơ chế GitHub Repository Secrets với khóa `REGISTRY_TOKEN`.
2. **Xác thực tự động:** Action `docker/login-action@v3` tự động nạp danh tính `${{ github.actor }}` và bí mật `${{ secrets.REGISTRY_TOKEN }}` để kết nối máy chủ `ghcr.io`.
3. **Đăng nhập thành công:** Kết quả thực thi ghi nhận `Login Succeeded` trong log chi tiết, cho phép runner sẵn sàng cho các tác vụ đóng gói và đẩy container image.

---

## 4. Hình ảnh minh chứng nộp bài

### Ảnh 1: Khai báo Repository Secret `REGISTRY_TOKEN` thành công:
![Repository Secrets](image/screenshot_secret.png)

### Ảnh 2: Log bước Đăng nhập Registry hiển thị `Login Succeeded`:
![Login Succeeded](image/screenshot_login.png)
