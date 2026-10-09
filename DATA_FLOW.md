# Office Life - Luồng dữ liệu

Tài liệu này mô tả đường đi của dữ liệu trong phiên bản hiện tại. Runtime dùng
MariaDB/MySQL (XAMPP); SQLite dành cho test/tùy chọn local, còn PostgreSQL
schema là tài liệu tham khảo.

## Đăng ký và đăng nhập

```text
Form Pygame
  → client/services/api_client.py
  → POST /api/auth/register hoặc /api/auth/login
  → FastAPI kiểm tra schema và database
  → password hash / verify
  → phản hồi access token JWT + user
  → API client dùng Bearer token cho request cần đăng nhập
```

Khi xác thực thành công, client tải snapshot từ
`GET /api/progress/state`. Khi đăng xuất, client bỏ token và khôi phục tiến độ
khách local; snapshot account trên server vẫn thuộc tài khoản đó.

## Gameplay và lưu tiến độ

```text
client/data/scenarios/content.json
  → BranchingEngine chọn scenario/hội thoại theo state
  → Người chơi chọn Choice
  → GameState cập nhật kỹ năng, quan hệ, XP, score và lịch sử hậu quả
  → Result screen giải thích phản ứng và thay đổi
  → lưu vào SaveManager (offline) hoặc PUT /api/progress/state (đã đăng nhập)
```

Các chỉ số kỹ năng bị giới hạn trong khoảng 0–100. Lựa chọn có thể xác định
scenario kế tiếp và làm thay đổi hội thoại sau đó thông qua trạng thái quan hệ.
Khi hoàn tất toàn bộ catalog, game hiển thị summary rồi quay về menu.

## Tiến độ và hồ sơ tài khoản

- Offline: `SaveManager` serialize `GameState` vào file save khách cục bộ.
- Online: client gửi snapshot JSON tới FastAPI; backend gắn snapshot với user
  từ JWT và lưu trong `game_saves`. Người dùng không thể ghi snapshot của user
  khác bằng cách truyền user ID tùy ý.
- Các endpoint progress, scores, achievements và analytics đọc dữ liệu theo
  user đã xác thực.
- Đặt lại tiến độ online xóa dữ liệu tiến độ/snapshot của tài khoản theo chức
  năng API; đăng xuất không xóa dữ liệu server.

## Leaderboard

```text
Điểm/tổng tiến độ đã lưu
  → API /api/leaderboard (công khai, phân trang)
  → danh sách điểm xếp hạng
  → client hiển thị và tô sáng tài khoản hiện tại nếu đã đăng nhập

Client đã đăng nhập
  → GET /api/leaderboard/me
  → hạng toàn cục của chính tài khoản
```

## Nội dung và phân quyền

Người chơi có thể đọc scenario/chapter qua content API. Các endpoint tạo, sửa
hoặc xóa nội dung kiểm tra role `admin`. API kiểm tra dữ liệu vào bằng Pydantic;
dependency xác thực JWT trước khi cho phép thao tác bảo vệ.

## Đặt lại mật khẩu

Người quản lý chạy `python -m server.reset_password` trực tiếp trên máy có
quyền truy cập database. Công cụ nhận username/email, đọc mật khẩu mới bằng
`getpass`, hash mật khẩu rồi cập nhật DB. Hiện chưa có xác minh email hoặc
luồng reset password công khai qua Internet.
