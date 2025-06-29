# GitOps triển khai với ArgoCD + Helm - Kiến Không Ngủ

Đây là repository triển khai ứng dụng theo mô hình **GitOps**, sử dụng [**ArgoCD**](https://argo-cd.readthedocs.io/en/stable/) để đồng bộ cấu hình từ Git lên Kubernetes. Ứng dụng được đóng gói và quản lý thông qua [**Helm**](https://helm.sh/).

![ArgoCD + Helm](https://media2.dev.to/dynamic/image/width=1280,height=720,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8vqktcxyjeyd1hoghvfu.png)

## Mục Lục

- [1. Cấu trúc thư mục](#1-cấu-trúc-thư-mục)
  - [1.2.1 deployment.yaml](#121-deploymentyaml)
  - [1.2.2 service.yaml](#122-serviceyaml)
  - [1.2.3 ingress.yaml](#123-ingressyaml)
- [2. GitOps triển khai với ArgoCD + Helm](#2-gitops-triển-khai-với-argocd--helm)
  - [Cách 1: Port-forward (Kiểm tra nhanh từ local)](#cách-1-port-forward-kiểm-tra-nhanh-từ-local)
  - [Cách 2: LoadBalancer Service (Expose ra Internet)](#cách-2-loadbalancer-service-expose-ra-internet)
  - [Cách 3: Ingress (Expose qua HTTP với domain)](#cách-3-ingress-expose-qua-http-với-domain)
- [3. Tài Liệu Tham Khảo](#3-tài-liệu-tham-khảo)

## 1. Cấu trúc thư mục

```plaintext
├── nonprod/                          # Môi trường non-prod (dev, staging...)
│   └── my-app/                       # Một ứng dụng cụ thể
│       ├── Chart.yaml                # Metadata của Helm chart
│       ├── values.yaml               # Thông số cấu hình mặc định cho chart
│       ├── templates/                # Template Kubernetes YAML (Pod, Service,...)
│       │   ├── deployment.yaml       # Định nghĩa Deployment – điều phối cách ứng dụng được chạy dưới dạng Pod
│       │   ├── service.yaml          # Định nghĩa Service – tạo endpoint nội bộ để truy cập ứng dụng
│       │   └── ingress.yaml          # Định nghĩa Ingress – cấu hình HTTP routing để truy cập từ bên ngoài
├── prod/                             # Môi trường production (tương tự nonprod)
│   └── my-app/
│       └── ...
└── README.md                         # Tài liệu mô tả
```

### 1.2 Thư mục `templates/` trong Helm Chart

#### 1.2.1 deployment.yaml

- **Mục đích**: Tạo đối tượng `Deployment` trong Kubernetes để quản lý số lượng bản sao (replicas) của Pod.
- **Bao gồm**: thông tin như image container, số lượng replicas, biến môi trường, port, volume,…
- **Helm sẽ render các biến** như: `.Values.image.repository`, `.Values.replicaCount`, v.v.

#### 1.2.2 service.yaml

- **Mục đích**: Tạo một `Service` để expose ứng dụng bên trong cluster thông qua DNS nội bộ (`ClusterIP`) hoặc mở ra bên ngoài (`NodePort`, `LoadBalancer`).
- **Chức năng**: Mapping từ port bên trong Pod ra ngoài thông qua Service.

#### 1.2.3 ingress.yaml

- **Mục đích**: Định tuyến các request HTTP từ bên ngoài vào đúng service theo domain hoặc path.
- **Cấu hình**: Các rule như `host: myapp.example.com`, `path: /api`,…
- **Tùy chọn**: Có thể thêm annotation để dùng ingress controller như NGINX hoặc AWS ALB Ingress Controller.


## 2. GitOps triển khai với ArgoCD + Helm

### Cách 1: Port-forward (Kiểm tra nhanh từ local)

Với **type: ClusterIP**: Chỉ cho phép truy cập bên trong cụm (từ các Pod khác hoặc qua proxy nội bộ).

```bash
kubectl port-forward svc/fake-data-rest-api -n argocd 8080:80
```
Ca
Giải thích:
- Lệnh này ánh xạ port 8080 trên máy local tới port 80 của Service fake-data-rest-api trong namespace abc.
- Service sẽ chuyển tiếp yêu cầu tới port 8080 của container.

Truy cập:

- http://127.0.0.1:8080/swagger-ui/index.html

- http://localhost:8080/api/ping

- http://localhost:8080/actuator/health

### Cách 2: LoadBalancer Service (Expose ra Internet)

Chuyển Service sang **type: LoadBalancer** để AWS tạo một Elastic Load Balancer (ELB) cung cấp public endpoint.

Cập nhật nonprod/values.yaml

```yaml
appName: fake-data-rest-api
namespace: argocd
replicas: 1
image:
  repository: 674059452637.dkr.ecr.us-east-1.amazonaws.com/fake-data
  tag: "setup_github_actions_15941804215"
containerPort: 8080
springProfile: nonprod
service:
  type: LoadBalancer # Đổi từ ClusterIP thành LoadBalancer
  port: 80
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "200m"
    memory: "256Mi"
```

Lấy endpoint LoadBalancer:

```bash
kubectl get svc -n argocd fake-data-rest-api -o wide
```

### Cách 3: Ingress (Expose qua HTTP với domain)

Sử dụng Ingress để expose ứng dụng qua HTTP với domain hoặc path, yêu cầu cài đặt Ingress Controller (như AWS ALB Controller).

## 3. Tài Liệu Tham Khảo

- [📘 Argo CD Documentation](https://argo-cd.readthedocs.io/en/stable/)  
  Tài liệu chính thức hướng dẫn cài đặt, sử dụng và tích hợp ArgoCD vào quy trình GitOps.

- [📘 Helm Documentation](https://helm.sh/docs/)  
  Hướng dẫn sử dụng Helm – công cụ quản lý Kubernetes bằng template hóa YAML.

- [📘 GitOps với ArgoCD và Helm – Hướng dẫn thực hành](https://argo-cd.readthedocs.io/en/stable/user-guide/helm/)  
  Hướng dẫn triển khai GitOps kết hợp ArgoCD và Helm để tự động hóa CI/CD cho Kubernetes.
