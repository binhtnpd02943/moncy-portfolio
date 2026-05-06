## Expected Output Shape

Output tốt nên gần với cấu trúc dưới đây.

## 1. Current Shape And Evidence

- runtime target hiện tại là Linux VM chạy Docker Compose
- production deploy trigger theo `push` vào `main`
- compose đang pull image `latest`
- deploy hiện tại là update tại chỗ qua SSH
- chưa thấy rollback contract và health verification rõ ràng
- môi trường là shared host nên rủi ro đụng service khác là có thật

## 2. Acceptable Parts

- workflow đã có CI/CD nền tảng ban đầu
- target runtime đã rõ
- deploy path SSH + Compose phù hợp nếu team chưa cần orchestration phức tạp

## 3. Risk By Severity

`Critical`
- production dùng `latest`, không có exact artifact hoặc digest
- chưa có rollback path thực tế

`High`
- không có health check sau deploy
- production deploy có thể chạy chồng nếu nhiều merge gần nhau
- deploy script chưa kiểm soát shared-host guardrails

`Medium`
- workflow tách CI và deploy chưa đủ rõ
- secret source hoặc identity model chưa được diễn đạt rõ

## 4. Recommended Target State

- giữ nguyên Linux VM + Docker Compose
- build image theo commit SHA
- deploy exact tag hoặc digest
- thêm health check, rollback script, concurrency control
- chỉ patch những phần nguy hiểm nhất, không rewrite toàn bộ

## 5. Smallest Patch Plan

- sửa workflow để build và push image theo SHA
- serialize production deploy
- sửa `docker-compose.prod.yml` dùng exact tag hoặc digest
- thêm `healthcheck.sh` và `rollback.sh`
- thêm validation để chỉ restart đúng service mục tiêu
