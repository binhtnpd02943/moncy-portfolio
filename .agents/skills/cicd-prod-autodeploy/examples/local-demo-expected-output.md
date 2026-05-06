## Expected Output Shape

Đây không phải đáp án duy nhất, nhưng output tốt nên gần với cấu trúc dưới đây.

## 1. Assumptions And Discovery

- production target là Linux VM có quyền SSH từ CI runner
- ứng dụng chạy dưới `systemd`, ví dụ service `techzen-app.service`
- nginx đã route domain sẵn, pipeline không tự sửa nginx
- artifact `.jar` được build ở CI rồi copy sang host
- app mới phải dùng thư mục và service name riêng để không đụng 2 hệ thống đang tồn tại

Các điểm cần khảo sát thêm:

- service name chính xác
- port ứng dụng mới
- đường dẫn deploy, ví dụ `/opt/techzen-app`
- health check URL
- thông tin SSH deploy user
- cách quản lý `.env` hoặc file config production

## 2. CI/CD Flow

- `feature/*` -> PR vào `stg`
- CI trên `stg`: test, build, package artifact
- artifact nên gắn với commit SHA hoặc release ID rõ ràng
- team review trên staging
- tạo PR từ `stg` sang `main`
- chỉ PO hoặc circle lead được approve và merge
- merge vào `main` kích hoạt GitHub Actions production deploy
- production deploy nên được serialize để tránh 2 job chạy chồng nhau
- workflow upload `.jar` sang server
- `deploy.sh` tạo release mới, đổi symlink `current`, restart service, chạy health check
- nếu health check fail thì gọi `rollback.sh`

Output tốt cũng nên nói rõ trong case này không cần ép container hóa nếu target thực tế đang phù hợp hơn với `.jar` + `systemd` + release directory.

## 3. Branch Protection

- bật branch protection cho `main`
- require pull request before merge
- disable direct push
- require status checks pass trước merge
- require tối thiểu 1 approval
- restrict merge cho team hoặc user đại diện PO và circle lead

## 4. Example Workflow

Output tốt nên sinh được một file gần kiểu này:

```yaml
name: prod-deploy

on:
  push:
    branches:
      - main

concurrency:
  group: prod-deploy
  cancel-in-progress: false

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
      - name: Build jar
        run: ./gradlew clean build
      - name: Copy artifact
        run: scp build/libs/app-${{ github.sha }}.jar ${{ secrets.DEPLOY_USER }}@${{ secrets.DEPLOY_HOST }}:/tmp/app-${{ github.sha }}.jar
      - name: Deploy
        run: ssh ${{ secrets.DEPLOY_USER }}@${{ secrets.DEPLOY_HOST }} 'bash /opt/techzen-app/bin/deploy.sh'
```

## 5. Example Shell Scripts

Output tốt nên sinh được script có các đặc điểm:

- `#!/usr/bin/env bash`
- `set -Eeuo pipefail`
- validate env vars và binaries
- dùng release dir dạng timestamp
- cập nhật symlink `current`
- restart `systemd`
- health check bằng `curl`
- rollback về release trước nếu fail
- không đụng nginx dùng chung
- nhận artifact theo release ID hoặc commit SHA, không ghi đè mơ hồ

Ví dụ rút gọn:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

APP_ROOT=/opt/techzen-app
RELEASES_DIR="$APP_ROOT/releases"
CURRENT_LINK="$APP_ROOT/current"
SERVICE_NAME=techzen-app.service
HEALTHCHECK_URL=http://127.0.0.1:18080/actuator/health
```

## 6. Validate Checklist

- deploy không đụng cấu hình nginx dùng chung
- không reuse port của hệ thống khác
- `main` không nhận direct push
- merge `stg -> main` dẫn tới deploy prod
- artifact release có định danh rõ ràng
- không có 2 production deploy chạy song song
- service restart thành công
- health check pass
- rollback chạy được nếu release lỗi
