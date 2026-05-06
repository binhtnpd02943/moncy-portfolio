## Review Checklist For Local Demo

Use this checklist to judge whether the skill output is good enough before presenting it.

- [ ] Có nhận diện đây là shared environment không
- [ ] Có nêu rõ assumption hoặc câu hỏi khảo sát còn thiếu không
- [ ] Có chọn chiến lược deploy phù hợp thay vì over-engineer không
- [ ] Có giữ đúng operating mode: review/hardening hay greenfield không
- [ ] Có nêu rõ flow `feature/* -> stg -> main` không
- [ ] Có chặn direct push vào `main` không
- [ ] Có giới hạn người merge vào `main` không
- [ ] Có trigger deploy khi merge hoặc push vào `main` sau PR từ `stg` không
- [ ] Có pipeline YAML cụ thể không
- [ ] Có image tag, artifact version hoặc digest bất biến không
- [ ] Có `deploy.sh` và `rollback.sh` cụ thể không
- [ ] Có cơ chế serialize deploy production, tránh 2 job chạy đè nhau không
- [ ] Có nói rõ repo hoặc module nào sở hữu pipeline, compose, script deploy không
- [ ] Có giới hạn deploy vào đúng service thay đổi thay vì restart cả stack không
- [ ] Có policy cho migration dữ liệu hoặc schema không
- [ ] Có health check sau deploy không
- [ ] Có rollback path rõ ràng không
- [ ] Có tách bạch app rollback và data rollback không
- [ ] Có chiến lược secrets rõ ràng, không nhét secret vào repo không
- [ ] Có đối chiếu workflow/provider syntax với official docs khi phần đó có thể thay đổi không
- [ ] Có validation cụ thể cho YAML, Docker, Compose, script nếu tooling sẵn có không
- [ ] Có tránh sửa các tài nguyên dùng chung như nginx, port, service name của hệ khác không
- [ ] Nếu input là hệ thống sẵn có, có phân loại `keep / patch / replace` không

## Failure Signs

Nếu output có một trong các dấu hiệu sau thì chưa nên đem demo:

- chỉ nói lý thuyết CI/CD mà không sinh file mẫu
- bỏ qua rủi ro môi trường đang có hệ thống khác chạy
- rewrite cả hệ thống dù chỉ cần hardening cục bộ
- không có rollback
- deploy production bằng `latest` hoặc artifact không cố định
- chạy migration phá huỷ dữ liệu tự động trong luồng auto-deploy
- cho phép nhiều deploy production chạy song song
- cho phép push trực tiếp vào `main`
- nói "dùng Kubernetes" mà không có lý do rõ ràng
