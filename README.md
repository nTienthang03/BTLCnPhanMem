# BTLCnPhanMem
# LINK YOUTUBE : https://www.youtube.com/watch?si=fLnitMejW4h-tFq-&v=qN1tEEYMaCo&feature=youtu.be
#Cách chạy
# Volunteer Management — Hướng dẫn chạy nhanh (Local)

Tài liệu ngắn này hướng dẫn cách chạy nhanh backend và frontend trên môi trường local (Windows / PowerShell). Nội dung tập trung vào các bước cần thiết để khởi động, cấu hình database, seed admin và kiểm thử nhanh API.

**Nội dung:**
- Yêu cầu môi trường
- Chuẩn bị & chạy database
- Cài dependencies và chạy backend
- Seed admin
- Chạy frontend tĩnh
- Tài khoản mẫu và kiểm thử nhanh
- Ghi chú bảo mật & khắc phục lỗi

---

**Yêu Cầu Môi Trường**
- Python 3.8+ (khuyến nghị 3.10+)
- SQL Server (local hoặc remote) với quyền chạy script và tạo DB
- Trình duyệt hiện đại

Nếu dùng conda/venv, kích hoạt môi trường trước (PowerShell):

```powershell
# conda
conda activate <env>
# hoặc venv PowerShell activation
.\venv\Scripts\Activate.ps1
```

**Chuẩn Bị Database**
- Schema và dữ liệu mẫu nằm ở `database/init_db.sql` (có sẵn một tài khoản admin mẫu `admin@volunteer.vn` / `admin123`).

Bạn có thể chạy file SQL bằng SQL Server Management Studio hoặc `sqlcmd`:

```powershell
sqlcmd -S <SERVER_NAME> -U <USER> -P <PASSWORD> -i "database\init_db.sql"
```

Lưu ý: cấu hình kết nối cơ sở dữ liệu của ứng dụng nằm trong `backend/database.py`. Nếu bạn chạy DB ở server/credentials khác, chỉnh file đó hoặc quản lý bằng biến môi trường trước khi khởi động backend.

**Cài Dependencies & Khởi Chạy Backend**

1) Cài dependencies Python:

```powershell
pip install -r backend\requirements.txt
```

2) Chạy backend (development) bằng Uvicorn (từ thư mục dự án):

```powershell
python -m uvicorn backend.main:app --reload
```

Mặc định server chạy tại `http://localhost:8000`. OpenAPI/Swagger có tại `http://localhost:8000/docs`.

**Seed Admin (tùy chọn)**
- `database/init_db.sql` thường đã tạo sẵn admin mẫu. Để tạo admin khác dùng script:

```powershell
python backend\scripts\create_admin.py --email admin@local.test --password admin123 --name "Admin Local"
```

Script này idempotent (nếu email đã tồn tại sẽ không ghi đè).

**Chạy Frontend (tập tin tĩnh)**
- Frontend nằm trong thư mục `frontend/`. Đề nghị phục vụ bằng static server thay vì mở file trực tiếp.

```powershell
cd frontend
python -m http.server 8001
# Truy cập http://localhost:8001/index.html
```

Hoặc dùng extension Live Server (VSCode) để phục vụ.

**Tài Khoản Admin Mẫu**
- Email: `admin@volunteer.vn`
- Mật khẩu: `admin123`

Gửi `POST /api/auth/dang-nhap` để lấy token. Sau khi có token, thêm header `Authorization: Bearer <token>` để gọi các endpoint yêu cầu quyền.

**Một số endpoint quan trọng (tóm tắt)**
- `GET /api/admin/organizations?status=pending` — danh sách tổ chức chờ duyệt
- `POST /api/admin/organizations/{ma_to_chuc}/approve` — duyệt/từ chối tổ chức (body: `{ "approve": true|false, "reason"?: "..." }`)
- `GET /api/admin/hoat-dong?status=pending` — hoạt động chờ duyệt
- `POST /api/admin/hoat-dong/{ma_hoat_dong}/approve` — duyệt/từ chối hoạt động
- `GET /api/admin/stats` — thống kê (tổng TNV, tổ chức, hoạt động...)
- `GET /api/admin/logs?limit=50&offset=0` — nhật ký hành động admin (audit log)

Huy hiệu (Badges):
- `GET /api/huy-hieu` — toàn bộ huy hiệu
- `GET /api/tinh-nguyen-vien/huy-hieu` — huy hiệu của TNV (cần token)
- `POST /api/tinh-nguyen-vien/huy-hieu/check` — TNV trigger kiểm tra & trao huy hiệu
- `POST /api/admin/huy-hieu` — Admin tạo huy hiệu mới
- `POST /api/admin/huy-hieu/check/{ma_tnv}` — Admin trigger trao huy hiệu cho TNV

**Kiểm Thử Nhanh (PowerShell)**

1) Khởi động backend:

```powershell
python -m uvicorn backend.main:app --reload
```

2) (Tùy chọn) Tạo admin mới:

```powershell
python backend\scripts\create_admin.py --email admin@local.test --password admin123 --name "Admin Local"
```

3) Lấy token (ví dụ PowerShell):

```powershell
$body = '{"email":"admin@volunteer.vn","mat_khau":"admin123"}'
Invoke-RestMethod -Method Post -Uri http://localhost:8000/api/auth/dang-nhap -Body $body -ContentType 'application/json'
```

Sau khi nhận `access_token`, thêm header `Authorization: Bearer <token>` để kiểm thử các endpoint admin.

**Ghi chú bảo mật & cấu hình**
- `SECRET_KEY` hiện đang hard-coded trong `backend/auth.py` / `backend/main.py`. Trong production dùng biến môi trường và giá trị mạnh.
- CORS hiện set `allow_origins=["*"]` cho dev; giới hạn domain khi deploy thực tế.
- DB credentials nằm trong `backend/database.py` — KHÔNG đẩy thông tin nhạy cảm lên kho công khai.
- Xem xét dùng cookie `httpOnly` cho token thay vì lưu client-side khi cần bảo mật cao.

**Troubleshooting nhanh**
- Lỗi kết nối DB: kiểm tra cấu hình trong `backend/database.py`, SQL Server đang chạy, firewall và ODBC driver.
- Lỗi 401: kiểm tra token, header `Authorization` và thời hạn token.
- Lỗi 403: kiểm tra vai trò (`vai_tro`) trong payload token (cần `quản trị viên`).


