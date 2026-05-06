## Demo Goal

Use this prompt to test the skill locally before showing it to a leader or team.

## Input

Thiết kế lại CI/CD cho một hệ thống web đang chuẩn bị đưa lên môi trường production mới.

Điều kiện hiện tại:
- Ứng dụng backend là Java Spring Boot, đóng gói file `.jar`
- Server production là Linux VM
- Trên cùng server đang có 2 hệ thống khác chạy bằng `systemd`
- Nginx reverse proxy đã tồn tại sẵn để route theo domain
- Không muốn đội vận hành phải deploy tay nữa

Yêu cầu:
- dùng AI để đề xuất flow CI/CD và sinh shell script deploy tự động
- nhánh `feature/*` merge vào `stg`
- khi merge từ `stg` sang `main` thì tự động deploy production
- không cho push trực tiếp vào `main`
- chỉ PO dự án hoặc trưởng circle mới được merge vào `main`
- phải có health check và rollback
- không được làm ảnh hưởng các hệ thống khác đang chạy trên cùng server

Hãy trả lời theo thứ tự:
1. assumptions và các điểm cần khảo sát thêm
2. CI/CD flow đề xuất
3. branch protection / approval rules
4. ví dụ pipeline GitHub Actions
5. ví dụ `deploy.sh` và `rollback.sh`
6. checklist validate sau deploy

## Expected Behavior

Khi skill hoạt động đúng, AI phải:

- nhận diện đây là shared environment
- không chọn giải pháp quá nặng như Kubernetes nếu chưa có lý do
- không ép Docker hoặc Compose nếu target phù hợp hơn với VM-native `systemd` + `.jar`
- ưu tiên `release directories + symlink switch`
- chặn direct push vào `main`
- nói rõ merge `stg -> main` là trigger deploy prod
- giữ release identity rõ ràng, ví dụ artifact theo commit SHA hoặc release ID
- tránh để 2 production deploy chạy chồng nhau
- sinh ra file pipeline và shell script có thể copy để tinh chỉnh tiếp
