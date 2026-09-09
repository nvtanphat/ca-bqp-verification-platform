# Hướng dẫn quy trình phát triển dành cho thành viên team (Onboarding Guide)

Chào mừng bạn tham gia phát triển dự án **CA/BQP Verification Platform** (`ca-bqp-verification-platform`).  
Tài liệu này hướng dẫn chi tiết từng bước từ cài đặt môi trường ban đầu cho đến quy trình làm việc chuẩn Agile/Scrum.

---

## 1. Chuẩn bị ban đầu (First-time Setup)

### 1.1. Clone Repository & Chuyển sang branch `develop`

```bash
# 1. Clone repository về máy local
git clone https://github.com/nvtanphat/ca-bqp-verification-platform.git
cd ca-bqp-verification-platform

# 2. Chuyển sang branch tích hợp develop
git checkout develop
git pull origin develop
```

---

### 1.2. Khởi chạy môi trường Local bằng Docker Compose

```bash
# 1. Tạo file cấu hình môi trường từ template
cp .env.example .env

# 2. Khởi chạy toàn bộ hệ thống bằng Docker Compose (Backend + PostgreSQL + Redis)
docker compose up --build
```

**Kiểm tra môi trường:**
- Backend API: Truy cập `http://localhost:8000/health` (Phải nhận được response `{"status": "ok", "service": "ca-bqp-backend"}`).
- PostgreSQL: Port `5432` (User: `cabqp`, Database: `cabqp`).
- Redis: Port `6379`.

---

## 2. Quy trình làm việc với Ticket mới (Daily Workflow)

### Bước 1: Tiếp nhận Ticket trên Jira
- Mở Jira ticket được giao (Ví dụ: `SCRUM-8`).
- Chuyển trạng thái ticket từ `To Do` ➔ **`In Progress`**.
- Comment trên Jira thông báo bắt đầu làm việc.

---

### Bước 2: Tạo Branch tính năng từ `develop`

```bash
# 1. Luôn cập nhật code mới nhất từ develop trước khi rẽ nhánh
git checkout develop
git pull origin develop

# 2. Tạo branch mới theo đúng quy ước đặt tên
git checkout -b feature/SCRUM-<id>-<ten-ngan-gon>
# Ví dụ: git checkout -b feature/SCRUM-8-ocr-pipeline
```

#### Quy ước đặt tên branch:
- Tính năng mới: `feature/SCRUM-<id>-<short-name>`
- Sửa lỗi: `fix/SCRUM-<id>-<short-name>`
- *Lưu ý:* Tên nhánh viết thường, dùng dấu gạch ngang `-`, tuyệt đối không dùng tên mơ hồ như `test`, `abc`, `fix1`.

---

### Bước 3: Phát triển Code & Kiểm thử (Development & Testing)

1. Triển khai logic nghiệp vụ trong các module tương ứng (`apps/backend/src/cabqp/modules/`).
2. Viết Unit Test bổ sung trong `apps/backend/tests/`.
3. Chạy kiểm thử tự động tại máy local:

```bash
# Kiểm thử thông qua Docker Container
docker compose exec backend pytest

# Hoặc nếu chạy ngoài Docker (cần cài virtualenv):
python -m pytest apps/backend/tests
```

---

### Bước 4: Commit theo chuẩn Conventional Commits

Chỉ commit khi code chạy thành công và test pass.

```bash
git add .
git commit -m "<type>(<scope>): <mô tả ngắn bằng tiếng Anh>"
```

#### Ví dụ commit chuẩn:
- `feat(ocr): add PDF text extraction pipeline`
- `fix(registry): correct fuzzy search score threshold`
- `docs(readme): update local setup steps`
- `chore(deps): update fastapi dependency version`

---

### Bước 5: Đẩy code & Mở Pull Request (PR)

```bash
# Push branch cá nhân lên GitHub
git push -u origin feature/SCRUM-<id>-<ten-ngan-gon>
```

#### Cấu hình Pull Request trên GitHub:
- **Target Branch (Base):** **`develop`** *(Tuyệt đối không push trực tiếp vào `main`)*
- **Source Branch (Compare):** `feature/SCRUM-<id>-<ten-ngan-gon>`
- **Title PR:** `[SCRUM-<id>] <Tên ticket>`
- **Mô tả (Description):** Liệt kê các thay đổi và kết quả kiểm thử.

---

### Bước 6: Review & Merge
1. Gửi link PR cho ít nhất **1 teammate** để Review code.
2. Kiểm tra **CI Pipeline (GitHub Actions)** chạy xanh (`All checks have passed`).
3. Khi đã có approval và CI Pass ➔ Bấm **Merge Pull Request** vào `develop`.
4. Cập nhật Jira ticket sang **`Done`**.

---

## 3. Kiến trúc thư mục dự án (Repository Structure)

```text
ca-bqp-verification-platform/
├── apps/
│   ├── backend/          # Core FastAPI Application & Unit Tests
│   └── web/              # Frontend Application (Future Sprint)
├── workers/              # Background Worker Tasks (Celery/Redis)
├── pipelines/            # Ingestion, Synthetic Data & ML Pipelines
├── contracts/            # Data Contracts & Schemas
├── datasets/             # Manifests & Sample Test Datasets
├── docs/                 # System Architecture & Business Docs
├── compose.yaml          # Docker Compose Stack
├── Makefile              # Phím tắt câu lệnh (make up, make test)
└── .github/workflows/    # CI/CD Automations
```

---

## 4. Các câu lệnh phím tắt thông dụng (Makefile)

| Câu lệnh | Công dụng |
|---|---|
| `make up` | Khởi chạy toàn bộ stack ngầm (`docker compose up --build -d`) |
| `make down` | Tắt và dừng toàn bộ containers (`docker compose down`) |
| `make logs` | Xem log hệ thống thời gian thực (`docker compose logs -f`) |
| `make test` | Chạy toàn bộ unit test backend (`docker compose exec backend pytest`) |
| `make config` | Kiểm tra tính hợp lệ của file `compose.yaml` |
