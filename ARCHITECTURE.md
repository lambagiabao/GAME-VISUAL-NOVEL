# Office Life - Kiến trúc hệ thống

## Trạng thái hiện tại

```text
Người chơi
   ↓
Client Pygame (client/)
   ├─ Offline: JSON scenario + local save
   └─ HTTP/JSON + JWT
          ↓
     FastAPI (server/)
          ↓
     MariaDB/MySQL (XAMPP)
```

- **Client Pygame** xử lý giao diện, input, gameplay và chạy độc lập ở chế độ
  offline.
- Video nền menu được giải mã bởi `imageio-ffmpeg` ở chiều ngang tối đa 960 px;
  soundtrack tạm được phát qua Pygame mixer. Nút loa nhỏ luôn nằm trên clip;
  thanh âm lượng fade hiện khi rê chuột vào góc dưới bên phải. File video nằm ở
  `client/assets/videos/office_life_intro.mp4`; các mốc cắt/lặp được cấu hình
  bằng `INTRO_END_SECONDS` và `LOOP_START_SECONDS` trong
  `client/managers/intro_video_manager.py`.
- **FastAPI** cung cấp API xác thực, nội dung, tiến độ, leaderboard, thành tựu
  và analytics. Client không truy cập database trực tiếp.
- **MariaDB/MySQL** là database runtime được hỗ trợ mặc định qua PyMySQL.
- **SQLite** dùng cho test và có thể chọn thủ công bằng `DATABASE_URL`.
- `server/database/schema.sql` là schema PostgreSQL tham khảo; chưa có adapter
  PostgreSQL/Supabase trong runtime hiện tại.

## Trách nhiệm theo thư mục

| Thư mục | Trách nhiệm |
|---|---|
| `client/game/` | Trạng thái phiên chơi, hiệu ứng và thuật toán branching |
| `client/managers/` | Lưu local, localization và âm thanh |
| `client/services/` | HTTP API client |
| `client/screens/` | Chưa tách màn hình riêng; UI hiện được điều phối từ `client/main.py` |
| `client/data/` | Catalog scenario, save và cài đặt local |
| `client/localization/` | Chuỗi tiếng Việt/tiếng Anh |
| `server/routers/` | Endpoint auth, nội dung và gameplay |
| `server/schemas/` | Kiểm tra request/response |
| `server/models/`, `server/services/` | Kiểu entity và helper nghiệp vụ |
| `server/auth/`, `server/dependencies.py` | Hash/token, xác thực và phân quyền |
| `server/database/` | Kết nối, schema runtime MariaDB và schema PostgreSQL tham khảo |
| `tests/` | Test game, API, cấu hình database và cấu trúc nội dung |

## Ranh giới và bảo mật

- Khi offline, game đọc scenario từ `client/data/scenarios/content.json` và
  lưu tiến độ khách ở `client/data/saves/progress.json`.
- Khi đăng nhập, save tài khoản được đọc/ghi qua FastAPI và liên kết với user
  đã xác thực; save khách được giữ tách biệt.
- Endpoint cá nhân dùng JWT. Endpoint ghi nội dung yêu cầu quyền admin.
- Mật khẩu không lưu dạng văn bản thuần; chỉ lưu password hash.
- Thông tin kết nối database và JWT secret phải được cấu hình trong `.env`,
  không commit hoặc chia sẻ.

Xem giải thích chi tiết theo file trong [README_CODE.md](README_CODE.md) và
các luồng hoạt động trong [DATA_FLOW.md](DATA_FLOW.md).
