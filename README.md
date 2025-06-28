# README

## Cách 1: Port-forward (Kiểm tra nhanh từ local)

Với **type: ClusterIP**: Chỉ cho phép truy cập bên trong cụm (từ các Pod khác hoặc qua proxy nội bộ).

```bash
kubectl port-forward svc/fake-data-rest-api -n argocd 8080:80
```

Giải thích:
- Lệnh này ánh xạ port 8080 trên máy local tới port 80 của Service fake-data-rest-api trong namespace abc.
- Service sẽ chuyển tiếp yêu cầu tới port 8080 của container.

Truy cập API: http://127.0.0.1:8080/swagger-ui/index.html

http://localhost:8080/hello

## Cách 2: LoadBalancer Service (Expose ra Internet)

Chuyển Service sang **type: LoadBalancer** để AWS tạo một Elastic Load Balancer (ELB) cung cấp public endpoint.

Cập nhật nonprod/values.yaml

```yaml
appName: fake-data-rest-api
namespace: argocd
replicas: 1
image:
  repository: 674059452637.dkr.ecr.us-east-1.amazonaws.com/nonprod-fake-data
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

## Cách 3: Ingress (Expose qua HTTP với domain)

Sử dụng Ingress để expose ứng dụng qua HTTP với domain hoặc path, yêu cầu cài đặt Ingress Controller (như AWS ALB Controller).


