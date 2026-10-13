# BÁO CÁO CHẨN ĐOÁN VÀ KHẮC PHỤC SỰ CỐ PIPELINE CI/CD

## 1. Chi tiết phân tích & Khắc phục 3 lỗi

### Lỗi 1: Permission Denied khi thực thi Gradle Wrapper
- **Mô tả lỗi:** 
  Runner ném ngoại lệ: `/bin/bash: ./gradlew: Permission denied` khi thực hiện câu lệnh `./gradlew test`.
- **Nguyên nhân gốc rễ:** 
  Tệp `gradlew` không giữ cờ thực thi (`executable bit`) khi đẩy từ hệ điều hành Windows lên Git repository.
- **Biện pháp xử lý:** 
  Bổ sung bước cấp quyền `chmod +x` trước khi gọi lệnh gradle:
  ```yaml
  - name: Cấp quyền thực thi cho gradlew
    run: chmod +x gradlew
  ```

---

### Lỗi 2: Authentication Failed khi Đăng nhập Docker Registry
- **Mô tả lỗi:** 
  Bước `docker/login-action` báo lỗi `Error: Username and password required` hoặc `unauthorized: authentication required`.
- **Nguyên nhân gốc rễ:** 
  Tệp kịch bản gọi sai tên biến bí mật (`secrets.REGISTRY_TOKEN` không tồn tại hoặc sai chính tả).
- **Biện pháp xử lý:** 
  Sử dụng token mặc định hệ thống `secrets.GITHUB_TOKEN` kèm quyền `packages: write`:
  ```yaml
  - name: Đăng nhập GitHub Container Registry
    uses: docker/login-action@v3
    with:
      registry: ${{ env.REGISTRY }}
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
  ```

---

### Lỗi 3: Build Context Error (Không tìm thấy Dockerfile)
- **Mô tả lỗi:** 
  Lỗi `failed to read dockerfile: open Dockerfile: no such file or directory` xuất hiện tại bước `docker/build-push-action`.
- **Nguyên nhân gốc rễ:** 
  Khai báo sai đường dẫn thư mục ngữ cảnh build (`context`) hoặc trỏ sai đường dẫn tệp Dockerfile.
- **Biện pháp xử lý:** 
  Cấu hình tường minh `context: .` (ngữ cảnh tại gốc repo) và `file: ./Dockerfile`:
  ```yaml
  - name: Build and Push Docker Image
    uses: docker/build-push-action@v5
    with:
      context: .
      file: ./Dockerfile
      push: true
      tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
  ```

---

## 2. Kết quả Nghiệm thu (Verification)

Sau khi chỉnh sửa và đẩy commit mới lên GitHub:
- Toàn bộ 2 jobs (`build-and-test` và `docker-build`) hoàn thành thuận lợi.
- Pipeline chuyển trạng thái hoàn toàn sang **Success** (Tích xanh).

//

---

## 3. Kết luận
- Pipeline khôi phục trạng thái hoạt động tự động liên tục.
- Nắm vững cách đọc log chi tiết để xác định các lỗi phổ biến về phân quyền, biến môi trường bí mật và ngữ cảnh build container.