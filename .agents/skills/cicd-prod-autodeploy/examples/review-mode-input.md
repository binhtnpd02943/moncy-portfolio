## Demo Goal

Use this prompt to test the audit and hardening mode of the skill.

## Input

Review khách quan pipeline CI/CD hiện tại cho một backend Node.js đã chạy production bằng Docker Compose trên Linux VM.

Hiện trạng:
- repo đã có `Dockerfile`, `docker-compose.prod.yml`, `.github/workflows/prod.yml`
- workflow đang deploy mỗi khi có `push` vào `main`
- production compose đang dùng image `myapp:latest`
- deploy script SSH vào server rồi chạy `docker compose pull && docker compose up -d`
- chưa thấy health check rõ ràng
- chưa có rollback script
- trên cùng VM còn 3 service khác đang chạy

Yêu cầu:
- đánh giá cái gì đang ổn, cái gì rủi ro
- xếp hạng mức độ `Critical` / `High` / `Medium`
- đề xuất patch nhỏ nhất để đưa workflow về mức production-safe
- không rewrite cả hệ thống nếu chưa cần

Hãy trả lời theo thứ tự:
1. current shape và evidence
2. điểm đang ổn
3. các risk theo severity
4. target state đề xuất
5. patch plan nhỏ nhất

## Expected Behavior

Khi skill hoạt động đúng, AI phải:

- vào review/hardening mode thay vì greenfield mode
- không khen hoặc chê chung chung; phải bám vào evidence
- bắt được rủi ro `latest`, thiếu health check, thiếu rollback, và shared-host risk
- không ép chuyển sang Kubernetes, ECS, hay platform khác nếu user không yêu cầu
- ưu tiên patch workflow, image tagging, rollback, và concurrency control trước
