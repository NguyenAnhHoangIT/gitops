# Báo cáo Minh chứng Thực hành (Lab Evidence)

Tài liệu này tổng hợp hình ảnh minh chứng kết quả triển khai hệ thống GitOps, giám sát và phát hành tự động.

---

## Lab 1: Cài Prometheus + Argo Rollouts — qua GitOps

Giám sát cụm Kubernetes sử dụng Prometheus Operator và quản lý triển khai bằng Argo Rollouts. Các tài nguyên được cài đặt tự động qua các ứng dụng ArgoCD.

### Trạng thái các Pods đang hoạt động trong cụm:
- **Namespace `argo-rollouts`**: Các pods điều khiển Argo Rollouts hoạt động ổn định.
- **Namespace `prometheus`**: Các pods thu thập số liệu giám sát (operator, state-metrics, prometheus-server, node-exporter) đang chạy đầy đủ.

![Triển khai Prometheus và Argo Rollouts](assets/installed-pods.png)

---

## Lab 2: Tự động Hủy & Lùi phiên bản Canary (Canary Auto-Abort & Rollback)

### 1. Chi tiết Lỗi Đo lường AnalysisRun
Khi đẩy phiên bản lỗi `v6` (tỷ lệ lỗi 50%), hệ thống giám sát Prometheus phát hiện tỷ lệ thành công sụt giảm sâu xuống còn **87.2%**, vi phạm ngưỡng tối thiểu **95%** và kích hoạt trạng thái **Failed** ngay chu kỳ quét thứ 3.

![Chi tiết Phép đo Thất bại](assets/canary-abort-rollback.png)

### 2. Sơ đồ Cấu trúc Tài nguyên ArgoCD sau khi Rollback
Sau khi phát hiện lỗi, hệ thống tự động:
1. Đánh dấu trạng thái Rollout là **Degraded** (Auto-Abort).
2. Xóa bỏ/Scale down toàn bộ các pod của ReplicaSet lỗi (`api-7959b686cf`).
3. Khôi phục hoạt động của 4 pod thuộc phiên bản ổn định trước đó (`api-f4d6975dc`) để tiếp nhận 100% traffic (Auto-Rollback).

![Sơ đồ Tài nguyên tự động Quay lui](assets/argocd-resource-tree.png)
