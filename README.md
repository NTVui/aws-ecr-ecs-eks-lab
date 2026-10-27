# aws-ecr-ecs-eks-lab
Triển khai ứng dụng container trên AWS (lab cá nhân) | T10/2026
# Triển khai ứng dụng container trên AWS (ECR, ECS Fargate, EKS)

Lab cá nhân thực hiện tháng 10/2026 trên tài khoản AWS cá nhân, region ap-southeast-1 (Singapore).
Toàn bộ tài nguyên đã được xóa sau khi làm xong để tránh phát sinh chi phí.

## Mục tiêu
Đóng gói một ứng dụng web (nginx) thành Docker image, lưu trên ECR, rồi triển khai trên cả ECS Fargate và EKS để so sánh hai cách chạy container trên AWS.

## Kiến trúc
- VPC 10.0.0.0/16, 2 Availability Zone, mỗi AZ có 1 public subnet và 1 private subnet
- Public subnet: NAT Gateway, Application Load Balancer, máy EC2 làm việc
- Private subnet: ECS task (Fargate), node EKS
- Security Group: lab-alb-sg (mở port 80 cho Internet), lab-task-sg (chỉ nhận port 80 từ lab-alb-sg)
- Truy cập máy EC2 bằng Session Manager, không mở port SSH

Internet -> Internet Gateway -> ALB -> ECS task (private subnet)
Internet -> Load Balancer của Service EKS -> Pod (private subnet)
ECR (image v1, v2) -> ECS Fargate và EKS

Sơ đồ chi tiết xem trong báo cáo PDF.

## Công nghệ sử dụng
AWS VPC, EC2, ECR, ECS Fargate, EKS, ALB, IAM, NAT Gateway, Docker, kubectl

## Những gì đã thực hiện
1. Thiết kế VPC 2 AZ, gắn tag subnet để EKS tự tạo load balancer đúng chỗ
2. Tạo IAM role và máy EC2 làm việc, kết nối bằng Session Manager
3. Build Docker image, tạo repository ECR, push image v1 và v2
4. ECS Fargate: Target Group (type IP), ALB, Cluster, Task definition, Service 2 task,
   kiểm tra health check và truy cập qua DNS của ALB
5. EKS: tạo cluster và managed node group (2 node ở private subnet), cấp quyền cho EC2
   bằng access entry, Deployment 2 replica, Service type LoadBalancer
6. Scale số replica, rolling update từ v1 lên v2 không gián đoạn, rollback bằng
   kubectl rollout undo
7. Dọn dẹp tài nguyên theo đúng thứ tự phụ thuộc

## Các lỗi thực tế đã gặp và cách xử lý
Tổng hợp 11 lỗi trong mục 3 của báo cáo, ví dụ:
- ECS task báo CannotPullContainerError: Image URI trong Task definition thiếu tag :v1
- Pod Pending "Too many pods": node t3.micro giới hạn số pod, đổi sang loại node lớn hơn
- kubectl báo Unauthorized: chưa tạo access entry cho IAM role của EC2
- Node group kẹt ở Creating: instance type không thuộc Free Tier của tài khoản
- Xóa VPC báo DependencyViolation: còn load balancer, NAT, ENI chưa xóa

## Tài liệu
- Báo cáo đầy đủ (sơ đồ, các bước, lệnh, ảnh chụp): Bao_cao_lab_AWS_VPC_ECR_ECS_EKS.pdf

## Ghi chú
- Đây là lab học tập, không phải hệ thống production
